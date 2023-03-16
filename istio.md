<h1>Chapter 1. Overview of Service Mesh and Istio</h1>

**Chapter Overview**

Welcome to the course *Introduction to Istio*!

Service meshes are becoming a vital component of a company's
infrastructure. [Istio](https://istio.io/) is an open-source project and
the leading service mesh implementation that is gaining rapid adoption.
To understand why service meshes are so important and why they are
becoming increasingly common, we must look back at the shift from
monoliths to microservices and cloud-native applications, and understand
the problems that stemmed from this shift.

We must also review other technologies that were developed as a response
to these problems, and try to understand why these other solutions, in
many ways, fall short of their objective.

We will then be in a position to review the architecture of Istio to
understand both how and why service meshes elegantly address these
problems.

Service meshes elegantly solve problems in the areas of security,
observability, high availability, and scalability. They enable
enterprises to respond to situations quickly without requiring
development teams to modify and redeploy their applications to effect a
policy change. A service mesh serves as the foundation for running
cloud-native applications.

# Learning Objectives

By the end of this chapter, you should be able to explain:

   -   What problems stemmed from the shift to cloud-native
            applications.

   -   How these problems were mitigated before service meshes
            existed.

   -   How service meshes address these problems.

   -   The design and architecture of Istio.

# The Shift to Cloud-Native Applications

The term \"cloud-native\" represents a list of characteristics that are
desirable in a software system, traits such as:

-   High availability

-   The ability to scale horizontally

-   Zero-downtime deployments and upgrades

-   Security built-in

Enterprises\' bottom lines became increasingly dependent on system
uptime and availability, and the old model of large monolithic
applications presented a variety of obstacles to becoming cloud-native.
Too many developers in a single codebase complicates continuous
integration. And so the reluctant move from monolith to microservices
began. Patterns and strategies emerged for how to go about \"strangling
the monolith.\"

The landscape is littered with business cases of enterprises\' difficult
journeys toward microservices. In the end, these journeys took the
enterprise to a better place, one where the use of automation had
increased, where continuous delivery was taking place with increasing
frequency. Development and operations became less siloed from one
another, allowing developers to self-serve the deployment of their
applications, and to become more involved with operating their
applications in production. Teams got better at monitoring their systems
and were able to lower mean time to detection and improved their mean
time to recovery.

Teams shared their experiences, and before long, many were striving to
emulate these successes in their own organizations.

Platform companies such as Heroku shared their experience and provided
guidance for moving to cloud-native with the publication of
the [[Twelve-Factor App]{.underline}](https://12factor.net/). Others
built on this foundation, and added to the wisdom by raising the
importance of API-first development, having security \"built-in\", and
the importance of telemetry. To read more about this topic, check
out [*[Beyond the Twelve-Factor
App.]{.underline}*](https://tanzu.vmware.com/content/blog/beyond-the-twelve-factor-app)

These transitions took a long time and required significant effort,
primarily because the new microservices architecture, also known as
distributed applications, brought its own challenges.

# New Problems

Microservices brought several benefits. Organizations were able to
organize into smaller teams. Codebases didn\'t all have to be written in
the same language. Individual codebases shrank and consequently became
simpler and easier to maintain and deploy. The deployment of a single
microservice entailed less risk. Continuous integration got easier.
There now existed contracts in the form of APIs for accessing other
services, and developers had fewer dependencies to contend with.

But the new architecture also brought with it a new set of challenges.
Certain operations that used to be simple became difficult. What used to
be a method call away became a call over the network. The address of
that target service in an increasingly dynamic environment became
difficult to resolve. How does one service know if another is available?

Here are some of the challenges that the new architecture posed:

**Service discovery:**\
How does a service discover the network address of other services?

**Load balancing:\
**Given that each service is scaled horizontally, the problem of
load-balancing was no longer an ingress-only problem.

**Service call handling:**\
How to deal with situations where calls to other services fail or take
an inordinately long time? An increasing portion of developers\'
codebases had to be dedicated to handling the failures and dealing with
long response times, by sprinkling in retries and network timeouts.

**Resilience:**\
Developers had to learn (perhaps the hard way) to build distributed
applications that are resilient and that prevent cascading failures.

**Security:\
**How do we secure our systems given this new architecture has a much
larger attack surface?\
Can a service trust calls from other services?\
How does a service identify its caller?

**Programming models:** \
Developers began exploring alternative models to traditional
multithreading to deal more efficiently with network IO (input and
output, see ReactiveX).

**Diagnosis and troubleshooting:** \
Stack traces no longer provided complete context for diagnosing an
issue. Logs were now distributed. How does a developer diagnose issues
that span multiple microservices?

**Resource utilization: **\
Managing resource utilization efficiently became a challenge, given the
larger deployment footprint of a system made up of numerous smaller
services.

**Automated testing:** \
End-to-end testing became more difficult.

**Traffic management:** \
The ability to route requests flexibly to different services under
different conditions started becoming a necessity.

# Early Solutions

Netflix was one such company that, out of necessity, had made the move
to cloud-native. Netflix was growing at such a rapid pace that they had
but a few months to migrate their systems to AWS before they ran out of
capacity in their data center.

In the process of that transition, they ran into many of the
above-described problems. Netflix teams began addressing their issues by
developing several projects, which they chose to open-source.

Netflix teams built the [Eureka service
registry](https://github.com/Netflix/eureka) to solve the problems of
service discovery. They
wrote [Ribbon](https://github.com/Netflix/ribbon) to support client-side
load balancing. They developed the Hystrix library (and dashboards) to
deal with cascading failures, and the Zuul proxy was designed to give
them the routing flexibility they needed for a variety of needs from
blue-green deployments to failover, troubleshooting, and chaos
engineering.

Shortly thereafter, the Spring engineering team adapted these projects
to the popular Spring framework under the umbrella name [Spring
cloud](https://spring.io/projects/spring-cloud). These new services
became accessible to any Spring developer by adding client libraries as
dependencies to their Spring projects.

Many enterprises adopted these projects. But their use implied certain
constraints.

To participate in this ecosystem, services had to be written for the JVM
and had to use the Spring framework. A growing list of third-party
dependencies had to be added to the footprint of each application.

Using these services was not a completely transparent operation either.
Developers often had to annotate their applications to enable features
and configure them. Specific methods required custom annotations, and in
other cases, specific client APIs were required to use a feature.

The use of these infrastructure services represented an important
improvement over the previous state of affairs. For example, development
teams no longer had to write their own retry logic. At the same time,
use of these infrastructure services was not transparent to
participating applications and represented an added burden on
development teams. Dependencies had to be kept up to date and versions
in sync, adding a certain measure of configuration fragility.

What if these infrastructural concerns could be removed from these
individual microservice applications, and \"pushed down\" into the
fabric of the underlying platform?

# Service Meshes

Engineers struggled with similar problems at Lyft, moving away from
monoliths toward a cloud-native architecture.

At Lyft, Matt Klein and others proposed a different approach to solving
these same problems: to bundle infrastructural capabilities completely
out of process, in a manner separate from the main running application.
Essentially every service in a distributed system would be accompanied
by its own dedicated proxy running out-of-process.

By routing requests in and out of a given application through its proxy,
the proxy would have the opportunity to perform services on behalf of
its application in a transparent fashion. This proxy could be made to
retry failed requests, it could be configured with specific network
timeouts, and circuit-breaking logic. The proxy could also be made to
encapsulate the logic of performing client-side load balancing requests
to other services.

The list doesn\'t stop there. The proxy could act as a security gateway,
also known as a Policy Enforcement Point. Connections can be upgraded
from plain HTTP to encrypted traffic with mutual TLS. The proxy could be
made to collect metrics such as request counts, durations, response
codes and more, and expose those metrics to a monitoring system, thereby
removing the burden of managing metrics collection and publishing from
the development teams.

The application of this proxy at Lyft helped solve many of the problems
that its development teams were running into, and helped make their
migration away from monoliths a success.

Matt Klein subsequently open-sourced the project and named
it [Envoy](https://envoyproxy.io/).

At the same time, the advent of containerization (Docker) and container
orchestrators (Kubernetes) began addressing problems of resource
utilization and freed operators from the mundane task of determining
where to run workloads.

Kubernetes Pods provided an important intermediary construct that, on
the one hand, allowed for isolation within the container but on the
other, for multiple loosely coupled containers to be bundled together as
a single unit.

Kubernetes came from Google, and Google was looking to build this same
out-of-process infrastructural capability on top of Kubernetes. The
Istio project started at Google, and as it turns out, Istio saw in Envoy
the perfect building block for its service mesh. The Istio control plane
would automate the configuration and synchronization of proxies deployed
onto Kubernetes as sidecars inside each Pod.

The Kubernetes API server\'s capabilities could be leveraged to automate
service discovery and communicate the locations of service endpoints
directly to each proxy. Fine-grained configuration of proxies could be
performed by exposing Kubernetes Custom Resource Definitions (CRDs).

The Istio project was born.

![The Istio Architecture, in a
nutshell](https://github.com/nadidurna/Istio/blob/master/images/image1.png)


# Istio Architecture

As shown in the illustration in the previous section, the basic idea
behind Istio is to push microservices concerns into the infrastructure
by leveraging Kubernetes. This is implemented by bundling the Envoy
proxy as a sidecar container directly inside every Pod.

***Note:** in the Advanced Topics chapter, we show how a service mesh
can be extended to include workloads running on VMs, outside
Kubernetes.*

In terms of implementation, Istio\'s main concerns are, therefore,
solving the following problems:

1.  Ensuring that each time a workload is deployed, an Envoy sidecar is
    deployed alongside it.

2.  Ensuring traffic into and out of the application is transparently
    diverted through the proxy.

3.  Assigning each workload a cryptographic identity as the basis for a
    more secure computing environment.

4.  Configuring the proxies with all the information they need to handle
    incoming and outgoing traffic.

We will explore each of these concerns more in-depth in the following
sections.

# Sidecar Injection

Modifying Kubernetes deployment manifests to bundle proxies as sidecars
with each pod is both a burden to development teams, error-prone and not
maintainable.

Part of Istio\'s codebase is dedicated to providing the capability to
automatically modify Kubernetes deployment manifests to include sidecars
with each pod.

This capability is exposed in two ways, the first and simpler mechanism
is known as *manual sidecar injection*, and the second is
called *automatic sidecar injection*.

## Manual sidecar injection

Istio has a command-line interface (CLI) named **istioctl** with the
subcommand **kube-inject**. The subcommand processes the original
deployment manifest to produce a modified manifest with the sidecar
container specification added to the pod (or pod template)
specification. The modified output can then be applied to a Kubernetes
cluster with the **kubectl apply -f** command.

With manual injection, the process of altering the manifests is
explicit.

## Automatic sidecar injection

With automatic injection, the bundling of the sidecar is made
transparent to the user.

This process relies on a Kubernetes feature known as [Mutating Admission
Webhooks](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook),
a mechanism that allows for the registration of a webhook that can
intercept the application of a deployment manifest and mutate it before
the final, modified specification is applied to the Kubernetes cluster.

The webhook is triggered according to a simple convention, where the
application of the label **istio-injection=enabled** to a Kubernetes
namespace governs whether the webhook should modify any deployment or
pod resource applied to that namespace to include the sidecar.

In the next chapter, after installing Istio, you will work through a lab
where you will deploy an application using automatic sidecar injection.

# Routing Application Traffic Through the Sidecar

With the sidecar deployed, the next problem is ensuring that the proxy
transparently captures the traffic. The outbound traffic should be
diverted from its original destination to the proxy, and inbound traffic
should arrive at the proxy before the application has a chance to handle
the incoming request.

This is performed by
applying [iptables](https://en.wikipedia.org/wiki/Iptables) rules. The
video by Matt Turner titled [*Life of a Packet through
Istio*](https://youtu.be/oZrZlx2fmcM) explains elegantly how this
process works.

In addition to the Envoy sidecar, the sidecar injection process injects
a Kubernetes init container. This **init-container** is a process that
applies these **iptables** rules before the Pod containers are started.

Today Istio provides two alternative mechanisms for configuring a Pod to
allow Envoy to intercept requests. The first is the
original **iptables** method and the second uses a [Kubernetes CNI
plugin](https://istio.io/latest/docs/setup/additional-setup/cni/).

# Assigning Workloads an Identity

The basis for a secure mesh is strong identity. We often associate the
concept of identity with an end user. But services also bear identity.
For example, when shopping on barnesandnoble.com, the server offers your
browser a certificate that allows it to assert the server\'s identity.

In Istio, each workload is assigned an X.509 cryptographic identity that
adheres to the [SPIFFE](https://spiffe.io/) (Secure Production Identity
Framework for Everyone) framework. 

Based on the SPIFFE framework, Istio encodes a SPIFFE ID into a
service\'s certificate.  The SPIFFE ID is a URL in the
form **spiffe://\<trust domain\>/\<workload identifier\>**.

In Istio, the trust domain value is typically drawn from the Kubernetes
cluster\'s domain, while the workload identity is a combination of the
service\'s namespace and service account fields.

Inside each sidecar, an Istio agent bootstraps Envoy and the service
identity by sending a certificate signing request (CSR) to Istio, and
making the resulting signed certificate accessible to Envoy securely
(via its xDS API).

This course dedicates an entire chapter to Istio security.

# Configuring Envoy

When an application makes a call to another service, that call is now
intercepted by its sidecar. But how does Envoy know how to route that
request? In the other direction, when a request arrives from another
service at a sidecar, how does Envoy know whether that request should be
allowed through?

The job of configuring the proxies with all the information they need to
handle both incoming and outgoing traffic falls to the Istio control
plane.

It is important to point out that only Envoy is in the path of live
requests and responses between services; the Istio control plane is not.

Let us explore a number of simple scenarios in order to better
understand the kind of configuration that the sidecars require.

-   -   1.  Imagine a sidecar for an instance of service A intercepting
            an outgoing request to service B. Service B may be backed by
            a Kubernetes Deployment with, say, three replicas. Service
            A\'s sidecar must know about all three endpoints: their
            network address, whether they\'re healthy, optionally the
            desired load balancing strategy when calling service B, and
            optionally other network configuration parameters such as
            request timeouts, number of retries, outlier detection, and
            more.

        2.  Imagine an operator specifying that all mesh traffic should
            be encrypted. The sidecar needs to be aware of this
            configuration in order to decide whether or not to upgrade
            the connection.

        3.  Imagine a situation where we\'re doing A/B testing. We have
            two subsets of service B\'s endpoints, with a rule to send
            certain types of requests to one subset and the rest to the
            other. Those subsets and rules must be communicated to the
            Envoy sidecar in order for it to adhere to this policy.

Imagine a service mesh with over a hundred microservices, and your job
is to configure each sidecar manually. That proposition is untenable.
This, in a nutshell, is the job that Istio performs. One could say that
Istio automates the configuration of all sidecars in the mesh to do
their job of routing traffic according to a defined network policy,
security policy, routing policy, and so on.

One point to appreciate is that the configuration of sidecars is not a
one-time, static operation. It\'s a dynamic reconciliation process,
because Kubernetes is a dynamic environment.

Imagine a scenario where a deployment is auto-scaled from two to three
replicas. Information about the newly-created service endpoint must be
communicated to all the sidecars in the mesh, so that the new endpoint
can join the pool of load balancing endpoints that can be reached from
other services.

This brings us to a final and important point about Envoy: Envoy has the
ability to receive configuration updates via API and to reload its
configuration \"live\", without requiring a restart. This API is known
as Envoy\'s discovery API, often abbreviated *xDS*.

Istio is then the control plane that continuously pushes configuration
updates to sidecars each time the mix or number of services in the mesh
changes, or each time we update policies that affect the mesh.

# Envoy at the Edge

No service mesh is an island.

Envoy, after all, is a proxy, and Istio leverages Envoy not only for
proxying requests within the mesh, but also as the mechanism for
handling ingress and egress traffic (i.e., traffic coming from a source
outside the mesh, or traffic bound to a destination outside the mesh).

Indeed, there exist today multiple open-source and commercial
implementations of Kubernetes Ingress controllers built on
Envoy. [[Contour]{.underline}](https://projectcontour.io/) and [[Emissary-ingress]{.underline}](https://www.getambassador.io/products/api-gateway/) are
two examples.

***Aside**:  The Envoy project recently [[announced the Envoy Gateway
project]{.underline}](https://www.cncf.io/blog/2022/05/16/introducing-envoy-gateway/),
a collaborative effort to develop an open-source solution for Ingress
based on Envoy. *

Two additional important components of the Istio architecture are
Istio\'s Ingress Gateway and its Egress Gateway. Both are based on
Envoy. They support the original Kubernetes Ingress CRD. However, Istio
provides its own Gateway Custom Resource Definition (CRD) for
configuring ingress and egress more flexibly.

We delve into these topics in more detail in the chapter on traffic
management.

The illustration below captures all of the Istio components, including
the edge gateways.

 

![The Istio architecture, depicting all
components](https://github.com/nadidurna/Istio/blob/master/images/image2.png)
 

In the next chapter, we get practical and explain how to install Istio
on a Kubernetes cluster and begin the journey of exploration.

[\
[Chapter 2. Installing
Istio]{.underline}](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS144x+3T2022/block-v1:LinuxFoundationX+LFS144x+3T2022+type@sequential+block@1a4e5f79ddd146c38b53d81e46115bc6)

# Chapter Overview

Now that we understand the high-level architecture of Istio, we can dive
into the installation. In this chapter, we will explain the different
approaches one can take for installing Istio service mesh to a
Kubernetes cluster. We will learn about the Istio Operator API, Istio
installation profiles, and Helm. 

In the lab, we will show how to download Istio and install it to the
Kubernetes cluster using Istio CLI. We will also show how to update an
existing installation of Istio and how to uninstall it.

# Learning Objectives

By the end of this chapter, you should be able to:

-   Discuss different ways to install Istio on a Kubernetes cluster.

-   Understand the basics of the Istio Operator API.

-   Understand the different Helm charts used for installation.

-   Understand the differences between Istio installation profiles.

# Installation Configuration Profiles

Istio service mesh has numerous configuration settings that operators
can update before installing Istio. To group the most common
configuration settings into a higher-level abstraction, Istio uses the
concept of configuration profiles. 

The configuration profiles contain different configuration settings for
the control plane as well as the data plane of Istio. The installation
configuration profiles are expressed through the Istio Operator API and
the IstioOperator resource.

Six configuration profiles are currently available, as shown in the list
below. To get an up-to-date list of Istio configuration profiles, run
the **istioctl profile list** command.

-   -   -   **Default profile\
            **The default profile is meant for production deployments
            and deployments of primary clusters in multi-cluster
            scenarios. It deploys the control plane and ingress gateway.

        -   **Demo profile**\
            The demo profile is intended for demonstration deployments.
            It deploys the control plane and ingress and egress gateways
            and has a high level of tracing and access logging enabled.

        -   **Minimal profile**\
            The minimal profile is equivalent to the default profile but
            without the ingress gateway. It deploys the control plane.

        -   **External profile**\
            The external profile is used for configuring remote clusters
            in a multi-cluster scenario. It does not deploy any
            components.

        -   **Empty profile**\
            The empty profile is used as a base for custom
            configuration. It does not deploy any components.

        -   **Preview profile**\
            The preview profile contains experimental features. It
            deploys the control plane and ingress gateway.

To install Istio using the Istio CLI, we can use the **\--set** flag and
specify the profile like this:

**istioctl install \--set profile=demo**

Later, we will cover how to install and customize Istio by creating an
IstioOperator resource and installing it using the Istio CLI. We will
also cover how to use [[Helm]{.underline}](https://helm.sh/) and deploy
the Istio Helm charts.

# Exploring the Configuration Profile Details

The Istio CLI offers convenience commands that allow us to get a full
dump of Istio configuration profiles and the differences between the two
profiles.

For example, to get the complete YAML configuration of the demo profile,
we can run the following command:

**istioctl profile dump demo**

The command will output the YAML of the IstioOperator resource to the
console. To get the configuration for a different profile, replace the
profile name in the above command.

Another useful command when exploring the different profiles is
the **diff** command. The **diff** command lists differences between the
configuration profiles.

For example, this command compares the default profile with the demo
profile:

**istioctl profile diff demo default**

The output is in the traditional diff format, including lines marked
with **+** or **-** to indicate the differences between the two
profiles. For example:

**The difference between profiles:**\
** apiVersion: install.istio.io/v1alpha1**\
** kind: IstioOperator**\
** metadata:**\
**   creationTimestamp: null**\
**   namespace: istio-system**\
** spec:**\
**   components:**\
**     base:**\
**       enabled: true**\
**     cni:**\
**       enabled: false**\
**     egressGateways:**\
**-    - enabled: true**\
**-      k8s:**\
**-        resources:**\
**-          requests:**\
**-            cpu: 10m**\
**-            memory: 40Mi**\
**+    - enabled: false**\
**       name: istio-egressgateway**\
**     ingressGateways:**\
**     - enabled: true**

The above output shows that the **default** profile has the egress
gateway disabled (**enabled: false**) while the **demo** profile enables
it.

# Using Istio Operator API

The Istio Operator API and the [IstioOperator
resource]{.underline} allow us to install and configure Istio on a
Kubernetes cluster. At a high level, we can separate the configuration
in the IstioOperator resource into the following sections:

1.  **Global**\
    The global section allows us to configure the profile name, root
    Docker image path, image tags, namespace, revision, and so on.

2.  **Mesh configuration (meshConfig)**\
    The **meshConfig** section includes the configuration of the control
    plane components. For example, in this section, we can configure
    access log format, log encoding, set up default proxy configuration,
    discovery selectors, trust domains, and more.

3.  **Component configuration (components)**\
    The **components** section allows us to enable or disable individual
    components, install additional components (multiple ingresses or
    egress gateways, for example), and configure Kubernetes resource
    settings for individual components. For example, for each component
    (e.g., pilot, ingress, or egress gateways), we can configure the CPU
    and memory requests and limits, annotations, labels, replica counts,
    and other settings in the Kubernetes resources.

Within the IstioOperator resource, we specify the desired state of Istio
components. We can apply or deploy the resource to the Kubernetes
cluster using the Istio CLI and the **install** command. 

Once we have created the IstioOperator resource, we can install it on
the cluster using the **install** command:

**istioctl install -f my-operator-resource.yaml**

# Using Helm

[[Helm]{.underline}](https://helm.sh/) is a Kubernetes package manager
that helps install and upgrade complex applications on Kubernetes. A
fundamental building block of Helm is a Helm Chart, a collection of YAML
manifests.

When using Helm, there are three different Helm charts we need to be
aware of, listed in the order we would install them:

1.  1.  1.  **Base chart (istio/base)**\
            The **base** chart includes cluster-wide resources such as
            the validating webhook configuration resource, service
            accounts, cluster roles and bindings, and other resources to
            ensure backward compatibility.

        2.  **Istiod chart (istio/istiod)**\
            The **istiod** chart contains Istio's control plane
            installation. It includes the **istiod** deployment and
            service, mutating webhook configuration (facilitates
            automatic sidecar injection into deployments), and other
            resources for the control plane.

        3.  **Gateway chart (istio/gateway)**\
            The **gateway** chart is used for deploying ingress and
            egress gateways to the cluster. It includes the service and
            deployment resources and other supporting resources. 

Before installing the charts, we need to manually create the root
namespace (i.e., **istio-system**) and use the **helm install** command
to install the individual charts. Typically, we install
the **base** and **istiod** charts to the **istio-system** namespace
and **gateway** charts into separate namespaces.

Here is how we could install the **istiod** chart, for example:

**helm install istiod istio/istiod -n istio-system**

The first parameter in the above command is the release name, followed
by the chart name.

To check on the installation progress, we can pass the release name
(e.g.istiod) to the status command: 

**helm status istiod -n istio-system**

# Using Helm: Updating the Configuration

We can provide custom configuration settings to individual Helm charts
at installation time. To review settings that can be updated, we can use
the **show values** command like this:

**helm show values istio/istiod**\
**#.Values.pilot for discovery and mesh wide config**

**\## Discovery Settings**\
**pilot:**\
**  autoscaleEnabled: true**\
**  autoscaleMin: 1**\
**  autoscaleMax: 5**\
**  replicaCount: 1**\
**  rollingMaxSurge: 100%**\
**  rollingMaxUnavailable: 25%**

**  hub: \"\"**\
**  tag: \"\"**

**  \# Can be a full hub/image:tag**\
**  image: pilot**\
**  traceSampling: 1.0**

**  \# Resources for a small pilot install**\
**  resources:**\
**    requests:**\
**      cpu: 500m**\
**      memory: 2048Mi**

**  env: {}**

**  cpu:**\
**    targetAverageUtilization: 80**\
**...**

Similarly, we can get the values of other Helm charts. To apply the
configuration updates to individual chart installations, we would create
a separate YAML file with the configuration value we want to update.
Then, use the **install** command with the **-f** flag to install the
individual chart with the provided configuration settings:

**helm install istiod istio/istiod -n istio-system -f
my-config-values.yaml**

# Using Helm: Uninstalling Istio

Uninstalling Istio that was deployed using Helm involves listing all
installed Istio charts using the **helm ls** command and then running
the **helm delete** command.

For example:

**helm delete istiod -n istio-system**

Once all releases are deleted, make sure to delete the namespaces if not
needed anymore.

# Installing Istio: Prerequisites

We will install Istio on your Kubernetes cluster in this lab using the
Istio Operator API.

To install Istio, we will need a running instance of a Kubernetes
cluster. All cloud providers have a managed Kubernetes cluster offering
that can be used for this purpose.

Alternatively, you can run a Kubernetes cluster locally on your computer
using one of the following platforms:

-   -   -   [[Minikube]{.underline}](https://istio.io/latest/docs/setup/platform-setup/minikube/)

        -   [[Docker
            Desktop]{.underline}](https://istio.io/latest/docs/setup/platform-setup/docker/)

        -   [[kind]{.underline}](https://istio.io/latest/docs/setup/platform-setup/kind/)

        -   [[MicroK8s]{.underline}](https://istio.io/latest/docs/setup/platform-setup/microk8s/)

When using a local Kubernetes cluster such as Minikube, ensure your
computer meets the minimum requirements for Istio installation (e.g.,
16384 MB RAM and 4 CPUs). Also, ensure the Kubernetes cluster version is
v1.20.2 or higher.

Lab exercises in this course have been tested in a GCP environment. On
GCP, the following command will provision a GKE cluster of adequate size
for the course (though a cluster with a small number of worker nodes
should work just fine):

**gcloud container clusters create my-istio-cluster \\**\
**  \--cluster-version latest \\**\
**  \--machine-type \"n1-standard-2\" \\**\
**  \--num-nodes \"3\" \\**\
**  \--network \"default\"**

If using a cloud provider like GCP, AWS, or Azure, you should be able to
complete the lab exercises using the free tier or credits provided to
you. However, you may incur charges if you exceed the credits initially
allocated by the cloud provider.

# Install Kubernetes CLI (kubectl)

If you need to install the Kubernetes CLI, follow [these
instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

We can run the command **kubectl version** to check that the CLI was
installed. You should see output similar to this:

**\$ kubectl version \--short\
Client Version: v1.24.3**\
**Kustomize Version: v4.5.4**\
**Server Version: v1.22.10-gke.600**

# Download Istio

Throughout this course, we will be using **Istio 1.14.3**. The first
step to installing Istio is downloading the Istio CLI (**istioctl**),
installation manifests, samples, and tools.

The easiest way to install the latest version is to use
the **downloadIstio** script:

1.  1.  1.  Open a terminal window and navigate to the folder where you
            want to download Istio

        2.  Run the download script:\
            \
            **\$ curl -L https://istio.io/downloadIstio \|
            ISTIO_VERSION=1.14.3 sh -**

The Istio release is downloaded and unpacked to the folder
called **istio-1.14.3**. 

***Note:** If your organization requires FIPS-certified distributions of
Istio, you can read more about
them [[here]{.underline}](https://tetr8.io/istio-fips). *

To run the  **istioctl** from any folder, we should include its
fully-qualified path in the **PATH** environment variable, as shown
here:

**cd istio-1.14.3**\
**export PATH=\$PWD/bin:\$PATH**

To check that the Istio CLI is on the path, run **istioctl version**.
You should see output resembling this:

**istioctl version**\
**no running Istio pods in \"istio-system\"**\
**1.14.3**

# Install Istio

To install Istio, we have to create the *IstioOperator* resource and
specify the configuration profile we want to use.

Create a file called **demo-profile.yaml** with the following contents:

**apiVersion: install.istio.io/v1alpha1**\
**kind: IstioOperator**\
**metadata:**\
**  namespace: istio-system**\
**  name: demo-installation**\
**spec:**\
**  profile: demo**

***Note: **You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

The last thing we need to do is to deploy the *IstioOperator* resource
using the **istioctl install** command:

**istioctl install -f demo-profile.yaml**

**This will install the Istio 1.14.3 demo profile with \[\"Istio core\"
\"Istiod\" \"Ingress gateways\" \"Egress gateways\"\] components into
the cluster. Proceed? (y/N) y\
✔ Istio core installed\
✔ Istiod installed\
✔ Egress gateways installed\
✔ Ingress gateways installed\
✔ Installation complete\
Making this installation the default for injection and validation.\
Thank you for installing Istio 1.14.  Please take a few minutes to tell
us about your install/upgrade experience! 
https://forms.gle/yEtCbt45FZ3VoDT5A**

Another option we have to install Istio using any configuration profile
is to use the **istioctl install** command with the **\--set** flag, for
example:

**istioctl install \--set profile=demo**

In both cases, we will get prompted to proceed with the installation,
and once we confirm, the Istio service mesh will be deployed.

To check the deployed resource, we can look at the status of the pods in
the **istio-system** namespace:

**\$ kubectl get po -n istio-system**

**NAME                                    READY   STATUS    RESTARTS 
 AGE\
istio-egressgateway-6db9994577-sn95p    1/1     Running   0         
79s\
istio-ingressgateway-58649bfdf4-cs4fk   1/1     Running   0         
79s\
istiod-dd4b7db5-nxrjv                   1/1     Running   0         
111s**

# Enable Sidecar Injection

As we learned in the previous section, service mesh needs the sidecar
proxies running alongside each application.

To inject the sidecar proxy into an existing Kubernetes deployment, we
can use **kube-inject** sub-command of the Istio CLI.

Alternatively, we can enable automatic sidecar injection on any
Kubernetes namespace. By labeling the namespace with the
label **istio-injection=enabled**, the Istio control plane will monitor
that namespace for new Kubernetes deployments. It will automatically
intercept the deployments and inject Envoy sidecars into each pod.

Enable automatic sidecar injection on the default namespace by setting
the **istio-injection** label:

**kubectl label namespace default istio-injection=enabled**\
**namespace/default labeled**

To check that the namespace is labeled, run the command below:

**\$ kubectl get namespace -L istio-injection\
NAME              STATUS   AGE    ISTIO-INJECTION\
default           Active   114m   enabled\
istio-system      Active   29m\
kube-node-lease   Active   114m\
kube-public       Active   114m\
kube-system       Active   114m**

The default namespace should be the only one with the
value **enabled** in the **ISTIO-INJECTION** column.

We can now try creating a Kubernetes deployment in the default namespace
and observe the injected proxy. 

We will create a deployment called **my-nginx** with a single container
using the Docker image **nginx**:

**kubectl create deploy my-nginx \--image=nginx**\
**deployment.apps/my-nginx created**

List the pods in the default namespace, and notice that there are two
containers ready in the pod as specified by the value **2/2** in
the **READY** column:

**kubectl get po**\
**NAME                        READY   STATUS    RESTARTS   AGE\
my-nginx-6b74b79f57-gh5fp   2/2     Running   0          62s**

Similarly, describing the pod shows how Kubernetes created both
an **nginx** container and an **istio-proxy** container. The latter was
injected by the Istio control plane.

**kubectl describe po my-nginx-6b74b79f57-gh5fp\
...**\
**Events:**\
**  Type    Reason     Age   From               Message**\
**  \-\-\--    \-\-\-\-\--     \-\-\--  \-\-\--             
 \-\-\-\-\-\--**\
**  Normal  Scheduled  70s   default-scheduler  Successfully assigned
default/my-nginx-6b74b79f57-gh5fp to
gke-cluster-1-default-pool-c2743eca-sts7**\
**  Normal  Pulled     69s   kubelet            Container image
\"docker.io/istio/proxyv2:1.14.\" already present on machine**\
**  Normal  Created    69s   kubelet            Created container
istio-init**\
**  Normal  Started    69s   kubelet            Started container
istio-init**\
**  Normal  Pulling    68s   kubelet            Pulling image
\"nginx\"**\
**  Normal  Pulled     64s   kubelet            Successfully pulled
image \"nginx\" in 4.334525037s**\
**  Normal  Created    63s   kubelet            Created container
nginx**\
**  Normal  Started    63s   kubelet            Started container
nginx**\
**  Normal  Pulled     63s   kubelet            Container image
\"docker.io/istio/proxyv2:1.14.3\" already present on machine**\
**  Normal  Created    63s   kubelet            Created container
istio-proxy**\
**  Normal  Started    63s   kubelet            Started container
istio-proxy**

To clean up and delete the Kubernetes deployment we created, run the
following command:

**kubectl delete deployment my-nginx**\
**deployment.apps \"my-nginx\" deleted**

# Updating the Installation

To update the installation, we can modify the existing IstioOperator
resource we deployed and re-apply it to the cluster. For example, if we
wanted to remove the egress gateway, we could update the IstioOperator
resource like this:

**apiVersion: install.istio.io/v1alpha1**\
**kind: IstioOperator**\
**metadata:**\
**  namespace: istio-system**\
**  name: demo-istio-install**\
**spec:**\
**  profile: demo**\
**  components:**\
**    egressGateways:**\
**    - name: istio-egressgateway**\
**      enabled: false**

Save the above YAML to **iop-egress.yaml** and apply it using **istioctl
install -f iop-egress.yaml**.

Just like before, we will get prompted to proceed with the installation.
If you list the pods in the **istio-system** namespace, you will notice
that the egress gateway is no longer in the list.

Another option for updating the Istio installation is to create a
separate IstioOperator resource. That way, we can have one resource for
the base installation and then separately apply different operators
using an empty installation profile.

For example, we could create a separate IstioOperator resource that only
deploys an internal ingress gateway. Note that this example assumes you
use a Kubernetes cluster running on GCP.

**apiVersion: install.istio.io/v1alpha1**\
**kind: IstioOperator**\
**metadata:**\
**  name: internal-gateway-only**\
**  namespace: istio-system**\
**spec:**\
**  profile: empty**\
**  components:**\
**    ingressGateways:**\
**      - namespace: some-namespace**\
**        name: ilb-gateway**\
**        enabled: true**\
**        kabel:**\
**          istio: ilb-gateway**\
**        k8s:**\
**          serviceAnnotations:**\
**            networking.gke.io/load-balancer-type: \"Internal\"**

# Uninstall Istio

To completely remove the Istio installation, we can use the uninstall
command:

**istioctl x uninstall \--purge**

[[Chapter 3.
Observability]{.underline}](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS144x+3T2022/block-v1:LinuxFoundationX+LFS144x+3T2022+type@sequential+block@aeb3a6c2bfcf4853b4d5197ba0010a22)

# Chapter Overview

Istio as a platform addresses and provides significant value in the four
areas of traffic management, security, observability, and
extensibility.  In this chapter, we explore observability.

# Learning Objectives

By the end of this chapter, you should be able to:

-   -   -   Understand the difference between monitoring and
            observability.

        -   Understand the value that Istio provides with respect to
            metrics collection and observability.

        -   Learn how metrics are collected in Istio.

        -   Learn what specific metrics are collected.

        -   Learn how to query the Prometheus metrics store using
            promQL.

        -   Gain familiarity with the Grafana monitoring dashboards for
            Istio.

        -   Understand the concept and purpose of distributed tracing.

        -   Visualize traffic flow with the Kiali console.

# From Monitoring Monoliths to Observability for Microservices

Monoliths have existed for much longer than microservices, and therefore
monolith monitoring and diagnosis tooling is more mature. Profilers can
display memory consumption and help detect memory leaks. We can monitor
CPU utilization and other vitals specific to a running process,
including memory, threads, and garbage collection operations. Debuggers
allow you to inspect a section of code, display the entire stack trace,
and allow one to view and navigate the call hierarchy easily.

Fundamentally, moving from monolith to microservices means that a call
stack that used to be a simple, in-process chain of method calls is now
a series of network hops across multiple services.

With microservices, we no longer monitor a single application. Vitals
and metrics from many services must be captured and aggregated to
provide a complete picture of a running system.

Logs complement metrics and often provide more concrete information
about what may be going on inside a particular service at a particular
point in time.

With microservices, the value of a stack trace greatly diminishes
because it is scoped to a single process. In place of stack traces, we
employ new methods that capture distributed traces that span multiple
services and, indeed, the entire request-response flow.

# Monitoring vs Observability

The term *observability* is broader than monitoring. It encompasses not
only monitoring but anything that assists system engineers in
understanding how a system behaves. Observability in microservices
includes not only the collection and visualization of metrics but also
log aggregation and dashboards that help visualize distributed traces.

Each facet of observability complements the other. Dashboards exposing
metrics may indicate a performance problem somewhere in the system.
Distributed traces can help locate the performance bottleneck to a
specific service. Finally, a service\'s logs can provide the context
necessary to determine what specifically may be the issue.

This trio: metrics, logs, and distributed traces are the foundation for
modern distributed systems observability.

# How Service Meshes Simplify and Improve Observability

Before service meshes, the burden of capturing, exposing, and publishing
metrics was on the shoulders of application developers. So was the
burden of constructing dashboards for the purpose of monitoring
application health. Not only was this an added burden, but it created a
situation where the implementation of observability from one team to the
next was not uniform. What metrics are exposed, how they are captured,
what they are named, and even what monitoring system is used could all
be different from one application to another.

With Istio, solutions to cross-functional problems such as observability
are truly orthogonal to the applications themselves. Through the Envoy
sidecar, Istio is able to collect metrics and expose scrape endpoints
that allow for the collection of a uniform set of metrics for all
microservices. Istio further allows for the development of dashboards
that are uniform across all services.

The burden is lifted from the shoulders of the developers responsible
for a given microservice, and the end result is a uniform treatment of
metrics collection and observability across the entire platform.

# How Istio Exposes Workload Metrics (1)

In this lab, we will deploy an application to the mesh, review the
Prometheus scrape endpoint, and study the metrics that are exposed for
collection.

Ensure that Istio is deployed with the demo profile:

**istioctl install \--set profile=demo**

***Aside:** why the demo profile?*\
*Typically in production, sampling one percent of traces is sufficient
for capturing multiple distinct and representative distributed traces.
Istio\'s demo installation profile configures distributed trace sampling
at 100%, to facilitate capturing traces in a test environment that
doesn\'t see much traffic in the first place and where there is little
concern for performance.*

Next, ensure that the default namespace is labeled for sidecar
injection:

**kubectl label ns default istio-injection=enabled**

Verify that the label has been applied with the following command:

**kubectl get ns -Listio-injection**

Navigate to the base directory of your Istio distribution:

**cd \~/istio-1.14.3**

Deploy the [BookInfo]{.underline} sample application that is bundled
with the Istio distribution.

**kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml**

Finally, deploy the
bundled [[sleep]{.underline}](https://github.com/istio/istio/tree/master/samples/sleep) sample
service:

**kubectl apply -f samples/sleep/sleep.yaml**

# How Istio Exposes Workload Metrics (2)

At this point, verify that the pods running in the default namespace
each have two containers: the workload proper and the Envoy sidecar.

**kubectl get pod**

**NAME                              READY   STATUS    RESTARTS   AGE**\
**details-v1-b48c969c5-pjcvv        2/2     Running   0          41s**\
**productpage-v1-74fdfbd7c7-sc2v2   2/2     Running   0          39s**\
**ratings-v1-b74b895c5-phvs7        2/2     Running   0          41s**\
**reviews-v1-68b4dcbdb9-tpv95       2/2     Running   0          40s**\
**reviews-v2-565bcd7987-sdk2j       2/2     Running   0          40s**\
**reviews-v3-d88774f9c-w8g24        2/2     Running   0          40s**\
**sleep-5887ccbb67-t9f9k            2/2     Running   0          34s**

With the BookInfo application running, make a token HTTP request against
its **productpage** service.

First, identify the name of the pod corresponding to
the **productpage** deployment:

**PRODUCTPAGE_POD=\$(kubectl get pod -l app=productpage
-ojsonpath=\'{.items\[0\].metadata.name}\')**

Next, use **curl** to call the **productpage** service\'s main page from
the running **sleep** pod:

**SLEEP_POD=\$(kubectl get pod -l app=sleep
-ojsonpath=\'{.items\[0\].metadata.name}\')**

**kubectl exec \$SLEEP_POD -it \-- curl productpage:9080/productpage \|
head**

The output should show the start of the HTML response from
the **productpage** service, like so:

**\<!DOCTYPE html\>**\
**\<html\>**\
**  \<head\>**\
**    \<title\>Simple Bookstore App\</title\>**\
**\<meta charset=\"utf-8\"\>**\
**\<meta http-equiv=\"X-UA-Compatible\" content=\"IE=edge\"\>**\
**\<meta name=\"viewport\" content=\"width=device-width,
initial-scale=1.0\"\>**

**\<!\-- Latest compiled and minified CSS \--\>**\
**\<link rel=\"stylesheet\"
href=\"static/bootstrap/css/bootstrap.min.css\"\>**

The name of the sidecar container in Istio is **istio-proxy**. This can
be determined with the following command:

**kubectl get pod \$PRODUCTPAGE_POD
-ojsonpath=\'{.spec.containers\[\*\].name}\'**

**productpage istio-proxy**

One of the benefits of running a sidecar is that it can expose a metrics
collection endpoint, also known as the Prometheus \"scrape endpoint,\"
on behalf of the workload it proxies.

We can query the scrape endpoint as follows:

**kubectl exec \$PRODUCTPAGE_POD -c istio-proxy curl
localhost:15090/stats/prometheus**

Another way to access this endpoint is via the Envoy administrative
dashboard corresponding to a specific pod or deployment.

Use the below **istioctl dashboard** command to open the Envoy dashboard
web page:

**istioctl dashboard envoy deploy/productpage-v1.default**

In the dashboard, click on the hyperlink in the first column titled
\"stats/prometheus.\"

 

![Envoy dashboard in the
browser](https://github.com/nadidurna/Istio/blob/master/images/image3.png)

# How Istio Exposes Workload Metrics (3)

The resulting output is rather lengthy, showing a list of over a hundred
distinct metrics. Some of the salient standard metrics that Istio
collects about services include request count
(**istio_requests_total**), request duration, request size, and response
size (see [Istio Standard
Metrics](https://istio.io/latest/docs/reference/config/metrics/) for
more information).

**\# TYPE envoy_cluster_assignment_stale counter**

**envoy_cluster_assignment_stale{cluster_name=\"xds-grpc\"} 0**

**\# TYPE envoy_cluster_assignment_timeout_received counter**

**envoy_cluster_assignment_timeout_received{cluster_name=\"xds-grpc\"}
0**

**\# TYPE envoy_cluster_bind_errors counter**

**envoy_cluster_bind_errors{cluster_name=\"xds-grpc\"} 0**

**\# TYPE envoy_cluster_default_total_match_count counter**

**envoy_cluster_default_total_match_count{cluster_name=\"xds-grpc\"} 1**

**\# TYPE envoy_cluster_http2_dropped_headers_with_underscores counter**

**envoy_cluster_http2_dropped_headers_with_underscores{cluster_name=\"xds-grpc\"}
0**

**...**

In the next lab, we study how these metrics are collected and stored in
Prometheus.

# See the Metrics Collected in Prometheus (1)

In this lab, we deploy Prometheus and run some queries directly against
the Prometheus server.   Prometheus is configured to collect metrics
from all workloads at a regular 15-second interval.

The Istio distribution bundles a manifest for deploying Prometheus with
the configuration necessary to collect the metrics from the scrape
endpoints decorating each workload.

***Aside: **Prometheus is also configured to gather metrics pertaining
to the Istio control plane, ***istiod***. In a subsequent lab, we will
explore those metrics directly from Grafana.*

Run the following command, which will deploy Prometheus to
the **istio-system** namespace:

**kubectl apply -f samples/addons/prometheus.yaml**

With Prometheus now running and collecting metrics, send another request
to the **productpage** service:

**SLEEP_POD=\$(kubectl get pod -l app=sleep
-ojsonpath=\'{.items\[0\].metadata.name}\')**

**kubectl exec \$SLEEP_POD -it \-- curl productpage:9080/productpage \|
head**

Run the following command, which Istio provides as a convenience to
expose the Prometheus server\'s dashboard locally:

**istioctl dashboard prometheus**

Enter the metric name **istio_requests_total** into the search field and
press the button labeled *Execute*.

 

![Prometheus dashboard displaying the metric
istio_requests_total](https://github.com/nadidurna/Istio/blob/master/images/image4.png)

 

The output can be viewed either in tabular form or graph form.
Prometheus collects these metrics along with an additional set of labels
that provide context and allow for querying.

For example, to find out the subset of requests that returned an HTTP
200 response code, the query would be:

**istio_requests_total{response_code=\"200\"}**

By collecting this extra metadata with each metric, we can obtain
answers to many interesting questions. For example, to locate the call
we made earlier from the **sleep** pod to the **productpage** service,
the query would be:

**istio_requests_total{source_app=\"sleep\",destination_app=\"productpage\"}**

# See the Metrics Collected in Prometheus (2)

A more interesting question that applies to COUNTER-type metrics in
general, is the rate of incoming requests (over a particular time
window, say the last 5 minutes) against a particular service:

**rate(istio_requests_total{destination_app=\"productpage\"}\[5m\])**

 

![Rate of requests to the productpage
service](https://github.com/nadidurna/Istio/blob/master/images/image5.png)

 

Look at the output from the **Graph** tab.

Since there currently exists no load against our service, the rate
should be zero.

If, on the other hand, we query the **productpage** service every 1-2
seconds, like so\...

**while true; do kubectl exec \$SLEEP_POD -it \-- curl
productpage:9080/productpage; sleep 1; done**

..then, within a couple of minutes, the rate of requests will rise from
zero to a value between 0.5 and 1.0.

Prometheus\' PromQL query language is powerful and can help diagnose
issues, as illustrated by Karl Stoney in his blog entry Istio: [503\'s
with UC\'s and TCP Fun
Times](https://karlstoney.com/2019/05/31/istio-503s-ucs-and-tcp-fun-times/).

But Prometheus queries are no substitute for a set of properly designed
monitoring dashboards, to which we turn our attention in the next lab.

# Grafana Dashboards for Istio (1)

[[Grafana]{.underline}](https://grafana.com/) is a popular open-source
tool that makes it easy to construct custom monitoring dashboards from a
backing metrics source. Grafana has built-in support for Prometheus.

The Istio project team has developed a set of Grafana dashboards
specifically designed for monitoring a service mesh.

Deploy Grafana with the following command:

**kubectl apply -f samples/addons/grafana.yaml**

Launch the Grafana UI with the following command:

**istioctl dashboard grafana**

To view the Istio dashboards in Grafana, navigate from the main page
with the aid of the navigation bar on the left-hand side of the screen.
Under *Dashboards* select *Browse*. To the right of the folder
labeled *Istio*, click on the link captioned *Go to folder*.

Inside that folder, you will find six dashboards:

-   -   -   **Mesh**: provides a high-level overview of the health of
            services in the mesh.

        -   **Service**: for monitoring a specific service. The metrics
            shown here are aggregated from multiple workloads.

        -   **Workload**: this allows you to inspect the behavior of a
            single workload.

        -   **Control Plane**: designed to monitor the health
            of **istiod**, the Control plane itself. It helps determine
            whether the control plane is healthy and able to synchronize
            the Envoy sidecars to the state of the mesh.

        -   **Performance**: this allows you to monitor the resource
            consumption of **istiod** and the sidecars.

        -   **Wasm Extension**: For monitoring the health of custom Web
            Assembly extensions deployed to the mesh.

# Grafana Dashboards for Istio (2)

Let us send some traffic to the **productpage** service so that we have
something to observe.

Capture the name of the **sleep** pod:

**SLEEP_POD=\$(kubectl get pod -l app=sleep
-ojsonpath=\'{.items\[0\].metadata.name}\')**

Run the following simple script to make repeated calls to
the **productpage** endpoint:

**while true; do kubectl exec \$SLEEP_POD -it \-- curl
productpage:9080/productpage; sleep 0.3; done**

Begin by navigating to the **Istio Mesh Dashboard**. This dashboard is a
perfect place to get our bearings and see what services are running,
inspect their health, and view global stats such as the global request
volume.

 

![The Istio Mesh Dashboard in
Grafana](https://github.com/nadidurna/Istio/blob/master/images/image6.png)

 

You should see a global request volume of 2-3 operations per second, no
4xx or 5xx errors, and all services should show a 100% success rate.
Some other metrics will show \"N/A\" (\"Not Applicable\") for the
moment. Later in this course, you will define custom Istio resources,
including Gateways, Virtual Services, and Destination Rules, and at that
time, the count of each type of resource will display on this dashboard.

# Grafana Dashboards for Istio (3)

Next, visit the **Istio Service Dashboard**, select the service
named **productpage.default.svc.cluster.local**, and expand
the *General* panel. There you will find the typical *golden signals*,
including request volume, success rate (or errors), and request
duration. The other two panels, *Client Workloads* and *Service
Workloads,* break down incoming requests to this service by source and
by destination, respectively.

 

![Istio service dashboard in
Grafana](https://github.com/nadidurna/Istio/blob/master/images/image7.png)

**Istio service dashboard in Grafana**

 

The final and most specific dashboard we will discuss is the **Istio
Workload Dashboard**, which allows one to focus on a single workload.
For example, the **reviews** service consists of three different
versions: v1, v2, and v3.

Imagine, for example, that we recently deployed v3 of
the **reviews** service and wish to determine how it is performing. We
could navigate to the **Workload Dashboard**, select
the **reviews-v3** workload from the pulldown menu, and inspect its
vitals to the exclusion of other workloads that are members of the
same **reviews** service. In the **General** panel, you will find
request volume, success rate, and request duration metrics. The
subsequent two panels focus on incoming and outgoing requests from the
workload, respectively.

 

![Istio workload dashboard showing reviews-v3
workload](https://github.com/nadidurna/Istio/blob/master/images/image8.png)

**Istio workload dashboard showing reviews-v3 workload**

# Distributed Tracing

Distributed tracing is an important component of observability that
complements metrics dashboards.

The idea is to provide the capability to \"see\" the end-to-end
request-response flow through a series of microservices and to draw
important information from it.

From a view of a distributed trace, developers can discover potential
latency issues in their applications.

## Terms

The end-to-end request-response flow is known as a *trace*. Each
component of a trace, such as a single call from one service to another,
is called a *span*. Traces have unique IDs, and so do spans. All spans
that are part of the same trace bear the same trace ID.

The IDs are propagated across the calls between services in HTTP headers
whose names begin with **x-b3** and are known as *B3 trace
headers* (see [B3
Propagation](https://github.com/openzipkin/b3-propagation)).

When Envoy sidecars receive the initial request that does not contain a
B3 header and realize that this span represents the beginning of a new
trace, they assign the request a new trace ID.

However, the propagation of these headers onto other services cannot be
performed automatically by Envoy, and so developers must ensure that
they propagate these headers in upstream calls to other services
(see [Istio /
FAQ](https://istio.io/latest/about/faq/#how-to-support-tracing)). This
task is often easily accomplished by including a tracing client library
as a dependency to the services.

# Deploy Jaeger and Review Some Traces (1)

The Istio **demo** profile configures Istio with distributed tracing
turned on and with full trace sampling. In production settings, however,
trace sampling is often set to 1% so as to minimize impact on
performance.

The Envoy sidecars are configured to send their trace information to a
distributed tracing collector.

Multiple distributed tracing systems exist,
including Zipkin, [Jaeger](https://www.jaegertracing.io/),
and Lightstep. In this lab, you will use Jaeger. Though this same
exercise can be easily repeated with Zipkin or Lightstep.

Begin by deploying Jaeger with the following command, invoked from the
Istio distribution base directory:

**kubectl apply -f samples/addons/jaeger.yaml**

The resulting deployment can be seen in the **istio-system** namespace.

As in previous labs, store the name of the **sleep** pod in an
environment variable:

**SLEEP_POD=\$(kubectl get pod -l app=sleep
-ojsonpath=\'{.items\[0\].metadata.name}\')**

# Deploy Jaeger and Review Some Traces (2)

Next, run the following command to send requests to
the **productpage** service every 1-2 seconds:

**while true; do kubectl exec \$SLEEP_POD -it \-- curl
productpage:9080/productpage; sleep 1; done**

Requests will be tagged with trace and span IDs and be sent and
consequently collected by Jaeger.

To view the distributed traces, open the Jaeger dashboard:

**istioctl dashboard jaeger**

From the search form on the left-hand side of the UI, select the
service **productpage.default**, and click on the button **Find
Traces** at the bottom of the form.

From the search results, select a trace that you might wish to study.
The figure below shows a sample trace.

 

![Tracing a request through the BookInfo
application](https://github.com/nadidurna/Istio/blob/master/images/image9.png)

 

The UI displays the end-to-end trace duration (29.3 ms), the number of
services involved, the depth of the trace, and the total number of
spans.

With such a view, one can easily see exactly where time is spent, which
services might have performance issues, and whether there are ways to
improve performance by calling certain services in parallel (in a
situation where the two calls have no interdependencies).

For example, in the figure, the **productpage** service appears to call
the **details** service first, to fetch the product details, and it
isn\'t until after that call returns that the reviews service is
subsequently called. Perhaps those two service calls can take place in
parallel and the response be made to arrive faster to its original
caller?

From the view of the trace, we also learn the basic business logic: that
the **reviews** service is called and that sometimes
the **reviews** service will turn around and make a call to
the **ratings** service. All of this information is then aggregated and
displayed to the end-user on an HTML page.

# Discover the Kiali Console (1)

[Kiali](https://kiali.io/) is an open-source graphical console
specifically designed for Istio and includes numerous features.

Through alerts and warnings, it can help validate that the service mesh
configuration is correct and that it does not have any problems.

With Kiali, one can view Istio custom resources, services, workloads, or
applications.

As an alternative to drafting and applying Istio custom resources by
hand, Kiali exposes actions that allow the operator to define routing
rules, perform traffic shifting, configure timeouts and inject faults.

Kiali relies on the metrics collected in Prometheus. In addition, Kiali
has the ability to combine information from metrics, traces, and logs to
provide deeper insight into the functioning of the mesh.

One feature of Kiali that stands out is the *Graph* section, which
provides a live visualization of traffic inside the mesh.

Begin by deploying Kiali to your Kubernetes cluster:

**kubectl apply -f samples/addons/kiali.yaml**

As in previous labs, store the name of the **sleep** pod in an
environment variable:

**SLEEP_POD=\$(kubectl get pod -l app=sleep
-ojsonpath=\'{.items\[0\].metadata.name}\')**

Next, run the following command to send requests to
the **productpage** service at a 1-2 second interval:

**while true; do kubectl exec \$SLEEP_POD -it \-- curl
productpage:9080/productpage; sleep 1; done**

Finally, launch the Kiali dashboard:

**istioctl dashboard kiali**

In the UI, select the **Graph** option from the sidebar, and select
the *default* namespace.

The following picture will appear:

 

![The BookInfo application in the Kiali
console](https://github.com/nadidurna/Istio/blob/master/images/image10.png)

**The BookInfo application in the Kiali console**

 

Through the *Display* options, interesting bits of additional
information can be overlaid on the graph, including whether calls
between services are encrypted with mutual TLS, traffic rate, and
traffic animations that reflect the relative rate of requests between
services.

One can also navigate directly from the *Graph* view to a particular
service. The screenshot below is a view of the ratings service, where
one can clearly see that both reviews-v2 and reviews-v3 call this
service and that those calls are indeed using mutual TLS encryption. The
green color of the arrows linking the services indicate that requests
are succeeding with HTTP 200 response codes.

 

![Kiali service
overview](https://github.com/nadidurna/Istio/blob/master/images/image11.png)

 

Feel free to peruse through this console to discover its details
further. If you have extra time, check out the following [video
interview](https://youtu.be/Y87cDA_JFfo?t=392) with Kiali committer
Lucas Ponce.

To clean up the deployed resources, run: 

**kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml**\
**kubectl delete -f samples/sleep/sleep.yaml**

# Alternative Monitoring Solutions

Monitoring solutions are often referred to as Application Performance
Monitoring (APM tools). There exist many alternative APM tools and
solutions out in the marketplace. One popular open-source option is
the [Apache Foundation\'s Skywalking
project](https://skywalking.apache.org/).

Apache Skywalking supports monitoring service meshes. This [blog
entry](https://skywalking.apache.org/blog/2020-12-03-obs-service-mesh-with-sw-and-als/) provides
a tutorial for installing Apache Skywalking on a Kubernetes cluster and
configuring it to work with Istio.

Skywalking can be installed with the popular [Helm package
manager](https://helm.sh/). Up-to-date instructions for installing
Skywalking with Helm can be found on [Apache Skywalking\'s GitHub
repository](https://github.com/apache/skywalking-kubernetes).

Once Apache Skywalking is up and running, we can proceed to access its
dashboard, which provides features similar to some of the other
dashboards we visited in this chapter. Below is a screenshot of the
Topology view from the Apache Skywalking dashboard showing traffic
making its way through Istio\'s bookinfo sample application.

 

![Apache Skywalking Dashboard displaying traffic coursing through the
services of the bookinfo sample
application](https://github.com/nadidurna/Istio/blob/master/images/image12.png)

Apache Skywalking Dashboard displaying traffic coursing through the
services of the bookinfo sample application**

[[Chapter 4. Traffic
Management]{.underline}](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS144x+3T2022/block-v1:LinuxFoundationX+LFS144x+3T2022+type@sequential+block@1f36c3ccc9574e79a938ad6e3e3bed74)

# Chapter Overview

This chapter will explain how to configure traffic routing to bring
traffic inside the mesh as well as how to split the traffic between
services running inside the mesh. We will also learn about how Istio can
help with service resiliency and how to inject failures into requests. 

# Learning Objectives

By the end of this chapter, you should be able to:

-   Understand how to expose services from the cluster.

-   Understand how Istio knows where to route the traffic and how the
    traffic gets routed.

-   Understand how to split traffic based on weight and other request
    properties.

-   Understand how service resilience, failure injection, and circuit
    breaking features work.

-   Understand how to bring external services to the mesh using the
    ServiceEntry resource.

# Gateways (1)

In the earlier installation lab, when we installed Istio using
the **demo** profile, it included the ingress and egress gateways. 

Both gateways are Kubernetes deployments that run an instance of the
Envoy proxy, and they operate as load balancers at the edge of the mesh.
The ingress gateway receives inbound connections, while the egress
gateway receives connections going out of the cluster.

Using the ingress gateway, we can apply route rules to the inbound
traffic entering the cluster. As part of the ingress gateway, a
Kubernetes service of type LoadBalancer is deployed, giving us an
external IP address. 

 

![Ingress and egress gateways are instances of Envoy, running at the
edge of the
cluster](https://github.com/nadidurna/Istio/blob/master/images/image13.png)

 

We can configure both gateways using a Gateway resource. The Gateway
resource describes the exposed ports, protocols, SNI (Server Name
Indication) configuration for the load balancer, etc.

Under the covers, the Gateway resource controls how the Envoy proxy
listens on the network interface and which certificates it presents.

Here\'s an example of a Gateway resource:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: my-gateway**\
**  namespace: default**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**  - port:**\
**      number: 80**\
**      name: http**\
**      protocol: HTTP**\
**    hosts:**\
**    - dev.example.com**\
**    - test.example.com**

The above Gateway resource sets up the Envoy proxy as a load balancer
exposing port 80 for ingress. The gateway configuration gets applied to
the Istio ingress gateway proxy, which we deployed to
the **istio-system** namespace and has the label **istio:
ingressgateway** set.  The **hosts** field acts as a filter and will let
through only traffic destined
for **dev.example.com** and **test.example.com**.

To control and forward the traffic to an actual Kubernetes service
running inside the cluster, we have to configure a VirtualService with
matching hostnames (**dev.example.com** and **test.example.com**, for
example) and then attach the Gateway resource to it.

 

![Inbound traffic is matched by the hosts in the Gateway and
VirtualService
resources](https://github.com/nadidurna/Istio/blob/master/images/image14.png)

 

The Ingress gateway we deployed as part of the **demo** Istio
installation created a Kubernetes service with the LoadBalancer type
that gets an external IP assigned to it, for example:

**kubectl get svc -n istio-system**

**NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     
PORT(S)                                                                 
    AGE**\
**istio-egressgateway    ClusterIP      10.0.146.214   \<none\>         
 80/TCP,443/TCP,15443/TCP                                               
     7m56s**\
**istio-ingressgateway   LoadBalancer   10.0.98.7      XX.XXX.XXX.XXX 
 15021:31395/TCP,80:32542/TCP,443:31347/TCP,31400:32663/TCP,15443:31525/TCP 
 7m56s**\
**istiod                 ClusterIP      10.0.66.251    \<none\>         
 15010/TCP,15012/TCP,443/TCP,15014/TCP,853/TCP                         
      8m6s**

***NOTE:** How the LoadBalancer Kubernetes service type works depends on
how and where we run the Kubernetes cluster. For a cloud-managed cluster
(GCP, AWS, Azure, etc.), a load balancer resource gets provisioned in
your cloud account, and the Kubernetes LoadBalancer service will get an
external IP address assigned to it. Suppose we are using Minikube or
Docker Desktop. In that case, the external IP address will either be set
to ***localhost*** (Docker Desktop) or, if we are using Minikube, it
will remain pending, and we will have to use
the ***minikube*** ***tunnel*** command to get an IP address.*

In addition to the ingress gateway, we can deploy an egress gateway to
control and filter traffic leaving our mesh.

We can use the same Gateway resource to configure the egress gateway
like we configured the ingress gateway. The egress gateway allows us to
centralize all outgoing traffic, logging, and authorization.

# Gateways (2)

In this lab, we will deploy a Hello World application to the cluster. We
will then deploy a Gateway resource and a VirtualService that binds to
the Gateway to expose the application on an external IP address.

Ensure you have a Kubernetes cluster with Istio installed, and
the **default** namespace labeled for Istio sidecar injection before
continuing.

 

![Exposing Hello world application through the ingress
gateway](https://github.com/nadidurna/Istio/blob/master/images/image15.png)

**Exposing Hello world application through the ingress gateway**

# Deploying the Gateway Resource

Let us start by deploying the Gateway resource. We will set
the **hosts** field to **\*** (**\*** is a wildcard matcher) to access
the ingress gateway directly from the external IP address, without any
hostname.

If we wanted to access the ingress gateway through a domain name, we
could set the hosts\' value to a domain name (e.g., **example.com**) and
add the external IP address as an A record in the domain\'s DNS
settings.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: gateway**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**    - port:**\
**        number: 80**\
**        name: http**\
**        protocol: HTTP**\
**      hosts:**\
**        - \'\*\'**

***NOTE: **You can download the supporting YAML and other files from
this [Github repo]{.underline}.*

Save the above YAML to **gateway.yaml** and deploy the Gateway
using **kubectl apply -f gateway.yaml**

If we try to access the ingress gateway\'s external IP address, we will
get back an HTTP 404 because there aren\'t any VirtualServices bound to
the Gateway. The ingress proxy doesn\'t know where to route the traffic
as we have not defined any routes yet.

To get the ingress gateway\'s external IP address, run the command below
and look at the **EXTERNAL-IP** column value:

**kubectl get svc -l=istio=ingressgateway -n istio-system**\
**NAME                   TYPE           CLUSTER-IP   EXTERNAL-IP     
PORT(S)                                                                 
    AGE**\
**istio-ingressgateway   LoadBalancer   10.0.98.7    \[GATEWAY_IP\] 
 15021:31395/TCP,80:32542/TCP,443:31347/TCP,31400:32663/TCP,15443:31525/TCP 
 9h**

Throughout the rest of the course and labs, we will
use **GATEWAY_IP** in examples and text when talking about the ingress
gateway\'s external IP. You can set the environment variable with
the **GATEWAY_IP** address like this:

**export GATEWAY_IP=\$(kubectl get svc -n istio-system
istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

# Create the Hello World Deployment and Service

The next step is to create the **hello-world** deployment and service.
The **hello-world** application is a simple website that shows the
sentence \"Hello world\".

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: hello-world**\
**  labels:**\
**    app: hello-world**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: hello-world**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: hello-world**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/hello-world:1.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: hello-world**\
**  labels:**\
**    app: hello-world**\
**spec:**\
**  selector:**\
**    app: hello-world**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 3000**

Save the above YAML to **hello-world.yaml** and create the deployment
and service using **kubectl apply -f hello-world.yaml**. 

Looking at the created Pods, you will notice two containers running. One
is the Envoy sidecar proxy, and the second one is the application. We
have also created a Kubernetes service called **hello-world**:

**kubectl get po,svc -l=app=hello-world**\
**NAME                               READY   STATUS    RESTARTS   AGE**\
**pod/hello-world-6bf9d9bdb6-r8bb4   2/2     Running   0          78s**

**NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP 
 PORT(S)   AGE**\
**service/hello-world   ClusterIP   10.0.155.147   \<none\>       
80/TCP    78s**

# Create a VirtualService

The next step is to create a VirtualService for
the **hello-world** service and bind it to the Gateway resource:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: hello-world**\
**spec:**\
**  hosts:**\
**    - \"\*\"**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: hello-world.default.svc.cluster.local**\
**            port:**\
**              number: 80**

We use the **\*** in the **hosts** field, just like in the Gateway
resource. We have also added the Gateway resource we created earlier
(**gateway**) to the **gateways** array. We say that we have attached
the Gateway to the VirtualService. 

Finally, we specify a single route with a destination that points to the
Kubernetes service **hello-world.default.svc.cluster.local**. 

Using a fully qualified service name is the preferred way to reference
the services in Istio resources. We can also use a short name
(e.g., **hello-world**), which might lead to confusion and unexpected
behavior if we have multiple services with the same name running in
different namespaces. 

Save the above YAML to **vs-hello-world.yaml** and create the
VirtualService using **kubectl apply -f vs-hello-world.yaml**. 

If you look at the deployed VirtualService, you should see output
similar to the following:

**kubectl get vs**\
**NAME          GATEWAYS    HOSTS   AGE**\
**hello-world   \[gateway\]   \[\*\]     3m31s**

If we run cURL against **GATEWAY_IP** or open it in the browser, we will
get back a response **Hello World:**

**curl -v http://\$GATEWAY_IP/\
\*   Trying \$GATEWAY_IP\...**\
**\* TCP_NODELAY set**\
**\* Connected to \$GATEWAY_IP (\$GATEWAY_IP) port 80 (#0)**\
**\> GET / HTTP/1.1**\
**\> Host: \$GATEWAY_IP**\
**\> User-Agent: curl/7.64.1**\
**\> Accept: \*/\***\
**\>**\
**\< HTTP/1.1 200 OK**\
**\< date: Mon, 18 Jul 2022 02:59:37 GMT**\
**\< content-length: 11**\
**\< content-type: text/plain; charset=utf-8**\
**\< x-envoy-upstream-service-time: 1**\
**\< server: istio-envoy**\
**\<**\
**\* Connection #0 to host \$GATEWAY_IP left intact**\
**Hello World\* Closing connection 0**

Also, notice the **server** header set to **istio-envoy**, indicating
that the request went through the sidecar proxy.

# Cleanup

To clean up, delete the Deployment, Service, VirtualService, and the
Gateway:

**kubectl delete deploy hello-world**\
**kubectl delete service hello-world**\
**kubectl delete vs hello-world**\
**kubectl delete gateway gateway**

# Traffic Routing in Istio

Istio features a couple of resources we can use to configure how traffic
is routed within the mesh. We have already mentioned the VirtualService
and the Gateway resource in the Gateway section. 

We can use the VirtualService resource to configure routing rules for
services within the Istio service mesh. For example, in the
VirtualService resource, we match the incoming traffic based on the
request properties and then route the traffic to one or more
destinations. For example, once we match the traffic, we can split it by
weight, inject failures and/or delays, mirror the traffic, and so on.

The DestinationRule resource contains the rules applied after routing
decisions (from the VirtualService) have already been made. With the
DestinationRule, we can configure how to reach the target service. For
example, we can configure outlier detection, load balancer settings,
connection pool settings, and TLS settings for the destination service.

The last resource we should mention is the ServiceEntry. This resource
allows us to take an external service or an API and make it appear as
part of the mesh. The resource adds the external service to the internal
service registry, allowing us to use Istio features such as traffic
routing, failure injection, and others against external services.

# Where to Route the traffic?

Before the Envoy proxy can decide where to route the requests, we need a
way to describe what our system and services look like. 

Let us look at an example where we have a **web-frontend** service and
two versions (**v1** and **v2**) of a **customers** service running in
the cluster. Regarding resources, we have the following deployed in the
cluster:

-   -   -   Kubernetes
            deployments: **customers-v1**, **customers-v2** and **web-frontend**.

        -   Kubernetes services: **web-frontend** and **customers**

To describe the different service versions, we use the concept of labels
in Kubernetes. The pods created from the two customer deployments have
the labels **version: v1** and **version: v2** set.

Note that we only have a single **customers** service that load-balances
the traffic across all customer service pods (regardless of which
deployment they were created from). How does Istio know or distinguish
between the different versions of the service?

We can set different labels in the pod spec template in each versioned
deployment and then use these labels to make Istio aware of the two
distinct versions or destinations for traffic. The labels are used in a
construct called **subset** that can be defined in the DestinationRule.

To describe the two versions of the service, we would create a
DestinationRule that looks like this:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: DestinationRule**\
**metadata:**\
**  name: customers**\
**spec:**\
**  host: customers.default.svc.cluster.local**\
**  subsets:**\
**    - name: v1**\
**      labels:**\
**        version: v1**\
**    - name: v2**\
**      labels:**\
**        version: v2**

Under the hood, when Istio translates these resources into Envoy proxy
configuration, unique Envoy clusters get created that correspond to
different versions. Istio takes the Kubernetes Service endpoints and
applies the labels defined in the subsets to create separate collections
of endpoints. The logical collection of these endpoints in Envoy is
called a cluster.

The Envoy clusters are named by concatenating the traffic direction,
port, subset name, and service hostname. 

For example:

**outbound\|80\|v1\|customers.default.svc.cluster.local**\
**outbound\|80\|v2\|customers.default.svc.cluster.local**

Now that Envoy has an addressable group of endpoints, we can decide how
we want to route the traffic to those destinations. This is where the
VirtualService resource comes in.

# How to Route the Traffic?

In the VirtualService, we can specify the traffic matching and routing
rules that decide which destinations traffic is routed to.

We have multiple options when deciding on how we want the traffic to be
routed:

-   -   -   Route based on weights

        -   Match and route the traffic

        -   Redirect the traffic (HTTP 301)

        -   Mirror the traffic to another destination

The above options of routing the traffic can be applied and used
individually or together within the same VirtualService resource.

Additionally, we can add, set or remove request and response headers,
and configure CORS settings, timeouts, retries, and fault injection.

Let us look at some examples:

**1. Weight-based routing**

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers-route**\
**spec:**\
**  hosts:**\
**  - customers.default.svc.cluster.local**\
**  http:**\
**  - name: customers-v1-routes**\
**    route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v1**\
**      weight: 70**\
**  - name: customers-v2-routes**\
**    route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v2**\
**      weight: 30**

In this example, we split the traffic based on weight to two subsets of
the same service, where 70% goes to subset **v1** and 30% to
subset **v2**.

**2. Match and route the traffic**

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers-route**\
**spec:**\
**  hosts:**\
**  - customers.default.svc.cluster.local**\
**  http:**\
**  - match:**\
**    - headers:**\
**        user-agent:**\
**          regex: \".\*Firefox.\*\"**\
**    route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v1**\
**  - route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v2**

In this example, we provide a regular expression and try to match
the **User-Agent** header value. If the header value matches, we route
the traffic to subset **v1**. Otherwise, if the **User-Agent** header
value doesn't match, we route the traffic to subset **v2**.

**3. Redirect the traffic**

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers-route**\
**spec:**\
**  hosts:**\
**  - customers.default.svc.cluster.local**\
**  http:**\
**  - match:**\
**    - uri:**\
**        exact: /api/v1/helloWorld**\
**    redirect:**\
**      uri: /v1/hello**\
**      authority: hello-world.default.svc.cluster.local**

In this example, we combine the matching on the URI and then redirect
the traffic to a different URI and a different service.

**4. Traffic mirroring**

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers-route**\
**spec:**\
**  hosts:**\
**    - customers.default.svc.cluster.local**\
**  http:**\
**  - route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v1**\
**      weight: 100**\
**    mirror:**\
**      host: customers.default.svc.cluster.local**\
**      subset: v2**\
**    mirrorPercentage:**\
**      value: 100.0**

 In this example, we mirror 100% of the traffic to the **v2** subset.
Mirroring takes the same request sent to subset **v1** and "mirrors" it
to the **v2** subset. The request is "fire and forget". Mirroring can be
used for testing and debugging the requests by mirroring the production
traffic and sending it to the service version of our choice.

# Weight-Based Traffic Routing

In this lab, we will learn how to route traffic between different
service versions using weights. We will start by deploying a
web-frontend application and a customers backend service version v1. We
will then deploy the customers service version v2 and split the traffic
between the two versions using subsets.

Let us start by deploying the Gateway:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: gateway**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**    - port:**\
**        number: 80**\
**        name: http**\
**        protocol: HTTP**\
**      hosts:**\
**        - \'\*\'**

 ***NOTE:** You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

 Save the above YAML to **gateway.yaml** and deploy the Gateway
using **kubectl apply -f gateway.yaml**.

# Deploying the \"web-frontend\" Application

Next, we will create the **web-frontend** and the **customers** service
deployments and corresponding Kubernetes services. Let us start with
the **web-frontend** first:

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: web-frontend**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: web-frontend**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/web-frontend:1.0.0**\
**          imagePullPolicy: Always**\
**          name: web**\
**          ports:**\
**            - containerPort: 8080**\
**          env:**\
**            - name: CUSTOMER_SERVICE_URL**\
**              value: \'http://customers.default.svc.cluster.local\'**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  selector:**\
**    app: web-frontend**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 8080**

Notice we are setting an environment variable
called **CUSTOMER_SERVICE_URL** that points to the **customers** service
we will deploy next. The **web-frontend** uses that URL to make a call
to the **customers** service.

Save the above YAML to **web-frontend.yaml** and create the deployment
and service using **kubectl apply -f web-frontend.yaml**.

# Deploying the \"customers\" Application

Now we can deploy version v1 of the **customers** service. Notice how we
set the **version: v1** label in the pod template. However, the
Kubernetes Service only uses **app: customers** label in the selector.
That\'s because we will create the subsets in the DestinationRule, and
those will apply the additional version label to the selector, allowing
us to reach the pods running specific versions.

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v1**\
**  labels:**\
**    app: customers**\
**    version: v1**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v1**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:1.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: customers**\
**  labels:**\
**    app: customers**\
**spec:**\
**  selector:**\
**    app: customers**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 3000**

Save the above to **customers-v1.yaml** and create the deployment and
service using **kubectl apply -f customers-v1.yaml**.

Both applications should be deployed and running:

**kubectl get po**\
**NAME                            READY   STATUS    RESTARTS   AGE**\
**customers-v1-7857944975-5lxc8   2/2     Running   0          36s**\
**web-frontend-659f65f49-jz58r    2/2     Running   0          3m38s**

# Configuring the VirtualService

Create a VirtualService for the **web-frontend** and bind it to the
Gateway resource:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: web-frontend**\
**spec:**\
**  hosts:**\
**    - \'\*\'**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: web-frontend.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above YAML to **web-frontend-vs.yaml** and create the
VirtualService using **kubectl apply -f web-frontend-vs.yaml**.

Next, we can set the environment variable with
the **GATEWAY_IP** address like this: 

**export GATEWAY_IP=\$(kubectl get svc -n istio-system
istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

And open the **GATEWAY_IP** in the browser. The **web-frontend** shows
the customers list from the **customers** service, as shown in the
figure below.

 

![Customers list from the customers service displayed by the
web-frontend
service](https://github.com/nadidurna/Istio/blob/master/images/image16.png)

 

If we deployed the **customers** service version **v2**, the responses
we would get back when calling
the **http://customers.default.svc.cluster.local** would be random. They
would either come from the **v2** or **v1** version of
the **customers** service. That is because the selector label in the
Kubernetes service does not have the version label set.

# Defining Service Subsets

We have to create the DestinationRule for the **customers** service and
define the two subsets representing **v1** and **v2** versions. Then, we
can create a VirtualService and route all traffic to the **v1** subset.
After that, we can deploy the **v2** version of
the **customers** service without impacting the existing services or
traffic.

Let us start with the DestinationRule and two subsets:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: DestinationRule**\
**metadata:**\
**  name: customers**\
**spec:**\
**  host: customers.default.svc.cluster.local**\
**  subsets:**\
**    - name: v1**\
**      labels:**\
**        version: v1**\
**    - name: v2**\
**      labels:**\
**        version: v2**

Save the above to **customers-dr.yaml** and create the DestinationRule
using **kubectl apply -f customers-dr.yaml**.

We can create the VirtualService and specify the **v1** subset in the
destination:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v1**

Whenever a request gets sent to the Kubernetes **customers** service, it
will get routed to the same service\'s **v1** subset.

Save the above YAML to **customers-vs.yaml** and create the
VirtualService using **kubectl apply -f customers-vs.yaml**.

# Deploying the \"customers v2\" Application

We are now ready to deploy the **v2** version of
the **customers** service. The **v2** version returns the same list of
customers as the previous version, with the addition of the City name.

Let us create the **customers-v2** deployment. We do not need to deploy
the Kubernetes Services because we have already deployed one with
the **v1** version.

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v2**\
**  labels:**\
**    app: customers**\
**    version: v2**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v2**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v2**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:2.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**

The deployment is nearly identical to the **v1** deployment. The
differences are in the Docker image version and the **v2** value set in
the **version** label.

Save the above YAML to **customers-v2.yaml** and create the deployment
using **kubectl apply -f customers-v2.yaml**.

# Splitting Traffic Between the Versions

Because of the VirtualService we created earlier, all traffic will go to
the subset **v1**. Let us use the **weight** field and modify the
VirtualService. We will send 50% of the traffic to the **v1** subset and
the other 50% to the **v2 **subset.

To do that, we will create a second **destination** with the same
hostname but a different subset. We will also add the **weight: 50** to
both destinations to split the traffic between the versions equally.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v1**\
**          weight: 50**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v2**\
**          weight: 50**

Save the above YAML to **customers-50-50.yaml** and update the
VirtualService using **kubectl apply -f customers-50-50.yaml**.

Open the **GATEWAY_IP** in the browser and refresh the page several
times to see the different responses. The figure below shows the
response from the **customers-v2**.

 

![Response from
customers-v2](https://github.com/nadidurna/Istio/blob/master/images/image17.png)

 

To change the proportion of the traffic sent to one or the other
version, we can update the VirtualService. Similarly, we could
add **v3** or **v4** versions and split the traffic between those
versions.

# Cleanup

To clean up the resources from the cluster, run: 

**kubectl delete deploy web-frontend customers-{v1,v2}**\
**kubectl delete svc customers web-frontend**\
**kubectl delete vs customers web-frontend**\
**kubectl delete dr customers**\
**kubectl delete gateway gateway**

# Advanced Traffic Routing

Earlier, we learned how to route traffic between multiple subsets using
the proportion of the traffic (**weight** field). In some cases, pure
weight-based traffic routing or splitting is enough. However, there are
scenarios and cases where we might need more granular control over how
the traffic is split and forwarded to destination services.

Istio allows us to use parts of the incoming requests and match them to
the defined values. For example, we can check the **URI prefix** of the
incoming request and route the traffic based on that.

The table below shows the different properties we can match on.

+-----------------------------------+-----------------------------------+
| ### Property                      | ### Description                   |
+===================================+===================================+
| **uri**                           | Matches the request URI to the    |
|                                   | specified value                   |
+-----------------------------------+-----------------------------------+
| **scheme**                        | Match the request schema (HTTP,   |
|                                   | HTTPS, ...)                       |
+-----------------------------------+-----------------------------------+
| **method**                        | Match the request method (GET,    |
|                                   | POST, ...)                        |
+-----------------------------------+-----------------------------------+
| **authority**                     | Match the request authority       |
|                                   | headers                           |
+-----------------------------------+-----------------------------------+
| **headers**                       | Match the request headers.        |
|                                   | Headers must be lower-case and    |
|                                   | separated by hyphens              |
|                                   | (e.g., **x-my-request-id**).      |
|                                   | Note, if we use headers for       |
|                                   | matching, other properties are    |
|                                   | ignored                           |
|                                   | (**uri**, **sch                   |
|                                   | eme**, **method**, **authority**) |
+-----------------------------------+-----------------------------------+

Each of the above properties can get matched using one of these methods:

-   -   -   Exact match: e.g., **exact: \"value\"** matches the exact
            string

        -   Prefix match: e.g., **prefix: \"value\"** matches the prefix
            only

        -   Regex match: e.g., **regex: \"value\"** matches based on the
            ECMAScript style regex

For example, let\'s say the request URI looks like
this: **https://dev.example.com/v1/api**. To match the request URI, we
could write the configuration like this:

**http:**\
**- match:**\
**  - uri:**\
**      prefix: /v1**

The above snippet would match the incoming request, and the request
would get routed to the destination defined in that route.

Another example would be using Regex and matching on a header:

**http:**\
**- match:**\
**  - headers:**\
**      user-agent:**\
**        regex: \'.\*Firefox.\*\'**

The above match will match any request where the **User Agent** header
matches the Regex.

# Rewriting and Redirecting Traffic

In addition to matching on properties and then directing the traffic to
a destination, sometimes we also need to rewrite the incoming URI or
modify the headers.

For example, let us consider a scenario where w match the incoming
requests to the **/v1/api** path. Once matched, we want to route the
requests to a different URI, **/v2/api**, for example. We can do that
using the rewrite functionality.

**\...**\
**http:**\
**  - match:**\
**    - uri:**\
**        prefix: /v1/api**\
**    rewrite:**\
**      uri: /v2/api**\
**    route:**\
**      - destination:**\
**          host: customers.default.svc.cluster.local**\
**\...**

The above snippet will match the prefix and then rewrite the matched
prefix portion with the URI we provide in the **rewrite** field. Even
though the destination service doesn\'t expose or listen on
the **/v1/api** endpoint, we can rewrite those requests to a different
path, **/v2/api** in this case.  

We also can redirect or forward the request to a completely different
service. Here is how we could match on a header and then redirect the
request to another service:

**\...**\
**http:**\
**  - match:**\
**    - headers:**\
**        my-header:**\
**          exact: hello**\
**    redirect:**\
**      uri: /hello**\
**      authority: my-service.default.svc.cluster.local:8000**\
**\...**

The **redirect** and **destination** fields are mutually exclusive. If
we use the **redirect**, there is no need to set the destination.

# Manipulating Headers

When redirecting or rewriting the requests, there's also a requirement
to add or modify the request (or response) headers. The headers can be
modified either for individual destinations or all destinations in the
VirtualService.

Let us consider the following example that shows both scenarios:

**apiVersion: networking.istio.io/v1beta1**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**  - customers.default.svc.cluster.local**\
**  http:**\
**  - headers:**\
**      request:**\
**        set:**\
**          debug: \"true\"**\
**    route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v2**\
**      weight: 20**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        subset: v1**\
**      headers:**\
**        response:**\
**          remove:**\
**          - x-api-key**\
**      weight: 80**

In the above example, we set a request header **debug: true** for all
traffic sent to the host. Additionally, we split the traffic between two
subsets by weight, and in one destination, we are removing a response
header called **x-api-key**. So, whenever the traffic reaches the
subset **v1**, the response from the service will not include
the **x-api-key** header. 

In addition to adding and removing the headers, we can also use
the **set** to overwrite existing header values.

# AND and OR Semantics

Regardless of the property we are matching on, we can either use AND or
OR semantics. Let\'s take a look at the following snippet:

**http:**\
**  - match:**\
**    - uri:**\
**        prefix: /v1**\
**      headers:**\
**        my-header:**\
**          exact: hello**\
**\...**

The above snippet uses the **AND** semantics. It states that both the
URI prefix needs to match **/v1** **AND** the header **my-header** has
to match the value **hello**. When both conditions are true, the traffic
will be routed to the destination.

To use the **OR** semantic, we can add another **match** entry, like
this:

**\...**\
**http:**\
**  - match:**\
**    - uri:**\
**        prefix: /v1**\
**    \...**\
**  - match:**\
**    - headers:**\
**        my-header:**\
**          exact: hello**\
**\...**

In the above snippet, the matching will be done on the URI prefix first,
and if it matches, the request gets routed to the destination.

If the first match does not evaluate to true, the algorithm moves to the
second **match** field and tries to match the header. If we omit
the **match** field on the route, it will continually evaluate to true.

When using either of the two options, make sure you provide a fallback
route if applicable. That way, if traffic doesn't match any of the
conditions, it could still be routed to a "default" route.

# Advanced Traffic Routing

In this lab, we will learn how to use request properties to route the
traffic between multiple service versions. 

We will start by deploying the Gateway:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: gateway**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**    - port:**\
**        number: 80**\
**        name: http**\
**        protocol: HTTP**\
**      hosts:**\
**        - \'\*\'**

***NOTE:** You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above YAML to **gateway.yaml** and deploy the Gateway
using **kubectl apply -f gateway.yaml**.

# Deploying the Applications

Next, we will deploy
the **web-frontend**, **customers-v1**, **customers-v2**, and the
corresponding VirtualServices and DestinationRule.

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: web-frontend**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: web-frontend**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/web-frontend:1.0.0**\
**          imagePullPolicy: Always**\
**          name: web**\
**          ports:**\
**            - containerPort: 8080**\
**          env:**\
**            - name: CUSTOMER_SERVICE_URL**\
**              value: \'http://customers.default.svc.cluster.local\'**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  selector:**\
**    app: web-frontend**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 8080**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: web-frontend**\
**spec:**\
**  hosts:**\
**    - \'\*\'**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: web-frontend.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above YAML to **web-frontend.yaml **and create the deployment
and service using **kubectl apply -f web-frontend.yaml**.

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v1**\
**  labels:**\
**    app: customers**\
**    version: v1**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v1**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:1.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v2**\
**  labels:**\
**    app: customers**\
**    version: v2**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v2**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v2**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:2.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: customers**\
**  labels:**\
**    app: customers**\
**spec:**\
**  selector:**\
**    app: customers**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 3000**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v1**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: DestinationRule**\
**metadata:**\
**  name: customers**\
**spec:**\
**  host: customers.default.svc.cluster.local**\
**  subsets:**\
**    - name: v1**\
**      labels:**\
**        version: v1**\
**    - name: v2**\
**      labels:**\
**        version: v2**

Save the above YAML to **customers.yaml** and create the resources
with **kubectl apply -f customers.yaml**.

Once we deploy everything, all traffic gets routed to
the **customers-v1** service. To ensure everything is deployed and works
correctly, open the **GATEWAY_IP** in a web browser and ensure you are
getting the responses back from the **customers-v1** service. You should
only see the **NAME** column on the resulting page.

You can set the environment variable **GATEWAY_IP** like this:

**export GATEWAY_IP=\$(kubectl get svc -n istio-system
istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

# Routing Based on Headers

We will update the **customers** VirtualService so that the traffic is
routed between two versions of the **customers** service.

Let us look at a YAML that routes the traffic to **customers-v2**, if
the request contains a header **user: debug**. If the header is not set,
we route the traffic to the **customers-v1** service.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**  - match:**\
**    - headers:**\
**        user:**\
**          exact: debug**\
**    route:**\
**    - destination:**\
**        host: customers.default.svc.cluster.local**\
**        port:**\
**          number: 80**\
**        subset: v2**\
**  - route:**\
**      - destination:**\
**          host: customers.default.svc.cluster.local**\
**          port:**\
**            number: 80**\
**          subset: v1**

Save the above YAML to **customers-vs-headers.yaml** and update the
VirtualService with **kubectl apply -f customers-vs-headers.yaml**.

***NOTE: **The destinations in the VirtualService would also work if we
did not provide the port number. That\'s because the service has a
single port defined.*

If we open the **GATEWAY_IP**, we should still get the response
from **customers-v1**. If we add the header **user: debug** to the
request, you will notice that the response comes from **customers-v2**.

We can use the [ModHeader extension]{.underline} to modify the headers
from the browser. Alternatively, we can use cURL and add the header to
the request, like this:

**curl -H \"user: debug\" http://\$GATEWAY_IP/ \| grep
-E \'CITY\|NAME\'**

**\...**\
**\<th class=\"px-4 py-2\"\>CITY\</th\>**\
**\<th class=\"px-4 py-2\"\>NAME\</th\>**

If we look through the response, we will notice the two columns
- **CITY** and **NAME**, which indicates the response was sent by
the **customers-v2** service.

Similarly, if you open **GATEWAY_IP** in the browser, the response will
be from the **customers-v1** service. If you add the **user:
debug** header (using the [ModHeader]{.underline} or other similar
extensions) and refresh the page, you will notice the response will be
coming from the **customers-v2** service.

# Cleanup

To clean up the Deployments, Services, VirtualServices, DestinationRule,
and the Gateway, run the following commands:

**kubectl delete deploy web-frontend customers-{v1,v2}**\
**kubectl delete svc customers web-frontend**\
**kubectl delete vs customers web-frontend**\
**kubectl delete dr customers**\
**kubectl delete gateway gateway**

# Service Resiliency

Resiliency is the ability to provide and maintain an acceptable level of
service in the face of faults and challenges to regular operation. It\'s
not about avoiding failures. It\'s responding to them, so there\'s no
downtime or data loss. The goal of resiliency is to return the service
to a fully functioning state after a failure occurs.

A crucial element in making services available is
using **timeouts** and **retry policies** when making service requests.
We can configure both in the VirtualService resource.

Using the **timeout** field, we can define a timeout for HTTP requests.
If the request takes longer than the value specified in
the **timeout** field, the Envoy proxy will drop the request and mark it
as timed out (return an **HTTP 408** to the application). The
connections remain open unless outlier detection is triggered.

Here\'s an example of setting a timeout for a route:

**\...**\
**- route:**\
**  - destination:**\
**      host: customers.default.svc.cluster.local**\
**      subset: v1**\
**  timeout: 10s**\
**\...**

In addition to timeouts, we can configure a more granular retry policy.
We can control the number of retries for a given request, the timeout
per try, and the specific conditions that should trigger a retry. Both
retries and timeouts happen on the client side. 

For example, we can only retry the requests if the upstream server
returns any **5xx** response code, retry only on gateway errors (HTTP
502, 503, or 504), or even specify the retriable status codes in the
request headers. When Envoy retries a failed request, the endpoint that
initially failed and caused the retry is no longer included in the load
balancing pool.

Let\'s say the Kubernetes service has three endpoints (Pods), and one of
them fails with a retriable error code. When Envoy retries the request,
it won\'t resend the request to the original endpoint anymore. Instead,
it will send the request to one of the two endpoints that have not
failed.

Here\'s an example of how to set a retry policy for a particular route:

**\...**\
**- route:**\
**  - destination:**\
**      host: customers.default.svc.cluster.local**\
**      subset: v1**\
**  retries:**\
**    attempts: 10**\
**    perTryTimeout: 2s**\
**    retryOn: connect-failure,reset**\
**\...**

The above retry policy will attempt to retry any request that fails with
a connect timeout (**connect-failure**) or if the server does not
respond at all (**reset**).

We set the per-try attempt timeout to 2 seconds and the number of
attempts to 10. Note that if we set both retries and timeouts, the
timeout value will be the most the request will wait. If we had a
10-second timeout specified in the above example, we would only ever
wait 10 seconds maximum, even if there are still attempts left in the
retry policy.

For more details on retry policies, see the [x-envoy-retry-on
documentation](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/router_filter#x-envoy-retry-on).

# Circuit Breaking with Outlier Detection

Another pattern for creating resiliency applications is circuit
breaking. It allows us to write services to limit the impact of
failures, latency spikes, and other network issues.

Outlier detection is an implementation of a circuit breaker, and it's a
form of passive health checking. It's called passive because Envoy isn't
actively sending any requests to determine the health of the endpoints.
Instead, Envoy observes the performance of different pods to determine
if they are healthy or not. If the pods are deemed unhealthy, they are
removed or ejected from the healthy load balancing pool.

The pods' health is assessed through consecutive failures, temporal
success rate, latency, and so on. 

Outlier detection in Istio is configured in the DestinationRule
resource. Here's a snippet that configures outlier detection:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: DestinationRule**\
**metadata:**\
**  name: customers**\
**spec:**\
**  host: customers**\
**  trafficPolicy:**\
**    connectionPool:**\
**      tcp:**\
**        maxConnections: 1**\
**      http:**\
**        http1MaxPendingRequests: 1**\
**        maxRequestsPerConnection: 1**\
**    outlierDetection:**\
**      consecutive5xxErrors: 1**\
**      interval: 1s**\
**      baseEjectionTime: 3m**\
**      maxEjectionPercent: 100**

The above snippet defines thresholds for TCP and HTTP connections in
the **connectionPool** field. The circuit breaker trips if we exceed 1
TCP connection or one pending HTTP request, or more than one request per
connection. When the circuit breaker trips, the service will start
responding with HTTP 503 (service unavailable) responses.

In addition to the connection pool settings, we have also configured the
outlier detection. When a pod is determined to be an outlier (i.e., it
exceeds the configured threshold, for example, consecutive 5xx errors),
Envoy checks whether it needs to be ejected from the healthy load
balancing pool of pods. The **maxEjectionPercent** field is used here,
and it specifies the maximum percentage of pods that can be ejected. So,
when the thresholds in the connection pool are exceeded, and we get more
than 1 consecutive 5xx error (the **consecutive5xxErrors**  field) and
the pod can be ejected, the outlier detection will eject a pod. 

Each pod gets ejected for a predetermined amount of time. We can
configure the ejection time using the **baseEjectionTime** value. This
value is multiplied by the number of times the pod has been ejected in a
row. If the pod continues to fail, it gets ejected for longer and longer
periods. 

Envoy checks the health of each pod at an interval specified in
the **interval** field. For every check, the endpoint is healthy, the
ejection multiplier gets decremented. After the ejection time passes,
the pod returns to the healthy load balancing pool.

# Failure Injection

Another feature to help us with service resiliency is **fault
injection**. We can apply the fault injection policies on HTTP traffic
and specify one or more faults to inject when forwarding the request to
the destination.  

There are two types of fault injection. We can **delay** the requests
before forwarding and emulate a slow network or overloaded service, and
we can **abort** the HTTP request and return a specific HTTP error code
to the caller. With the abort, we can simulate a faulty upstream
service.

Here\'s an example that aborts HTTP requests and returns HTTP 404, for
30% of incoming requests:

**- route:**\
**  - destination:**\
**      host: customers.default.svc.cluster.local**\
**      subset: v1**\
**  fault:**\
**    abort:**\
**      percentage:**\
**        value: 30**\
**      httpStatus: 404**

The Envoy proxy will abort all requests if we don\'t specify the
percentage. Note that the fault injection affects services that use that
VirtualService. It does not affect all consumers of the service. For
example, if we configure a fault for a specific host name
(e.g. **example.com**), then any request using the
hostname **hello.com** will not be subjected to the fault injection.

Similarly, we can apply an optional delay to the requests using
the **fixedDelay** field:

**- route:**\
**  - destination:**\
**      host: customers.default.svc.cluster.local**\
**      subset: v1**\
**  fault:**\
**    delay:**\
**      percentage:**\
**        value: 5**\
**      fixedDelay: 3s**

The above setting applies a 3-second delay to 5% of incoming requests.

Note that the fault injection will not trigger any retry policies we
have set on the routes. For example, if we inject an HTTP 500 error, the
retry policy configured to retry on the HTTP 500 will not be triggered.

# Observing Failure Injection and Delays in Grafana, Jaeger, and Kiali

In this lab, we will deploy
the **web-frontend **and **customers-v1** service. We will then inject a
failure and a delay and observe them in Jaeger, Kiali, and Grafana.

 

![Architecture of the
lab](https://github.com/nadidurna/Istio/blob/master/images/image18.png)

**Architecture of the lab**

# Deploying the Gateway

Start by deploying the Gateway:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: gateway**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**    - port:**\
**        number: 80**\
**        name: http**\
**        protocol: HTTP**\
**      hosts:**\
**        - \'\*\'**

***NOTE**: You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above YAML to **gateway.yaml** and create the Gateway
using **kubectl apply -f gateway.yaml**.

# Deploying the Applications

Next, deploy the **web-frontend**, its corresponding Kubernetes Service,
and VirtualService.

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: web-frontend**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: web-frontend**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/web-frontend:1.0.0**\
**          imagePullPolicy: Always**\
**          name: web**\
**          ports:**\
**            - containerPort: 8080**\
**          env:**\
**            - name: CUSTOMER_SERVICE_URL**\
**              value: \'http://customers.default.svc.cluster.local\'**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  selector:**\
**    app: web-frontend**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 8080**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: web-frontend**\
**spec:**\
**  hosts:**\
**    - \'\*\'**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: web-frontend.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above YAML to **web-frontend.yaml** and create the resources
using **kubectl apply -f web-frontend.yaml**.

Lastly, deploy the **customers-v1** and related resources.

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v1**\
**  labels:**\
**    app: customers**\
**    version: v1**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v1**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:1.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: customers**\
**  labels:**\
**    app: customers**\
**spec:**\
**  selector:**\
**    app: customers**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 3000**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: DestinationRule**\
**metadata:**\
**  name: customers**\
**spec:**\
**  host: customers.default.svc.cluster.local**\
**  subsets:**\
**    - name: v1**\
**      labels:**\
**        version: v1**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v1**

Save the above YAML to **customers.yaml** and create the resources
using **kubectl apply -f customers.yaml**.

# Injecting Delays

With the applications deployed, let\'s inject a 5-second delay to
the **customers-v1** service for 50% of all requests. Setting this on
the **customers-v1** service means that 50% of the calls made to that
service will experience a 5-second delay. 

We will configure the delay in the **customers** VirtualService:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v1**\
**      fault:**\
**        delay:**\
**          percentage:**\
**            value: 50**\
**          fixedDelay: 5s**

Save above YAML to **customers-delay.yaml** and update the
VirtualService using **kubectl apply -f customers-delay.yaml**.

# Observing Delays in Grafana

To generate some traffic, open a separate terminal window and start
making requests to the **GATEWAY_IP** in an endless loop:

**export GATEWAY_IP=\$(kubectl get svc -n istio-system
istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

**while true; do curl http://\$GATEWAY_IP/; done**

You should start noticing some of the requests taking longer than usual.
Let\'s open Grafana from a separate terminal window and observe these
delays.

***NOTE:** You can follow the lab in the observability chapter to learn
how to install Prometheus, Grafana, Jaeger, and Kiali.*

**istioctl dash grafana**

When Grafana opens, click **Home**, **istio** folder, and the **Istio
Service Dashboards**. On the dashboard, select
the **customers.default.svc.cluster.local** in the **Service** dropdown
and **source** in the **Reporter** dropdown.

You can see the same delay is reported on the
web-frontend.default.svc.cluster.local service side by selecting it in
the Service dropdown.

If you expand the **Client Workloads** panel, you will notice the
increased duration on the **Client Request Duration** graph, as shown in
the figure below. 

 

![Increased request duration due to the delay
injection](https://github.com/nadidurna/Istio/blob/master/images/image19.png)

**Increased request duration due to the delay injection**

 

You can see the same delay is reported on
the **web-frontend.default.svc.cluster.local** service side by selecting
it in the **Service** dropdown.

# Observing Delays in Jaeger

Let\'s see how this delay shows up in Jaeger. Open Jaeger
with **istioctl dash jaeger**.

On the main screen, select the **web-frontend.default** from
the **Service** dropdown. Then, enter **5s** in the **Min
Duration** text box. Click the **Find Traces** button to find the
traces. Because we entered the minimum duration of traces, all traces in
the list took at least 5 seconds. Click any of the traces to open the
details. On the details page, we will notice the total duration is 5
seconds.

The single trace has four spans - expand the third span (one with the 5s
duration) that represents the request made from the **web-frontend** to
the **customers** service. 

You will notice in the details that the **response_flags** tag gets set
to **DI**. \"**DI**\" stands for \"delay injection\" and indicates that
a delay was injected into the request.

 

![Injected delay as it shows up in a Jaeger
trace](https://github.com/nadidurna/Istio/blob/master/images/image20.png)

**Injected delay as it shows up in a Jaeger trace**

# Injecting Failures

Let\'s update the VirtualService again, and this time, we will inject a
fault and return HTTP 500 for 50% of the requests.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**\
**            subset: v1**\
**      fault:**\
**        abort:**\
**          httpStatus: 500**\
**          percentage:**\
**            value: 50**

Save the above YAML to **customers-fault.yaml** and update the
VirtualService with **kubectl apply -f customers-fault.yaml**.

# Observing Failures in Grafana

Just like before, we will start noticing failures from the request loop.
If we go back to Grafana and open the **Istio Service Dashboard** (make
sure you select **source** from the **Reporter** dropdown), we will
notice the client success rate dropping and the increase in the 500
responses on the **Incoming Requests by Source and Response
Code** graph, as shown in the figure.

 

![A screenshot of a computer Description automatically generated with
medium
confidence](https://github.com/nadidurna/Istio/blob/master/images/image21.png){width="6.5in"
height="3.907638888888889in"}

**Increased HTTP 500 responses due to failure injection**

# Observing Failures in Jaeger

There\'s a similar story in Jaeger. If we search for traces again (we
can remove the min duration), we will notice the traces with errors, as
shown below.

 

![Graphical user interface, text Description automatically
generated](https://github.com/nadidurna/Istio/blob/master/images/image22.png)


**Jaeger traces with HTTP 500 responses**

# Observing Failures in Kiali

Open Kiali with **istioctl dash kiali**. Look at the service graph by
clicking the **Graph** item. You will notice how
the **web-frontend** service has a red border, as shown below.

 

![Failures in
Kiali](https://github.com/nadidurna/Istio/blob/master/images/image23.png)

**Failures in Kiali**

 

If you click on the **web-frontend** service and look at the sidebar on
the right, you will notice the HTTP request details. The graph shows the
percentage of successes and failures. Both numbers are around 50%,
corresponding to the percentage value we set in the VirtualService.

# Cleanup

Run the following commands to clean up the Deployments, Services,
VirtualServices, DestinationRule, and the Gateway:

**kubectl delete deploy web-frontend customers-v1**\
**kubectl delete svc customers web-frontend**\
**kubectl delete vs customers web-frontend**\
**kubectl delete gateway gateway**

# Using ServiceEntry

When a service is created inside the mesh, Istio reacts by creating an
entry for the service in its service registry. This act is part of
Istio\'s service discovery process.

There exist situations where automatic service discovery does not or
cannot take place, but where the service must be made known to Istio and
entered into its registry. Examples include legacy workloads, or a
database cluster that mesh services need to reach that is not explicitly
part of the mesh.

Istio provides the ServiceEntry custom resource to allow us to define
these service entries explicitly.

Istio makes the distinction between mesh-internal and mesh-external
services. A legacy workload is an example of a mesh-internal service,
whereas a third-party API is mesh-external.

Creating a ServiceEntry for a third-party API allows us to define
routing and traffic policies for that service via Virtual Services and
Destination Rules. Examples include retries, timeouts, mirroring, fault
injection, load balancing strategy, outlier detection, and so on.

Imagine a situation where a third-party API is not reliable. Defining
timeouts and retries for such a service becomes necessary, and Istio
lets us do that in one place, without having to alter any of the source
code backing the workloads that make requests to the service in
question.

Below is an example ServiceEntry for **www.googleapis.com** that defines
these third-party APIs in the registry as **MESH_EXTERNAL**.

**apiVersion: networking.istio.io/v1beta1**\
**kind: ServiceEntry**\
**metadata:**\
**  name: googleapis-svc-entry**\
**spec:**\
**  hosts:**\
**  - www.googleapis.com**\
**  location: MESH_EXTERNAL**\
**  resolution: DNS**\
**  ports:**\
**  - number: 443**\
**    name: https**\
**    protocol: TLS**

The **resolution** field allows us to control how the actual backing
endpoints for this service are resolved. In this case, it\'s done with a
simple DNS resolution.

But imagine a different situation where we wish to make a call to a
mesh-internal database cluster where we wish to furnish the cluster\'s
IP addresses directly, for example:

**apiVersion: networking.istio.io/v1beta1**\
**kind: ServiceEntry**\
**metadata:**\
**  name: external-svc-mongocluster**\
**spec:**\
**  hosts:**\
**  - mymongodb.somedomain \# not used**\
**  addresses:**\
**  - 192.192.192.192/24 \# VIPs**\
**  ports:**\
**  - number: 27018**\
**    name: mongodb**\
**    protocol: MONGO**\
**  location: MESH_INTERNAL**\
**  resolution: STATIC**\
**  endpoints:**\
**  - address: 2.2.2.2**\
**  - address: 3.3.3.3**

In the above example, the value of the location field
is **MESH_INTERNAL**. In lieu of DNS resolution, **resolution** is set
to **STATIC**, and the endpoints are furnished explicitly as static IP
addresses using the **endpoints** field (the hosts field is actually not
used in this case). An alternative to using **endpoints** is
the **workloadSelector** field which allows for endpoint selection by
matching labels.

# Service Entries for Registry-Only Outbound Traffic Policy (1)

Securing egress traffic is an important part of securing a service mesh.
The mesh\'s outbound traffic policy can be configured to allow calls
only to services that are present in the registry.

When the mesh is configured in this way, a ServiceEntry resource must be
created for each mesh-external service in order for calls to that
service to be permitted.

In this lab, we will create a ServiceEntry for a mesh-external service
when the outbound traffic policy is set to \"registry-only.\"

The first step will be to configure the mesh for \"registry-only\"
outbound traffic, and to confirm that a service that runs inside the
mesh is no longer able to call out to an arbitrary external service,
such as github.com.

You should already have a running Kubernetes cluster with Istio
installed, and configured to use the ***demo*** profile.

Run the following command to override the outbound traffic policy mode
to **REGISTRY_ONLY**.

**istioctl install \--set profile=demo \\\
    \--set meshConfig.outboundTrafficPolicy.mode=REGISTRY_ONLY**

The Istio CLI is intelligent enough to realize that Istio is already
running, and so the configuration of the existing installation will be
updated.

# Service Entries for Registry-Only Outbound Traffic Policy (2)

Inspect the updated Istio configuration by displaying the value of the
ConfigMap named Istio in the **istio-system** namespace.

**kubectl get cm -n istio-system istio -o yaml**

Verify that **outboundTrafficPolicy.mode** is set to the
value **REGISTRY_ONLY**.

To send a token outbound call to an external service, deploy Istio\'s
sleep sample application.

Navigate to the base directory of your Istio distribution and deploy
the **sleep** workload:

**kubectl apply -f samples/sleep/sleep.yaml**

Next, capture the name of the running **sleep** pod:

**SLEEP_POD=\$(kubectl get pod -l app=sleep
-ojsonpath=\'{.items\[0\].metadata.name}\')**

And finally, make an HTTP request from within the mesh-internal service
to github.com:

**kubectl exec \$SLEEP_POD -it \-- curl -v https://github.com**\
**Confirm that the request fails, as shown by this output**\
**\*   Trying 140.82.114.4:443\...**\
**\* Connected to github.com (140.82.114.4) port 443 (#0)**\
**\* ALPN: offers h2**\
**\* ALPN: offers http/1.1**\
**\*  CAfile: /cacert.pem**\
**\*  CApath: none**\
**\* TLSv1.3 (OUT), TLS handshake, Client hello (1):**\
**\* OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to
github.com:443**\
**\* Closing connection 0**\
**curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to
github.com:443**\
**command terminated with exit code 35**

# Service Entries for Registry-Only Outbound Traffic Policy (3)

Let us now proceed to define an explicit ServiceEntry resource for that
mesh external service.

Inspect the following resource:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: ServiceEntry**\
**metadata:**\
**  name: github-external**\
**spec:**\
**  hosts:**\
**  - github.com**\
**  ports:**\
**  - number: 443**\
**    name: https**\
**    protocol: HTTPS**\
**  resolution: DNS**\
**  location: MESH_EXTERNAL**

***NOTE:** You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Note how the location field is set to **MESH_EXTERNAL** and how the
specification matches the hostname and port we wish to access.

Save the above to a file named **service-entry.yaml** and apply it to
your cluster:

**kubectl apply -f service-entry.yaml**

With the service entry created, we can now attempt to call out to
github.com one more time.

**kubectl exec \$SLEEP_POD -it \-- curl -v https://github.com**

This time the request succeeds, and we obtain the HTML response from
github.com.

Configuration of external services does not stop there. We can deploy
Istio with its egress gateway component and route all outbound traffic
through this gateway through the use of the custom resources Gateway,
VirtualService, and DestinationRule.

Furthermore, thanks to Istio\'s workload identity capabilities, this
configuration can be overlaid with AuthorizationPolicy resources that
specify which internal mesh services are allowed to call which external
services.

This entire process is summarized in [[this video interview with Michael
Acostamadiedo, on Tetrate\'s Tech Talks (Episode
15)]{.underline}](https://youtu.be/u-6Ejgeu8Xk).

# Cleanup

To clean up, run the following commands:

**kubectl delete -f samples/sleep/sleep.yaml**\
**kubectl delete serviceentry github-external**

[[Chapter 5.
Security]{.underline}](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS144x+3T2022/block-v1:LinuxFoundationX+LFS144x+3T2022+type@sequential+block@8bc9c35231ef49cf8a84dc1681fd8291)

# Chapter Overview

This chapter explains the security concepts in Istio. We will learn what
Istio uses for identity in Kubernetes and how the certificates are
issued and renewed. Then, we will learn about access control questions,
authorization and authentication, and mutual TLS.

# Learning Objectives

By the end of this chapter, you should be able to:

-   Understand what authentication, authorization, and access control
    question are and what Istio uses for identity.

-   Understand what mTLS is and how to configure TLS configuration for
    sidecar proxies and gateways.

-   Understand how service and user authentication are done in Istio.

-   Discuss how certificates get issued in Istio.

# Access Control

The question access control tries to answer is: can
a **principal** perform an **action** on an **object**? 

For example, consider the following two questions:

*Can a user delete a file?*\
*Can a user execute a script?*

The *user* in the above example is
the *principal*, *actions* are *deleting* and *executing*, and
an *object* is a *file* and a *script*.

If we stated a similar example using Kubernetes and Istio, we would ask:

*Can service A perform an action on service B?*

The key terms are authentication and authorization when discussing
security and answering the access control question.

# Authentication (authn)

**Authentication** is all about the **principal** or, in our case, about
services and their identities. It's an act of validating a credential
and ensuring the credential is valid and authentic. Once the
authentication is performed, we can say that we have an **authenticated
principal**.

In Kubernetes, each workload automatically gets assigned a unique
identity. This identity is used when workloads communicate. The identity
in Kubernetes takes the form of a Kubernetes service account. The pods
in Kubernetes use the service account as their identity and present it
at runtime.

Specifically, Istio uses the X.509 certificate from the service account,
creating a new identity according to the specification
called [SPIFFE](https://spiffe.io/) (Secure Production Identity
Framework For Everyone).

SPIFFE  defines a universal naming scheme that's based on a URI. For
example **spiffe://cluster.local/ns/default/sa/my-service-account**. The
spec describes how to take the identity and encode it into different
documents. The document we care about in this case is the X.509
certificate. 

Based on the SPIFFE specification, when we create an X.509 certificate
and fill out the Subject Alternative Name (SAN) field based on the
naming scheme above and check that name during the certificate
validation, we can authenticate the identity encoded in the certificate.
We get a valid SPIFFE identity which is an authenticated principal.

The Envoy proxy sidecars are modified and can perform the extra
validation step required by SPIFFE (i.e., checking the SAN), allowing
those authenticated principals to be used for policy decisions.

# Mutual TLS

Once workloads have a strong identity, we can use them at runtime to do
mutual TLS authentication (mTLS).

Traditionally, TLS is done one way. Let's take the example of going
to [https://google.com](https://google.com/). If you navigate to the
page, you will notice the lock icon, and you can click on it to get the
certificate details. However, we did not give any proof of identity to
google.com, we just opened the website. This is where mTLS is
fundamentally different. 

When two services try to communicate using mTLS, it's required that both
of them provide certificates to each other. That way, both parties know
the identity of who they are talking to.

 

![Using mTLS both client and server verify each others'
identities](https://github.com/nadidurna/Istio/blob/master/images/image24.png)

**Using mTLS both client and server verify each others' identities**

 

As we already learned, all communication between workloads goes through
the Envoy proxies. When a workload sends a request to another, Istio
re-routes the traffic to the sidecar Envoy proxy, regardless of whether
mTLS or plain text communication is used.

In the case of mTLS, once the mTLS connection is established, the
request is forwarded from the client Envoy proxy to the server-side
Envoy proxy. Then, the sidecar Envoy starts an mTLS handshake with the
server-side Envoy. The workloads themselves aren't performing the mTLS
handshake - it's the Envoy proxies doing the work.

During the handshake, the caller does a secure naming check to verify
the service account in the server certificate is authorized to run the
target service. After the authorization on the server-side, the sidecar
forwards the traffic to the workload.

The sidecars are involved in intercepting the incoming inbound traffic
and facilitating or sending the outbound traffic. For that reason, two
distinct resources control the inbound and outbound traffic.
The **PeerAuthentication** for inbound traffic and
the **DestinationRule** for outbound traffic.

 

![PeerAuthentication is used to configure mTLS settings for inbound
traffic, and DestinationRule for configuring TLS settings for outbound
traffic](https://github.com/nadidurna/Istio/blob/master/images/image25.png)

**PeerAuthentication is used to configure mTLS settings for inbound
traffic, and DestinationRule for configuring TLS settings for outbound
traffic**

# Inbound Traffic to the Proxy

The resource that configures what type of mTLS traffic the sidecar
accepts is **PeerAuthentication**. It supports four operating modes, as
shown in the table below.

+-----------------------------------+-----------------------------------+
| ### mTLS mode name                | ### Description                   |
+===================================+===================================+
| **UNSET**                         | Setting is inherited from the     |
|                                   | parent (e.g., mesh or namespace   |
|                                   | level settings); otherwise        |
|                                   | defaults to permissive mode       |
+-----------------------------------+-----------------------------------+
| **DISABLE**                       | mTLS is disabled                  |
+-----------------------------------+-----------------------------------+
| **PERMISSIVE **(default)          | Connection between the workloads  |
|                                   | can either be plaintext or mTLS   |
+-----------------------------------+-----------------------------------+
| **STRICT**                        | Connection between workloads must |
|                                   | be mTLS (i.e., both parties must  |
|                                   | present certificates)             |
+-----------------------------------+-----------------------------------+

The default mTLS mode is **PERMISSIVE**, which means that if a client
tries to connect to a service via mTLS, the Envoy proxy will serve mTLS.
The server will respond in plain text if the client uses plain text.
With this setting, we are allowing the client to choose whether to use
mTLS or not. The permissive mode is useful when onboarding existing
applications to the service mesh as it allows us to roll out mTLS to all
workloads gradually.

Once all applications are onboarded, we can turn on the **STRICT** mode.
The strict mode says that workloads can only communicate using mTLS. The
connection will be closed if a client tries to connect without
presenting their certificate.

We can configure the mTLS mode at
the **mesh**, **namespace**, **workload**, and **port** level. For
example, we can set the **STRICT** mode at the namespace level and then
individually set permissive mode or disable mTLS at either workload
level using selectors or at the individual port level.

Here's an example of how we could set **STRICT** mTLS mode for all
workloads matching a specific label:

**apiVersion: security.istio.io/v1beta1**\
**kind: PeerAuthentication**\
**metadata:**\
**  name: default**\
**  namespace: foo**\
**spec:**\
**  selector:**\
**    matchLabels:**\
**      app: hello-world**\
**  mtls:**\
**    mode: STRICT**

# Outbound Traffic from the Proxy

We use the **DestinationRule** resource to control and configure what
type of TLS traffic the sidecar sends.

The supported TLS modes are shown in the table below.

+-----------------------------------+-----------------------------------+
| ### TLS mode                      | ### Description                   |
+===================================+===================================+
| **DISABLE**                       | Does not set up a TLS connection  |
+-----------------------------------+-----------------------------------+
| **SIMPLE**                        | Originates the traditional TLS    |
|                                   | connection                        |
+-----------------------------------+-----------------------------------+
| **MUTUAL**                        | Uses provided key and certificate |
|                                   | to originate the mTLS connection  |
+-----------------------------------+-----------------------------------+
| **ISTIO_MUTUAL **(default)        | Uses Istio certificates for TLS   |
|                                   | connection                        |
+-----------------------------------+-----------------------------------+

Based on the configuration, the traffic can be sent as plain text
(**DISABLE**), or a TLS connection can be initiated using **SIMPLE** for
TLS and **MUTUAL** or **ISTIO_MUTUAL** for mTLS. 

If the DestinationRule and the TLS mode are not explicitly set, the
sidecar uses Istio's certificates to initiate the mTLS connection. That
means, by default, all traffic inside the mesh is encrypted.

The configuration can be applied to individual hosts by their fully
qualified domain names
(e.g., **hello-world.my-namespace.svc.cluster.local**). A workload
selector with labels can be set up to select specific workloads on which
to apply the configuration.

We can configure specific ports on the workloads to apply the settings
for even more fine-grained control.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: DestinationRule**\
**metadata:**\
**  name: tls-example**\
**spec:**\
**  host: \"\*.example.com\"**\
**  trafficPolicy:**\
**    tls:**\
**      mode: SIMPLE**

The above YAML specifies that TLS connection should be initiated when
talking to services whose domain matches the **\*.example.com**.

# Gateways and TLS

In the case of gateways, we also talk about inbound and outbound
connections.

With **ingress gateways**, the inbound connection typically comes from
the outside of the mesh (i.e., a request from the browser, cURL, etc.),
and the outgoing connection goes to services inside the mesh.

The inverse applies to the **egress gateway**, where the inbound
connection typically comes from within the mesh, and outbound
connections go outside the mesh.

In both cases, the configuration and resources used are identical.
Behind the scenes, the mesh\'s ingress and egress gateway deployments
are identical - it's the configuration that specializes and adapts them
for either ingress or egress traffic. The configuration of gateways is
controlled with the Gateway resource. 

The **tls** field in the Gateway resource controls how the gateway
decodes the inbound traffic. The **protocol** field specifies whether
the inbound connection is
plaintext **HTTP**, **HTTPS**, **GRCP**, **MONGO**, **TCP**, or **TLS**.

When the inbound connection is TLS, we have a couple of options to
control the behavior of the gateway. Do we want to terminate the TLS
connection, pass it through, or do mTLS?

These additional options can be configured using the TLS mode in the
Gateway resource. The table below describes the different TLS modes.

+-----------------------------------+-----------------------------------+
| ### TLS mode                      | ### Description                   |
+===================================+===================================+
| **PASSTHROUGH**                   | Do not terminate TLS; route       |
|                                   | matching in VirtualService is     |
|                                   | done based on the SNI             |
|                                   | information. Connection is        |
|                                   | forwarded to the destination      |
|                                   | as-is                             |
+-----------------------------------+-----------------------------------+
| **SIMPLE**                        | Standard TLS connection           |
+-----------------------------------+-----------------------------------+
| **MUTUAL**                        | Perform mTLS with the             |
|                                   | destination,                      |
|                                   | requires **caCertif               |
|                                   | icates** or **credentialName** to |
|                                   | be set                            |
+-----------------------------------+-----------------------------------+
| **AUTO_PASSTHROUGH**              | Similar to passthrough, except it |
|                                   | does not require an associated    |
|                                   | VirtualService to map from the    |
|                                   | SNI to service in the service     |
|                                   | registry as destination details   |
|                                   | (service, subset, port) are       |
|                                   | encoded in the SNI value          |
+-----------------------------------+-----------------------------------+
| **ISTIO_MUTUAL**                  | Similar to **MUTUAL** but uses    |
|                                   | the certificates generated by     |
|                                   | Istio (i.e., the identity used in |
|                                   | the certificates is the gateway   |
|                                   | workload)                         |
+-----------------------------------+-----------------------------------+

Other TLS-related configurations in the Gateway resource, such as TLS
versions, can be found in
the [documentation](https://istio.io/latest/docs/reference/config/networking/gateway/#ServerTLSSettings).

When configuring TLS settings for outgoing traffic through the egress
gateway, the same configuration applies to the regular Envoy sidecar
proxies in the mesh - the DestinationRule is used.

# Mutual TLS

In this lab, we will deploy the sample application
(**web-frontend **and **customers** service). The **web-frontend** will
be deployed without an Envoy proxy sidecar, while
the **customers** service will have the sidecar injected. With this
setup, we will see how Istio can send both mTLS and plain text traffic
and change the TLS mode to **STRICT**.

Let\'s start by deploying a Gateway resource:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: gateway**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**    - port:**\
**        number: 80**\
**        name: http**\
**        protocol: HTTP**\
**      hosts:**\
**        - \'\*\'**

*** NOTE**: You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above YAML to **gateway.yaml** and deploy the Gateway using the
following command:

**kubectl apply -f gateway.yaml**

Next, we will create the **web-frontend**,
the **customers-v1** deployments, and related Kubernetes services.

# Disabling Sidecar Injection

Before deploying the applications, we will disable the automatic sidecar
injection in the **default** namespace, so the proxy doesn\'t get
injected into the **web-frontend** deployment. Before we deploy
the **customers-v1** service, we will enable the injection again, so the
workload gets the proxy injected.

We are doing this to simulate a scenario where one workload is not part
of the mesh.

**kubectl label namespace default istio-injection-**\
**namespace/default labeled**

# Deploying the Applications

With injection disabled, let\'s create the **web-frontend **deployment,
service, and the VirtualService resource:

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: web-frontend**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: web-frontend**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/web-frontend:1.0.0**\
**          imagePullPolicy: Always**\
**          name: web**\
**          ports:**\
**            - containerPort: 8080**\
**          env:**\
**            - name: CUSTOMER_SERVICE_URL**\
**              value: \'http://customers.default.svc.cluster.local\'**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  selector:**\
**    app: web-frontend**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 8080**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: web-frontend**\
**spec:**\
**  hosts:**\
**    - \'\*\'**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: web-frontend.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above YAML to **web-frontend.yaml** and create the deployment,
service, and VirutalService using **kubectl apply -f
web-frontend.yaml**. 

If we look at the running Pods, we should see one Pod with a single
container running, indicated by the **1/1** in the **READY** column:

**kubectl get po**\
**NAME                           READY   STATUS    RESTARTS   AGE**\
**web-frontend-659f65f49-cbhvl   1/1     Running   0          7m31s**

Let\'s re-enable the automatic sidecar proxy injection:

**kubectl label namespace default istio-injection=enabled**\
**namespace/default labeled**

And then deploy the **customers-v1** workload: 

**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v1**\
**  labels:**\
**    app: customers**\
**    version: v1**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v1**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v1**\
**    spec:**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:1.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: customers**\
**  labels:**\
**    app: customers**\
**spec:**\
**  selector:**\
**    app: customers**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 3000**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**             number: 80**

Save the above to **customers-v1.yaml** and create the resources using
the following command:

**kubectl apply -f customers-v1.yaml**

We should have both applications deployed and running -
the **customers-v1** pod will have two containers, and
the **web-frontend** pod will have one: 

**kubectl get po**\
**NAME                            READY   STATUS    RESTARTS   AGE**\
**customers-v1-7857944975-qrqsz   2/2     Running   0          4m1s**\
**web-frontend-659f65f49-cbhvl    1/1     Running   0          13m**

# Permissive Mode in Action

Let's set the environment variable called GATEWAY_IP that stores the
gateway IP address:

**export GATEWAY_IP=\$(kubectl get svc -n istio-system
istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

If we try to navigate to the **GATEWAY_IP**, we will get the web page
with the customer service\'s response.

Accessing the **GATEWAY_IP** works because of the permissive mode, where
plain text traffic gets sent to the services that do not have the proxy.
In this case, the ingress gateway sends plain text traffic to
the **web-frontend** because there\'s no proxy.

# Observing the Permissive Mode in Kiali

If we open Kiali with **istioctl dash kiali** and look at the Graph, you
will notice that Kiali detects calls made from the ingress gateway to
the web frontend. Make sure you select both the **default** namespace
and the **istio-system** namespace.

However, the calls made to the **customers** service are coming
from **unknown** service. This is because there\'s no proxy next to
the **web-frontend**. Therefore Istio doesn\'t know who, where or what
that service is.

 

![Calls to customers-v1 service are coming from an unknown service
because there's no proxy
injected](https://github.com/nadidurna/Istio/blob/master/images/image26.png)

**Calls to customers-v1 service are coming from an unknown service
because there's no proxy injected**

# Exposing the \"customers\" Service

Let\'s update the **customers** VirtualService and attach the gateway to
it. Attaching the gateway allows us to make calls directly to the
service.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above to **vs-customers-gateway.yaml** and update the
VirtualService using **kubectl apply -f vs-customers-gateway.yaml**.

# Generating Traffic to the \"customers\" Service

We can now specify the **Host** header and send the requests through the
ingress gateway (**GATEWAY_IP**) to the **customers** service:

**curl -H \"Host: customers.default.svc.cluster.local\"
http://\$GATEWAY_IP**\
**\[{\"name\":\"Jewel Schaefer\"},{\"name\":\"Raleigh
Larson\"},{\"name\":\"Eloise Senger\"},{\"name\":\"Moshe
Zieme\"},{\"name\":\"Filiberto Lubowitz\"},{\"name\":\"Ms.Kadin
Kling\"},{\"name\":\"Jennyfer Bergstrom\"},{\"name\":\"Candelario
Rutherford\"},{\"name\":\"Kenyatta Flatley\"},{\"name\":\"Gianni
Pouros\"}\]**

To generate some traffic to both
the **web-frontend **and **customers-v1** deployments through the
ingress, open the two terminal windows, and run one command in each:

**// Terminal 1 \
while true; do curl -H \"Host: customers.default.svc.cluster.local\"
http://\$GATEWAY_IP; done**

**// Terminal 2\
while true; do curl http://\$GATEWAY_IP; done**

# Observing the Traffic in Kiali

Open Kiali and look at the Graph. From the Display dropdown, make sure
you check the Security option. You should see a graph similar to the one
in the following figure.

 

![Traffic between istio-ingressgateway and customers-v1 is using mTLS as
indicated by the padlock
icon](https://github.com/nadidurna/Istio/blob/master/images/image27.png)

**Traffic between istio-ingressgateway and customers-v1 is using mTLS as
indicated by the padlock icon**

 

Notice a padlock icon between the **istio-ingressgateway** and
the **customers** service, which means the traffic gets sent using mTLS.

***NOTE**: If you do not see the padlock icon, click the Display
dropdown and ensure the \"**Security**\" option is selected. *

However, there\'s no padlock between the **unknown** and
the **customers-v1** service, as well as
the **istio-ingress-gateway** and **web-frontend**. Proxies send plain
text traffic between services that do not have the sidecar injected.

# Enabling \"STRICT\" mode

Let\'s see what happens if we enable mTLS in **STRICT** mode. We expect
the calls from the **web-frontend** to the **customers-v1** service to
fail because there\'s no proxy injected to do the mTLS communication.

On the other hand, the calls from the ingress gateway to
the **customers-v1** service will continue working.

**apiVersion: security.istio.io/v1beta1**\
**kind: PeerAuthentication**\
**metadata:**\
**  name: default**\
**  namespace: default**\
**spec:**\
**  mtls:**\
**    mode: STRICT**

Save the above YAML to **strict-mtls.yaml** and create the
PeerAuthentication resource using **kubectl apply -f strict-mtls.yaml**.

If we still have the request loop running, we will see
the **ECONNRESET** error message from the **web-frontend**. This error
indicates that the **customers-v1** service closed the connection. In
our case, it was because it was expecting an mTLS connection.

On the other hand, the requests we make directly to
the **customers-v1** service continue to work because
the **customers-v1** service has an Envoy proxy running next to it and
can do mTLS.

If we delete the PeerAuthentication resource deployed earlier
using **kubectl delete peerauthentication default**, Istio returns to
its default, the **PERMISSIVE** mode, and the errors will disappear.

# Cleanup

To clean up the resources, run:

**kubectl delete deploy web-frontend customers-v1**\
**kubectl delete svc customers web-frontend**\
**kubectl delete vs customers web-frontend**\
**kubectl delete gateway gateway**

# Authenticating Users

While the PeerAuthentication resource is used to control service
authentication, the RequestAuthentication resource is used for end-user
authentication. The authentication is done per request and verifies the
credentials attached to the request in JSON Web Tokens (JWTs).

Just like we used the SPIFFE identity to identify services, we can use
JWT to authenticate users. 

Let's look at an example RequestAuthentication resource:

**apiVersion: security.istio.io/v1beta1**\
**kind: RequestAuthentication**\
**metadata:**\
**name: customers-v1**\
**namespace: default**\
**spec:**\
** selector:**\
**   matchLabels:**\
**     app: customers-v1**\
** jwtRules:**\
** - issuer: \"issuer@tetrate.io\"**\
**   jwksUri: \"someuri\"**

 The above resource applies to all workloads in the default namespace
that match the selector labels. The resource is saying that any requests
made to those workloads will need a JWT attached to the request.

The RequestAuthentication resource configures how the token and its
signature are authenticated using the settings in
the **jwtRules** field. If the request doesn't have a valid JWT
attached, the request will be rejected because the token doesn't conform
to JWT rules. The request will not be authenticated if we do not provide
a token. 

If the token is valid, then we have an authenticated principal. We can
use the principal to configure authorization policies.

# Authorization (authz)

**Authorization** is answering the access control question. Is a
principal allowed to perform an action on an object? Even though we
might have an authenticated principal, we still might not be able to
perform a certain action. 

For example, you can be an authenticated user, but you won't be able to
perform administrative actions if you are not authorized. Similarly, a
service can be authenticated but is not allowed to make POST requests to
other services, for example.

To properly enforce access control, we need
both **authentication** and **authorization**. If we only authenticate
the principles without authorizing them, the principals can perform any
action on any objects.

Similarly, if we only authorize the actions or requests, a principal can
pretend to be someone else and perform all actions again.

# Authorization Policies

Whether we get the principals from the PeerAuthentication resource or
RequestAuthentication, we can use these identities to write
authorization policies with the AuthorizationPolicy resource.

We can use the **principals** field to write policies based on service
identities and the **requestPrincipals** field to write policies based
on users. 

Let's look at an example:

**apiVersion: security.istio.io/v1beta1**\
**kind: AuthorizationPolicy**\
**metadata:**\
** name: require-jwt**\
** namespace: default**\
**spec: selector:**\
**   matchLabels:**\
**     app: hello-world**\
** action: ALLOW**\
** rules:**\
** - from:**\
**   - source:**\
**      requestPrincipals: \[\"\*\"\]**

The policy applies to all workloads in the default namespace matching
the selector labels. Each AuthorizationPolicy includes an action - that
says whether the action is allowed or denied based on the rules set in
the **rules** field. Note that there are other actions we can set, and
we will explain them later.

In this example, we aren't checking for any specific principal. Instead,
we just need a principal to be set. With this combination and a
corresponding RequestAuthentication resource, we guarantee that only
authenticated requests will reach the workloads in
the **default** namespace, labeled with the **app: hello-world label**.

Let's look at the three possible scenarios we can encounter in this
case.

1.  **Token is not present**\
    If the token is not present in the request, it means that the
    request is not authenticated (that's what RequestAuthentication
    resource ensures). Because the request is not authenticated,
    the **requestPrincipals** field value won't be set. However, because
    we require a request principal to be set (the "**\***" notation) in
    the AuthorizationPolicy, but there isn't one, the request will be
    denied.

2.  **Token is present but invalid**\
    The validity of the JWT is checked by the RequestAuthentication. If
    that fails, the AuthorizationPolicy won't even be processed.

3.  **Token is valid**\
    If we provided a valid token, the **requestPrincipals** field will
    be set. In the **rules** field of the AuthorizationPolicy, we only
    allow the calls to be made from sources with the request principal
    set. Since that is set, we can send the requests to the workloads
    with the **app: hello-world** labels set.

# Sources Identities (\"from\" field)

Using the **from** field we can set the source identities of a request.
The sources we can use are:

-   principals

-   request principals

-   namespaces

-   IP blocks

-   remote IP blocks

We can also provide a list of negative matches for each one of the
sources and combine them with logical AND.

For example:

**rules:**\
**- from:**\
**  - source:**\
**      ipBlocks: \[\"10.0.0.1\", "10.0.0.2"\]**\
**  - source:**\
**      notNamespaces: \[\"prod\"\]**\
**  - source:**\
**      requestPrincipals:**\
**      - \"tetrate.io/peterj\"**

The above rules would apply to sources coming from one of the two IP
addresses that are not in the prod namespace and have the request
principal set to **tetrate.io/peterj**.

# Request Operation (\"to\" field)

The **to** field specifies the operations of a request. We can use the
following operations:

-   -   -   hosts

        -   ports

        -   methods

        -   paths

For each of the above, we can set the list of negative matches and
combine them with the logical AND. 

For example:

**to:**\
**  - operation:**\
**      host: \[\"\*.hello.com\"\]**\
**      methods: \[\"DELETE\"\]**\
**      notPaths: \[\"/admin\*\"\]**

The above operation matches if the host ends with **\*.hello.com** and
the method is **DELETE**, but the path doesn't start with **/admin**.

# Conditions (\"when\" field)

With the **when** field, we can specify any additional conditions based
on the [Istio
attributes](https://istio.io/latest/docs/reference/config/security/conditions/).
The attributes include headers, source and remote IP address, auth
claims, destination port and IP addresses, and others.

For example:

**when:**\
**   - key: request.auth.claims\[iss\]**\
**     values: \[\"https://accounts.google.com\"\]**\
**   - key: request.headers\[User-Agent\]**\
**     notValues: \[\"curl/\*\"\]**

The above condition evaluates to true if the **iss** claim from the
authenticated JWT equals the provided value and the **User
Agent** header doesn't start with **curl/**.

# Configure the \"action\" Field

Once we have written the rules, we can also configure
the **action** field. We can either **ALLOW** or **DENY** the requests
matching those rules. The additional supported actions
are **CUSTOM** and **AUDIT**.

For example,

**spec:**\
**  action: DENY**\
**  rules:**\
**  - from:**\
**    to:**\
**    when:**\
**    \...**

The **CUSTOM** action is when we specify our custom extension to handle
the request. The custom extension needs to be configured in
the **MeshConfig**. An example of using this is if we wanted to
integrate a custom external authorization system to delegate the
authorization decisions to it. Note that the CUSTOM action is
experimental, so it might break or change in future Istio versions.

The **AUDIT** action can be used to audit a request that matches the
rules. If the request matches the rules, the **AUDIT** action triggers
logging that request. This action doesn't affect whether the requests
are allowed or denied. Only **DENY**, **ALLOW**, and **CUSTOM** actions
can do that. A sample scenario for when one would use
the **AUDIT** action is when you are migrating workloads
from **PERMISSIVE** to **STRICT** mTLS mode.

# How Are Rules Evaluated?

If we set any authorization policies, everything will be allowed.
However, requests that do not conform to the allow policy rules will be
denied as soon as we set any **ALLOW** policies.

The rules get evaluated in the following order:

1.  **CUSTOM** policies

2.  **DENY** policies

3.  **ALLOW** policies

If we have multiple policies defined, they are aggregated and evaluated,
starting with the **CUSTOM** policies. So, if we have two rules - one
for **ALLOW** and one for **DENY**, the **DENY** rule will be enforced.
If we use all three actions, the **CUSTOM** action is evaluated first. 

Assume we have defined policies that use all three actions. We will
evaluate the **CUSTOM** policies and then deny or allow the request
based on that. Next, the **DENY** policies matching the request get
evaluated. If we get past those, and if there are no **ALLOW** policies
set, we will allow the request. If any **ALLOW** policies match the
request, we will also allow it. If they do not, the request gets denied.
For a complete flowchart of authorization policy precedence, [[refer to
this
page]{.underline}](https://istio.io/latest/docs/concepts/security/#implicit-enablement).  

A good practice is to create a policy denying all requests. Once we have
that in place, we can create individual **ALLOW** policies and
explicitly allow communication between services.

# Access Control

In this lab, we will learn how to use an authorization policy to control
access between workloads. Let\'s start by deploying the Gateway:

**apiVersion: networking.istio.io/v1alpha3**\
**kind: Gateway**\
**metadata:**\
**  name: gateway**\
**spec:**\
**  selector:**\
**    istio: ingressgateway**\
**  servers:**\
**    - port:**\
**        number: 80**\
**        name: http**\
**        protocol: HTTP**\
**      hosts:**\
**        - \'\*\'**

***NOTE: **You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above YAML to **gateway.yaml** and deploy the Gateway
using **kubectl apply -f gateway.yaml**.

# Deploying the Applications (1)

Next, we will create the **web-frontend** deployment, service account,
Kubernetes service, and a VirtualService. 

**apiVersion: v1**\
**kind: ServiceAccount**\
**metadata:**\
**  name: web-frontend**\
**\-\--**\
**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: web-frontend**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: web-frontend**\
**        version: v1**\
**    spec:**\
**      serviceAccountName: web-frontend**\
**      containers:**\
**        - image: gcr.io/tetratelabs/web-frontend:1.0.0**\
**          imagePullPolicy: Always**\
**          name: web**\
**          ports:**\
**            - containerPort: 8080**\
**          env:**\
**            - name: CUSTOMER_SERVICE_URL**\
**              value: \'http://customers.default.svc.cluster.local\'**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: web-frontend**\
**  labels:**\
**    app: web-frontend**\
**spec:**\
**  selector:**\
**    app: web-frontend**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 8080**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: web-frontend**\
**spec:**\
**  hosts:**\
**    - \'\*\'**\
**  gateways:**\
**    - gateway**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: web-frontend.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above YAML to **web-frontend.yaml** and create the resources
using **kubectl apply -f web-frontend.yaml**.

# Deploying the Applications (2)

Finally, we will deploy the **customers v1** service:

**apiVersion: v1**\
**kind: ServiceAccount**\
**metadata:**\
**  name: customers-v1**\
**\-\--**\
**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: customers-v1**\
**  labels:**\
**    app: customers**\
**    version: v1**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: customers**\
**      version: v1**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: customers**\
**        version: v1**\
**    spec:**\
**      serviceAccountName: customers-v1**\
**      containers:**\
**        - image: gcr.io/tetratelabs/customers:1.0.0**\
**          imagePullPolicy: Always**\
**          name: svc**\
**          ports:**\
**            - containerPort: 3000**\
**\-\--**\
**kind: Service**\
**apiVersion: v1**\
**metadata:**\
**  name: customers**\
**  labels:**\
**    app: customers**\
**spec:**\
**  selector:**\
**    app: customers**\
**  ports:**\
**    - port: 80**\
**      name: http**\
**      targetPort: 3000**\
**\-\--**\
**apiVersion: networking.istio.io/v1alpha3**\
**kind: VirtualService**\
**metadata:**\
**  name: customers**\
**spec:**\
**  hosts:**\
**    - \'customers.default.svc.cluster.local\'**\
**  http:**\
**    - route:**\
**        - destination:**\
**            host: customers.default.svc.cluster.local**\
**            port:**\
**              number: 80**

Save the above to **customers-v1.yaml** and create the deployment and
service using **kubectl apply -f customers-v1.yaml**. 

We can set the environment variable called **GATEWAY_IP** that stores
the gateway IP address:

**export GATEWAY_IP=\$(kubectl get svc -n istio-system
istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

If we open the **GATEWAY_IP**, the web page with the data from
the **customers-v1** service is displayed as shown in the figure.

 

![Response from web-frontend and customers-v1
service](https://github.com/nadidurna/Istio/blob/master/images/image28.png)

# Denying All Requests

Let\'s start by creating an AuthorizationPolicy resource that denies all
requests between services in the **default** namespace.

**apiVersion: security.istio.io/v1beta1**\
**kind: AuthorizationPolicy**\
**metadata:**\
** name: deny-all**\
** namespace: default**\
**spec:**\
**  {}**

Save the above to **deny-all.yaml** and create the policy
using **kubectl apply -f deny-all.yaml**.

If we try to send a request to **GATEWAY_IP** we will get back the
access denied response:

**curl \$GATEWAY_IP**\
**RBAC: access denied**

Similarly, if we run a pod inside the cluster and make a request from
within the **default** namespace to either the **web-frontend** or
the **customers-v1** service, we will get the same error.

Let\'s try that:

**kubectl run curl \--image=radial/busyboxplus:curl -i \--tty**\
**If you don\'t see a command prompt, try pressing enter.**\
**\[ root@curl:/ \]\$ curl customers**\
**RBAC: access denied**\
**\[ root@curl:/ \]\$ curl web-frontend**\
**RBAC: access denied**

In both cases, we get back the access denied error. You can
type **exit**, to exit the container.

# Enabling Requests from Ingress to \"web-frontend\"

We will first allow requests to be sent from the ingress gateway to
the **web-frontend** workload using the **ALLOW** action.

The rules section specifies the source namespace (**istio-system**)
where the ingress gateway is running and the ingress gateway\'s service
account name in the **principals** field.

**apiVersion: security.istio.io/v1beta1**\
**kind: AuthorizationPolicy**\
**metadata:**\
**  name: allow-ingress-frontend**\
**  namespace: default**\
**spec:**\
**  selector:**\
**    matchLabels:**\
**      app: web-frontend**\
**  action: ALLOW**\
**  rules:**\
**    - from:**\
**        - source:**\
**            namespaces: \[\"istio-system\"\]**\
**        - source:**\
**            principals:
\[\"cluster.local/ns/istio-system/sa/istio-ingressgateway-service-account\"\]**

Save the above YAML to **allow-ingress-frontend.yaml** and deploy the
policy using **kubectl apply -f allow-ingress-frontend.yaml**.

Suppose we try to send a request to the **GATEWAY_IP**. In that case, we
will get a different error:

**curl http://\$GATEWAY_IP**\
**\"Request failed with status code 403\"**

***Note: ** It takes a couple of seconds for Istio to distribute the
policy to all proxies, so you might still see the **RBAC: access
denied** message for the first couple of requests.*

This error is coming from the **customers-v1** service - remember, we
allowed calls from the ingress gateway to the **web-frontend**.
However, **web-frontend** still isn't allowed to make calls to
the **customers-v1** service.

If we go back to the curl pod we are running inside the cluster and try
to send a request to **http://web-frontend** we will get an RBAC error:

**kubectl run curl \--image=radial/busyboxplus:curl -i \--tty**\
**curl http://web-frontend**\
**RBAC: access denied**

The initial **DENY** policy we deployed is still in effect. We only
allow calls to be made from the ingress gateway to the **web-frontend**.
Calls between other services (including the curl pod) will be denied.

When we deployed the **web-frontend**, we created a service account and
assigned it to the pod. The service account represents the pods'
identity.

Each namespace in Kubernetes has a **default** service account. If we do
not explicitly assign a service account to the pod, the default service
account from the namespace is used. It is good practice to create a
separate service account for each deployment. Otherwise, all pods in the
same namespace will have the same identity, which is useless when trying
to enforce access policies.

# Enabling Requests from \"web-frontend\" to \"customers\"

We can use the service account to configure which workloads can make
requests to the **customer**.

**apiVersion: security.istio.io/v1beta1**\
**kind: AuthorizationPolicy**\
**metadata:**\
**  name: allow-web-frontend-customers**\
**  namespace: default**\
**spec:**\
**  selector:**\
**    matchLabels:**\
**        app: customers**\
**        version: v1**\
**  action: ALLOW**\
**  rules:**\
**  - from:**\
**    - source:**\
**        namespaces: \[\"default\"\]**\
**      source:**\
**        principals: \[\"cluster.local/ns/default/sa/web-frontend\"\]**

Save the above YAML to **allow-web-frontend-customers.yaml** and create
the policy using **kubectl apply -f
allow-web-frontend-customers.yaml**. 

As soon as the policy gets created, we can send a couple of requests
to **\$GATEWAY_IP**, and we will get back the responses
from **customers** service. If we would go back to the **curl** and try
sending requests to **web-frontend** or **customers**, we would still
get back the *RBAC: access denied* error message because
the **curl** pod is not permitted to make calls to those services.

In this lab, we have used multiple authorization policies to explicitly
allow calls from the ingress to the front end and the front end to the
backend service. Using a **deny-all** approach is an excellent way to
start because we can control, manage, and then explicitly allow the
communication we want to happen between individual services.

# Cleanup

To clean up and delete all deployed resources, run:

**kubectl delete sa customers-v1 web-frontend**\
**kubectl delete deploy web-frontend customers-v1**\
**kubectl delete svc customers web-frontend**\
**kubectl delete vs customers web-frontend**\
**kubectl delete gateway gateway**\
**kubectl delete authorizationpolicy allow-ingress-frontend
allow-web-frontend-customers deny-all**\
**kubectl delete pod curl**

# Provisioning Identities at Runtime

The process of provisioning identities involves the following components
of the service mesh:

-   -   -   Istio agent, running in the sidecar

        -   Envoy's Secret Discovery Service (SDS)

        -   Citadel (part of Istio control plane)

The Istio agent (a binary called pilot-agent) runs in the sidecar and
works with Istio's control plane to automate key and certificate
rotation. Even though the agent runs in the sidecar containers, it's
still considered part of the control plane.

The Envoy proxy in the sidecar gets its initial bootstrap configuration
as a JSON file. Amongst other settings, the JSON file configures the
Istio agent as the SDS server, telling the Envoy proxy to go to the SDS
server for any certificate/key needs. For example, the SDS server will
automatically push the certificate to the Envoy proxy, removing the need
for creating the certificates as Kubernetes secrets and mounting them
inside the containers' file system.

The Istio agent is responsible for launching the Envoy proxy. When the
Envoy proxy starts, it reads its bootstrap configuration and sends the
SDS request to the Istio agent, telling it the workloads service
account.\</p

 

![Certificate issuance flow in
Istio](https://github.com/nadidurna/Istio/blob/master/images/image29.png)

**Certificate issuance flow in Istio**

 

The Istio agent sends a request to the certificate authority (CA), which
in this case is Citadel, for a new workload certificate. The Citadel
component plays the role of the certificate authority. By default,
Citadel uses a self-signed root certificate to sign all workload
certificates. We can change that by providing our root certificate,
signing certificate, and key for Citadel to use. 

The request to the CA involves creating a certificate signing request
(CSR) and includes proof of the workloads' service account. In
Kubernetes, that's the pods' service account JWT.

Next, Citadel performs authentication and authorization and responds
with a signed X.509 certificate. 

The Istio agent takes the signed certificate and caches it in memory.
Additionally, the Istio agent registers the certificate for automatic
rotation before the certificate expires. The default expiration of mesh
certificates is 24 hours. This value is configurable and can be
changed. 

In the last step, the Istio agent streams back the signed certificate to
the Envoy proxy via SDS over a Unix domain socket. This allows Envoy
proxy to use the certificate when needed.

[[Chapter 6. Extending the
Mesh]{.underline}](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS144x+3T2022/block-v1:LinuxFoundationX+LFS144x+3T2022+type@sequential+block@2dc12437a8664185bdb5750d26474191)

# Chapter Overview

This chapter explains a couple of ways to extend Istio service mesh
functionality. We'll learn about the basics of Envoy proxy, how to use
EnvoyFilter resource to customize the Envoy configuration and how to use
Wasm plugins to extend the Envoy proxy functionality.

# Learning Objectives

By the end of this chapter, you should be able to:

-   Understand the basic building blocks of the Envoy proxy.

-   Understand how to customize the Envoy proxy configuration using the
    EnvoyFilter resource.

-   Understand how to build a basic Wasm plugin using Go and Proxy-Wasm
    SDK.

-   Understand how to use WasmPlugin resource to deploy a Wasm plugin to
    Istio service mesh.

# Overview of Envoy Proxy

Before explaining the EnvoyFilter resource, we need an overview of the
Envoy configuration and its building blocks.

Envoy configuration is a JSON file that's divided into multiple
sections. The basic building blocks and concepts we need to understand
are shown in the figure below.

 

![Envoy building
blocks](https://github.com/nadidurna/Istio/blob/master/images/image30.png)

**Envoy building blocks**

# Envoy Listeners

The **listeners** are named network locations, typically an IP and port.
Envoy listens on these locations, where it receives the connections and
requests. Istio generates multiple listeners for each sidecar.

For example, there's a listener bound to **0.0.0.0:15006** - this is
where all incoming traffic to the pod gets routed. The second example is
listener **0.0.0.0:15001**, where all traffic exiting the pod gets
routed.

Between the listeners and routes, the requests in Envoy go through a
chain of filters. The filters can process and augment requests, generate
statistics, translate protocols, modify the requests, and so on.

# Envoy Filters

The request starts at the Envoy listener and flows through an ordered
list of filters called a **filter chain.**

 

![Filters in
Envoy](https://github.com/nadidurna/Istio/blob/master/images/image31.png)

 

Each filter in the chain can augment the request and decide whether to
hand off the request to the next filter or stop processing the request.
The last filter in the chain is responsible for routing the request to
its destination.

At a high level, there are three types of filters in Envoy:

-   **Listener filters**\
    Listener filters have access to raw data and can manipulate metadata
    of layer 4 of the OSI model (L4) connections during the initial
    connection phase. An example of a listener filter is a TLS inspector
    filter that identifies whether the connection is TLS-encrypted and,
    if so, extracts relevant TLS information from it.

-   **Network filters**\
    Network filters work with raw data in the form of TCP packets. The
    most popular network filter is the HTTP connection manager filter
    (HCM). Other example network filters are a rate limit filter, Redis
    proxy filter, Mongo proxy, connection limit filter, and others. See
    the complete list of built-in network
    filters [[here]{.underline}](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/network_filters).

-   **HTTP filters**\
    The HTTP filters operate at L7 with the HTTP data. The HTTP
    connection manager filter optionally creates HTTP filters. The HCM
    filter translates from raw data to HTTP data so that the HTTP
    filters can manipulate HTTP requests and responses.

# Envoy Routes

When routing the HTTP traffic, the last HTTP filter in the chain must be
the router filter. This filter is responsible for routing traffic, which
brings us to the second configuration block - the **routes**.

Within the route configuration, we define the virtual hosts and then
write the routes that can match the incoming requests based on the URI,
headers, etc.; based on that, specify where the traffic is sent. Within
the routes, we can send traffic to clusters.

# Envoy Clusters and Endpoints

In the Envoy proxy world, a **cluster** is a group of familiar upstream
hosts that accept traffic. A concept similar to the clusters is a
Kubernetes Service. 

***NOTE: **We use the term "upstream" when we mean a destination or a
server, and "downstream" when talking about a client or source of the
request.*

The **endpoints** are part of clusters and the addresses where traffic
can be sent. For example, an IP address or a hostname.

Here's an example of an Envoy configuration that features the different
building blocks we described:

**static_resources:**\
**  listeners:**\
**  - name: listener_0**\
**    address:**\
**      socket_address:**\
**        address: 0.0.0.0**\
**        port_value: 10000**\
**    filter_chains:**\
**    - filters:**\
**      - name: envoy.filters.network.http_connection_manager**\
**        typed_config:**\
**          \"@type\":
type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager**\
**          stat_prefix: hello_world**\
**          http_filters:**\
**          - name: envoy.filters.http.router**\
**          route_config:**\
**            name: my_first_route**\
**            virtual_hosts:**\
**            - name: my_vhost**\
**              domains: \[\"\*\"\]**\
**              routes:**\
**              - match:**\
**                  prefix: \"/\"**\
**                route:**\
**                  cluster: hello_world_service**\
**  clusters:**\
**  - name: hello_world_service**\
**    connect_timeout: 5s**\
**    load_assignment:**\
**      cluster_name: hello_world_service**\
**      endpoints:**\
**      - lb_endpoints:**\
**        - endpoint:**\
**            address:**\
**              socket_address:**\
**                address: 127.0.0.1**\
**                port_value: 8000**

The above configuration defines a listener called **listener_0**,
listening on port **10000**. We have a single filter in the network
chain, the HCM filter. Within the HCM filter, we declare the router
filter and provide the route configuration. 

In the route configuration, we're matching on any virtual host
(the **\*** in the domains array) and matching on the prefix "**/**". If
the request matches these conditions, we route it to a cluster
called **hello_world_service**. In the cluster definition, we're
specifying a list of load balancing endpoints, with a single endpoint
at **127.0.0.1:8000**.

# Inspecting Envoy Configuration

Let's look at an example of how to use Istio CLI to look at the sidecar
Envoy configuration for workloads in the mesh. We'll explain the
concepts using the **web-frontend** and **customers** applications we
used in previous labs. We\'ll use the CLI outputs to explain how the
requests get routed within Envoy.

# Inspecting the Listeners

Using the **istioctl proxy-config** command, we can list all listeners
defined in the sidecar proxy in the web-frontend pod. Here's how the
output looks like:

**istioctl proxy-config listeners web-frontend-64455cd4c6-p6ft2**\
**ADDRESS      PORT  MATCH   DESTINATION**\
**10.124.0.10  53    ALL     Cluster:
outbound\|53\|\|kube-dns.kube-system.svc.cluster.local**\
**0.0.0.0      80    ALL     PassthroughCluster**\
**10.124.0.1   443   ALL     Cluster:
outbound\|443\|\|kubernetes.default.svc.cluster.local**\
**10.124.3.113 443   ALL     Cluster:
outbound\|443\|\|istiod.istio-system.svc.cluster.local**\
**10.124.7.154 443   ALL     Cluster:
outbound\|443\|\|metrics-server.kube-system.svc.cluster.local**\
**10.124.7.237 443   ALL     Cluster:
outbound\|443\|\|istio-egressgateway.istio-system.svc.cluster.local**\
**10.124.8.250 443   ALL     Cluster:
outbound\|443\|\|istio-ingressgateway.istio-system.svc.cluster.local**\
**10.124.3.113 853   ALL     Cluster:
outbound\|853\|\|istiod.istio-system.svc.cluster.local**\
**0.0.0.0      8383  ALL     PassthroughCluster**\
**0.0.0.0      15001 ALL     PassthroughCluster**\
**0.0.0.0      15006 ALL     Inline Route: /\***\
**0.0.0.0      15010 ALL     PassthroughCluster**\
**10.124.3.113 15012 ALL     Cluster:
outbound\|15012\|\|istiod.istio-system.svc.cluster.local**\
**0.0.0.0      15014 ALL     PassthroughCluster**\
**0.0.0.0      15021 ALL     Non-HTTP/Non-TCP**\
**10.124.8.250 15021 ALL     Cluster:
outbound\|15021\|\|istio-ingressgateway.istio-system.svc.cluster.local**\
**0.0.0.0      15090 ALL     Non-HTTP/Non-TCP**\
**10.124.7.237 15443 ALL     Cluster:
outbound\|15443\|\|istio-egressgateway.istio-system.svc.cluster.local**\
**10.124.8.250 15443 ALL     Cluster:
outbound\|15443\|\|istio-ingressgateway.istio-system.svc.cluster.local**\
**10.124.8.250 31400 ALL     Cluster:
outbound\|31400\|\|istio-ingressgateway.istio-system.svc.cluster.local**

The request from the **web-frontend** to **customers** is an outbound
HTTP request to port **80**. The request gets handed off to
the **0.0.0.0:80** virtual listener. We can use Istio CLI to filter the
listeners by address and port. We can add the **-o json** flag to the
command and get a JSON representation of the listener:

**istioctl proxy-config listeners web-frontend-58d497b6f8-lwqkg
\--address 0.0.0.0 \--port 80 -o json**\
**\...**\
**\"rds\": {**

**   \"configSource\": {**\
**      \"ads\": {},**\
**      \"resourceApiVersion\": \"V3\"**\
**   },**\
**   \"routeConfigName\": \"80\"**\
**},**\
**\...**

The listener uses RDS (Route Discovery Service) to find the route
configuration (**80** in our case). Routes are attached to listeners and
contain rules that map **virtual hosts** to clusters.

We can create traffic routing rules because Envoy can look at headers or
paths (the request metadata) and route traffic.

# Inspecting the Routes

As we mentioned, a route selects a **cluster**, and a cluster is a group
of similar upstream hosts that accept traffic. In this example, the
collection of all instances of the **web-frontend** service is a cluster
in the Envoy configuration. We can configure resiliency features within
a cluster, such as circuit breakers, outlier detection, and TLS config.

Using the **routes** command, we can get the route details by filtering
all routes by the name:

**istioctl proxy-config routes web-frontend-58d497b6f8-lwqkg \--name 80
-o json\
\[**\
**    {**\
**        \"name\": \"80\",**\
**        \"virtualHosts\": \[**\
**            {**\
**                \"name\":
\"customers.default.svc.cluster.local:80\",**\
**                \"domains\": \[**\
**                    \"customers.default.svc.cluster.local\",**\
**                    \"customers.default.svc.cluster.local:80\",**\
**                    \"customers\",**\
**                    \"customers:80\",**\
**                    \"customers.default.svc.cluster\",**\
**                    \"customers.default.svc.cluster:80\",**\
**                    \"customers.default.svc\",**\
**                    \"customers.default.svc:80\",**\
**                    \"customers.default\",**\
**                    \"customers.default:80\",**\
**                    \"10.124.4.23\",**\
**                    \"10.124.4.23:80\"**\
**                \],**\
**                \],**\
**                \"routes\": \[**\
**                    {**\
**                        \"match\": {**\
**                            \"prefix\": \"/\"**\
**                        },**\
**                        \"route\": {**\
**                            \"cluster\":
\"outbound\|80\|v1\|customers.default.svc.cluster.local\",**\
**                            \"timeout\": \"0s\",**\
**                            \"retryPolicy\": {**\
**                                \"retryOn\":
\"connect-failure,refused-stream,unavailable,cancelled,retriable-status-codes\",**\
**                                \"numRetries\": 2,**\
**                                \"retryHostPredicate\": \[**\
**                                    {**\
**                                        \"name\":
\"envoy.retry_host_predicates.previous_hosts\"**\
**                                    }**\
**                                \],**\
**                                \"hostSelectionRetryMaxAttempts\":
\"5\",**\
**                                \"retriableStatusCodes\": \[**\
**                                    503**\
**                                \]**\
**                            },**\
**                            \"maxGrpcTimeout\": \"0s\"**\
**                        },**\
**\...**

The route **80** configuration has a virtual host for each service.
Assuming we're sending a request to
c**ustomers.default.svc.cluster.local**, Envoy selects the virtual host
that matches one of those domains
(**customers.default.svc.cluster.local:80**).

# Inspecting the Clusters

Once the domain is matched, Envoy looks at the routes and picks the
first one that matches the request. Since we do not have any special
routing rules defined, it matches the first (and only) described route.
Then, it instructs Envoy to send the request to the cluster
named **outbound\|80\|v1\|customers.default.svc.cluster.local**. 

***NOTE:** the **v1** in the cluster\'s name is because we have a
DestinationRule deployed that creates the **v1** subset. If there are no
subsets for a service, that part would be left empty, for
example: **outbound\|80\|\|customers.default.svc.cluster.local**.*

We can look up more details now that we have the cluster name. To get an
output that clearly shows the FQDN, port, subset and other information,
we can omit the **-o json** flag:

**istioctl proxy-config cluster web-frontend-58d497b6f8-lwqkg \--fqdn
customers.default.svc.cluster.local**\
**SERVICE FQDN                            PORT     SUBSET     DIRECTION 
   TYPE     DESTINATION RULE**\
**customers.default.svc.cluster.local     80       -          outbound 
    EDS      customers.default**\
**customers.default.svc.cluster.local     80       v1         outbound 
    EDS      customers.default**

Finally, using the cluster name, we can look up the actual endpoints the
request will end up at:

**istioctl proxy-config endpoints web-frontend-58d497b6f8-lwqkg
\--cluster \"outbound\|80\|v1\|customers.default.svc.cluster.local\"**\
**ENDPOINT            STATUS      OUTLIER CHECK     CLUSTER**\
**10.120.0.4:3000     HEALTHY     OK               
outbound\|80\|v1\|customers.default.svc.cluster.local**

The endpoint address equals the pod IP where the Customer application
runs. If we scale the **customers** deployment, additional endpoints
show up in the output, like this:

**istioctl proxy-config endpoints web-frontend-58d497b6f8-lwqkg
\--cluster \"outbound\|80\|v1\|customers.default.svc.cluster.local\"**\
**ENDPOINT            STATUS      OUTLIER CHECK     CLUSTER**\
**10.120.0.4:3000     HEALTHY     OK               
outbound\|80\|v1\|customers.default.svc.cluster.local**\
**10.120.3.2:3000     HEALTHY     OK               
outbound\|80\|v1\|customers.default.svc.cluster.local**

We can also visualize the above flow using the figure below.

 

![Visualization of a request sent to customers service
](https://github.com/nadidurna/Istio/blob/master/images/image32.png)

**Visualization of a request sent to customers service**

# EnvoyFilter Resource

Istio automatically generates the Envoy configuration for each proxy in
the mesh.  To customize the Envoy proxy configuration, we can use
the **[EnvoyFilter]{.underline}** resource. 

EnvoyFilters can update configuration values, add or remove filters,
create new listeners, clusters, etc.

We can apply the EnvoyFilter resource at three levels:

1.  Global, affecting all proxies in the mesh

2.  Namespace, affecting all proxies in the namespace

3.  Workload, affecting specific workloads

The way EnvoyFilter resource works is by allowing us to customize
portions of the Envoy proxy configuration. We can customize it by
patching the existing configuration.

# Where to Apply the Patch

First, we need a way to describe where in the configuration we want to
apply the patch. We can do this with the **applyTo** field. For example,
we can apply the patch to a listener (**LISTENER**), network filter
(**NETWORK_FILTER**), HTTP filter (**HTTP_FILTER**), virtual host
(**VIRTUAL_HOST**), etc. You can find the complete list in
the [reference
documentation](https://istio.io/latest/docs/reference/config/networking/envoy-filter/#EnvoyFilter-ApplyTo).

Optionally, we can provide a more exact location for the patch using the
match field. The field allows us to specify one or more match conditions
that have to be met before the patch gets applied. We can construct the
match conditions using the values in the table below.

+-----------------------------------+-----------------------------------+
| ### Field                         | ### Description                   |
+===================================+===================================+
| **context**                       | Specifies which proxies to apply  |
|                                   | the patch to. We can apply the    |
|                                   | patch to inbound                  |
|                                   | (**SIDECAR_INBOUND**) or outbound |
|                                   | (**SIDECAR_OUTBOUND**) proxies,   |
|                                   | gateways (**GATEWAY**), or any    |
|                                   | proxy, regardless if running as a |
|                                   | sidecar or a gateway (**ANY**).   |
+-----------------------------------+-----------------------------------+
| **proxy**                         | Match on proxy properties. For    |
|                                   | example, we can match the proxy   |
|                                   | version or the node metadata set  |
|                                   | on the proxy.                     |
+-----------------------------------+-----------------------------------+
| **listener**                      | Match on Envoy listener           |
|                                   | attributes. For example, we can   |
|                                   | match the port number, name of    |
|                                   | the listener, and specific        |
|                                   | filters in the chain and their    |
|                                   | properties.                       |
+-----------------------------------+-----------------------------------+
| **routeConfiguration**            | Match on HTTP route configuration |
|                                   | attributes. For example, we can   |
|                                   | match on route name, port number  |
|                                   | and port name, gateway name, and  |
|                                   | individual virtual hosts.         |
+-----------------------------------+-----------------------------------+
| **cluster**                       | Match on cluster attributes. For  |
|                                   | example, we can match the cluster |
|                                   | name, subset, fully qualified     |
|                                   | service name, and port number.    |
+-----------------------------------+-----------------------------------+

Let's look at an example configuration snippet:

**configPatches:**\
**  - applyTo: HTTP_FILTER**\
**    match:**\
**      context: SIDECAR_INBOUND**\
**      listener:**\
**        portNumber: 9000**\
**        filterChain:**\
**          filter:**\
**            name: \"envoy.filters.network.http_connection_manager\"**\
**            subFilter:**\
**              name: \"envoy.filters.http.router\"**

The above snippet states that we want to apply a patch for inbound
sidecars for a specific listener with port **9000**, and we're targeting
the router filter within the HCM.

# How to Apply the Patch

Once we have the patch\'s location, we also need to know what and how we
should apply it.

The **patch** field consists of an **operation** and the **value** of
the patch. The operation specifies how to apply the patch. For example,
we can merge the patch with the existing configuration, add it to the
existing list, remove it, insert it before or after specific objects,
and so on. We can find the full list of operations and different
conditions they apply to in the [reference
documentation](https://istio.io/latest/docs/reference/config/networking/envoy-filter/#EnvoyFilter-Patch-Operation).

The **value** field is where we specify the actual patch value. It's the
YAML configuration of the patch. 

Let's consider the following snippet:

**- applyTo: EXTENSION_CONFIG**\
**    patch:**\
**      operation: ADD **\
**      value:**\
**        name: my-wasm-extension**\
**        typed_config:**\
**          \"@type\":
type.googleapis.com/envoy.extensions.filters.http.wasm.v3.Wasm**\
**          config:**\
**            root_id: my-wasm-root-id**\
**            vm_config:**\
**              vm_id: my-wasm-vm-id**\
**              runtime: envoy.wasm.runtime.v8**\
**              code:**\
**                remote:**\
**                  http_uri:**\
**                    uri: http://my-wasm-binary-uri**

The above snippet uses the **ADD** operation to apply the patch (Wasm
extension configuration) to the **EXTENSION_CONFIG** section of the
Envoy configuration. Comparing this snippet to the previous one, you
will notice that we do not have any specific match conditions here.
We're applying the patch directly to the extension configuration
section.

# Extending the Envoy Proxy

One way we showed how to configure or modify Envoy is by augmenting the
configuration. Another option is implementing a custom filter that
processes or augments the requests. 

We mentioned HTTP filters, such as the external authz filter and other
filters built into the Envoy binary.

We can also write our own filters that Envoy dynamically loads and runs.
We can decide where we want to run the filter in the filter chain by
declaring it in the correct order.

We have a couple of options for extending Envoy. By default, we write
Envoy filters in C++. However, we can also write them in Lua script or
use WebAssembly (WASM), which allows us to develop Envoy filters in
other programming languages.

Note that the Lua and Wasm filters are limited in their APIs compared to
the C++ filters.

1.  **Native C++ API**\
    The first option is to write native C++ filters and package them
    with Envoy. This would require us to recompile Envoy and maintain
    our own version. This route makes sense if we\'re trying to solve
    complex or high-performance use cases.

2.  **Lua filter**\
    The second option is using the Lua script. An HTTP filter in Envoy
    allows us to define a Lua script either inline or as an external
    file and execute it during both the request and response flows.

3.  **Wasm filter**\
    The last option is using a Wasm filter to run Wasm plugins.  We
    write the custom Wasm plugin, and Envoy can load it dynamically at
    run time using the Wasm filter.

# What is WebAssembly (WASM)?

[[WebAssemby]{.underline}](https://webassembly.org/) (Wasm) is a
portable binary format for executable code that relies on an open
standard. It allows developers to write in their preferred programming
language and compile the code into a Wasm module or a plugin.

The Wasm plugins are isolated from the host environment and executed in
a memory-safe sandbox called a **virtual machine (VM)**. Wasm modules
use an API to communicate with the host environment.

The original goal of Wasm was to enable high-performance applications on
web pages. For example, let\'s say we\'re building a web application
with Javascript. We could write some in Go (or other languages) and
compile it into a binary file, the Wasm plugin. Then, we could run the
compiled Wasm plugin in the same sandbox as the Javascript web
application.

To compile a custom filter into a Wasm plugin that's compatible with
Envoy proxy, we can use one of the available SDKs:

-   -   -   [[Go
            SDK]{.underline}](https://github.com/tetratelabs/proxy-wasm-go-sdk)

        -   [[AssemblyScript
            SDK]{.underline}](https://github.com/solo-io/proxy-runtime)

        -   [[C++
            SDK]{.underline}](https://github.com/proxy-wasm/proxy-wasm-cpp-sdk)

        -   [[Rust
            SDK]{.underline}](https://github.com/proxy-wasm/proxy-wasm-rust-sdk)

        -   [[Zig
            SDK]{.underline}](https://github.com/mathetake/proxy-wasm-zig-sdk)

Since all SDKs implement the
same [[specification]{.underline}](https://github.com/proxy-wasm/spec),
the choice of an SDK comes down to the preferred programming language.

Once we've compiled the code into a **.wasm** file, we need a way to
inject the module into the Envoy proxy configuration. Historically, the
way to do this was to use the EnvoyFilter resource we discussed earlier.
However, with Istio 1.12, a [WasmPlugin]{.underline} resource was
introduced that allows us to configure Wasm plugins for the workloads in
the mesh.

# How to Deploy Wasm Plugins to the Mesh?

Deploying Wasm plugins requires two steps (assuming we already have a
compiled **.wasm** file).

The first step is to make the .wasm file available and accessible to the
Istio mesh. A component called an image fetcher is implemented in the
Istio agent, and it knows how to download .wasm files from an
OCI-compliant registry. The agent downloads the .wasm plugin and places
it in the local volume of the Envoy proxy. Pushing the .wasm file to the
OCI registry involves creating a simple Dockerfile, building the image,
and pushing it to a registry.

The second step is configuring the Wasm plugin and telling Istio which
workloads to apply the Wasm plugin, where to pull the **.wasm** file
from and to provide any optional configuration for the Wasm plugin.

Here's an example WasmPlugin resource:

**apiVersion: extensions.istio.io/v1alpha1**\
**kind: WasmPlugin**\
**metadata:**\
**  name: hello-world-wasm**\
**  namespace: default**\
**spec:**\
**  selector:**\
**    labels:**\
**      app: hello-world**\
**  url: oci://my-registry/hello-world:v1**\
**  pluginConfig:**\
**    greeting: hello**\
**    debug: true**

The selector field specifies which proxies we want to apply the Wasm
plugin to. Next, we specify the location of the compiled Wasm plugin.
Using the OCI-compliant registry, we prefix the location
with **oci://**. Alternatively, we can use an HTTP location of the Wasm
file or a path to a local file. Finally, under the **pluginConfig** we
can specify any plugin-specific configuration values.

Once we deploy the resource, Istio will download the .wasm file and then
generate and modify the Envoy configuration for the proxies we specify
with the selector labels. In comparison, the WasmPlugin resource offers
a much easier and cleaner way to configure Wasm plugins than an
EnvoyFilter resource.

Note that we can still use an EnvoyFilter if we want to. The difficulty
here is that we need to provide a verbose Envoy configuration. Moreover,
we can't use the OCI registry as a destination because the Envoy proxy
doesn't know how to download the images. Instead, we need to provide a
remote (HTTP) location or a local address from which the proxy can load
the Wasm file.

In any case, using a WasmPlugin is a much better choice as the resource
abstracts away the complexity of Envoy configuration.

# Deploying Wasm Plugins to Istio

In this lab, we will learn how to create a basic Wasm plugin and deploy
it to workloads running in the Kubernetes cluster.

We\'ll use TinyGo and [Proxy Wasm Go
SDK](https://github.com/tetratelabs/proxy-wasm-go-sdk) to build the Wasm
plugin.

TinyGo powers the SDK we will use as Wasm doesn\'t support the official
Go compiler.

Let\'s download and install the TinyGo:

**wget
https://github.com/tinygo-org/tinygo/releases/download/v0.25.0/tinygo_0.25.0_amd64.deb**\
**sudo dpkg -i tinygo_0.25.0_amd64.deb**

You can run **tinygo version** to check the installation is successful:

**tinygo version**\
**tinygo version 0.25.0 linux/amd64 (using go version go1.18.4 and LLVM
version 14.0.0)**

# Creating the Wasm Plugin

We will start by creating a new folder for our extension, initializing
the Go module, and downloading the SDK dependency:

**mkdir wasm-extension && cd wasm-extension**\
**go mod init wasm-extension**

Next, let\'s create the **main.go** file where the code for the Wasm
plugin will live. The plugin reads the additional response headers
(key/value pairs) we will provide through the configuration in the
WasmPlugin resource. Any values set in the configuration are added to
the response.

Here is what the code looks like:

**package main**

**import (**\
**    \"github.com/valyala/fastjson\"**

**    \"github.com/tetratelabs/proxy-wasm-go-sdk/proxywasm\"**\
**    \"github.com/tetratelabs/proxy-wasm-go-sdk/proxywasm/types\"**\
**)**

**func main() {**\
**    proxywasm.SetVMContext(&vmContext{})**\
**}**

**// Override types.DefaultPluginContext.**\
**func (ctx pluginContext) OnPluginStart(pluginConfigurationSize int)
types.OnPluginStartStatus {**\
**    data, err := proxywasm.GetPluginConfiguration()**\
**    if err != nil {**\
**        proxywasm.LogCriticalf(\"error reading plugin configuration:
%v\", err)**\
**    }**

**    var p fastjson.Parser**\
**    v, err := p.ParseBytes(data)**\
**    if err != nil {**\
**        proxywasm.LogCriticalf(\"error parsing configuration: %v\",
err)**\
**    }**

**    obj, err := v.Object()**\
**    if err != nil {**\
**        proxywasm.LogCriticalf(\"error getting object from json value:
%v\", err)**\
**    }**

**    obj.Visit(func(k \[\]byte, v \*fastjson.Value) {**

**        ctx.additionalHeaders\[string(k)\] =
string(v.GetStringBytes())**\
**    })**

**    return types.OnPluginStartStatusOK**\
**}**

**type vmContext struct {**\
**    // Embed the default VM context here,**\
**    // so that we do not need to reimplement all the methods.**\
**    types.DefaultVMContext**\
**}**

**// Override types.DefaultVMContext.**\
**func (\*vmContext) NewPluginContext(contextID uint32)
types.PluginContext {**\
**    return &pluginContext{contextID: contextID, additionalHeaders:
map\[string\]string{}}**\
**}**

**type pluginContext struct {**\
**    // Embed the default plugin context here,**\
**    // so that we do not need to reimplement all the methods.**\
**    types.DefaultPluginContext**\
**    additionalHeaders map\[string\]string**\
**    contextID         uint32**\
**}**

**// Override types.DefaultPluginContext.**\
**func (ctx \*pluginContext) NewHttpContext(contextID uint32)
types.HttpContext {**\
**    proxywasm.LogInfo(\"NewHttpContext\")**\
**    return &httpContext{contextID: contextID, additionalHeaders:
ctx.additionalHeaders}**\
**}**

**type httpContext struct {**\
**    // Embed the default http context here,**\
**    // so that we do not need to reimplement all the methods.**\
**    types.DefaultHttpContext**\
**    contextID         uint32**\
**    additionalHeaders map\[string\]string**\
**}**

**func (ctx \*httpContext) OnHttpResponseHeaders(numHeaders int,
endOfStream bool) types.Action {**\
**    proxywasm.LogInfo(\"OnHttpResponseHeaders\")**

**    for key, value := range ctx.additionalHeaders {**\
**        if err := proxywasm.AddHttpResponseHeader(key, value); err !=
nil {**\
**            proxywasm.LogCriticalf(\"failed to add header: %v\",
err)**\
**            return types.ActionPause**\
**        }**\
**        proxywasm.LogInfof(\"header set: %s=%s\", key, value)**\
**    }**

**    return types.ActionContinue**\
**}**

***NOTE:** You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above to **main.go**, then download the dependencies and use
TinyGo to build the plugin:

**go mod tidy**\
**tinygo build -o main.wasm -scheduler=none -target=wasi main.go**

# Building the Wasm Plugin Image

The next step is creating the Dockerfile, building the Wasm plugin
image, and pushing it to the registry. First, let\'s create
the **Dockerfile** with the following contents:

**FROM scratch**\
**COPY main.wasm ./plugin.wasm**

Since we\'ve already built the **main.wasm** file, we can now use Docker
to build and push the Wasm plugin to the registry:

**docker build -t \[REPOSITORY\]/wasm:v1 .**\
**docker push \[REPOSITORY\]/wasm:v1**

***NOTE:** The **\[REPOSITORY\] **is the name of the repository you\'ve
created in the Docker registry. You can use any OCI-compliant registry
such as [DockerHub]{.underline} or [[GCP's container
registry]{.underline}](https://cloud.google.com/container-registry).*

With the Wasm plugin in the registry, we can now craft the WasmPlugin
resource:

**apiVersion: extensions.istio.io/v1alpha1**\
**kind: WasmPlugin**\
**metadata:**\
**  name: wasm-example**\
**  namespace: default**\
**spec:**\
**  selector:**\
**    matchLabels:**\
**      app: httpbin**\
**  url: oci://\[REPOSITORY\]/wasm:v1**\
**  pluginConfig:**\
**    header_1: \"first_header\"**\
**    header_2: \"second_header\"**

Before saving the above YAML, replace the **\[REPOSITORY\]** with the
name of the repository you\'ve created in the registry.

In the **pluginConfig** field we're setting the configuration values we
read in the Wasm plugin and add to the response.

Once you've replaced the repository name, save the YAML
to **wasm-plugin.yaml** and then use the **kubectl** command to create
the WasmPlugin resource:

**kubectl apply -f wasm-plugin.yaml**

# Deploying the Sample Workload

We\'ll deploy a sample workload to try out the Wasm plugin. We\'ll
use **httpbin**. Make sure the default namespace is labeled for Istio
sidecar injection (**kubectl label ns default istio-injection=enabled**)
and then deploy the **httpbin** workload.

**apiVersion: v1**\
**kind: ServiceAccount**\
**metadata:**\
**  name: httpbin**\
**\-\--**\
**apiVersion: v1**\
**kind: Service**\
**metadata:**\
**  name: httpbin**\
**  labels:**\
**    app: httpbin**\
**    service: httpbin**\
**spec:**\
**  ports:**\
**  - name: http**\
**    port: 8000**\
**    targetPort: 80**\
**  selector:**\
**    app: httpbin**\
**\-\--**\
**apiVersion: apps/v1**\
**kind: Deployment**\
**metadata:**\
**  name: httpbin**\
**spec:**\
**  replicas: 1**\
**  selector:**\
**    matchLabels:**\
**      app: httpbin**\
**      version: v1**\
**  template:**\
**    metadata:**\
**      labels:**\
**        app: httpbin**\
**        version: v1**\
**    spec:**\
**      serviceAccountName: httpbin**\
**      containers:**\
**      - image: docker.io/kennethreitz/httpbin**\
**        imagePullPolicy: IfNotPresent**\
**        name: httpbin**\
**        ports:**\
**        - containerPort: 80**

Save the above YAML to **httpbin.yaml** and deploy it using **kubectl
apply -f httpbin.yaml**.

Before continuing, let\'s check that the **httpbin** Pod is up and
running:

**kubectl get po**\
**NAME                       READY   STATUS        RESTARTS   AGE**\
**httpbin-66cdbdb6c5-4pv44   2/2     Running       1          11m**

You can look at the logs from the **istio-proxy **container to see if
something went wrong with downloading the Wasm plugin.

# Wasm Plugin in Action

Let\'s try out the deployed Wasm plugin!

We will create a single pod inside the cluster, and from there, we will
send a request to **http://httpbin:8000/get**.

**kubectl run curl \--image=curlimages/curl -it \--rm \-- /bin/sh**\
**Defaulted container \"curl\" out of: curl, istio-proxy, istio-init
(init)**\
**If you do not see a command prompt, try pressing enter.**\
**/ \$**

Once you get the prompt to the curl container, send a request to
the **httpbin** service:

**curl -v http://httpbin:8000/headers**\
**\...**\
**\< HTTP/1.1 200 OK**\
**\< content-length: 13**\
**\< content-type: text/plain**\
**\< header_1: first_header**\
**\< header_2: second_header**\
**\< server: envoy**\
**\...**

In the output above, you can see that the Wasm plugin added the two
headers set in the configuration to the response.

# Cleanup

To clean everything up, run the following commands:

**kubectl delete -f wasm-plugin.yaml**\
**kubectl delete -f httpbin.yaml**

[[Chapter 7. Advanced
Topics]{.underline}](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS144x+3T2022/block-v1:LinuxFoundationX+LFS144x+3T2022+type@sequential+block@397c4d3bec874bce8818d403fb6021cf)

# Chapter Overview

This chapter will explain a couple of advanced Istio topics. We'll learn
about the Sidecar resource and how to use it to improve the mesh
performance, how to onboard virtual machines, and how to think about
multi-cluster Istio deployments.

# Learning Objecties

By the end of this chapter, you should be able to:

-   Understand the purpose of the Sidecar resource.

-   Understand the process of onboarding virtual machines to the mesh.

-   Understand the different facets of deploying Istio in multi-cluster
    scenarios.

# The Sidecar Resource

Istio, by default, watches all workloads in all namespaces and will
update every sidecar (with information regarding the whereabouts of
other workloads) in the mesh when any new workload is introduced or
deleted.

In a typical microservice application, each service communicates with a
small subset of the entire list of services deployed to the mesh. And
so, in most situations, it is wasteful to update each microservice with
information about every other microservice in the mesh.

The [Sidecar](https://istio.io/latest/docs/reference/config/networking/sidecar/) resource
is a configuration that can inform Istio exactly what services a given
service needs to know about (this is unrelated to security policy, and
the absence of a service does not prevent it from being called).

This is roughly analogous to organizations where every member is
informed on a \"need to know\" basis (though the motive in each
situation is entirely different: performance vs. secrecy).

Let us learn more about the Sidecar resource through the following lab.

# Sidecars for Improved Mesh Performance

Ensure that the default namespace is labeled for automatic sidecar
injection and, from the folder with the Istio distribution, deploy
the **bookinfo** application to the default namespace.

**kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml**

Study the sample application BookInfo from the Istio distribution.

 

![The BookInfo sample
application](https://github.com/nadidurna/Istio/blob/master/images/image33.png)

 

From the above illustration, we can derive the below table.


|Service|Services it needs to know about|
|:----|:----|
|productpage|reviews-v1, reviews-v2, reviews-v3, details|
|reviews-v1|--|
|reviews-v2|ratings|
|reviews-v3|ratings|
|details|--|
|ratings|--|


For six services, by default, Istio will inform each service of every
other. Not taking replicas into account, that\'s 6\*5 or thirty
service-to-service connections. In contrast, the above table informs us
that we need only six. That\'s a ratio of 5 to 1. In other words, we can
eliminate roughly 80% of the work of keeping the mesh up to date when we
know exactly who needs to communicate with who.

The **istioctl proxy-config** command allows us to spy on each sidecar
and see precisely the list of endpoints a sidecar is configured with.

For example, this command will list all endpoints that
the **productpage** deployment sidecar has in its list:

**istioctl proxy-config endpoints deploy/productpage-v1.default**

The output will display not only other **bookinfo** endpoints but also
endpoints in the **istio-system** namespace.

Here is a slightly pruned listing that lists only **bookinfo** services:

**ENDPOINT         STATUS    OUTLIER CHECK  CLUSTER**\
**10.32.0.7:9080   HEALTHY   OK           
 outbound\|9080\|\|details.default.svc.cluster.local**\
**10.32.0.8:9080   HEALTHY   OK           
 outbound\|9080\|\|reviews.default.svc.cluster.local**\
**10.32.0.9:9080   HEALTHY   OK           
 outbound\|9080\|\|reviews.default.svc.cluster.local**\
**10.32.1.7:9080   HEALTHY   OK           
 outbound\|9080\|\|ratings.default.svc.cluster.local**\
**10.32.1.8:9080   HEALTHY   OK           
 outbound\|9080\|\|reviews.default.svc.cluster.local**\
**10.32.1.9:9080   HEALTHY   OK           
 outbound\|9080\|\|productpage.default.svc.cluster.local**

The endpoints listing for the **ratings** service is similar, but the
issue is even more pronounced since **ratings** does not make any
outbound requests.

**istioctl proxy-config endpoints deploy/ratings-v1.default**

# Drafting the Sidecars

Let us now apply Sidecar resources for each of the **bookinfo** services
according to the \"need to know\" table in the previous section.

The basic recipe is to use the **workloadSelector** field to target each
service in turn and to list the desired target services under
the **egress.hosts** field.

**\-\--**\
**apiVersion: networking.istio.io/v1beta1**\
**kind: Sidecar**\
**metadata:**\
**  name: productpage-sidecar**\
**  namespace: default**\
**spec:**\
**  workloadSelector:**\
**    labels:**\
**      app: productpage**\
**  egress:**\
**  - hosts:**\
**    - \"./reviews.default.svc.cluster.local\"**\
**    - \"./details.default.svc.cluster.local\"**\
**    - \"istio-system/\*\"**\
**\-\--**\
**apiVersion: networking.istio.io/v1beta1**\
**kind: Sidecar**\
**metadata:  name: reviews-v1-sidecar**\
**  namespace: default**\
**spec:**\
**  workloadSelector:**\
**    labels:**\
**      app: reviews**\
**      version: v1**\
**  egress:**\
**  - hosts:**\
**    - \"istio-system/\*\"**\
**\-\--**\
**apiVersion: networking.istio.io/v1beta1**\
**kind: Sidecar**\
**metadata:**\
**  name: reviews-v2-sidecar**\
**  namespace: default**\
**spec:**\
**  workloadSelector:**\
**    labels:**\
**      app: reviews**\
**      version: v2**\
**  egress:**\
**  - hosts:**\
**    - \"./ratings.default.svc.cluster.local\"**\
**    - \"istio-system/\*\"**\
**\-\--**\
**apiVersion: networking.istio.io/v1beta1**\
**kind: Sidecar**\
**metadata:**\
**  name: reviews-v3-sidecar**\
**  namespace: default**\
**spec:**\
**  workloadSelector:**\
**    labels:**\
**      app: reviews**\
**      version: v3**\
**  egress:**\
**  - hosts:**\
**    - \"./ratings.default.svc.cluster.local\"**\
**    - \"istio-system/\*\"**\
**\-\--**\
**apiVersion: networking.istio.io/v1beta1**\
**kind: Sidecar**\
**metadata:**\
**  name: details-sidecar**\
**  namespace: default**\
**spec:**\
**  workloadSelector:**\
**    labels:**\
**      app: details**\
**  egress:**\
**  - hosts:**\
**    - \"istio-system/\*\"**\
**\-\--**\
**apiVersion: networking.istio.io/v1beta1**\
**kind: Sidecar**\
**metadata:**\
**  name: ratings-sidecar**\
**  namespace: default**\
**spec:**\
**  workloadSelector:**\
**    labels:**\
**      app: ratings**\
**  egress:**\
**  - hosts:**\
**    - \"istio-system/\*\"**

Above, we are pruning the list of other **bookinfo** services that each
service needs to know about, but preserving these workloads\' need to
know about services in the **istio-system** namespace.

The above implementation could be refined further by creating a Sidecar
definition without a **workloadSelector** field that applies
by **default** to all workloads in the default namespace and then
overriding that policy for specific workloads that deviate from the
default.

***Note:** You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above to a file named **sidecars.yaml** and apply it to your
Kubernetes cluster.

**kubectl apply -f sidecars.yaml**

Verify that the list of endpoints for each service has been narrowed.
The **productpage** service should only know
about **reviews** and **details**, and so on.

**istioctl proxy-config endpoints deploy/productpage-v1.default**

# Cleanup

To clean up, run the following commands:

**kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml**\
**kubectl delete sidecar
{details,productpage,ratings,reviews-v1,reviews-v2,reviews-v3}-sidecar**

# Summary

In the previous example, one can argue that creating all of these
Sidecar resources is perhaps not worth the effort since the savings in
computation and memory resources are negligible.

But imagine a situation where the number of services grows, to say 100.
The number of connections grows in the order of the square of the number
of services. So whereas with six services, we had a maximum of 30
connections, with 100 services growing to 9,900 (100 \* 99)!

In other words, each of the 100 services will be informed about the
other 99. Each time a single endpoint is added or removed, 100 sidecars
will be updated when in fact, it\'s more likely that only a dozen
sidecars care about this event and need to receive updates. That\'s
roughly an 8:1 ratio! That is, we can potentially reduce nearly 90% of
the work that Istio would otherwise perform.

In an application architecture, services are often organized
hierarchically. That is, services exposed to outside traffic via ingress
gateways are typically the ones calling other services and not the other
way around. It would be wasteful to send updates about services at the
top of that hierarchy to other services in the mesh that do not need
this information.

For an example of how the Sidecar resource becomes a necessity in large
clusters, see the talk by Cathal Conroy (from Workday) to the Istio
Meetup group on [Scaling Istio in Large
Clusters](https://youtu.be/wcJPC_bXYBA).

# Onboarding VMs

So far, we have assumed that all our workloads run inside Kubernetes
clusters as pods.

In production, you will most likely have a services architecture with a
mix of workloads: some will be containerized and running on a Kubernetes
cluster, but others will be running on Virtual Machines (VMs).

It\'s important for a service mesh to support this use case and to be
able to onboard workloads running on VMs.

Let us explore how a workload running on a VM can be made a part of the
Istio service mesh.

The VM workload will need to be accompanied by a sidecar. Istio makes
its **istio-proxy** sidecar available as a Debian (.deb) or CentOS
(.rpm) package and can simply be installed on a Linux VM and configured
as a **systemd** service.

Services on the cluster should be able to call a service backed by a
workload running on a VM, using the same fully-qualified hostname
resolution used for on-cluster workloads. Conversely, a workload running
on the VM should be able to call a service running on the Kubernetes
cluster.

In Kubernetes, the endpoints of a service consist of pod IP addresses. A
workload running on a VM is akin to a pod, so it\'s important to be able
to target these workloads using labels as selectors.

A pod running on Kubernetes is associated with a Kubernetes service
account and a namespace. That is the basis for its SPIFFE identity.
Similarly, the workload running on a VM will require an associated
namespace and service account. It is worth mentioning that [[Istio
version 1.14 introduced support for
SPIRE,]{.underline}](https://istio.io/latest/news/releases/1.14.x/announcing-1.14/#support-for-the-spire-runtime) which
opens the door to basing the SPIFFE identity on alternative pieces of
information.

Istio provides the [WorkloadEntry]{.underline} custom resource as a
mechanism for configuring the VM workload and providing all of these
details: the namespace, labels, and service account.

Istio also provides the [WorkloadGroup]{.underline} custom resource as a
template for WorkloadEntry resources, which can be automatically created
when Istio registers a workload with the mesh.

The WorkloadGroup also has a provision for specifying a readiness probe
for VM workloads to support health-checking.

 

![A service mesh with VM
workloads](https://github.com/nadidurna/Istio/blob/master/images/image34.png)


**A service mesh with VM workloads**

 

On the left-hand side in the above illustration, we see a Kubernetes
cluster with Istio installed and a service (Service A) running inside
the cluster. On the right are two additional workloads (Services B and
C), each running on a VM.

Note how these VMs have an Envoy proxy running as a sidecar. The
east-west gateway depicted in the center allows communication between
the sidecars running on the VMs and the Istio control plane.

The green arrows show how, in this particular configuration, the
services communicate directly with one another (because they all exist
within a single network). In a scenario where the VMs reside in a
separate network, that traffic would route through the east-west
gateway.

The general procedure for onboarding a VM can be summarized by the
following steps:

1.  Create and configure the east-west gateway as described above

2.  Construct the WorkloadGroup resource

3.  Install the sidecar on the VM

4.  Generate the configuration files for the sidecar and copy them to
    the VM.

5.  Place all configuration files in their proper locations on the VM.

6.  Start the sidecar service on the VM

This lab walks us through these steps in detail.

# Connect a VM Workload to the Istio Mesh

This lab demonstrates how a workload running on a VM can join the Istio
service mesh.

## Prerequisites

This exercise was developed on GCP, and some of the steps shown are
specific to that infrastructure. However, feel free to use different
cloud infrastructure and adapt this exercise to your environment.

## Overview

We begin with a Kubernetes cluster with Istio installed, and running
the BookInfo sample application. Next, we will turn off
the **ratings** deployment running inside the Kubernetes cluster, and in
its place, we will provision a VM to run the **ratings** application.

# Create a Kubernetes Cluster

Armed with your GCP (or other) cloud account, create a Kubernetes
cluster:

**gcloud container clusters create my-istio-cluster \\**\
**  \--cluster-version latest \\**\
**  \--machine-type \"n1-standard-2\" \\**\
**  \--num-nodes \"3\" \\**\
**  \--network \"default\"**

***Note**: make sure you specify a zone or region close to you in the
command above. *

Wait until the cluster is ready.

# Install Istio

Run the following command to install Istio:

**istioctl install \\**\
**  \--set
values.pilot.env.PILOT_ENABLE_WORKLOAD_ENTRY_AUTOREGISTRATION=true \\**\
**  \--set
values.pilot.env.PILOT_ENABLE_WORKLOAD_ENTRY_HEALTHCHECKS=true**

The essential difference between previous installations of Istio and the
above command are the two new pilot configuration options that enable
workload entry auto-registration and health checks.

Auto-registration means that when a workload is created on a VM, Istio
will automatically create a WorkloadEntry custom resource.

# Deploy BookInfo

Finally, deploy the BookInfo sample application, as follows:

1.  1.  1.  Turn on sidecar-injection:\
            **kubectl label ns default istio-injection=enabled**

        2.  Deploy the BookInfo sample application:\
            **cd \~/istio-1.14.3\
            kubectl apply -f
            samples/bookinfo/platform/kube/bookinfo.yaml**

 

![Topology of the BookInfo
application](https://github.com/nadidurna/Istio/blob/master/images/image33.png)

**Topology of the BookInfo application**

 

**Some Tweaks**

In the following steps, we will focus specifically on
BookInfo\'s **ratings** service.  We will be inspecting the \"star\"
ratings shown on BookInfo\'s product page as a means of determining
whether the service is up and reachable.

Notice in the above illustration that the product page load balances
requests to the **reviews** service between three different versions of
the application and that **reviews-v1** never calls ratings.

To ensure that we can see the ratings each time we request the product
page, scale down **reviews-v1** to zero replicas, effectively turning
off that endpoint:

**kubectl scale deploy reviews-v1 \--replicas=0**

# Expose BookInfo

Expose the BookInfo application via the ingress gateway:

**kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml**

Grab your load balancer public IP address:

**GATEWAY_IP=\$(kubectl get svc -n istio-system istio-ingressgateway
-ojsonpath=\'{.status.loadBalancer.ingress\[0\].ip}\')**

Open a browser and visit the BookInfo product page at
http://\$GATEWAY_IP/productpage.

**\***Make sure to replace the **\$GATEWAY_IP** in the URL with an
actual IP address you got from running the previous command.

 

![The BookInfo product
page](https://github.com/nadidurna/Istio/blob/master/images/image35.png)

**The BookInfo product page**

 

Verify that you can see ratings stars on the page.

With the BookInfo application fully deployed with ingress and
functioning, we can now turn our attention to the VM.

# Turn Off the Ratings Endpoints

Scale down the **ratings-v1** deployment running inside the Kubernetes
cluster to zero replicas:

**kubectl scale deploy ratings-v1 \--replicas=0**

Refresh the page in your browser and ensure the ratings stars are *now
gone* and have been replaced with the message **\"Ratings service is
currently unavailable.\"**

This indicates that the **ratings** service has no available endpoints
to handle requests.

In the subsequent steps, we replace this workload with an instance
running on a VM.

# Create the VM

Start by creating a VM in the same network that the Kubernetes cluster
is running:

**gcloud compute instances create my-mesh-vm \--tags=mesh-vm \\**\
**  \--machine-type=n1-standard-2 \\**\
**  \--network=default \--subnet=default \\**\
**  \--image-project=ubuntu-os-cloud \\**\
**  \--image-family=ubuntu-2204-lts**

The above command is specific to the GCP cloud environment; please adapt
the command to your specific environment.

Wait for the machine to be ready.

# Install the Ratings Service on the VM

The **ratings** service is a Node.js application.

1.  SSH onto the VM\
    **gcloud compute ssh ubuntu@my-mesh-vm**

2.  Install Node.js. The **ratings** service is a Node.js application.\
    **sudo apt-get update**\
    **sudo apt-get install nodejs npm jq**

3.  Grab a copy of the **ratings** app source code from the Istio GitHub
    repository.\
    **mkdir ratings && cd ratings**\
    **wget
    https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/src/ratings/package.json**\
    **wget
    https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/src/ratings/ratings.js**

4.  Install the application\'s dependencies:\
    **npm install**

5.  Run the app:\
    **npm run start 9080 &**\
    **\> start**\
    **\> node ratings.js \"9080\"**\
    \
    **Server listening on: http://0.0.0.0:9080\
    \
    ***\*Note that the ampersand (**&**) in the above command causes the
    process to run in the background. If desired, the **fg** command can
    be used to bring the process back to the foreground*.

6.  Test the app by retrieving a rating:\
    **curl http://localhost:9080/ratings/123 \| jq**\
    \
    The output should resemble this:\
    **{**\
    **  \"id\": 123,**\
    **  \"ratings\": {**\
    **    \"Reviewer1\": 5,**\
    **    \"Reviewer2\": 4**\
    **  }**\
    **}**

# Allow POD-to-VM Traffic on Port 9080

Open another terminal window and set up a variable that captures the
CIDR IP address range of the pods in the Kubernetes cluster.

**CLUSTER_POD_CIDR=\$(gcloud container clusters describe
my-istio-cluster \--format=json \| jq -r \'.clusterIpv4Cidr\')**

***Note:** use **\--zone** or **\--region** to specify the location for
the above command.*

The variable **CLUSTER_POD_CIDR** is used as an input to the following
command, which creates a firewall rule to allow communication from the
pods to the VM.

**gcloud compute firewall-rules create \"cluster-pods-to-vm\" \\**\
**  \--source-ranges=\$CLUSTER_POD_CIDR \\**\
**  \--target-tags=mesh-vm \\**\
**  \--action=allow \\**\
**  \--rules=tcp:9080**

Above, the **target-tags=mesh-vm** matches the tag given to the VM when
it was created.

# Install the East-West Gateway and Expose Istiod

An east-west gateway is necessary to enable communication between the
sidecar that will be running on the VM and istiod, the Istio control
plane (see the [[Istio
documentation]{.underline}](https://istio.io/latest/docs/ops/deployment/vm-architecture/)).

1.  Install the east-west gateway:\
    **./samples/multicluster/gen-eastwest-gateway.sh \--single-cluster
    \| istioctl install -y -f -**\
    \
    If you list the pods in the **istio-system** namespace you'll notice
    the **istio-eastwestgateway** instance was created.

2.  Expose istiod though the east-west gateway:\
    **kubectl apply -n istio-system -f
    ./samples/multicluster/expose-istiod.yaml**

# Create the WorkloadGroup

A [WorkloadGroup]{.underline} is a template for WorkloadEntry objects.

**apiVersion: networking.istio.io/v1alpha3**\
**kind: WorkloadGroup**\
**metadata:**\
**  name: ratings**\
**  namespace: default**\
**spec:**\
**  metadata:**\
**    labels:**\
**      app: ratings**\
**  template:**\
**    serviceAccount: bookinfo-ratings**

***Note:** You can download the supporting YAML and other files
from [[this Github
repo]{.underline}](https://tetr8.io/cncf-source-files).*

Save the above to a file named **ratings-workloadgroup.yaml** and apply
it:

**kubectl apply -f ratings-workloadgroup.yaml**

This WorkloadGroup will ensure that our VM workload is labeled
with **app: ratings** and associated with the service
account **bookinfo-ratings** (this service account was one of the
resources deployed together with the BookInfo application).

We now focus on installing and configuring the sidecar on the VM.

# Generate VM Artifacts

The Istio CLI provides a command to automatically generate all artifacts
needed to configure the VM:

1.  1.  1.  Create a subdirectory to collect the artifacts to be
            generated:\
            **mkdir vm_files**

        2.  Run the command to generate the artifacts:\
            **istioctl x workload entry configure \\**\
            **   \--file ratings-workloadgroup.yaml \\\
               \--output vm_files \\\
               \--autoregister**

        3.  Inspect the contents of the folder **vm_files**.  There, you
            will find five files, including a root certificate, an
            addition to the VM\'s **hosts** file that resolves the
            istiod endpoint to the IP address of the east-west gateway,
            a token used by the VM to securely join the mesh, an
            environment file containing metadata about the workload
            running on the VM (**ratings**), and a mesh configuration
            file necessary to configure the proxy.

In the next few steps, we copy these files to the VM and install them to
their proper locations.

# VM Configuration Recipe

1.  Copy the generated artifacts to the VM:\
    **gcloud compute scp vm_files/\* ubuntu@my-mesh-vm:**

2.  SSH onto the VM\
    **gcloud compute ssh ubuntu@my-mesh-vm**

3.  On the VM, run the following commands (see reference):\
    **\# place the root certificate in its proper place:**\
    **sudo mkdir -p /etc/certs**\
    **sudo cp \~/root-cert.pem /etc/certs/root-cert.pem**\
    \
    **\# place the token to the correct location on the file system:**\
    **sudo  mkdir -p /var/run/secrets/tokens**\
    **sudo cp \~/istio-token /var/run/secrets/tokens/istio-token**\
    \
    **\# fetch and install the istio sidecar package:**\
    **curl -LO
    https://storage.googleapis.com/istio-release/releases/1.14.3/deb/istio-sidecar.deb**\
    **sudo dpkg -i istio-sidecar.deb**\
    \
    **\# copy over the environment file and mesh configuration file:**\
    **sudo cp \~/cluster.env /var/lib/istio/envoy/cluster.env**\
    **sudo cp \~/mesh.yaml /etc/istio/config/mesh**\
    \
    **\# add the entry for istiod to the /etc/hosts file:**\
    **sudo sh -c \'cat \$(eval echo \~\$SUDO_USER)/hosts \>\>
    /etc/hosts\'**\
    **sudo mkdir -p /etc/istio/proxy**\
    \
    **\# make the user \"istio-proxy\" the owner of all these files:**\
    **sudo chown -R istio-proxy /etc/certs /var/run/secrets
    /var/lib/istio /etc/istio/config /etc/istio/proxy**

The VM is now configured.

# Exercise 1

Watch the WorkloadEntry get created as a consequence of the VM
registering with the mesh.

1.  Run the following **kubectl** command against your Kubernetes
    cluster:\
    **kubectl get workloadentry \--watch**

2.  On the VM, start the sidecar (**istio-proxy**) service:\
    **sudo systemctl start istio\
    \
    **The workload entry will appear in the listing (this can take up to
    a minute), and it should look similar to this:\
    **NAME                  AGE   ADDRESS**\
    **ratings-10.138.0.53   0s    10.138.0.53**

If we were to stop the sidecar service, we would see the WorkloadEntry
resource removed.

With the VM on-boarded to the mesh, we can proceed to verify
communications between the VM and other services.

# Exercise 2

Although the **ratings** service does not need to call back into the
mesh, we can manually test communication from the VM to the mesh.

For example, we can call the details service from the VM with:

**curl details.default.svc:9080/details/123 \| jq**

The above command should produce the following output:

**{**\
**    \"id\": 123,**\
**    \"author\": \"William Shakespeare\",**\
**    \"year\": 1595,**\
**    \"type\": \"paperback\",**\
**    \"pages\": 200,**\
**    \"publisher\": \"PublisherA\",**\
**    \"language\": \"English\",**\
**    \"ISBN-10\": \"1234567890\",**\
**    \"ISBN-13\": \"123-1234567890\"**\
**}**

This capability is supported via a [DNS
proxy](https://istio.io/latest/docs/ops/deployment/vm-architecture/#dns) bundled
with the sidecar.

Next, let us test the communication in the reverse direction: to
the **ratings** application running on the VM.

# Exercise 3

Head back to the web browser, and refresh
the **http://\$GATEWAY_IP/productpage** URL.

You should see that the **ratings** service is once more available, and
the ratings stars are displaying.

The workload entry has become an effective endpoint for
the **ratings** service!

We can verify this with the **istioctl proxy-config** command (which
lists the endpoints of the reviews service), as follows:

**istioctl proxy-config endpoints deploy/reviews-v2.default \| grep
ratings**

The output should resemble this:

**10.128.0.45:9080 HEALTHY OK
outbound\|9080\|\|ratings.default.svc.cluster.local**

Compare the above IP address of the endpoint with the address of the
WorkloadEntry for the VM workload:

**kubectl get workloadentry**

Here is the output:

**NAME                  AGE     ADDRESS**\
**ratings-10.128.0.45   5m55s   10.128.0.45**

The two addresses match.

# Multi-cluster Deployments

A multi-cluster deployment (two or more clusters) gives us greater
isolation and availability, but the cost we pay is increased complexity.

We will deploy clusters across multiple zones and regions if the
scenarios require high availability (HA).

The next decision we need to make is to decide if we want to run the
clusters within one network or if we want to use multiple networks.

The following figure shows a multi-cluster scenario (Cluster A, B, and
C) deployed across two networks.

 

![Multi-cluster scenario with two
networks](https://github.com/nadidurna/Istio/blob/master/images/image36.png)


**Multi-cluster scenario with two networks**

# Network Deployment Models

When multiple networks are involved, the workloads running inside the
clusters must use Istio gateways to reach workloads in other clusters.
Using various networks allows for better fault tolerance and scaling of
network addresses.

 

![East-west gateways used for communication between
clusters](https://github.com/nadidurna/Istio/blob/master/images/image37.png){width="6.5in"
height="3.3520833333333333in"}

**East-west gateways used for communication between clusters**

 

The gateways services use to communicate across or within the cluster
are called **east-west gateways**

# Control Plane Deployment Models

Istio service mesh uses the control plane to configure all
communications between workloads inside the mesh. The control plane the
workloads connect to depends on their configuration.

In the simplest case, we have a service mesh with a single control plane
in a single cluster. This is the configuration we\'ve been using
throughout this course.

The **shared control plane model** involves multiple clusters where the
control plane only runs in one cluster. That cluster is referred to as a
primary cluster, while other clusters in the deployment are called
remote clusters. These clusters don\'t have their control plane.
Instead, they are sharing the control plane from the primary cluster.

 

![Shared control plane between primary and remote
cluster](https://github.com/nadidurna/Istio/blob/master/images/image38.png)

**Shared control plane between primary and remote cluster**

 

Another deployment model is where we treat all clusters as remote
clusters controlled by an external control plane. The external control
plane gives us a complete separation between the control plane and the
data plane. A typical example of an external control plane is when a
cloud vendor manages it.

For high availability, we should deploy multiple control plane instances
across multiple clusters, zones, or regions, as shown in the figure
below.

 

![Multiple control planes deployed across multiple
clusters](https://github.com/nadidurna/Istio/blob/master/images/image39.png)

**Multiple control planes deployed across multiple clusters**

 

This model offers improved availability and configuration isolation. If
one of the control planes becomes unavailable, the outage is limited to
that one control plane. To improve that, you can implement failover and
configure workload instances to connect to another control plane in case
of failure.

For the highest availability possible, we can deploy a control plane
inside each cluster.

# Mesh Deployment Models

All scenarios we have discussed so far use a single mesh. In a single
mesh model, all services are in one mesh, regardless of how many
clusters and networks they are spanning.

A deployment model where multiple meshes are federated together is
called a **multi-mesh deployment**. In this model, services can
communicate across mesh boundaries. The model gives us a cleaner
organizational boundary and stronger isolation and reuses service names
and namespaces.

When federating two meshes, each mesh can expose a set of services and
identities that all participating meshes can recognize. To enable
cross-mesh service communication, we have to enable trust between the
two meshes. Trust can be established by importing a trust bundle to a
mesh and configuring local policies for those identities.

# Tenancy Models

A tenant is a group of users sharing common access and privileges to a
set of workloads. Isolation between the tenants gets done through
network configuration and policies. Istio supports namespace and cluster
tenancies. Note that the tenancy we discuss here is **soft
multi-tenancy**, not hard. There is no guaranteed protection against
noisy neighbor problems when multiple tenants share the same Istio
control plane.

Within a mesh, Istio uses **namespaces** as a unit of tenancy. If using
Kubernetes, we can grant permissions for workloads deployments per
namespace. By default, services from different namespaces can
communicate with each other through fully qualified names.

In the security section, we learned how to improve isolation using
authorization policies and restrict access to only the appropriate
callers.

In the multi-cluster deployment models, the namespaces in each cluster
sharing the same name are considered the same.
Service **customers** from namespace **default** in cluster A refers to
the same service as service **customers** from namespace **default** in
cluster B. Load balancing is done across merged endpoints of both
services when traffic is sent to service **customers**, as shown in the
following figure.

 

![Service load balancing across merged
endpoints](https://github.com/nadidurna/Istio/blob/master/images/image40.png)

**Service load balancing across merged endpoints**

 

To configure **cluster tenancy** in Istio, we must configure each
cluster as an independent service mesh. The meshes can be controlled and
operated by separate teams, and we can connect the meshes into a
multi-mesh deployment. Suppose we use the same example as before. In
that case, service **customers** running in the **default** namespace in
cluster A does not refer to the same service as
service **customers** from the **default** namespace in cluster B.

Another critical part of the tenancy is isolating configuration from
different tenants. At the moment, Istio does not address this issue.
However, it encourages it through the namespace scoped configuration.

A typical multi-cluster deployment topology is one where each cluster
has its control plane. For regular service mesh deployments at scale,
you should use multi-mesh deployments and have a separate system
orchestrating the meshes externally.

It is always recommended to use ingress gateways across clusters, even
if they span a single network. Direct pod-to-pod connectivity requires
populating endpoint data across multiple clusters, which can slow down
and complicate things. A more straightforward solution is to have
traffic flow through ingresses across clusters instead.

# Locality Failover

When dealing with multiple clusters, we need to understand the
definition of the term locality, which determines a geographical
location of a workload inside the mesh.

The locality comprises region, zone, and subzone, as shown in the figure
below.

 

![Hierarchical view of regions, zones, and
sub-zones](https://github.com/nadidurna/Istio/blob/master/images/image41.png)


**Hierarchical view of regions, zones, and sub-zones**

 

The region and zone are typically set automatically if your Kubernetes
cluster runs on cloud providers\' infrastructure. For example, nodes of
a cluster running GCP in the **us-west1** region and
zone **us-west1-a** will have the following labels set:

**topology.kubernetes.io/region=us-west1**\
**topology.kubernetes.io/zone=us-west1-a**

The sub-zones allow us to divide individual zones further. However, the
concept of sub-zones doesn't exist in Kubernetes. To use sub-zones, we
can use the label **topology.istio.io/subzone**.

Once set up, Istio can use the locality information to control load
balancing, and we can configure locality failover or locality weighted
distribution.

The failover settings can be configured in the DestinationRule under
the **localityLbSettings** field. For example:

**apiVersion: networking.istio.io/v1beta1**\
**kind: DestinationRule**\
**metadata:**\
**  name: helloworld**\
**spec:**\
**  host: helloworld.sample.svc.cluster.local**\
**  trafficPolicy:**\
**    loadBalancer:**\
**      simple: ROUND_ROBIN**\
**      localityLbSetting:**\
**        enabled: true**\
**        failover:**\
**          - from: us-west**\
**            to: us-east**\
**    outlierDetection:**\
**      consecutive5xxErrors: 100**\
**      interval: 1s**\
**      baseEjectionTime: 1m**

The above example specifies that when the endpoints within
the **us-west** region are unhealthy, the traffic should failover to any
zones and sub-zones in the **us-east** region. Note that the outlier
detection settings are required for the failover to function properly.
The outlier tells Envoy how to determine whether the endpoints are
unhealthy.

Similarly, we can use the locality information to control the
distribution of traffic using weights. Consider the following example:

**apiVersion: networking.istio.io/v1beta1**\
**kind: DestinationRule**\
**metadata:**\
**  name: helloworld**\
**spec:**\
**  host: helloworld.sample.svc.cluster.local**\
**  trafficPolicy:**\
**    loadBalancer:**\
**      simple: ROUND_ROBIN**\
**      localityLbSetting:**\
**        enabled: true**\
**        distribute:**\
**          - from: \"us-west1/us-west1-a/\*\"**\
**            to:**\
**      \"us-west1/us-west1-a/\*\": 50**\
**              \"us-west1/us-west1-b/\*\": 30**\
**              \"us-east1/us-east1-a/\*\": 20\
    outlierDetection:**\
**      consecutive5xxErrors: 100**\
**      interval: 1s**\
**      baseEjectionTime: 1m**

In the above example, we're distributing traffic that originates
in **us-west1/us-west1-a** in the following manner:

-   50% of the traffic is sent to workloads in **us-west1/us-west1-a**

-   30% of the traffic is sent to workloads in **us-west1/us-west1-b**

-   20% of the traffic is sent to workloads in **us-east1/us-east1-a **
