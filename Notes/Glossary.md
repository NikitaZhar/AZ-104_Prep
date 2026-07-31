2026-07-31 10:00
Tags: #AZ-104 #glossary

# AZ-104 Glossary

A collection of key terms. Each entry has a **definition** (first paragraph, drawn from the module text) and a **description** (second paragraph — the surrounding context, rules, and examples). Standalone reference; the module notes are left as clean reading text.

---

## Module 01 — Describe cloud computing

### Microsoft Azure
A cloud computing platform with an ever-expanding set of services to help you build solutions that meet your technical goals.

Supports everything from simple web services for internet-facing apps to fully virtualized computers and cloud-based services like remote storage, database hosting, and centralized account management. Also offers capabilities in artificial intelligence (AI) and Internet of Things (IoT).

### Cloud computing
The delivery of computing services over the internet.

Computing services include common IT infrastructure such as virtual machines, storage, databases, and networking, and expand to include IoT, machine learning (ML), and AI. Because it uses the internet, cloud computing isn't constrained by physical infrastructure the way a traditional datacenter is, so you can expand your IT footprint rapidly instead of building a new datacenter.

### Azure Fundamentals
A series of three learning paths (plus one applied path) that familiarize you with Azure and its many services and features.

Includes guided content and knowledge checks, requires no technical IT experience, and helps prepare for Exam AZ-900.

### AZ-900
Exam AZ-900: Microsoft Azure Fundamentals.

Covers three knowledge domains: describe cloud concepts (25–30%), describe Azure architecture and services (35–40%), and describe Azure management and governance (30–35%). Each domain maps to a learning path in Azure Fundamentals.

### Shared responsibility model
A model where responsibilities are shared between the cloud provider and the consumer, rather than one party owning everything as in an on-premises datacenter.

The cloud provider is always responsible for the physical datacenter, physical network, and physical hosts. The consumer is always responsible for the data and information stored in the cloud, the devices allowed to connect, and the accounts and identities in their environment. Responsibility for operating systems, network controls, applications, and identity and access depends on the service type: IaaS places the most responsibility on the consumer, SaaS the most on the provider, and PaaS falls in the middle.

### Cloud provider
The party responsible for the physical parts of the cloud — physical security, power, cooling, and network connectivity.

Under the shared responsibility model the provider always owns the physical datacenter, network, and hosts; with SaaS it takes on most of the remaining responsibility as well.

### Consumer
The customer using the cloud, who is always responsible for their data and information, connected devices, and accounts and identities, as well as access security.

How much additional responsibility the consumer holds depends on the service type — most under IaaS, least under SaaS.

### On-premises datacenter
A traditional datacenter where your team is responsible for everything — maintaining the physical space, ensuring security, and maintaining or replacing servers, plus all infrastructure, software, and patching.

Contrast with cloud computing, where these responsibilities shift to the cloud provider under the shared responsibility model.

### Cloud model (deployment model)
The models that define the deployment type of cloud resources. The three main models are private, public, and hybrid.

Multicloud is an increasingly common fourth scenario that uses multiple public cloud providers.

### Private cloud
A cloud environment used by a single entity.

Evolved naturally from the traditional datacenter model, delivering IT services over the internet while keeping resources dedicated to one organization. Provides much greater control but comes with greater cost and fewer public-cloud benefits. May be hosted on-site or in a dedicated offsite datacenter, potentially by a third party.

### Public cloud
A cloud built, controlled, and maintained by a third-party cloud provider, where anyone that wants to purchase cloud services can access and use resources.

General public availability is the key difference between public and private clouds.

### Hybrid cloud
A computing environment that uses both public and private clouds in an inter-connected environment.

Can let a private cloud surge into public cloud resources for increased, temporary demand, or provide an extra layer of security by letting users choose which services stay in the private cloud.

### Multicloud
A scenario in which you use multiple public cloud providers.

You might use different features from different providers, or be migrating from one provider to another; either way you manage resources and security across two or more environments.

### Azure Arc
A set of technologies that helps manage your cloud environment.

Can manage a public cloud solely on Azure, a private cloud in your datacenter, a hybrid configuration, or a multicloud environment running on multiple providers at once.

### Azure VMware Solution
A service that lets you run your VMware workloads in Azure with seamless integration and scalability.

Useful if you're already established with VMware in a private cloud but want to migrate to a public or hybrid cloud.

### Consumption-based model
A model where you pay for the IT resources you use and nothing more — renting compute power and storage and releasing those resources when you're done.

