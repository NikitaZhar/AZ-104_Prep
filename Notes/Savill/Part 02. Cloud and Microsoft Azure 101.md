2026-07-01 10:03
Tags: #

## Introduction

The goal of this session is to provide an introduction to the different types of cloud service, dive into specifics about Microsoft Azure and its offerings, explain what "as a service" offerings mean, and then cover how to actually acquire and leverage Azure services.

## Cloud Services

There are many different types of cloud service. The definition of what they mean can vary based on where the services are hosted, who provides the service, and how it is accessed.

### Private Cloud

With a private cloud, a company has full access to all aspects of the environment — the hardware, the operating system, the applications. There is a certain amount of capacity that is then exposed as different types of resource: this might be virtual machines, containers, or different types of database service, but it is typically limited.

What really differentiates "running a hypervisor" from "running a private cloud" is the management infrastructure layered on top of it to provide services to people within the company. Picture an organization on premises with various servers hosting virtual machines and, on top of that, perhaps containers. A management layer sits above all of this, exposing capabilities to the organization's people — for example, through a portal — so they can provision services, consume them, and stop and start them, rather than having to send a service ticket to the operations team asking for something to be created. This is a private cloud: infrastructure hosted within the environment that the company's own people can consume.

### Public Cloud

With a public cloud, an external party provides the service, primarily accessed over the internet, though very often there is also the option to connect via more private connectivity. Microsoft Azure, Amazon Web Services, and Google Cloud are examples.

Here there is a shift in responsibility for how the service is consumed and leveraged. Only specific types of resource are offered by the provider — not everything — and the types of resource available vary by cloud provider, but are typically far more diverse than what could be offered on premises. There is typically a very full control plane and a rich set of interactions, along with built-in capabilities such as policy, governance, and security solutions. Azure is a public cloud solution.

There is still infrastructure running behind a public cloud — this isn't magic — but none of it is visible to the consumer. Instead, there is a control plane and different types of resource: basic things like virtual machines, but also AI services, database services, and many other capabilities. The control plane is accessed through APIs, command-line interfaces, portals, and infrastructure-as-code, allowing resources to be declared rather than manually built. Access may be internet-facing, or a private connection may be established between an organization's locations and the cloud.

### Community Cloud

Some organizations share a certain set of infrastructure. Governments, for example, commonly use community clouds for their various agencies — not just one entity, but a group of organizations sharing the same infrastructure.

### Hybrid Cloud

Hybrid cloud means using a public cloud while still maintaining things on premises. Most companies will be in this state for quite some time. This doesn't necessarily mean completely separate control planes: it's possible to extend a cloud's control plane to resources in other clouds and on premises. In Azure this is called **Azure Arc**. There is also **Azure Local**, which provides the ability to run certain Azure services on-premises directly.

## What Is a Cloud

There are many different definitions of "cloud" — ask five people and you might get seven different answers. However, the US National Institute of Standards and Technology (NIST) defines five specific characteristics that something must have to be classified as a cloud:

**On-demand self-service** — as a consumer of a service, you can provision computing capabilities and storage on your own, without requiring interaction with another human. This can be daunting for organizations enabling self-service for their users, which is where governance comes in: the ability to provision on demand, within the budget and constraints that governance has laid out.

**Broad network access** — capabilities are available over the network and accessed through standard mechanisms, typically the internet, though many enterprises establish private connectivity for additional security and control. The goal is that the service should be highly accessible to everyone.

**Resource pooling** — the provider has islands of resource that are pooled together and shared by multiple consumers in a multi-tenant model. As a consuming company, you may actually be using the same physical server as another company, but strong isolation technologies — hypervisors and other hardware- and software-based capabilities — ensure complete isolation between tenants' resources. The provider maintains this pool of resources, and consumers don't need to worry about which building or rack their resources are in; they simply consume the services they need.

**Rapid elasticity** — the ability to create new instances of a service and remove them as needed, automatically scaling so that provisioned capacity matches incoming workload demand. Rather than running ten instances of something at all times, capacity can flex — three instances, then five, then seven, then three — changing as needed, whether based on incoming work or a schedule. The cloud can essentially be thought of as an infinite amount of resource that can be provisioned whenever needed. In reality there is a finite amount of capacity: the more specific the requirements (a particular region, zone, or VM type), the more likely a limit will be hit. The more flexible you can be with regions and SKUs, the closer that "infinite capacity" becomes to reality.

**Measured service** — there should be metering, and even budgets, showing exactly what is being consumed. Tagging can be used to add metadata to resources, showing that a specific department or application consumed a given amount, rather than receiving one large bill. Because you pay for what you consume, you can see exactly what has been used.

