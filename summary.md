# Summary DevOps

## 1. What is DevOps?

Practice of operations and development engineers participating *together* in the entire service lifecycle from design through development to production support. Whit the goal of improving and shortening the systems development lifecycle.

![DevOps lifecycle](./img/devops.png)

### Traditional development teams (before DevOps)

- Development teams
  - Requirement analysis
  - Software design
  - Planning
  - Software implementation
  - Testing
- Operation teams
  - Receives application from dev team
  - Deploy app on infrastructure
  - Manage infrastructure
  - Monitoring
  - Support

This approach led to a lot of *operation mismatch*:

- Defects released to production
- Hard to diagnose issues quickly
- Finger pointing
- ...

### DevOps core values

1. Culture and People > Process and tools
    - People become product owners, give them trust and responsibility
2. Automation (infrastructure as code)
    - Automation is critical as things need to move fast
3. Measurement (measure everything)
    - Knowledge of system is key
    - Know when and why things go wrong
4. Sharing - collaboration - feedback
    - Sharing knowledge between Devs and Ops

### Continuous delivery/deployment

- **Continuous integration**: build and test
- **Continuous delivery**: deploy and integration test
- **Continuous deployment**: build, test, deploy, integration test and deploy to prod

> 💡: Best practices
>
> - Build should pass within 5 min (☕ coffee test)
> - Commit small bits
> - Do not leave build broken
> - Deployment should go to a copy of production (🚧 staging) before production

![Continuous delivery/deployment](./img/continuous-development-deployment.png)

#### Automated testing for continuous deployment

- Unit tests: Test individual components of code
- Integration tests: Test how components work together
- Crossbrowser, performance, security tests

### CI/CD Tooling

- **Version control**: Git, BitBucket
- **CI systems**: Jenkins, TravisCI
- **Build**: Make, Maven, Packer
- **Test**: Junit, Cucumber
- **Artifact repository**: Dockerhub, Artifactory
- **Deployment**: Ansible

### Reliability Engineering

The goal is to have no down-time, design patterns exist for creating resilient systems.

The key is to: Build - Measure - Learn - Repeat!

#### Chaos Engineering

Chaos Engineering is a discipline that aims to proactively test and improve a system's resilience to unexpected and turbulent conditions. It involves deliberately introducing controlled and planned disruptions into a software or system environment to observe how the system responds under stress.

**Chaos monkey**
: Introduce random failures in production and see how resilient system is, or how fast engineer can act upon this problem.

### DevSecOps

Extension of DevOps that includes security from the get-go

- **Secure Coding**: Responsibility of the devs to write secure code.
- **Security testing tools integrated in CI/CD pipeline**: eg. scanning dependencies/containers for vulnerabilities
- **Shift-left testing**: Software testing is performed earlier in lifecycle.

#### Tools to implement DevSecOps

- **SAST - Static Application Security Testing**
  - Scan proprietary/custom code for errors
  - During the code, build and dev phases of the lifecycle
- **SCA - Software Component Analysis**
  - Scan source code and libraries for known vulnerabilities
  - Provide insight of security and licence risks
  - Integrates seamlessly into CI/CD pipeline
- **IAST - Interactive Application Security Testing**
  - Work in the background during tests to analyze runtime behavior
  - Observe request/response integration
  - Detect runtime vulnerabilities
- **DAST - Dynamic Application Security Testing**
  - Automated black box testing -> mimic how hacker would approach your system

## 2. Container Technology

### 2.1 Virtualisation: VM's and containers

**Why is there a need to containerize testing and deployment?**

- Operation teams must set-up a variety of different runtime engines, deal with versioning, ...
- No ability to constrain resources consumed by a single service instance
- Lack of isolation between multiple instances on the same machine

#### Virtual Machine Images

- ✅: Encapsulation and isolation
- ✅: Virtual hardware
- ❌: Less efficient resource utilization
- ❌: Slow deployments

#### Host Virtualisation: Virtual Machines and Containers

**Hardware virtualisation**
: Hypervisor arbitrates access to shared hardware. VM's are completely isolated, each VM requires its own  OS

![Hardware Virtualization](./img/hw-virtualization.png)