Benefits: no upfront costs for hardware or datacenter infrastructure, no need to buy capacity that may go underutilized, and the ability to add resources when demand increases and release them when it decreases. Because you pay as you consume, cloud spend is an operational expense.

### Capital expenditure (CapEx)
Up-front spending on physical infrastructure like servers, network hardware, and datacenter space.

A traditional IT budgeting term, contrasted with operational expenditure (OpEx).

### Operational expenditure (OpEx)
Ongoing spending on services over time.

Because you pay for cloud services as you consume them, cloud computing is classified as an operational expense rather than a capital one.

### Pay-as-you-go pricing
A pricing model where you typically pay only for the services you consume.

Helps you plan and manage operating costs, run infrastructure more efficiently, and scale as workload needs change, while the provider maintains the underlying infrastructure.

---

## Module 02 — Describe the benefits of using cloud services

### High availability
Ensuring maximum availability of resources, regardless of disruptions or events that may occur.

Azure is a highly available cloud environment with uptime guarantees that depend on the service; these guarantees are part of the service-level agreements (SLAs). You account for availability guarantees when architecting a solution.

### Scalability
The ability to adjust resources to meet demand — adding resources to handle increased demand and reducing them when demand drops.

Comes in vertical and horizontal forms. Because the cloud is consumption-based, scaling down also reduces cost, so you aren't overpaying for services.

### Vertical scaling
Increasing or decreasing the capabilities of a resource — for example, adding or removing CPU or RAM on a virtual machine (scaling up or down).

Used when a resource needs more (or less) processing power.

### Horizontal scaling
Adding or subtracting the number of resources — for example, adding or removing virtual machines or containers (scaling out or in), automatically or manually.

Used to respond to steep jumps or drops in demand.

### Service-level agreement (SLA)
The service availability/uptime guarantees that Azure provides, which depend on the service.

You account for these guarantees when architecting a solution for availability.

### Reliability
The ability of a system to recover from failures and continue to function.

One of the pillars of the Microsoft Azure Well-Architected Framework. The cloud's decentralized, global design naturally supports reliability: resources can be deployed in regions around the world, so if one region has a catastrophic event others keep running, and in some cases the environment shifts to a different region automatically.

### Predictability
Being able to move forward with confidence by predicting performance and cost in the cloud.

Split into performance predictability (predicting the resources needed to deliver a good experience, supported by autoscaling, load balancing, and high availability) and cost predictability (forecasting cloud spend using real-time tracking, monitoring, and tools like the Azure Pricing Calculator). Both are heavily influenced by the Well-Architected Framework.

### Microsoft Azure Well-Architected Framework
A framework whose pillars (including reliability) guide building solutions with predictable cost and performance.

Deploy a solution built around this framework and its cost and performance are predictable.

### Autoscaling
Automatically deploying additional resources to meet demand, then scaling back when the demand drops.

One of the cloud concepts that supports performance predictability.

### Load balancing
Redirecting some of the overload to less-stressed areas when traffic is heavily focused on one area.

One of the cloud concepts that supports performance predictability.

### Azure Pricing Calculator
A tool you can use to get an estimate of potential cloud spend.

Supports cost predictability by helping you forecast future costs.

### Governance
Using tools such as templates to ensure that deployed resources meet your technical standards and regulatory requirements, and updating resources at scale as standards change.

Cloud-based auditing helps flag resources that are out of compliance with your baseline and provides mitigation strategies. Depending on your operating model, software patches and updates may be applied automatically, helping with both governance and security. Establishing a good governance footprint early keeps your cloud footprint updated, secure, and well managed.

### Compliance
Conformance of resources to your baseline of technical standards and regulatory requirements.

Cloud-based auditing flags resources that are out of compliance and provides mitigation strategies.

### Distributed denial of service (DDoS)
A type of attack that overwhelms a service with traffic; cloud providers are typically well suited to handle it.

Handling DDoS attacks makes your network more robust and secure.

### Manageability in the cloud
How you're able to manage your cloud environment and resources — through a web portal, a command line interface (CLI), APIs, or PowerShell.

For example, an operations team can deploy from templates, monitor health in the portal, and automate recurring tasks with CLI or PowerShell scripts.

### Management of the cloud
How the cloud manages resources for you.

Includes automatically scaling resource deployment based on need, deploying resources from a preconfigured template, monitoring the health of resources and automatically replacing failing ones, and sending automatic alerts based on configured metrics.