As a user, interaction with the service provider and access to the underlying resources is very limited. In Azure, for example, there is no way to look directly at a physical host — nor would you want to. Instead, you are provided with a certain type and classification of resource: a disk of a certain performance level, a virtual machine with a certain amount of memory or compute optimization. You don't have to worry about the specifics of the node running it, the disk hosting it, or how many other VMs sit on the same network segment.

## Types of Service: The Pizza Analogy

The concept of "as a service" is really about shifting responsibility — who is responsible for providing a certain aspect of the service. As you move up through the types of service, more responsibility shifts away from you, and you typically pay more for it — but this may be offset by only paying for what you actually need.

Think of pizza: there's the dough, the sauce, the cheese, the toppings, the fire or oven, the electricity or gas, drinks, and a table to eat at.

- **Make it yourself** — you are responsible for all of it.
- **Take-and-bake** — you're given the dough, sauce, cheese, and toppings, but you're responsible for cooking it, providing drinks, and providing a table.
- **Delivery** — the pizza arrives cooked; you're responsible only for the drink and somewhere to eat it.
- **Restaurant** — they do everything; you're responsible for showing up and eating.

Generally, the less you are responsible for outside your core purpose, the better — all else being equal, most people would rather eat out.

## Types of Compute Service

The technology stack has layers: networking, storage, servers, a virtualization layer (hypervisor), the operating system running inside the virtual machine, middleware (runtimes such as Java Enterprise Edition or .NET, or messaging systems), the application providing a business function, and the data.

One of the key ideas behind "as a service" offerings is **cloud cadence**: constant, small, incremental capabilities being introduced, rather than the old model of a major new version every few years with one huge batch of new features.

**Infrastructure as a Service (IaaS)** — think of a VM in the cloud. The provider handles the physical network, physical storage, physical servers, and the hypervisor (in Azure, Hyper-V). You get a VM in the cloud and are responsible for the OS inside it, along with middleware updates, runtime updates, application updates, and data. Responsibility for an operating system includes patching, backup, antivirus, configuration, and high availability — Azure provides tools to help with these, but you are responsible for their deployment and configuration.

**Platform as a Service (PaaS)** — this simply provides a place to run code. You care only about your application and data. There is still probably a VM underneath in most cases, but you don't see it, and you don't worry about patching the operating system, antivirus, firewall rules, or runtime and middleware versions — it's maintained for you. It doesn't provide the business function itself, which is why it's "platform," not "software," as a service. Managed databases work the same way: managed Postgres or managed Azure SQL Database are still considered PaaS, because you still write the business logic that runs on top of the service.

**Software as a Service (SaaS)** — services like Microsoft 365 or Dynamics 365 provide the business function itself. In this model, you're not really responsible for anything except the data, and even that is a shared responsibility: the provider is responsible for durability and replication of the data, while you remain responsible for classification, labeling, data loss prevention, retention policies, ensuring data is deleted after a set period, and rights management.

Generally, if something is available as SaaS, that's the preferred option, since it carries almost no responsibility — this is why many companies use Microsoft 365 rather than patching their own Exchange or SharePoint servers; they still get the benefit of cloud cadence. If SaaS isn't available, PaaS is the next preference, avoiding the need to manage operating systems, middleware, and runtimes — app services, Azure Functions, Logic Apps, or containers might fit. Some solutions require more direct access to the operating system, for example third-party applications requiring OS-level interaction, which is where IaaS comes in. IaaS is also the simplest form of migration — a like-for-like move of an on-premises VM into a cloud VM — but it doesn't take advantage of the responsibility-shifting benefits that PaaS or SaaS offer.

An important point: none of this covers identity, accounts, and devices — those remain the customer's responsibility regardless of service model. There are tools that help: Intune for client devices, and for identity, Active Directory Domain Services on premises or Entra ID in the cloud (Azure always uses Entra ID). Even with Entra ID, you remain responsible for applying good practices: passkeys, multi-factor authentication, and conditional access policies to control access to resources.

Terms like **Desktop as a Service** (Windows 365, Azure Virtual Desktop), **Database as a Service**, and various AI service models follow the same logic — Copilots that just provide a solution are SaaS-like, Azure AI Foundry hosting models for you to build on is more PaaS-like, and running your own model directly in a VM or container is more IaaS-like. It always comes down to what you are responsible for.

Another example: running Minecraft in your own VM makes you responsible for everything. Minecraft Realms provides a pre-built world — closer to SaaS — but with less control over what you can do. There's often a balance between control and responsibility across these models.