**Software virtualisation**
: OS kernel allows multiple process spaces. Containers share the host OS kernel, each container has its own root file system.

![Software Virtualization](./img/sw-virtualization.png)

#### Container as deployable artifact

![Container architecture](./img/container-architecture.png)

- Encapsulation
- Isolation
- Resource contraints
- No virt hardware
- lightweight
- fast boot

### 2.2 Container Technologies

#### LXC/LXD

**LXC**:
Containers build into the Linux kernel. Low-level and difficult to configure.

**LXD**:
Next generation, build on top of LXC, locked in the Canonical ecosystem.

#### Docker

- Can be thought of as a software logistics provider tool (installing, removing, upgrading and running software)
- Docker CLI + Deamon
  - Builds images, runs and manages containers + REST API
  - Containers can only access their own memory and resources

![Docker Architecture](./img/docker-architecture.png)

Container = software that has to get started

Container-image = All the configuration necessary to start the container

##### Handling application dependencies

Without containers, application use the same dependencies (the ones located on the host computer). This can become problematic if different applications require different versions of the dependencies.

When using docker, every container will have it's own copy of the required dependencies.

##### Namespaces / Environment independence

- Every running program or container has a unique *PID*
- PID namespace is set of unique numbers that identify processes
  - Host can have multiple PID namespaces
  - Each PID namespace contains its own PID's
  - Docker created new PID namespace for each container
- Docker provides **environment independence**
  - Within docker, you can run everything independent of other processes on the computer (eg: you don't have to care about existing PID's or other open ports on system)

##### Environment-Agnostic

1. Read-only file systems
    - Attacker can not compromise files in the container
    - Confidence container will not change after changes to the files it contains
2. Environment variable injection
    - Docker `env` command can be used to inject variables (like credentials) into the container
3. Volumes

##### Container states

![Docker container states](./img/container-states.png)

##### OS layers in containers

**Why is there an OS-layer in a docker container**
: Containers contain a very minimalistic version of an OS. This layer is called the `Base Layer`. This layer contains essentials to run the application (like a file system, network, ...). Developers can choose which Base they want (centos, busybox, scratch, ...)

Windows hosts can also have an additional layer for a hypervisor, in this case, every container receives its own Hyper-V kernel.

![Docker container with Hyper-V](./img/docker-hyper-v.png)

> 💡: Containers with Hyper-V are also a bit safer. When an attacker gets access to the kernel, it will only access the virtual Hyper-V kernel.

##### Building container with Dockerfile

Containers can be made using a `Dockerfile` configuration file

```txt
FROM ubuntu:14.04       // Base image

COPY html /var/www/html // Copy files from host to container
ADD web-page-config.tar // Similar to COPY bur for tar and remote URL

ENV APACHE_LOG_DIR /var/log/apahce    // Set env varialbe
USER 73                 // Set userid in the image

EXPOSE 7373/udp 8080    // Expose network ports

RUN apt update -y       // Run bash command
ENTRYPOINT ["echo", "Dockerfile demo"]  // Run when container starts
```

##### Container vs Image

When starting a container from an image, Docker engine will add another layer. This layer will store information added to the container while it was running. If container gets restarted this information will be gone again.

##### Union file system

Some layers in a docker image (like the base layers) have their own read-only file-system. On top of that, Docker Engine adds another read-write file-system when building the container. These different layers get merged into one virtual layer (= **UnionFS**).

![UnionFS](./img/unionFS.png)

**Copy-on-write**
: Only copy file to upper layer that have been modified, this reduces space and startup time.

### 2.3 Container Management/Orchestration

**What does an orchestration framework do?**

It automates deployment, interconnection, scaling and maintenance of multiple containers on a cluster of nodes.

Some container orchestration systems:

- Kubernetes
- Docker orchestration
- Amazon EC2

### 2.4 Container Ecosystem Discussion

#### Low-level

- **Low-level runtime**:
  - Can: spin up a container and connect it to existing network
  - Can't: create network, manage images, prepare environment for container and manage local/persistent storage

- Main approaches (OCI runtime-spec compatible)
  - runc: cli tools for spanning and running OS-level virt container
  - kata-runtime: cli tool for spanning hardware-virt container (focus: security)

#### High-level runtime

