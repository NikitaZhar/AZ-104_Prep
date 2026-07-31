2026-07-02 16:42
Tags: #

## Introduction

Welcome to the resiliency module of the Azure Masterclass V3. It's critical to understand the considerations and options available to build a resilient, redundant architecture when using the cloud, or even in hybrid environments. This module dives into the types of resiliency available, some of the constructs Azure offers in terms of isolation boundaries when designing architectures, some of the replication capabilities and global balancing options, and then some of the backup-type capabilities.

## DR: Planned vs Unplanned

There are really three types of disaster recovery to consider. In a **planned** scenario — for example, a storm is coming and systems are moved over to the DR location before it arrives — there should be no data loss and no unexpected outage, since the data can be cleanly replicated over and the timing of the switchover is under your control.

This is very different from an **unplanned** scenario, where a disaster occurs unexpectedly and the data center simply disappears. In that case there is likely to be some data loss, since there wasn't time to copy over the most recent data, the failover wasn't under your control, and there's a greater chance of extended downtime, since recovering from unexpected circumstances is much harder than from full planning.

> **Exam tip:** Testing is essential — a real disaster is not the time to discover a gap in your failover process. Many organizations routinely perform a DR process, and some periodically flip the active site over and run from the DR location for an extended period (for example, a month) as the best way to build confidence in DR readiness. Lack of confidence in a DR location — not just concern about downtime — is often what causes organizations to delay failing over during a real incident, which extends the outage.

Make sure your DR plan is part of your change control process. Treat DR as a living document: whenever a change is made in production (a new resource type, a new application, a configuration change, an image update), the impact on DR must be considered as part of change control discussions.

## What are we protecting against?

To design resiliency correctly, you need to understand the different aspects that can actually fail, which determines where mitigation efforts need to be placed.

**Software.** Software exists at multiple layers — the operating system itself (which can have bugs, and a kernel-mode driver crash can bring down the whole OS), various agents (monitoring agents, antimalware components that call into drivers), runtimes and middleware (Java EE, .NET, etc.), and the application itself, along with any client applications that connect to it. If running inside a virtual machine, the VM itself depends on other components — for example, the hypervisor (Hyper-V in Azure), which has its own updates and patches, plus control-plane elements such as Azure Resource Manager and fabric controllers. Any of this software can introduce bugs that cause crashes, unexpected behavior, or performance problems.

**Hardware.** Hardware has many layers: the physical box has CPU, memory, storage, network capability, and power supply units — this is the host on which the OS or hypervisor runs. Typically there's some built-in resiliency (multiple NICs, multiple PSUs), but failures can still occur. The host lives in a rack, which has its own power supplies (often redundant) feeding the host, and a top-of-rack switch with (hopefully redundant) connectivity onward to other switches, the network, and the internet. All of these hardware components, and the data center systems around them, represent possible points of failure.

**Corruption.** Corruption can affect data, elements of the OS, or a database. It can be accidental — a bug, or a failure that causes a large number of erroneous updates — or it can result from an attack, such as ransomware encrypting data until a ransom is paid. When thinking about corruption, remember that attackers may also target backups specifically. Data at risk could be on file shares, local storage, in a database, or on other media.

**Attacks.** These include attempts to infiltrate a system, or denial-of-service attacks.