### Sustainability (in the cloud)
Supporting sustainability goals by actively optimizing how resources are deployed and used.

Cloud providers operate at large scale, which can improve resource utilization compared to isolated on-premises environments. Practices include scaling resources down when demand decreases, turning off or deallocating idle resources, choosing efficient services to reduce overprovisioning, and using governance and monitoring to track usage trends and optimize over time.

---

## Module 03 — Describe cloud service types

### Infrastructure as a service (IaaS)
The most flexible category of cloud services, providing the maximum amount of control for your cloud resources.

The cloud provider maintains the hardware, network connectivity to the internet, and physical security; you're responsible for operating system installation, configuration, and maintenance, network configuration, and database and storage configuration. You essentially rent the hardware and decide what to do with it. Common scenarios: lift-and-shift migration and testing and development.

### Platform as a service (PaaS)
A middle ground between renting infrastructure (IaaS) and paying for a complete deployed solution (SaaS).

The cloud provider maintains the physical infrastructure, physical security, internet connection, operating systems, middleware, development tools, and analytics services; you don't worry about licensing or patching for operating systems and databases. You focus on application code, data, and access. Common scenarios: a development framework and analytics or business intelligence.

### Software as a service (SaaS)
The most complete cloud service model from a product perspective — you rent or use a fully developed application.

Examples include email, financial software, messaging applications, and connectivity software. It's the least flexible but easiest to get up and running, requiring the least technical expertise. The provider manages almost the entire application stack; you primarily manage your data, identity and access settings, and device access posture, giving the lowest operational overhead.

### Lift-and-shift migration
Setting up cloud resources similar to your on-premises datacenter and then moving your workloads to the IaaS infrastructure.

A common IaaS scenario.

### Multitenant capability
A capability, included with PaaS along with scalability and high availability, that reduces the amount of coding developers must do.

Part of the built-in cloud features a PaaS development framework provides.

---

## Module 04 — Describe the core architectural components of Azure

### Datacenter
Facilities with servers arranged in racks, with dedicated power, cooling, and networking infrastructure — similar to an on-premises datacenter, but at a much larger scale.

You don't interact with individual datacenters directly. Azure groups them into regions and availability zones that provide resiliency and reliability for your workloads.

### Region
A geographical area on the planet that contains at least one, but potentially multiple datacenters that are nearby and networked together with a low-latency network.

When you deploy a resource, you often choose its region. Some services, VM sizes, and storage types are only available in certain regions; global services such as Microsoft Entra ID, Azure Traffic Manager, and Azure DNS don't require selecting a region.

### Availability Zone
Physically separate datacenters within an Azure region, each made up of one or more datacenters with independent power, cooling, and networking, set up as an isolation boundary. If one zone goes down, the others continue working.

Zone-enabled regions have a minimum of three zones, connected by high-speed private fiber-optic networks, but not all Azure regions support zones. You place resources in one zone and replicate them to others within the region for resiliency; duplicating services and transferring data between zones can add cost.

### Zonal services
Services you pin to a specific zone — for example, VMs, managed disks, and IP addresses.

One of three availability-zone service categories. Because the resource sits in a chosen zone, surviving a zone failure requires deliberately distributing or replicating it across zones.

### Zone-redundant services
Services the platform replicates automatically across zones — for example, zone-redundant storage and SQL Database.

One of three availability-zone service categories. The platform handles cross-zone replication for you, so the service tolerates the loss of a single zone without manual design.

### Non-regional services
Services always available from Azure geographies and resilient to both zone-wide and region-wide outages.

One of three availability-zone service categories. These services aren't tied to a single region, so they survive even region-wide events.

### Region pair
Most Azure regions are paired with another region within the same geography (such as US, Europe, or Asia) at least 300 miles away, allowing replication of resources across a geography to reduce the likelihood of interruptions from events like natural disasters or power outages.

Advantages: during a broad outage one region per pair is prioritized for faster restoration, planned platform updates roll out to paired regions one at a time, and data stays within the same geography for residency and compliance (except Brazil South). Not all services auto-replicate or fail over — the customer may need to configure it. Most pairs are bidirectional; some (like Brazil South) are one-directional, and some regions (Italy North, Poland Central, Israel Central) have no traditional pair and rely on availability zones and geo-redundant storage instead.

### Sovereign region
Instances of Azure that are isolated from the main instance of Azure, used for compliance or legal purposes.