**High-level runtime**: deals with creating the network, managing images, perparing environment, and managing local/persistent storage

- Main approaches (CRI-compatible)
  - Containerd: controlled by CNCF
  - CRI-O: bridge between Kubernetes and OCI-compliant runtimes by RedHat (default in OpenShift)
  - Docker

#### High-level management

**High-level container management**: deals with orchestrating containers on infrastructure

- Main approaches:
  - Kubernetes (*de-facto*)
  - Docker
  - Podman (RedHat)

![Current container ecosystem](./img/container-ecosystem.png)

#### Conclusion

- Container: go-to for micro-services
  - Rapid scaling
  - lightweight
  - Can run anywhere
- Container tech options
  - Dockerfiles
  - Docker compose
- Container orchestration
  - Kubernetes is the de-facto standard
- Fractured ecosystem

---

## 3. Kubernetes

**Kubernetes**
: Orchestration technology that abstracts the underlying hardware of the nodes and provides a uniform interface for workload to be deployed and to consume the shared resource pool.

> 💡: Kubernetes works **declarative** -> you declare what you want and kubernetes figures out the steps to get there

### 3.1 Decoupled infrastructure and scaling

- All services within kubernetes are **load balanced** -> easy to upscale
- **Self-healing** and seamless upgrades and rollbacks
- Auto schedule pods based on their resource needs
- Autoscale workloads: makes more instances of container if needed
- Blue/green deployment: runs two environments at once.
  - Blue: live environment
  - Green: Once new software is ready, switch traffic to green, and blue becomes idle.
- Fire off jobs and schedules cronjobs (eg. auto remove n-elements from log file after n minutes)
- Manage stateless and stateful apps
- ! use the same API across bare metal and every cloud provider

### 3.2 Kubernetes Key Concepts

#### Pods

- Smallest unit of work of Kubernetes
- Pods can contain one or more containers, they share volumes, network, namespace and are part of a single context.
- Serve as a unit of deployment, horizontal scaling and replication
  - Pods should remain small, usually a main containers per pod plus required sidecar containers
- Are REST objects
- Are Ephemeral (they can be destroyed anytime and not have fixed network addresses)
- Can have labels, specifying attributes meaningful to user

![Pod architecture](./img/pod.png)

#### Services

- Abstraction which defines a logical set of pods and a policy by which to access them
- Durable resource (no dynamic startups and shutdowns)
  - Static cluster IP
  - static namespaced DNS name
- Services are also REST objects
- Services is an internal load balancer to pods

![Kubernetes services](./img/k8s-services.png)

**Service registry**
: Database of services, their instances and their locations. Instances are registered on startup and removed on shutdown.

##### Zero-downtime upgrade techniques

1. **Rolling update**
    - ![Rolling update](./img/service-rolling-update.png)

2. **Canary release**: Have a small numbers of users test the new version
    - ![Canary release](./img/service-canary-release.png)

3. **A/B testing**: Users randomly get a version, collect business metrics (rather than technical testing)
    - ![A/B testing](./img/service-a-b-testing.png)

### 3.3 Kubernetes Architecture

![Kubernetes Architecture](./img/kubernetes-architecture.png)

#### Control Plane

![Control Plane](./img/kubernetes-ctrl-plane.png)

##### Kube-controller-manager

- Primary service/daemon that manages all core component control groups
- Monitors the cluster state via API server
- Steer cluster to desired state

##### Kube-scheduler

- Policy-rich engine, tries to place workload on matching resource
  - Decides which nodes should run which pods
- Select node for pods to run on
- Workload requirements example
  - labels
  - Affinity/anti-affinity (which services should be grouped)
  - HW requirements

##### Cloud-Control-Manager

- Daemon that provides cloud-provider specific integration into kubernetes core loop
- Controllers inside cloud-control-manager
  - Node controller: created new Node objects when new servers are created
  - Route controller: configures routes in the cloud
  - Service controller: integration with load balancers, IP address assignment, health checks

#### Node

**Node**
: component of the cluster that serves as a worker machine. It can be either a physical machine or a virtual machine and is responsible for running applications, handling containerized workloads, and providing resources necessary for running those workloads

![Kubernetes Node](./img/kubernetes-node.png)