An important point about public cloud: you are never placing an order for servers to be racked. If there's ever a "racking time," that's a colocation or hosting provider, not a true cloud. The whole point of the cloud is consuming the service you want at the moment you want it.

## When to Use Public Cloud

There's a huge shift happening across companies, though there's no single right answer for everyone. For most organizations, running IT is not their core function — it's there to support the business, and they don't want to be in the data center business.

A key goal is the ability to shift responsibility. Even just using IaaS eliminates concerns about physical servers, physical networking, physical SANs, and the hypervisor layer. From there, more responsibility can be shifted further — containers, app services, serverless — enabling modernization without running data centers, dealing with heating, ventilation, air conditioning, power, and generators.

Other motivations include proximity to customers (running ten different data centers isn't cost-effective), better disaster recovery and business continuity, and the fact that different cloud providers may operate at different price points or have different environmental goals.

Realistically, with economies of scale, it's very unlikely an organization can run its own data center — factoring in the building, physical security, power, networking, servers, and licensing — as efficiently and cost-effectively as major cloud operators can. This parallels how factories once generated their own power before utility companies could provide it more cheaply and reliably; compute has largely followed the same path.

Azure runs a very high Power Usage Effectiveness (PUE) — the ratio of power used to run machines versus other overhead — close to 1, meaning nearly all incoming power goes toward compute. For some companies, their own power bill alone exceeds their entire Azure bill.

The key point of public cloud is consumption-based billing: pay for the storage you use, not what you might use someday; pay for a VM only for the hours it runs, not because you own the hardware. Because workload for nearly every business fluctuates over time, matching resource to incoming work is the most cost-efficient approach. Requirements can also change over time — a purchased server locks you in for three to five years, but in the cloud you can delete one type of resource and create another: move from VMs to containers, from containers to app services or serverless technologies. This is a phenomenal degree of flexibility.

## Public Cloud Example Scenarios

A great illustration combines American football, the Super Bowl, and pizza. As a pizza restaurant, business has predictable busy periods — Saturday night might be three times busier than any other time of the week. Ordinarily, that would require three times the infrastructure just to handle a few peak hours, twice a week. The Super Bowl is even more extreme: pizza ordered between game breaks is roughly 50% busier than even a typical Friday night. Building permanent infrastructure to handle that one day a year would be unreasonable.

This is where cloud elasticity makes sense: normal capacity most of the time, a bump on weeknight evenings, a bigger bump on Friday and Saturday nights, and a spike for the Super Bowl — scaling back down afterward. Paying only for the hours of actual peak use is a massive benefit. The cloud can also be used to burst — running a baseline on premises and bursting into the cloud for peak periods like Fridays, Saturdays, or the Super Bowl.

### Scenarios Continued

**Predictable bursting** — like the pizza example: known busy periods (Friday and Saturday nights) versus known quiet periods (breakfast), allowing capacity to scale up and down on a known schedule.

**Startups avoiding large upfront investment** — cloud usage scales with success. As traffic and revenue grow, the bill grows proportionally. If the business doesn't succeed, there's no large upfront investment or locked-in equipment lost.

**Unpredictable bursting** — for example, an online retailer whose product suddenly gets mentioned in the news, or a news site during a major event. Capacity can scale automatically, for instance when CPU usage crosses a threshold like 80%.

**Seasonal or event-driven workloads** — a tax office busy for a couple of months a year, or a large sporting event with heavy streaming traffic during the event and little afterward. Public cloud can also supplement on-premises capacity in these cases.

Whenever there's fluctuation in required resources, or a need for flexibility in VM type, service type, database type, or AI services, the cloud's consumption-based pricing makes it very attractive.

## How to Start with Cloud

Companies typically adopt the cloud progressively, starting small, gaining confidence in both the platform and their ability to operate it, and gradually placing more important workloads there.

**Test and development** is a common first use case. It has high churn — environments are created and deleted constantly — which is awkward on premises but well-suited to the cloud. Final testing, however, should match the production environment; if production is on premises, final testing shouldn't use a completely different hypervisor or operating model.

**Disaster recovery** is another strong use case. Many companies either have no DR or a poor one because of cost — an on-premises DR site requires a fully built second location that, ideally, is never used. In the cloud, DR should be tested regularly (every few months), but most of the bill is tied to actually running compute, along with storage and possibly replication costs, depending on recovery point objectives.

**DMZ (demilitarized zone)** — offering certain services publicly from the cloud as a separation from the private network, while still requiring strong security such as web application firewalls and reduced allowed traffic types.

**Global reach** — creating instances of services across regions worldwide to serve customers, potentially using tools like Azure Front Door with cached static content close to customers.

**Special projects** — limited-duration needs, such as access to specific hardware or capabilities not available on premises (for example, large language models), spun up and deleted after use, paying only for the time actually used.

Most organizations end up going largely "all in," since running IT is generally a cost and an inconvenience relative to their core business, not the business itself. Even simply offloading physical compute, storage, network, and hypervisor is significant; using managed databases offloads further; using container offerings offloads Kubernetes management; using networking solutions offloads networking concerns. There remain certain responsibilities, but a large amount shifts away.

This connects to **Opex versus Capex**: consumption-based cloud billing (operating expense) versus buying infrastructure outright and depreciating it over years (capital expenditure). Customers typically prefer Opex.

Companies that go "all in" often start with virtual machines, then progress to containers, app services, moving databases from VMs into managed Postgres, managed SQL Database, or Cosmos DB, and adopting serverless technologies. Cloud solutions also bring strong governance — policies as guardrails, budgets, monitoring, and security — which tends to accelerate further adoption.

What typically remains on premises is "anchored" — for example, a mainframe that can't currently move to the cloud. Latency (a function of the distance between services — light travels only so fast, and even slower over fiber than in a vacuum) means a component split between on-premises and cloud will have higher latency than if both were co-located. Some workloads are latency-sensitive and some are not, which is part of why certain components remain on premises.

## Types of Azure Service

There is a massive range of service categories: compute, storage, networking, databases, AI and machine learning, analytics, security, DevOps, and IoT. Within AI and machine learning alone there are bot services, AI Search (for building semantic indexes used with large language models), general machine learning, video indexing, and broader OpenAI-based services.

Within compute there are Windows VMs, Linux VMs, VMware-based services, virtual machine scale sets, and container capabilities including container apps and other container-specific services. There are also many types of database and caching services.

Understanding what's available and what to actually use takes time.

## How to Get Azure

Regardless of the acquisition method, remember to pay for what you consume — shut things down or scale them down when not in use, and use tagging to see who's using what.

**Free account** — a one-month trial including $200 of credit, usable on nearly anything, with some limitations (such as Marketplace). In addition, a set of services are free for the first 12 months, and another set are always free — and importantly, the "12 months free" and "always free" tiers are also available on regular, non-trial accounts. Examples: AI Document Intelligence offers 500 free pages for 12 months; Cosmos DB offers 400 request units per second with 25 GB of storage free for 12 months. These are worth exploring to save money while learning.

**Visual Studio subscription benefits** — Azure credit is included depending on subscription level: $150/month for Visual Studio Enterprise, $100/month for a regular MSDN-equivalent subscription, $50/month for Visual Studio Professional or Test. These are intended for exploration, not production workloads.

**Pay-as-you-go** — creating a subscription and paying only for consumed resources. Multiple subscriptions can be organized into management groups under a tenant (an Entra ID tenant is the modern equivalent of an old Active Directory domain — all accounts, identities, and devices live there).

**Enterprise Agreement (EA)** — for larger organizations with many subscriptions, allowing subscriptions to be organized by department and account owner, controlling who can create subscriptions. Roles include the Enterprise Administrator, who can create and add accounts, and account owners, who can add subscriptions to their account.

**Cloud Solution Provider (CSP)** — essentially a reseller relationship: a partner entity provides Azure services, is paid directly, and may add additional value-added capabilities on top.

### EA Structures

There are many ways to structure subscriptions under an Enterprise Agreement — for example, by functional team (sales, legal, marketing), by business division, or geographically (North America, Europe). This is largely a matter of how the organization wants to do business, enabling subscriptions to be created per group or line of business. While tags help with billing visibility, richer policy and governance capabilities generally come through management groups.

## Limits and Quotas

Be mindful of spend and shut down unused VMs. Azure enforces limits and quotas, partly to protect organizations from themselves. There are **soft limits**, which can be raised on request, and **hard limits**, which are absolute maximums — for example, the number of cores usable in a subscription, the maximum size of a storage account, or the number of database instances allowed.

Many subscriptions start with fairly low limits specifically to constrain potential damage and cost from accidental overuse. These limits are documented, covering management groups, subscription limits (such as how many subscriptions or resource groups are allowed), Entra ID limits, and limits for every other resource type.

Limits can also vary by SKU. App Service, for example, has free, shared, basic, standard, premium (V1–V3), and isolated tiers, each with different numbers for how many instances are allowed, how much compute can be used, or which capabilities are included — one of the main reasons multiple SKUs exist for the same service.

Soft limits can be increased via a service request — in the portal, under the subscription's "Usage + quotas" page. On a free account, the spending limit can also be removed once the $200 credit is used, after which charges apply to the linked credit card.

## Reliability

Cloud reliability is approached differently than traditional on-premises reliability.

**On premises**, reliability has historically been built into hardware: centralized SAN storage with redundant disks and replication, clusters of hosts that fail over VMs, and live migration (or vMotion) during planned maintenance so operating systems experience no downtime. Very often this meant single instances of a workload. The challenge is that this only helps with planned scenarios or a full host crash — if the operating system, application, or a middleware component crashes internally while the VM keeps running, nothing protects against that unless deeper monitoring is in place. A single instance is therefore not ideal, though this is starting to shift on premises too, as containers introduce multiple instances for both resiliency and scale.

**In the cloud**, reliability is built into the software. There are distributed constructs with independent networking, cooling, power, and control-plane elements, and application instances are spread across these isolation boundaries — multiple instances for both compute and storage. VMs are typically not migrated during maintenance (though features like VM preservation can minimize downtime and may still move a VM occasionally). Resiliency comes from having multiple instances spread across update domains, so the service stays up even if a single node fails or the OS or app crashes on one instance — a much higher level of reliability.

This doesn't mean Azure's underlying infrastructure isn't highly reliable — it is, with published service level agreements (SLAs) defining expected availability. Azure data centers include large battery banks, generators, onsite fuel, and large water tanks for cooling. But rather than relying solely on that infrastructure reliability, the goal is to also design for **availability zones** — independent power, cooling, networking, and control planes within a region — so that even if an entire building goes down, the service remains available in another.

Microsoft continually adds new reliability features (for example, "Project Tigris" to improve reliability), including safer deployment practices and proactive detection of impending hardware failures. But the core principle at cloud scale — and it applies broadly, not just to Azure — is to avoid single instances and instead distribute multiple instances across fault domains, providing both resiliency and the ability to scale automatically based on demand. This has to be factored into application architecture from the start: designing around multiple instances and the ability to react to demand fluctuations.

## Why Use Azure

There are many cloud providers, and Gartner's Magic Quadrant is one useful reference — plotting ability to execute against completeness of vision. Microsoft sits in the Leaders quadrant across many categories: strategic cloud platform services (IaaS and PaaS), access management, cloud database management, AI development services, and various security capabilities.

Azure also provides strong hybrid solutions. Most companies don't want two completely separate monitoring and control planes. Microsoft's history with on-premises technologies (Windows Server, Hyper-V) carries forward into common solutions spanning on-premises and cloud — the Azure control plane, many Azure capabilities, and governance can be extended outside Azure via Azure Arc, and Azure Local allows running Azure-managed hardware, security services, and identity at a physical location. This is a significant advantage: a single control solution usable both in the cloud and on premises.

## Feature Focus

A common question is "what about feature X?" — evaluating clouds based on caring intensely about one particular feature. A useful lens here is the **Gartner Hype Cycle**, which applies not just to cloud technology but arguably to almost anything.

The pattern: a triggering event (large language models are a good recent example) leads to a **peak of inflated expectations** — the technology seems like it will do everything and solve every problem. Then, through actual use, comes the **trough of disillusionment** — it's not as transformative as first thought. Continued use leads to the **slope of enlightenment** — understanding where it's genuinely useful and where the real benefits lie — and finally the **plateau of productivity**, where it's actually put to practical, productive use.

At the peak of inflated expectations, evaluation tends to focus purely on individual features, without much thought to integration with the rest of the environment. Once something reaches the plateau of productivity, the focus shifts to integration — how it works with monitoring, security, provisioning, governance, and the rest of the environment holistically. Features matter, but any specific feature of interest today is likely to be less relevant tomorrow, and any feature missing from Azure today will likely arrive soon, since the platform is constantly evolving. The recommendation is to evaluate based on how something fits the whole environment, rather than any single feature in isolation.

## What Does This Mean

Azure is one of the few realistic major players in the market. AWS is another major player; Google Cloud and other regional offerings exist but are discussed far less. Building and populating data centers at this scale — including securing the GPUs and infrastructure needed for AI — takes billions of dollars in investment, so very few providers can operate at true global scale. Everyone else tends to be a niche player: a particular geographic market, a specific service, or a specialist capability. Azure and AWS — and to a lesser extent Google Cloud — are the true global cloud players in today's market, making them a relatively safe long-term bet.

## Close

That concludes this module. The next sessions will build on this foundation and move into more specific constructs.