Examples: US government regions (US DoD Central, US Gov Virginia, US Gov Arizona, and more) are network-isolated instances operated by screened U.S. personnel with additional compliance certifications; China regions (China East, China North, and more) are delivered through a partnership between Microsoft and 21Vianet, where Microsoft doesn't directly maintain the datacenters.

### Resource
The basic building block of Azure — anything you create, provision, or deploy.

Examples include VMs, virtual networks, databases, and Azure AI services. Every resource must belong to exactly one resource group.

### Resource group
Groupings of resources. Every resource must belong to exactly one resource group, and a resource is associated with only one group at a time (though some resources can be moved between groups).

Resource groups can't be nested, and they can't be renamed after creation, so choose a clear naming convention from the start. Actions applied to a group affect all resources inside it — deleting a group deletes everything in it, and granting or denying access applies to all its resources. This suits temporary environments (delete the whole group at once) and per-team/per-project separation.

### Subscription
A unit of management, billing, and scale that lets you organize resource groups and control billing separately from access.

Using Azure requires a subscription; it provides access to Azure products and services and links to an Azure account. Each subscription defines two boundaries — a billing boundary and an access control boundary. Reasons to create additional subscriptions include separating environments, team and workload boundaries, and billing.

### Billing boundary
Determines how an Azure account is billed. You can create multiple subscriptions for different billing requirements, and Azure generates separate billing reports and invoices for each subscription.

One of the two subscription boundary types.

### Access control boundary
The subscription level at which Azure applies access-management policies. For example, you might create one subscription for development work and another for production, each with different spending limits and access rules.

One of the two subscription boundary types.

### Environments
Subscriptions for lifecycle stages such as sandbox, development, test, and production. Access control occurs at the subscription level, making this a natural boundary.

One of the reasons the material lists for creating additional subscriptions.

### Team and workload boundaries
Giving each project its own subscription so costs are easy to track, or separating sandbox environments from production.

One of the reasons the material lists for creating additional subscriptions.

### Management group
Containers that sit above subscriptions. You organize subscriptions into management groups and apply governance conditions — like access policies or compliance rules — to the group, and all subscriptions in the group automatically inherit those conditions.

Management groups can be nested up to six levels deep (excluding the root and subscription levels); a single directory supports up to 10,000 management groups, and each management group and subscription has only one parent. Uses: apply a policy across many subscriptions (owners can't override it), or grant access across many subscriptions with a single Azure RBAC assignment.

### Microsoft Entra tenant
An organization's dedicated instance of Microsoft Entra ID. Every Microsoft Entra tenant has a single top-level Tenant Root Group.

All of a tenant's management groups and subscriptions fold up to that root group.

### Tenant Root Group
The single top-level management group that every Microsoft Entra tenant has.

All other management groups and subscriptions fold up to this root group, which lets you apply governance policies globally across the tenant.

### Microsoft Entra ID
Azure's identity and directory service. An Azure account is an identity in Microsoft Entra ID or in a directory that Microsoft Entra ID trusts.

Listed among the global Azure services (with Azure Traffic Manager and Azure DNS) that don't require selecting a particular region.

### Azure RBAC
Azure role-based access control — a way to grant access via role assignments at a chosen scope.

Assigning Azure RBAC at a management group lets all sub-management groups, subscriptions, resource groups, and resources underneath inherit those permissions, avoiding the need to script role assignments across individual subscriptions.

### Azure account
An identity in Microsoft Entra ID (or in a directory that Microsoft Entra ID trusts) that an Azure subscription links to.

Created when you first sign up for Azure, at which point a subscription is created for you; you can then create additional subscriptions (for example, separate dev, test, and production). An account can have multiple subscriptions but requires only one.

### Azure free account
Includes free access to popular Azure products for 12 months, a credit to use for the first 30 days, and access to more than 65 services that are always free.

To sign up you need a phone number, a credit card (used for identity verification only), and a Microsoft or GitHub account. You aren't charged for services until you upgrade to a paid subscription.

### Azure free student account
Includes free access to certain Azure services for 12 months, a credit to use in the first 12 months, and free access to certain software developer tools.

Gives $100 credit and free developer tools, and you can sign up without a credit card.

### Cloud Solution Provider
Microsoft partners that offer a range of complete managed-cloud solutions for Azure.

One of the ways to purchase Azure access — directly from Microsoft (via the website or a Microsoft representative) or through a Microsoft partner.

---