##### Kubelets 

- Node agent responsible for managing the lifecycle of every pod on its host
- Registers node with API server
- Kubelet interprets YAML pod manifest

##### Container Runtime Engine

- CRI (Container Runtime Interface) compatible application that executes ans manages containers
- Supports different container runtimes (Docker, Kata, Containerd, ...)

##### Kube-proxy

- Manages the network rules on each node
- Performs connection forwarding or load balancing for Kubernetes services

---

## 4. Cloud Native Computing Foundation (CNCF)

- A Linux Foundation project
- Goal: help advance container technology and align industry
- Organizes conferences
- Provide kubernetes certifications and education programs

### 4.1 Scheduling & Orchestration: Kubernetes

CNCF graduated project -> see chapter: [3. Kubernetes](#kubernetes)

### 4.2 Application definition & image build: HELM

- CLI package manager for K8s allowing devs to more easily configure and deploy packages onto K8s
- Helm is **Day 1** operator (deployment)
  - Install or upgrade software deps
  - Configure software deployment
  - Fetch software packages from repo
- K8s is **Day 2** operator (management tasks): monitoring and making backups of stateful/complex workloads
  - eg. Postgres, Kafka, Cassandra

![Helm and K8s operator](./img/helm-operator.png)

Helm or K8s operator?

- Just installing application -> Helm
- How much customization is needed?
  - Default config -> Helm
  - Special config -> operator
- How mature is cluster?
  - First setup: Helm
  - Advanced: Operator

### 4.3 Database: Vitess and TIKV

### 4.4 Coordiantion & service discovery: CoreDNS & ETCD

#### CoreDNS

Cloud native, authoritative DNS server written in Go. It has a  flexible and plugin-based, extensible request pipeline. So it becomes easy to perform tasks like logging, caching and collecting metric about the DNS server.

![CoreDNS](./img/coredns.png)

### 4.5 Service Proxy: Envoy

Cloud native edge and service proxy.

### 4.6 Service Mesh: Linkerd

**Service Mesh**
: Makes running services easier and safer by giving you runtime debugging, observability, reliability, and security.

Three basic components:

1. UI
2. Data plane: Transparent proxies that run next to each service instance (handles traffic from and to service)
3. control plane: set of services providing aggregation of telemetry data and control data to the data plane proxies

Linkerd exports Prometheus and Grafana dashboards.

![Linkderd](./img/linkerd.png)

### 4.7 Cloud native storage: Rook

Rook turns distributed storage systems into self-managing, self-scaling, self-healing storage services. It automates tasks of a storage administrator.

### 4.8 Container registry: Harbor

Open source registry that secures artifacts with policies and role-based access control,
ensures images are scanned and free from vulnerabilities, and signs images as trusted

### 4.9 Observability & analysis: Prometheus, Jaeger, Fluentd

**Prometheus**
: Metrics-based monitoring and alerting stack (all levels of the stack)

**Jaeger**
: End-to-end distributed tracing: Monitoring and troubleshooting transactions in complex distributed systems.

---

## 5. Software development models

**Software dev model**
: Structured set of activities required to develop a software system.

They involve:

- **Specification**: defining what the system should do
- **Design and implementation**: Define the organization of the system and implementing it
- **Validation**: Checking if it does what the customer wants
- **Evolution**: changing the system in response to changing customer needs

**Plan-driven process**
: Precesses where all activities are planned in advance and progress is measured against plan

**Agile process**
: Planning is incremental and it is easier to change the process to reflect changing customer requirements

### 5.1 Software Development Models

#### Waterfall model

Plan-driven model, separate and distinct phases of specification development.

![Waterfall Model](./img/devmodel-waterfall.png)

Phases:

1. **Requirement definition**: eg. What is the budget? Must it work in the Cloud?
2. **System and software design**: eg. making UML diagram
3. **Implementation and unit testing**
4. **Integration and system testing**: Do the different components work with each other?
5. **Operation and maintenance**: maintain the system, include maintenance costs in budget!

- ❌: Difficult to accommodate change after process is underway
  - Only use when requirements are well-understood
- ❌: A phase has to be complete before moving on to next phase
- ✅: When requirements are well defined, this model is typically faster

#### Incremental development

![Incremental Model](./img/devmodel-incremental.png)

- ✅: Cost of accommodating changing customer requirements is reduced
  - Less rewriting of analysis and documentation
- ✅: easier to get customer feedback on work that has been done
- ✅: More rapid delivery and deployment of *userful* software is possible
  - Customer can have an earlier functioning app that still lacks some less important features.
- ❌: progress is less visible
- ❌: System structure tends to degrade as new increments are added
  - Different features may not be designed to work together from the get-go

#### Reuse-oriented development

System that is mostly made from existing components or COTS (Commercial-off-the-shelf) systems.

![Reuse Model](./img/devmodel-reuse.png)

Key phases:

- Requirement specification
- Software discovery and evaluation: eval -> check if software does what is advertised
- Requirement refinement: negotiate with customer, see if potential shortcomings are acceptable.
- Application system configuration
- Component adaption and integration

- ✅: Reduced cost
- ✅: Reduced risk
- ✅: Fast delivery and deployment
- ❌: Requirement compromises are common
- ❌: Loss of control over evolution of reused system

### 5.2 Coping with changes

Changes are inevitable in large software projects. Changes lead to rework, so cost of change is dictated by cost of rework and cost of implementing new functionality.

#### Reducing cost of rework

- **Change avoidance**: anticipate changes in software process (eg. Develop prototype to show key-features to customer)
- **Change tolerance**: process is designed so that changes can be implemented at low cost (eg. incremental development)

#### Software prototyping

Initial version of system to demonstrate concepts and try out design options.

- ✅: Improve system usability
- ✅: Closer match to users needs
- ✅: Reduced development effort

![Prototype development process](./img/prototype-developement.png)

> ⚠️: Prototypes should be discarded after development, they don't form a good basis for the final system.

#### Incremental delivery

- Rather than delivering one final product, deliver in broken down increments.
- Implement highest priorities first
- Once development of increment starts, requirements are frozen, though requirements for later increments can continue to evolve.

![Incremental delivery](./img/incremental-delivery.png)

- ✅: Customer value can be delivered with each increment -> available earlier
- ✅: Earlier increment can function as prototype -> help elicit additional requirements
- ✅: Low risk of project failure
- ❌: Most parts of the system require set of basic functionality (big increments needed in the beginning)

---

## 6. Agile Software Development

**Origin of Agile development methods**:

Rapid development and delivery is now often the most important requirement for software systems. Plan-driven development does not meat this requirements.

**Agile development**:

- Program specification, design and implementation are interleaved
- System developed as series of versions with stakeholder involved in version evaluation.
- Extensive tool support for development (eg. automated testing)
- Minimal documentation - focus on working code

**Plan-driven vs Agile development**:

- Plan-driven:
  - Software engineering based around separate development stages (output of each stage is planned in advance)
  - waterfall, plan-driven and incremental development are all valuable options.
  - Iteration occurs within activities
- Agile:
  - Specification, design, implementation and testing are interleaved and outputs from the development process are decided through a process of negotiation during the software development process

![Plan-based vs Agile](./img/plan-based-vs-agile.png)

### 6.1 Agile methods

Aim of agile:

- Reduce overhead in software process (eg. by limiting documentation)
- Allow to respond quickly to changes

#### Agile manifesto

- individuals and interactions >> process and tools
- working software >> documentation
- customer collaboration >> contract negotiation
- responding to change >> following a plan

#### Principle of Agile methods

| Principle   | Description|
|--------------- | --------------- |
| Customer involvement | Custormer should be closesly involved (prioritize requirements and evaluate iterations) |
| Incremental delivery | Software developed in increments (customer specifies which requirements included in next increment) |
| Poeple not process | Skill of dev team (knowledge about tools) should be exploited -> let them shoose what tech stack |
| Embrace change | Expect system requirements to change |
| Maintain simplicity | Focus on simplicity in both process and development |

#### Where is Agile applicable?

- Small to medium-sized projects
- When there's clear commitment from customer to be involved in process
- Not a lot of external rules and regulations applicable

### 6.2 Agile development techniques

#### XP: Extreme programming

- New versions may be built several times a day
- Increments are delivered every 2 weeks
- All tests must run for every build, build only accepted on successful tests

![Extreme programming release cycle](./img/XP-release-cycle.png)

| Principle or practice  | Description |
|--------------- | --------------- |
| Incremental planning   | Requirs are recorded on *story cards*, story included in next release is detmined by priority. Dev breaks story down into *Tasks* |
| Small releases         | Minimal useful set of functinoality that provides business velue is developed first. Include finished funtionality in frist release |
| Simple design          | Enough design is carried out to meet the current requirs. and no more |
| Test-first develpment  | Write unittest for functionality before writing funtionality |
| Refactoring            | Refactor as soon as imporvements are found |
| Pair programming       | Work in pair, check eachothers progress |
| Collective ownership   | Pair of devs work on all areas of system (no island of expretise) |
| Continious integration | As soon as task is complete -> integrate into whole system |
| Sustainable pace       | Overtime is not acceptable (reduces code quality and productivity) |
| On-site customer       | Represtative of customer should be available full time for use of the XP team (for requirs. and reviews) |

### 6.3 Agile Project Management

Goal: deliver software on time within the planned budget.

#### SCRUM

Most used project management method for Agile development. SCRUM focuses on managing iterative development rather than specific agile practices.

Three phases:

1. **Initial phase**: planning phase, establish general objective for design and architecture
2. **Sprint cycles**: each cycle develops an increment of the system
3. **Project closure phase**: wrap up project and complete required documentation.

| Scrum term     | Definintion |
|--------------- | --------------- |
| Scrum          | A daily meeting of Scrum team that revies progress and goes over work to be done for the day (short face-to-face meeting)|
| ScrumMaster    | Oversees the Scrum process, guides Scrum teams and comunicates progress to rest of company |
| Sprint         | A development iteration (2-4 weeks long) |
| Velocity       | Estimate how much team can cover in a single sprint |

![Scrum sprint cycle](./img/scrum-cycle.png)

- ✅: Product is broken down into manageable chunks
- ✅: Unstable requirements do not hold up progress
- ✅: Whole team has visibility on everything
- ✅: Customers get delivered increments and give feedback
- ✅: Trust between customer and developers is established

#### Kanban

Tool to oversea progress (continuous, not in sprints). 3 rules:

1. Visualize workflow
2. Limit WIP
3. Measure flow

![Kanban board](./img/kanban-board.png)

#### Problems with Agile

- Development is incompatible with legal approach to contract definition commonly used in large companies.
- Agile is geared to new software not software maintenance.
- Agile geared to small co-located teams

---

## 7. Requirements Engineering

**Requirements Engineering**
: The process of establishing the service that the customer requires from a system, and the constraints under which the system operates and is developed.

**Types of requirements**

- User requirements
  - Statements in natural language plus diagrams of the services and its operational constraints.
  - Written for customer
- System requirements
  - Structured doc with detailed description of system's functions, services and operational constraints
  - Defines what should be implemented so may be part of a contract between client and contractor

### 7.1 Functional and non-functional requirements

- **Functional requirements**
  - Statements of service the system should provide, how it should react to certain inputs and how sys should behave in certain situations
  - May state what sys should *not* do
- **non-functional requirements**
  - Constraints on service or functions offered by system (eg. time constraints, dev precess, standards, etc...)
  - Often applies to sys as a whole rather than individual features or services
- **Domain requirements**
  - Constraints on sys from the domain of operations (eg. software for airplanes will have domain specific constraints)

#### non-functional classification

- Product requirements: specifies that delivered product should behave a particular way (eg. execution speed, reliability, ...)
- Organisational requirements: factors in org policies and procedures (eg. standards used, techstack, ...)
- External requirements: eg. GDRP

![non-functional Requirements](./img/non-functional-reqs.png)

Metrics for specifying non-functional requirements: speed, size, ease of use, reliability, robustness and portability.

### 7.2 Requirement Engineering Process

Generic activities common to all RE processes:

1. Requirements elicitation
2. Requirements analysis
3. Requirements validation
4. Requirements management

> 💡: In practice, RE is an iterative process

![spiral view of requirement engineering process](./img/req-engin-proces-spiral.png)

### 7.3 Requirements Elicitation

#### Process activities

1. Requirements discovery
    - interacting with stakeholders to discover requirements
    - Domain requirement discovery
2. Requirements classification and organisation
    - Group related requirements in coherent clusters
3. prioritisation and negotiation
    - Prioritising requirements and resolving requirements conflicts
4. Requirements specification
    - Requirements are documented and input into the next round of the spiral

![Requirements Elicitation](./img/requirement-elicitation.png)

#### Interviewing

- Closed: interviews based on pre-determined list of questions
- Open:  various issues are explored with stakeholders

### 7.4 Requirements Specification

The process of writing user and system requirements in a requirements document. The user requirements have to be understandable for the end-user. System requirements are more detailed. Requirements may also be part of a contract for the system development.

### 7.5 Requirements validation

Demonstrate that the requirements provided by the customer are the requirements the customer really wants. Requirements error costs are very high, so verifying that the requirements are good is very important.

#### Requirements checking

- **Validity**: Does the system provide the functions which best fit customers needs?
- **Consistency**: Are there requirement conflicts?
- **Completeness**: Are all functions required by the customer included?
- **Realism**: Is requirement possible given the available budget and technology?
- **Verifiability**: Can requirements be checked?

#### Requirements validation techniques

- Requirements reviews
  - Regularly manual analysis of requirements
  - Both client and contractor should be involved in reviews
  - Review checks:
    - **Verifiability**: is requirement testable
    - **Comprehensibility**: is requirement well understood
    - **Traceability**: is origin of requirement clearly stated
    - **Adaptability**: Can the requirement be changed without large impact on other requirements?

### 7.6 Requirements change

**Changes are almost inevitable**

- New hardware
- New kind of system it might need to interface with
- Business priorities may change
- Legislations may change
- Customer comes under new management (wants other requirements)

**Dealing with changed requirements**

-  keep track of individual requirements and maintain links between dependent requirements so that you can assess the impact of requirements changes
-  establish a formal process for making change proposals and linking these to system requirements

**Decide if a requirement change should be accepted**

![Requirement change](./img/requirement-change.png)

---

## 8. Design and implementation

### 8.1 Implementation issues

Goal: Reuse as much code as possible

Issues:

- Configuration management: keeping track of the many different versions of each software component in a configuration management system.
- Host-target development: Program gets developed on different computer than it's destined to run on.

#### Reuse level

1. **The abstract level**: Don't reuse software directly, rather use knowledge of successful abstractions in the design of the software.
2. **The object level**: directly use objects form a library rather than writing the code yourself
3. **The component level**: Reuse objects and object classes
4. **The system level (COTS)**: Reuse entire application system

#### Configuration management 

**Configuration management**
: The process of managing a changing software system.

Goal: support the system integration process so that devs can access the projects code and documentation in a controlled way, find out what changes have been made, and compile and link components to create a system.

**Configuration management activities**

- **Version management**: Keep track of different software components and coordinate development by several programmers.
- **System integration**: Help devs define which versions of components are used, and config to build system automatically
- **Problem tracking**: Allow users to report bugs, and let devs see who is working on which issue.

![Config management tool interaction](./img/config-managemt-tool-interaction.png)

#### Host-target development

Development machine usually has different installed software than execution machine.

![Host-target development](./img/host-target-dev.png)

### 8.2 Documentation

**Importance of documentation**: Most development is done in teams, so it's especially important to provide docs for API's and code. Luckily, there are tools that can generate API documentation from comments in the source code (eg. JavaDoc for codebase, Swagger for API's).

### 8.3 Open Source Development

An approach to software dev where code is made public for volunteers to participate.

#### Open Source Licensing

The source code being freely available doesn't mean that anyone can do as they wish with that code. Developers add licenses to their open source projects that explain what people can an cannot do in order to legally use the code.

Popular licenses:

- GPL (GNU General Public License): if use use code with GPL, you are required to make your code open source too.
- MIT: Do whatever you want as long as you keep original copyright and license notice.
- Apache 2.0: Do what you want, as long as you include copyright, license, notice and list changes.

#### License management

- Establish a system for maintaining information about open-source
components that are downloaded and used
- Have auditing systems in place that detect e.g. if you break the terms of a
license

---

## 9. Software Testing

Program testing goals:

- Validation testing: to demonstrate that software meets requirements
- Defect testing: to discover undesirable actions to certain inputs or to see if software does not conform to its specifications.

**Input-output model for program testing**

![Input-output model](./img/input-output-testing.png)

**Inspections vs testing**

- **Software testing**: Concerned with exercising and observing product behavior (dynamic verification)
- **Software inspections**: Concerned with analysis of the static system representation to discover problems (static verification)
  - Other people examining source to find anomalies and defects
  - Does not require execution of system -> used before implementation
  - May be used to any representation of the system (requirements, design, config, test data, ...)
  - Benefits
    - During testing, errors can hide other errors
    - Incomplete system -> inspect without additional cost
    - Detect compliance with standards, portability and maintainability

![Software inspections](./img/software-inspection.png)

> ⚠️: Inspections and testing are complementary, both should be used

**White Box vs Black Box testing**:

- White Box: Tester has knowledge about the system internals by looking at code
- Black Box: Tester has no knowledge about internals

**Code coverage**
: Percentage of code that is covered by tests. (Note: 100% code coverage doesn't mean secure code nor that everything is tested)

### 9.1 Development testing

Tests carried out by the team that is developing the system.

- Unit testing: Test individual methods or classes
- Component testing: test component interfaces
- System testing: some or all components of a system are integrated and tested as a whole.

#### Automated testing

- Unit test should run without manual intervention
- Tests consist of:
  - setup part: initialize the system with the test case (input and expected output)
  - A call part: call object or method to be tested
  - An assertion part: compere result from setup with the one from call

#### Testing strategies

- **Partition testing**: identify groups of inputs that have common characters and should be processed the same way.
  - Input data and output results fall into different classes (all members of a class are related)
  - Program behaves in equivalent way for each class member
  - Test cases should be chosen from each pratition
- **Guideline-based testing**: Use testing guidelines to choose test case
  - Guidelines reflect previous experience of kind of error a programmer often makes

#### Equivalence partitioning

![Equivalence participating](./img/equivalence-partitioning.png)

The blue surface means bad data. Bad input results in bad outputs.

#### Testing guidelines

- Sequences:
  - Test software with sequences which have only single value
  - Use sequence of different sized in different tests
  - Derive test so that first, middle and last element of sequence are tested
  - Test sequence of zero length
- General:
  - Choose input that force the sys to generate all error messages
  - Design inputs that cause input buffers to overflow
  - Repeat the same series of inputs a number of times (to check if buffers clear before new iteration)

### 9.2 Test-driven development

- TTD: inter-leave testing and code development
- Write test before feature
- Develop code incrementally, along with test for increment
- used in Agile methods like XP

![Test-driven development cycle](./img/TDD-cycle.png)

1. Identify the increment
2. Write test for functionality and implement as automated test
3. Run the test, along with other tests
4. Implement the functionality and re-run test
5. All tests success? -> repeat for next chunk of functionality

- ✅: Code coverage
- ✅: Regression testing (test to check that changes did not brake previously working code)
- ✅: simplified debugging
- ✅: system documentation (tests themselves are a form of documentation)

### 9.3 Release and User Testing

#### Release testing

- test a particular release of a sys that is intended for use outside of the dev team
- goal: convince the supplier of the system that it is good enough for use
- usually a *black-box* testing process

**Release test vs System test**

- A separate team should be responsible for release testing
- In system testing, team focuses on discovering bugs (defect testing)
- In release testing, focus on checking if requirements are met, and system is good enough for external use (validation testing)

#### Rquirement-based testing

Examine each requirement and develop test(s) for it.

#### Performance testing

To test performance and reliability. This test should reflect the profile of use of the system. Usually, performance tests involve a sequence of tests with incremental work load. It is also a good idea to perform **stress tests** to discover the limit of work load a system can manage.

#### User testing

Let user or consumer test by letting them provide input. There are 3 types of user testing:

- **Alpha testing**: users work with dev team to test software at the dev site.
- **Beta testing**: a release of the software is made available to users to allow them to experiment and raise problems.
- **Acceptance testing**: customers test system and decide if it is ready to be deployed

---