> **Exam tip:** Being resilient means being available. If a service isn't available, it isn't resilient — protection against attacks (infiltration or denial-of-service, including distributed denial-of-service that floods the service so it can't respond to legitimate requests) is part of the resiliency picture.

**Regulatory requirements.** Depending on country or industry, there can be steep penalties (including criminal liability) for failing to meet uptime or data retention requirements — for example, emergency services may need to be available within tight limits, or data may need to be retained for a set number of years.

**Humans.** Many outages are not caused by hardware failure at all, but by human-introduced changes — insufficient testing before a change, or a seemingly small change (a routing configuration change, a DNS change, a database schema change, a software change) that has a massive, unforeseen, and hard-to-undo impact. Protecting against human error is a critical part of resiliency planning.

## Doing stuff and remembering stuff

Components generally fall into one of two categories (some do both): those that **do stuff** — processing, computation, handling requests — with no state that matters (or none that can't be trivially recreated by resending the request), and those that **remember stuff** — maintaining data that matters, such as a file share or a database.

> **Exam tip:** Understanding whether each component in your architecture is stateless ("doing stuff") or stateful ("remembering stuff") is essential, because resiliency solutions differ significantly between the two — stateful components are generally much more complex to make resilient.

## What can we do against infra failures?

The most basic building block for resiliency against infrastructure failure (compute, storage, network) is simply having a copy of something. For stateful components, copying the state is a fundamental first step, though it introduces complexity around distance between copies (which affects replication time) and the tolerance for potential data loss.

For stateless components, resiliency is much easier: modern architectures are typically composed of multiple tiers or microservices rather than a single monolithic process, and as long as state is isolated to one part of the architecture (typically the database layer), the stateless tiers can simply be run as multiple instances, spread across data centers or regions, without needing to synchronize anything between the instances themselves.

For stateful resources, resiliency also requires the ability to go back to a previous point in time to protect against logical corruption — near-real-time replication alone will simply replicate a corruption to the copy as well. This is where **snapshots** (typically storing only the delta of changes since the previous snapshot, stored close to the primary data) and **backups** (stored on completely different infrastructure, often at a significant distance, with configurable retention such as weeks or months) come in. A snapshot gives you the ability to roll back to a recent point in time but doesn't protect against hardware failure; a backup, stored on separate infrastructure, provides both hardware failure protection and longer-term retention.

Configuration also matters — images used to create containers, VMs, or other workloads, and configuration used to define them, are typically stored in some kind of repository or registry with version control, giving you point-in-time views of configuration in addition to state itself.

> **Exam tip:** You almost always want both replication and backup, not one or the other — though for stateless components you may not need to back up "state" as such, just the configuration and images needed to recreate them. Good monitoring and a solid benchmark of normal resource behavior are also essential, since they make it much easier to spot anomalies (for example, a memory spike indicating a leak).

## Understanding replication requirements

**Distance.** The greater the distance between copies of data, the greater the replication delay, since light — and especially light in fiber — is not instant. A location a millisecond away behaves very differently from one 70 milliseconds away; that difference can severely impact application performance, so understanding distance and latency tolerance is fundamental to architecting the right solution.

**Recovery Time Objective (RTO).** This defines how long you can be down in the other location during a true disaster (an exceptional circumstance, not a small component failure). For very low RTO requirements (for example, 30 seconds), an active-passive architecture with startup time isn't viable — you need active-active, where all instances are running continuously.

**Recovery Point Objective (RPO).** This defines how much data you can afford to lose in an event. Reducing RPO towards zero can dramatically increase cost and architectural complexity, since it typically requires synchronous replication or similar techniques rather than "as fast as possible" asynchronous replication.

**Service Level Agreement (SLA).** This is your guaranteed uptime — for example, "four nines" (99.99%), which is financially backed, meaning there's a penalty (credit) if it isn't met.

> **Exam tip:** No SLA is realistically 100%. Even a contractually stated 100% SLA is just a financial commitment — it doesn't guarantee zero downtime in practice. The more nines required, the more expensive and architecturally complex the solution becomes (more components, more replication, more synchronization, more caching).

## What can we do against humans?

Humans should generally not be touching production directly (aside from genuine break-glass troubleshooting scenarios). Automation and orchestration tools help remove humans from the deployment path, but a caveat: automation amplifies both the good and the potential for harm — a single command run against the wrong target can take down large numbers of servers at once.

> **Exam tip:** "Human error" outages are, in most cases, really a failure of the system to have proper checks in place to prevent human accidents. Since humans will make mistakes, the system itself needs safeguards — for example, detecting and warning when a command targets an unusually large number of servers, or production instead of a test batch, and requiring a second authorization for risky operations.

A common pattern is version-controlled, pipeline-driven deployment: a human commits a change (application code, infrastructure as code, configuration, or a database schema change) to a version control system (a Git repository). This can trigger a **pipeline** that runs a sequence of checks — for example, scanning for bugs, vulnerabilities, or outdated libraries, building the code, deploying to a test environment behind a gate, then to a user acceptance test environment behind another gate, and only then, after satisfying criteria (such as no more than a defined number of open bugs), deploying to production. In no case does a human deploy directly to production.

**Monitoring** should extend beyond individual component health — the individual components (VM, app, database) might each be healthy while communication between them is broken. **Synthetic transactions** — fake transactions run through the entire end-to-end system, checking for a correct response within an acceptable time — are critical for catching this kind of failure, since they test the whole flow rather than any single component in isolation.

## Safe deployment practices

> **Exam tip:** When making any change, never roll it out everywhere all at once. Instead, gradually roll out the change through progressively larger and more important segments of the environment as confidence in the change grows. This applies to any type of change — VM resizing, database SKU changes, new architecture components, OS/runtime/app updates, Kubernetes version upgrades, configuration changes, antimalware updates — nothing happens everywhere at once.

Deployment patterns help achieve this progressive exposure, giving you the ability to find problems (or roll back) before hitting the critical population. Early testing often uses dedicated test machines, and pipelines should facilitate this progressive rollout rather than deploying straight to production.

**Canary** deployment refers to sending a change initially to a very small group; if that succeeds, the change proceeds further. **Rings of deployment** extend this idea, rolling a change out to progressively larger and more important segments of the population.

> **Exam tip:** Testing must still cover realistic, representative scenarios — it's not sufficient to test only unimportant machines and leave all of production for the final ring if only production runs at the scale or in the configuration where certain problems manifest. Either build high-quality test systems that replicate production's functional and scale characteristics, or scatter testing across regions/availability zones as appropriate — production testing cannot simply be skipped or deferred entirely.

**Blue-green deployment** uses two parallel environments: production traffic points to "blue" via a load balancer; the new version is deployed to "green," where it can be warmed up and tested before traffic is shifted over (all at once, gradually, via feature flags, or via A/B testing). If a problem is discovered, failing back to blue is fast, since it's still running. Since cloud infrastructure can be created on demand, "green" doesn't need to exist permanently — it can be created for each release and torn down afterward, reducing the cost of maintaining duplicate infrastructure.

Other considerations include your ability to remediate a problem quickly (avoid deploying initially to a remote site with no one available to fix issues) and avoiding deployments right before a weekend when problems may go unnoticed until they've compounded.

## What are we protecting against — summary

For **hardware and software failures**: have copies. If stateless, this is straightforward (recreate as needed, or run active-active); if stateful, replication is required.

For **corruption**: replication alone doesn't help, since it would just replicate the corruption. Snapshots (typically alongside the primary data) or backups are needed instead.

For **attacks**: isolate the backup's blast radius from the primary environment — different user credentials and permission structures for backups than for primary data, since attackers who compromise a user account will typically go after backups too. (Multi-user authorization in Azure Backup addresses this — covered later in this module.)

For **denial-of-service**: use protections such as DDoS Protection, intelligent firewalls, a regional Web Application Firewall, or Azure Front Door.

For **regulatory requirements**: understand and adhere to specific retention and availability requirements.

For **human error**: rely on good process — limits, checks, peer validation for commits, and tooling (including AI-assisted change review) rather than trusting individuals not to make mistakes.

## Avoid pets

Avoid treating infrastructure as unique, highly specific "pets" that require special care and manual recovery when something breaks. Instead, treat resources as **cattle** — replaceable, recreated on demand rather than nursed back to health.

> **Exam tip:** Use version control and declarative infrastructure-as-code for everything — infrastructure, workload configuration, application configuration, Kubernetes manifests, OS configuration — so that anything can simply be recreated and reconfigured by rerunning a pipeline. In a disaster scenario with a long enough RTO, non-stateful components don't need to be pre-provisioned in the DR location at all; they can simply be created when needed via the same pipeline.

Minimizing the number of stateful components (isolating state into as few places as possible) reduces the complexity of replication and reduces the number of components that need to be actively running in a DR location.

## Knowing services that need to be protected

Understanding which systems are critical to the business is essential, and this typically requires input from business units, since infrastructure and operations teams may not know which systems are true "must-haves" versus lower-priority systems.

Even once the important applications are identified, you need to understand each system's makeup — which tiers hold state, and which are stateless — and its dependencies, some of which may not be obvious to the business (for example, a web application firewall, a load balancer, a key vault holding a customer-managed key, and Entra ID for identity, on which most components ultimately depend).

> **Exam tip:** Even stateless caching layers (for example, Redis) matter — a failover with a cold cache can cause a temporary performance hit, so understand whether that's acceptable or whether cache mirroring is needed.

Regarding some specific dependencies:

> **Exam tip:** While Key Vault will replicate to its paired region if one exists, best practice is generally *not* to rely on that — use separate secrets per region/cell/stamp to limit blast radius. Entra ID is a global service and you cannot add resilience to it directly, but you should use managed identities, which get long-lived (24-hour) tokens that proactively refresh at the 12-hour mark, use a regional secure token service, and have backup capabilities — using the platform's built-in resiliency features is the way to be more resilient to Entra issues.

Tools that can help discover dependencies include Azure Migrate's discovery and assessment capability (analyzing network traffic and ports), Azure Monitor's Service Map (VM insights based on network connections), and Application Insights' Application Map (based on API calls between components).

Finally, don't forget to consider how the application is actually *consumed* — for example, jump boxes or VPN access points also need to be resilient, or a technically resilient and available application may still be unreachable during a disaster.

## Understand availability requirements

Understand your actual availability requirements and architect accordingly. Most Azure services publish a Service Level Agreement (SLA), and different services (and different configurations of the same service, such as VM disk type) have different SLAs.

> **Exam tip:** The cloud is not magic — resources still run on real hosts, racks, and switches that can fail, and maintenance operations (some of which require live migration, some of which don't) still occur. No Azure service SLA is 100%.

Pushing SLA requirements higher (e.g., from "three nines" to "four nines") typically involves a large increase in cost, which may not be justified by the actual business or financial impact of the additional downtime — though reputational or regulatory impact, not just direct financial loss, should also be weighed when deciding how much resiliency to invest in.

### Composite SLA

Your overall (composite) SLA depends on how resources are architected together, and specifically whether the relationship between components is an **AND** relationship (both are required, so overall availability is *lower* than either individual component — the probabilities multiply) or an **OR** relationship (only one of several redundant instances is required, so overall availability is *higher* — you multiply the probabilities of failure and subtract from 1).

For example, with two components each having a 99% SLA in an AND relationship (a dependency chain), the composite SLA drops to about 98% (0.99 × 0.99). With two instances of the same 99%-SLA component in an OR relationship (either instance is sufficient), the composite availability rises to about 99.99% (1 − (0.01 × 0.01)).

> **Exam tip:** Avoid single points of failure and single instances of anything to improve your composite SLA — but remember your overall SLA can never exceed the SLA of a shared dependency required by every component (for example, Entra ID). More redundancy costs more money and adds complexity, so architect to the actual SLA the business requires, not simply to the maximum theoretically achievable.

Consider client-side resilience too — caching the last-used endpoint or DNS name, circuit breaker patterns (to avoid hammering a failing dependency and making the problem worse), and bulkhead patterns (to prevent a failure from propagating to other components).

## Test, test, test

Testing should cover many dimensions: application testing, load testing, deployment process testing, failover testing, restore testing, security testing, unit testing, integration testing, and smoke testing.

> **Exam tip:** No amount of testing will be as creative as real-world usage. Test with good and bad data — junk data, illegal data, overloaded instances, expired certificates — not just the scenarios you expect to happen. Azure Load Testing (built on Apache JMeter) can help automate this.

Also test failover and backup restore processes explicitly, since gaps often surface only during real incidents (for example, a mismatch between production and DR backup tape formats). Any production change should be assessed for DR impact as part of change control.

## Chaos engineering

Since testing is never as creative as real user behavior, **chaos engineering** deliberately introduces failures into a system — especially complex systems — to find weak points and build confidence, ideally starting in development environments and progressing toward production as confidence grows.

> **Exam tip:** Azure Chaos Studio creates experiments that actually induce real failures (not simulated ones) — for example, exhausting CPU on a VM, creating disruptive pods in Kubernetes, making Key Vault unavailable, or making an entire availability zone unavailable — to test resilience under real conditions.

Consider also testing organizational resilience by removing key people temporarily (planned time off) to identify unhealthy dependence on specific individuals for operations or recovery.

## Baskets

Having multiple copies or replicas is of limited value if they run on the same physical infrastructure — the same node, rack, or data center. **Blast radius isolation** — using separate racks, data centers, power substations, cooling systems, and network control planes, and physical distance between them — is important so that a single incident cannot take out all your redundancy at once.

## Azure resiliency constructs

### Fault domains and availability sets

Azure infrastructure is built from clusters/stamps/scale units of compute and storage, made up of racks. A **rack** is a fault domain — if a rack fails, everything within it fails.

An **availability set** provides a 99.95% SLA by spreading VM instances across (typically three) fault domains within the same cluster/facility, giving resilience against a rack-level failure but not against an entire data center outage.

> **Exam tip:** An availability set requires a minimum of two instances (ideally three) to provide any benefit — a single instance in an availability set gains nothing. Managed disks can also be aligned per fault domain for additional storage-level resiliency. Availability sets have no concept of workload — mixing unrelated workloads (e.g., domain controllers and web servers) in the same availability set can, through bad luck, still concentrate a given workload on a single rack; create a separate availability set per unique workload to ensure proper distribution. There is also an update domain setting (5–20) used when rolling out updates.

### Availability zones

**Availability zones** provide a 99.99% SLA. Not every region supports availability zones, though the list is growing. An availability zone represents a group of data centers with independent power (typically different substations), independent cooling, and independent network capabilities — providing resilience against an entire data center (or its supporting infrastructure) becoming unavailable, not just a rack.

> **Exam tip:** There is no guaranteed physical distance between availability zones within a region, but the round-trip latency between them is guaranteed to be under 2 milliseconds. Which physical zone maps to "AZ1," "AZ2," or "AZ3" is a logical mapping that can differ between subscriptions. A minimum of two instances spread across zones is required for zone resiliency, and this is generally preferred over availability sets where available.

### Regions

Using multiple regions provides resilience against an entire regional outage. Azure regions are often paired within the same geopolitical boundary and at a significant distance apart, historically used for services like geo-replicated storage or certain Key Vault/backup replication.

> **Exam tip:** Paired-region usage is being deprecated in guidance. If your primary region fails and you try to fail over to your paired region, everyone else affected by the same regional outage is also trying to fail over there, creating contention ("run on the bank"). It is generally recommended to prefer regional and SKU flexibility — potentially using several regions (not just a pair) — rather than relying on a specific paired region, and to avoid global secrets/shared state tied to a single pairing (e.g., separate Key Vault secrets per region).

For true resilience, you should be thinking about multiple regions with meaningful geographic separation (to avoid a single natural disaster affecting both), in addition to availability zones for intra-region resilience. Azure has never had a complete regional outage, but the possibility must still be planned for.

A **proximity placement group** is unrelated to resiliency — it is essentially the opposite, guaranteeing very low latency and few network hops between resources that need to be extremely close together, rather than spreading resources apart for isolation.

## Async vs sync replication

Because light (and especially light in fiber, roughly 30% slower than in a vacuum) takes measurable time to travel, distance between replicas always introduces a latency penalty. Microsoft publishes network round-trip latency statistics between regions (for example, West US to East US is around 71 ms).

Within a region (including across availability zones), latency is typically around 1 millisecond, which is generally acceptable for most applications; latencies around 70ms between distant regions would significantly harm the performance of most applications for common operations.

For a relational database, within the region, **synchronous replication** is used: an application's write operation is committed to the primary and the local replica before the acknowledgment is returned to the application. This guarantees no data loss if the primary or its zone fails, and enables fast, automatable failover.

Between regions, given the much higher latency, **asynchronous replication** is used: the write is acknowledged to the application once durably stored locally, and replicated to the remote region in the background afterward. This avoids performance impact, but introduces a risk of data loss in an unplanned failover, since the most recent changes may not yet have replicated — this is precisely what the Recovery Point Objective accounts for.

> **Exam tip:** Synchronous replication across regions is rarely practical for relational applications — the latency impact is generally unacceptable. Read replicas (local read-write listeners and read-only listeners) can be used to offload read traffic and improve performance without compromising the replication model.

## Understanding Azure resource resilience support

Some Azure services are **global** and handle their own resiliency across regions — for example, Entra ID, Azure Front Door (anycast), Traffic Manager (DNS-based), and DNS zones.

Most resources are **regional**, and some offer a choice of deployment model:

- **Regional** — deployed somewhere in the region without zone-level guarantees.
- **Zone-redundant** — spread across multiple availability zones (requires at least two, ideally three, instances to actually be zone-redundant).
- **Zonal** — deployed to a specific availability zone you choose (useful for cell-based architectures, where you deploy separate zonal instances into each zone and balance across them yourself).

> **Exam tip:** Ensure all components in an architecture meet at least the same minimum resiliency level — a zone-redundant service that depends on a zonal-only resource (for example, NAT Gateway, which is zonal-only) introduces a hidden single point of failure. Architect subnets and dependencies to align with zones consistently and avoid cross-zone dependencies.

## Multi-region deployments

True resilience requires operating in at least two regions, and the architecture — active-passive or active-active — is driven largely by the data model.

For a relational database, an application instance running in a given region can typically read from a local replica but must write to the primary (read-write listener), accepting the added latency for writes. Applications must account for potential replication lag (for example, reading data immediately after writing it may require special handling, since it may not yet be present on a local replica).

> **Exam tip:** True active-active, multi-write replication for relational databases is technically possible but very complex — it introduces write replication lag and requires conflict resolution, undermining the strict consistency and rollback guarantees of a relational database. Where genuine multi-region active-active writes are needed, NoSQL solutions (such as Cosmos DB) are typically a better fit, offering configurable consistency models (e.g., session consistency with eventual consistency across regions) rather than the strict ACID guarantees of a relational database.

Don't forget other dependencies — shared image galleries support replication, repositories are typically globally replicated automatically, and you should avoid unnecessary duplication (for example, don't make a storage account geo-redundant if the application layer is already replicating that data itself). Some resources, like public IPs, are region-locked, requiring a global balancing solution.

### Global balancing solutions

- **DNS-based (Azure Traffic Manager)** — exposes its own DNS name and resolves to different endpoints using distribution methods such as performance-based routing; works with virtually any backend.
- **Azure Front Door** — a layer-7 (HTTP/HTTPS) global service using Microsoft's global network and points of presence, exposing an anycast IP so clients connect to the nearest point of presence, terminating the TCP/TLS session locally, then relaying to the healthiest backend. It also provides caching, effectively combining load balancing with a CDN.
- **Global load balancer (layer 4)** — also anycast-based, pointing to regional load balancers, providing balancing at a lower layer than Front Door.

## Infrastructure as code

Version-controlled infrastructure as code ensures consistency and enables rebuilding an environment reliably, rather than depending on manual portal configuration. For components with a sufficiently long RTO, stateless resources don't need to be pre-provisioned in a DR location at all — they can be created on demand via the pipeline during failover, saving cost, though this introduces the risk of contention if many organizations try to provision resources in the same region simultaneously during a widescale event, reinforcing the case for regional flexibility (or, where a specific SKU/region combination is mandatory, an on-demand capacity reservation).

## Replication preference

Where possible, use the highest-level native replication capability available for the specific technology (e.g., a database's built-in replication) over lower-level replication (e.g., OS-level or disk-level replication), since native capabilities are typically the most reliable and consistent — but note that higher-level replication generally requires the target compute resource to be running, adding cost, whereas disk-level replication does not.

> **Exam tip:** There isn't one "correct" universal replication method — choose per component, based on its importance, the acceptable RPO/RTO, and cost. Critical systems (e.g., a core SQL database, domain controllers) may warrant paying for an actively running replica; less critical systems may only need periodic disk replication or none at all (recreate on demand during a true disaster).

### Non-stateful components

If not running active-active, consider your RTO — non-stateful components can often simply be recreated on demand via infrastructure as code and a pipeline, rather than kept running continuously, though regional/SKU flexibility remains important to avoid contention during a true regional disaster, and specific SKU/region requirements may necessitate an on-demand capacity reservation. Also make sure all required artifacts (images, repositories) are actually available in the target region.

## VM replication

For regular virtual machines (on-premises Hyper-V, VMware, physical servers, or another cloud), **Azure Site Recovery (ASR)** provides replication into Azure without requiring a running VM in Azure until failover actually occurs — you pay for the replicated disk and the ASR license, not compute, until failover.

> **Exam tip:** Azure Site Recovery is free for the first 31 days per machine; if using Azure Migrate for a migration, the free period is around 180 days.

- **Hyper-V VMs** use the Hyper-V Replica capability with a host provider, replicating changes to a VHD in Azure (typically every 30 seconds to 5 minutes); on failover, a VM is created and attached to the replicated disk.
- **VMware or physical servers** use an ASR replication appliance and a Mobility service installed between the file system and volume driver, continuously replicating changes to a VHD in Azure. Failback in this scenario is only supported to ESXi — not back to bare metal or another cloud.
- **Azure-to-Azure** replication uses similar mobility-service technology, and supports **consistency groups** so that multiple VMs get crash-consistent or app-consistent snapshots at the same point in time (app-consistent uses Volume Shadow Copy Service on Windows, or pre/post scripts on Linux). ASR also supports **recovery plans** to automate multi-step failover sequences (including manual approval steps) rather than relying on manual runbooks.

## We don't pick one

Different components warrant different levels of protection, and picking the highest appropriate solution per component — not a lowest-common-denominator approach for the whole environment — gives the best outcome for the cost. Automation (such as ASR recovery plans) can still let you execute a complex failover with minimal manual effort even when using differentiated solutions per component. The right choice ultimately comes down to the RPO/RTO requirements and cost tolerance for each component.

## Clustering in Azure IaaS

Traditional clustering approaches (e.g., for SQL Server Always On) face a complication in Azure: IP addresses can't simply be moved between instances the way they can on-premises.

Historically, a load balancer was placed in front of cluster instances, owning the IP associated with the cluster's virtual network name, using a health probe to route traffic to the active node. Windows Server 2019 and later introduced the **Distributed Network Name (DNN)**, which binds to every node's IP directly, letting clients find the active node without requiring a load balancer. **Multi-subnet failover** further speeds up client reconnection across subnets.

Clusters often need a witness for quorum; Azure provides a **cloud witness**, using a storage account and a lockable blob file representing the cluster's unique ID, useful for placing a witness in a third region.

For shared storage scenarios, Azure offers shared disks (multiple nodes connecting to the same VHD) and Azure Elastic SAN (iSCSI-based shared storage). 

> **Exam tip:** Where possible, avoid shared storage architectures in the cloud — **Always On availability groups**, which replicate data to each node rather than relying on shared storage, are generally the preferred, more cloud-native approach.

## Azure Backup

Azure Backup uses two vault types: the original **Recovery Services vaults**, and the newer **Backup vaults**, used for newer workloads — visible together via the **Business Continuity Center**, which shows which vault type is used for a given workload.

Depending on the technology, the backup mechanism varies — some workloads use their own native snapshot/replication capability orchestrated by Azure Backup; others are copied to a managed storage account by Azure Backup itself (for example, using a dump-based mechanism run on a worker role, writing to block blob).

> **Exam tip:** Azure Backup VM extensions support app-consistent backups — Volume Shadow Copy Service on Windows, pre/post scripts on Linux — and use delta-based (incremental) storage rather than full copies each time. Retention policies are configurable for daily, weekly, monthly, and annual backups, each with its own retention duration.

Vaults themselves can use zone-redundant or geo-redundant storage for the backup data, and geo-redundancy enables cross-region restore even if the source region itself isn't down.

## Protecting backups

Since attackers who compromise an account often go after backups directly, backup protection is critical, even with minimum-permission practices and PIM elevation requirements for backup administrators.

> **Exam tip:** Azure Backup provides **soft delete** by default — a deleted backup is retained for 14 days at no extra cost and can be recovered during that window. Soft delete can normally be disabled, but an **always-on (irreversible) soft delete** option prevents it from ever being disabled, and the retention window can be extended up to 180 days (at additional storage cost).

**Multi-user authorization** provides stronger protection: critical operations on a backup vault are protected by a **resource guard**, managed by a completely separate identity (ideally in a different tenant, and at minimum a different subscription) than the backup administrator. The backup administrator must obtain temporary permission on the resource guard (for example, via PIM, requiring another human's approval) before performing any critical/protection-reducing operation — ensuring that a single compromised identity (the backup admin) cannot unilaterally disable backup protection.

**Immutable vaults** provide a write-once-read-many (WORM) guarantee: data cannot be deleted before its intended expiry. Immutability can initially be enabled in a reversible state for testing, but once set to **enabled and locked**, it cannot be disabled by anyone, regardless of permissions.

> **Exam tip:** Be certain of your retention settings before locking immutability, since an enabled-and-locked immutable vault with a long retention period cannot be shortened or disabled afterward, and you will continue to incur the associated storage cost for the full retention period.

Combining immutable vaults with multi-user authorization provides a strong overall backup security posture.

## Close

That concludes the resiliency module.
