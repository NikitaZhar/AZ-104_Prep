2026-07-03 17:42
Tags: #

## Introduction

This is the Azure Storage module of the masterclass, focused on the functionality of the Azure storage account — a building block that many other services in Azure are built on. The module covers the capabilities of Azure Storage, some capabilities of virtual machines (also a fundamental building block for many other Azure services), and some storage tools.

## Types of storage

When thinking about the types of storage available in Azure, there are several considerations:

- **Durability** — the ability to preserve data over time.
- **Latency** — how long a given operation takes. Latency is a function of both the distance between the client performing the operation and the storage itself, and the time the storage takes to perform the operation. In the days of hard disk drives there was seek time, as the head had to move to the right spot; SSDs have no seek time, but performing operations still takes some amount of time.
- **Structure of the data** and other factors — different workloads have different needs for different types of storage, both ephemeral and durable.

**Ephemeral** storage exists for a moment in time — it's temporary and volatile. This could be used for caching purposes, or something you'd store a page file on — but if you lose power on the compute workload, you'd lose the content.

**Durable** storage is persistent — this is data you want to keep, and it has to be stored on some type of long-term media, typically a disk, but sometimes even tape.

There are various types of data that vary greatly depending on what an application needs:

- **Unstructured** — for example, a media file: an image, a video, some unstructured piece of data. You could essentially store anything, including binary data.
- **Structured** — typically this means a database, because it has a schema. The schema describes what tables exist, what the columns (attributes) of the data are, and what type each is — a text field 16 characters long, a floating point, binary, and so on. This is a fixed format, and data placed into a particular table has to adhere to that schema. It's a very rigid format, common in relational databases, where data is normalized to be very efficient about how it's stored, with relationships between tables — foreign keys, where a record in one table relates to a record in a different table — enabling efficient storage.
- **Semi-structured / documents** — often self-describing, so there isn't a traditional schema, but you'll have something like an XML or JSON file that describes its own structure within it, and a single document can contain a mixture of different structures.

Depending on the type of data, different capabilities may be needed:

- **Indexing** — important in relational databases and even semi-structured documents, because you want to be able to interact with a specific record without having to trawl through a huge amount of data to find it. You can have one or more indexes that make it easy to find records based on the particular columns chosen to be indexed.
- **Snapshots** — the ability to capture a point-in-time view of the data.
- **Replication** — data may need to be replicated between different regions, different instances, from on-premises to the cloud, and so on.
- **APIs** (application program interfaces) — ways one application can talk to the data; different protocols may be supported, such as block level (where you see the blocks on the disk and manage the file system yourself) or file based (where you talk to an API and just deal with files and folders).

Requirements vary a lot depending on the use case, and there isn't one best answer. Very typically an application will use multiple different types of storage, because different parts of it will have different requirements — structured, unstructured, or semi-structured data; ephemeral storage for caching purposes; durable storage for long-term state. There's no single best option — it comes down to the particular requirement of the part of the solution you're trying to solve.

## Azure Storage 101

Azure does not use traditional storage area networks (SANs) or network attached storage (NAS) solutions the way we think of them in our own data centers, with big arrays and lots of fiber connections — that doesn't exist in Azure. The one exception (covered later) is Azure NetApp Files, the only solution where actual NetApp filers sit inside Azure data centers.

For Azure Storage, there's a special storage stamp — clusters of storage servers that provide the storage services. They do not use traditional SANs; it's all focused around software-defined storage. These storage services, powered by Azure Storage, are leveraged by pretty much all of the other capabilities in Azure — there's a hierarchy: Azure Storage sits at the base on these storage clusters, then Azure database services use that storage, and even more advanced services like Azure Key Vault use blob under the covers. These things all build on top of each other.

Azure's own storage architecture, running on these storage clusters, is a three-tier architecture that provides all the scale, capabilities, and replication needed, and it's actually built on work originally done for Bing (Cosmos was used in part for the replication).

- **Storage stamp** — a certain number of racks within a cluster, with different fault domains, redundant networking and power. Special types of storage stamps exist — for example, premium offerings are on a different type of storage stamp with a different type of disk.
- **Stream layer** — the bottom of the hierarchy — think of it as the actual bits on the disk. It's responsible for the distribution and replication of data across the servers in a particular cluster (stamp) to make it durable. A stream is just an ordered list of storage chunks made up of various blocks.
- **Partition layer** — understands the constructs we interact with: abstractions like blobs, tables, and queues. It provides a nice, scalable namespace.
- **Front end layer** — essentially stateless; it takes in the API request, looks up the account name, does the authentication, and routes it to the partition layer, which then takes care of the actual durable storage.

DNS is used for pretty much everything in Azure to some capacity, including the namespace for a storage account. You always want to use encryption, so it's `https` with TLS 1.2 or above. The name format is your account name, the type of service (blob, file, queue, table), `core.windows.net`, then a partition that varies depending on the type of service, and finally the actual object you want to interact with.

In the demo, one of the existing storage accounts is shown under `Settings` → `Endpoints`, where each service's endpoint is visible:

- For `blob`: `https://<account-name>.blob.core.windows.net` (using TLS encryption). Blob also has a secondary endpoint.
- For `file`: `https://<account-name>.file.core.windows.net`.
- For `queue`: `https://<account-name>.queue.core.windows.net`. Queue also has a secondary endpoint, as does table.
- Only `file` does not have a secondary endpoint.

Data is replicated in two ways. In your home region there are always three copies of the data, and that replication within the region happens at the stream layer, synchronously — a client writing data doesn't get an acknowledgment back until it's been durably stored across all three copies. If Zone Redundant replication is chosen, those three copies are split across the three availability zones in the region. If Geo Redundant replication is chosen, there are another three copies in the paired region, but that replication is asynchronous — those copies are replicated in the background as quickly as possible, but in an unplanned disaster scenario some of that data could be lost, while the three copies in the home region remain intact. That means six total copies with geo-redundancy.

## Storage account basics

The top-level namespace for all the various storage services is the storage account. It's created in a specific region, like almost every resource in Azure, and lives in a specific resource group. In the demo, an existing storage account is shown to confirm it lives in a certain location and a certain resource group. When creating a new storage account, you must give it a name, pick a resource group, and pick a region.

There are actually different types of storage account. The portal hides a lot of this today, asking general questions about what you're going to use it for and what performance tier you want. If you're using the CLI or templates, you have to specify the exact type.

- **Standard** and **Premium** — performance tiers, which change the backend storage stamp used to provide higher performance.
- **General Purpose V2 (GPv2)** — what you use for most general things; it supports blob, queue, table, and files — this is typically what you'll leverage.
- If you pick the premium offerings, you get into very specific account types that only support one particular type of service:
  - `Premium Blob` uses block blob storage.
  - `Premium Files` uses file storage type, or provisioned V2 page storage if you pick premium page.

In the demo of creating a storage account, the portal asks what the primary service will be — `blob`, `files`, `tables`, or `queues`. If `blob` is picked, performance stays `Standard` by default, meaning General Purpose V2 is used. Flipping to `Premium` brings up a choice of a very specific account type (block blobs, file shares, or page blobs). Selecting West US 2, `files`, and `Standard` also surfaces an additional choice about billing and performance — for example, `Provisioned V2`, which creates a provisioned V2 file storage account. There are specifics happening behind the scenes that the portal abstracts, but if you're using templates, CLI, or the REST API you need to understand those options.

After picking the SKU and performance, you choose a resiliency type. This drives different SLAs — for availability of the service, for reading from it, for writing to it — and is completely separate from the durability of the data. Remember the different layers: if the front-end layer had a problem, that doesn't mean you're losing data — the data is still highly durable on disk.

## Storage durability

Durability is the chance of actually losing data.

- Even locally redundant storage (`LRS`, the regular three copies) has 11 nines of durability; as far as is known, there has never actually been data loss in Azure.
- Zone-redundant storage (`ZRS`) gives 12 nines of durability.
- Geo-redundant storage (`GRS`) gives 16 nines of durability.

These are extremely high levels of durability. That doesn't mean you can always interact with the data, though — your data is safe, but your ability to actually interact with it can vary. That's where having additional copies replicated in different places helps ensure you can always interact with your data. Options include `LRS`, `GRS`, `ZRS`, and `GZRS`. If, for example, `GRS` is picked, there's also the option to get read access to that secondary copy, even without any regional problem going on at that moment.

## Resiliency options

> **Exam tip:** The key point to remember is that these different resiliency options (`LRS`, `ZRS`, `GRS`, `GZRS`) are about the ability to interact with the data, not the risk to its durability. Durability does increase as you spread the data out more, since more copies and their placement (e.g., across two regions) give you even greater assurance that your data will be available.

Let's dig into what these options actually mean for how your data is stored, starting within a single region (Region 1) and a single storage cluster (storage stamp):

- **LRS (Locally Redundant Storage)** — always three copies of your data, and all three copies live in the same storage cluster.
- **ZRS (Zone Redundant Storage)** — still three copies, but now distributed across storage clusters in the three availability zones (AZ1, AZ2, AZ3). This gives even higher resiliency: durability goes up a bit, and because different storage clusters have their own front-end services, your ability to interact with the data also improves, since you're now resilient to a failure in just one of them.
- **GRS (Geo Redundant Storage)** — uses paired regions, which Microsoft designs to be a significant distance apart. With GRS, you have three copies within a certain storage cluster (synchronous), which are then asynchronously replicated to another storage cluster on the paired region side, where another three copies exist.
- **GZRS (Geo-Zone Redundant Storage)** — a hybrid of ZRS and GRS: on the primary region, the three copies are distributed across the three availability zones, but on the paired region, the three copies still sit within a single storage cluster (it is not ZRS on the other side).

For GRS and GZRS there's an optional (slightly more expensive) capability to get read access to the secondary copy, using a secondary endpoint to talk to the services in that region — this works for blob, queue, and table, but not for Azure Files. In a real failover, the primary endpoint would just point to the secondary region — but as far as is known, Azure has never actually had a full regional failover, although there were some changes made in this area (covered next).

From a cost perspective, moving down these options costs more money: ZRS costs more than LRS; GRS costs more than the non-geo options because there are six copies instead of three; GZRS costs more than GRS because those three copies are spread across the AZs. You also pay for data egress leaving the region.

You can switch between resiliency levels — for example, between LRS and GRS (there are some special steps for switching to ZRS) — and you're not locked into what you picked at creation time.

## Storage account failover

Because Azure has never had a full regional outage, but there can be smaller-scale issues that impact you, there's a capability called customer-managed failover.

In the demo, under a storage account's data management section, you can see the current redundancy (in this case `GRS`), the primary and secondary regions, and a nice picture of where they are. There's an option called `Prepare for failover`, which lets you do an unplanned failover. It shows when the copies were last synchronized — any data after that point could be at risk of loss. There's also a grayed-out `planned` option, in preview, unavailable in the region being used.

- **Unplanned failover** — the secondary becomes the primary, but it becomes `LRS`; the direction doesn't automatically reverse, so you'd need to turn `GRS` back on afterward to reverse it.
- **Planned failover** (preview, only available in certain regions) — swaps the primary and secondary while keeping `GRS`, and reverses the direction of replication.

This is useful when Azure hasn't declared a regional outage, but an issue is impacting your service and you want to perform a customer-managed failover.

## APIs and other features

There are APIs available to interact with the services — that ability, and those SLAs, to actually talk to the data. The options available vary depending on the actual service:

- There are always base APIs for blob, queue, and table.
- `files` can be talked to via `SMB`, and also `NFS`.
- `blob` can also use `NFS` and `SFTP`.
- Data Lake can use `HDFS`.

Capacity, IOPS, and throughput vary based on a lot of different factors — the SKU picked, the performance tier picked, and whether pay-as-you-go or provisioned options were used. Higher performance always means paying more money, but provisioned options may give you more consistency in your bill, which can matter more to some customers than paying the absolute minimum.

Different services also have tiers available:

- For `blob`: `hot`, `cool`, `cold`, `archive`.
- For `files`: `transaction optimized`, `hot`, `cool`.

These tiers let you balance the need to store and have the data available against how much you intend to actually interact with it. Newer data you expect to interact with a lot may be better suited to `hot` or `cool`; data you need instant access to but expect to interact with rarely may be better suited to `cold`.

There's also monitoring and logging: different metrics available for different services, and diagnostic settings to control sending logs to another storage account, an event hub, or a Log Analytics workspace.

In the demo, the `Logs` and `Metrics` sections are shown: metrics exist at the account level and at the level of individual services (for `blob`, for example: capacity, container, account, and types of transaction). Values can be aggregated in different ways (average, minimum, maximum). This granularity matters because these are things you pay for.

## Object level replication

Object level replication is worth focusing on separately, because the paired-region approach used by GRS isn't always the flexibility you want. Object level replication gives that flexibility, and it applies to blobs.

It works at the container level (the structure blobs live in), enabling replication to a container in another storage account of your choice. It uses two features covered later — change feed and blob versioning — which track different versions of a blob as it's changed, and when those changes took effect, essentially acting as a notification that a blob has been changed and needs to be sent elsewhere. This only works for block blobs, not for append or page blobs.

The setup: there's `Storage account 1`, with a `blob` service and two containers — Container 1 and Container 2. In a different region (not necessarily the paired one) there's a completely separate storage account (Storage account 2) with its own Container 3, and there could be yet another storage account (Storage account 3) with its own container. Object level replication lets you say, for example, that Container 1 should replicate to a container in Storage account 2, while Container 2 should replicate to a container in a completely different storage account, Storage account 3.

Replication is asynchronous — there's no guarantee that a write is only acknowledged once it's been copied to the destination (that would hurt performance given the distance involved). It uses the change feed to know when something happens, and blob versioning to work out what changed and copy the right version across.

Key properties of object level replication:

- Filtering is supported — you can apply a filter to control what actually gets copied.
- It works for premium block blob accounts as well (unlike many other capabilities limited to general purpose accounts), but not if Data Lake (hierarchical namespace) is enabled.
- It is not tied to regional pairs — you can pick any region you want.
- There's flexibility in which blobs are replicated — you can specify a prefix that must match, and you can have a certain number of rules within a policy.

> **Exam tip:** A key point here is that you can only replicate to one destination per policy.

One nice feature is you can use different tiers for the source and the target — for example, hot on the source but cool on the target, giving you more flexibility.

In the demo, a replication rule is created: pick the destination storage account, pick the source container, pick the destination container in the other storage account, add filters (prefix match), and control which objects are copied — only new items, everything, or a custom set (e.g., only items created starting from a certain date). This gives flexibility without having to align to storage account regional pairings, but remember it uses the change feed, requires transactions (reading and writing) that you'll pay for, and network egress that you'll also pay for — so factor those costs into your planning.

## Storage account services

If the account type is General Purpose V2, all the storage services are available. If a more specific type was picked (e.g., block blob, page, or files), only that one service is supported.

### Blob offerings

Blob is probably the most fundamental type of object used across nearly everything in Azure — great for very large amounts of unstructured data. The main type to think about is block blob: different-sized blocks joined together to form the blob, and it's useful for storing pretty much anything.

By default, block blob uses a flat namespace — there's no concept of true directories. You've probably seen "directories" when working with block blob, but those are virtual directories: the directory name is just added as part of the blob's name; the system doesn't actually understand a directory hierarchy. When hierarchical namespace (HNS) is turned on — which is what Azure Data Lake Storage Gen 2 is — it sits on top of blob but adds true hierarchy.

When hierarchical namespace is enabled, additional capabilities become available:

- `NFS` (NFS 3.0) lets you interact with block blob differently, but you can only use it from a network connected to it, and hierarchical namespace has to be enabled.
- `SFTP` gives encrypted FTP access, also requiring Data Lake to be enabled.

The hierarchy: storage account → `blob` service → containers (the structure blobs are placed into). If hierarchical namespace is enabled, containers become true folders. The fundamental thing stored inside is a block blob (flat namespace, no true directories). If hierarchical namespace was turned on at creation time, this becomes a Data Lake (what ADLS Gen 2 is built on), unlocking additional capabilities — NFS 3.0 and SFTP, both requiring hierarchical namespace to be enabled — plus additional APIs: the Blob REST API, the Data Lake API, NFS access, and SFTP access.

In the demo of creating a storage account, when `blob` is picked, there's an option to enable hierarchical namespace, which makes the account a Data Lake Storage Gen 2 account. With hierarchical namespace on, you can additionally enable `SFTP` and `NFS v3`.

> **Exam tip:** Hierarchical namespace cannot be changed after the storage account is created — it's a fundamental decision made once, at creation time.

In `Storage browser`, a container might show what looks like a folder, e.g., `testing`, that you can navigate into — but really that's just part of the file's name. There's no true folder system unless hierarchical namespace is enabled — in a flat namespace, all "folders" are virtual and created on the fly.

Other blob types:

- **Page blob** — made up of pages of 512 bytes, optimized for random read/write access. In the early days of Azure, you'd work directly with page blobs, but today managed disks are used instead, so in most scenarios you no longer interact with page blobs directly.
- **Append blob** — essentially a block blob, but you can only add blocks to the end of it — you cannot update or delete existing blocks. This is useful for log-style scenarios, making it easy to keep adding data to the end of a file.

Additional blob capabilities:

- **Blob index tags** — key-value pairs, similar to Azure Resource Manager tags, but these are part of the data plane rather than the control plane. They can be used, among other things, as part of attribute-based access control. You can have up to 10 of these tags per blob. In the demo, a tag `Milestonerelated = true` is shown set on a specific file via `Storage browser`, and a filter (e.g., `milestonerelated = true`) is added under `Containers`, after which only blobs matching that filter are shown.
- **Blob inventory** — creates reports on a daily or weekly basis. In the demo, under `Data management` → `Blob inventory`, a rule is created that inventories `block`, `page`, and `append` blobs, with the option to include versions, set frequency, and choose the output format. The result is written into an `inventory` container, organized into date-based folder structures. Since those folders accumulate over time, it's worth using life cycle management (covered later) to clean them up.

### Files

The `files` service can be interacted with via various REST APIs, but it also exposes an `SMB` or `NFS` share — you choose SMB or NFS when creating a share.

For **SMB**: 2.1 or, ideally, 3.x is supported (the exact version depends on the client — up to 3.1.1 with Windows Server 2022 or Windows 11). From version 3 onward, encryption is used, letting you talk to it even from outside the region. SMB supports a number of authentication options, plus snapshot capability, surfaced through the Windows "Previous Versions" tab. SMB is built on Windows and is designed around Kerberos authentication: you can use your existing Active Directory Domain Services (your storage account gets an object within your AD DS), Entra ID's Kerberos capability (useful if a client doesn't have line of sight to your domain controllers), or, if you don't have AD DS at all, Entra Domain Services (a managed set of Active Directory), with the usual access control lists on files and folders.

For **NFS** (NFS 4.1): only usable from a trusted network, since there's no encryption in transit — this has to be picked up front. NFS 4.1 requires a premium files account and is designed only for Linux — Windows is not supported.

The hierarchy: storage account → `files` service → share (`SMB` or `NFS`, with NFS requiring premium) → folders and files underneath — the standard structure you'd expect from file-based storage.

### Table

Table and queue (below) are increasingly de-emphasized services.

Table is a key-value set of data — a kind of NoSQL with no set schema. There are entities, acting as rows, that can have whatever columns you want. A partition key controls how the data is sharded and partitioned, and together the partition and row key uniquely identify a particular record. Today, most customers can use Cosmos DB's key-value store instead, which offers far richer capabilities.

### Queue

Queues are just messages. Service Bus queues are generally a better option. Storage queues are first-in-first-out, but that order isn't guaranteed (Service Bus does guarantee it), and only get, put, and peek are supported, whereas Service Bus offers much richer publish/subscribe and topic concepts.

In the demo, `Storage browser` shows a `File share` (viewed with Entra authentication), a `Queue` (messages can have properties like a lifetime, and are pulled off first-in-first-out), and a `Table` (no schema — entities can have any properties; the interface suggests properties used previously on other objects to save time, but doesn't require them). Cosmos DB's capabilities in this space are far richer today, which is why queue and table are less heavily used now.

A common use case for queues is event-driven processing: something happens, you drop a message on a queue about it, another event-driven component picks it up, reads an ID, and goes and processes the corresponding object. In event-driven architectures, a queue is commonly used to trigger a serverless Azure Function, a Logic App, or something similar.

## Money

Standard performance and blob premium are billed on a consumption basis: you pay for the amount of capacity actually used and for the type of operations (transactions) performed.

In the demo, on the pricing page, hierarchical (Data Lake) and flat namespace pricing are essentially the same (there is a tiny price difference). You pay for storage, plus separately for write operations, list operations, read operations, data retrieval, and data write — essentially, for the work being done.

> **Exam tip:** Capacity cost increases the "fancier" the tier — premium costs more than hot, cool, or cold; archive is the exception (see below). Operation cost works the other way — the higher the tier, the cheaper the operations, and the lower the tier, the more expensive the operations. Archive is the exception because it's effectively offline storage: you need to keep the data, but don't need instant access to it.

A key point: for blob and files, on the pay-as-you-go model, tiering affects the balance between capacity cost and operational cost. That lets you tune whether you pay more for capacity but less for transactions (if you expect to interact with the data a fair amount), or pay the absolute minimum to just store something you don't expect to interact with much, accepting a higher cost if you do have to access it.

## Tiering

For **block blob**, an important point is that the tier is set per blob — many different blobs in the same container can have different tier levels. Options (aside from premium, which has no tiers — it's its own account type chosen for very high performance, paying more for capacity but much less for operations):

- `Hot`
- `Cool`
- `Cold`
- `Archive`

`Archive` is the deepest archive tier — essentially offline: it keeps the data for you, but you can't interact with it live. This suits, for example, a data retention scenario where you need to keep data for years but have almost no intention of interacting with it — you pay almost no money to store it, but if you do need to read it, you have to rehydrate it back into another tier first, and then access it.

`Cold` was introduced later because sometimes waiting up to 24 hours to move data back into an online tier isn't acceptable — cold is cheaper than hot, but is still available online immediately.

> **Exam tip:** Capacity cost is highest for premium and lowest for archive; operation cost works the other way — cheapest at the top (premium/hot) and increasing as you move down the tiers, with rehydration out of archive being significantly more expensive. As data ages, it's very common to want to move it down tiers over time; many people leave data sitting in hot when it could quite happily live in cold, but it can be hard to know the right option.

For **files** (pay-as-you-go), the tier is a share-level setting, not a per-file setting. Options (aside from a separate premium variant):

- `Transaction optimized`
- `Hot`
- `Cool`

As with blob: `transaction optimized` costs the most for capacity but the least for operations, moving toward `cool` reverses that.

In the demo, under `Containers`, files show an `Access tier` column: some are hot, some cool, some cold, some archived. A file in `hot` can be downloaded immediately; a file in `archive` is grayed out ("archived and cannot be downloaded") — it must be rehydrated (tier changed) first, which can take up to 24 hours. The tier can be changed at any time, and for blob this is done per file, while for file shares it's set at the share level (chosen when the share is created).

In the demo of files pricing (`Provisioned V1` = premium, pay-as-you-go), storage cost decreases moving from transaction optimized to hot to cool, but operations get progressively more expensive.

## Provisioned based billing services

The pay-as-you-go model, for standard performance and blob premium, means paying for capacity and operations. Files premium and page blob premium, however, are provision-based — you pay for the amount of capacity you provision for the service, not the amount you're actually using. For example, if you provision a 100 GB premium file share, you pay for the full 100 GB, even if you only have 10 MB stored in it.

Performance scales based on the amount of capacity provisioned — you may need to pick a much larger capacity than you actually need just to get the associated performance. This is an important distinction to understand: are you being charged for what you put in, or for the provisioned size, and you choose that when you create the account or the share.

## Provisioned v2 standard

There is a newer, now-regular option: **standard performance files provisioned V2**. This is chosen when creating the storage account. With this model, you separately pay for provisioned capacity, IOPS, and throughput, and you can dynamically change these at any time — think of it as three separate dials: one for capacity, one for IOPS, and one for throughput, and you pay for whatever you set each dial to.

IOPS and throughput can be changed dynamically. If you increase IOPS or throughput, you have to wait 24 hours before you can lower it again. This model is expected to eventually come to premium files as well, giving you much more flexibility in what you pay for and how.

> **Exam tip:** When planning Azure Storage costs, don't forget about data transfer — when data is replicated to another region, there's still underlying network egress cost to pay for.

In the demo, on the pricing page for files provisioned V2, billing is shown based on provisioned storage amount, provisioned IOPS, and provisioned throughput. Switching to `GRS` adds an additional line item — data transfer — the bandwidth used to replicate data to the secondary Azure region.

## Data Lake features

Azure Data Lake Storage Gen 1 was a completely separate service that didn't do very well. **Azure Data Lake Storage Gen 2** builds on blob, adding true hierarchical namespace through a real directory structure. You can store almost anything in it — Parquet files, Avro, database tables (structured data — Parquet, a columnar storage format, is very commonly used), semi-structured data (JSON, XML), or unstructured media. You get a lot of blob features — tiering, premium options, life cycle management — but not all of them.

In the demo, a page lists which blob capabilities are supported when hierarchical namespace is enabled. Some are unsupported or in preview: for example, **not supported**: blob index tags, custom domains, customer-provided keys, object level replication, and point-in-time restore for block blobs; there's also limited blob snapshot support. Coverage is fairly complete overall, but it's worth checking the current list of capabilities you actually need before designing around it.

Data Lake is typically used as a raw store. In today's data pipelines, instead of the classic extract-transform-load approach, it's now very common to dump raw data into the data lake as-is before processing, because storage has gotten much cheaper (in the old days storage was so expensive that you'd have to transform and shrink data down to the minimum possible before storing it). If you store the raw data before transforming, normalizing, validating, and pruning it, you can come back to the data lake later for a use case you hadn't thought of and re-analyze or re-transform it — that's why data lakes are so appealing: the cost efficiency lets you do analysis you hadn't yet thought of.

Because Data Lake is built on blob, you can use all the blob APIs. There's also a **DFS API** (distributed file system API), and tools like Hadoop, Spark, and Databricks have an **ABFS** driver that talks to the DFS driver — essentially standardized APIs that data analysis solutions expect, letting Hadoop, Spark, and Databricks plug in and leverage this capability.

The true hierarchical directory structure matters: with regular block blob, "folders" are really just part of the file name, so moving a file between virtual directories has no real move operation — it has to copy all the blocks to a new file and delete the old one, which is very expensive for a large file. With true directory structure (Data Lake), moving is an actual metadata change, so it's very fast — very useful in data analytics, where you often move data between stages of processing.

Data Lake also gives you POSIX-style ACLs, letting you use familiar types of access control, and you can also use Entra ID identities as part of those access control lists.

In the demo, an account with Data Lake enabled shows a `Manage ACLs` section with POSIX-style owner, owning group, other, mask, and the ability to add additional principals — this richer set of permission capabilities comes from hierarchical namespace being turned on. If you have a workload designed around a true file system that wants to manipulate directories, hierarchical namespace will be a big benefit.

From a cost perspective, since it's built on blob, it will always cost the same or more than blob — it should never be cheaper. On the pricing page, the hierarchical namespace pricing is what applies. For any kind of analytics or data transformation, Data Lake is a great place to dump raw data as-is, and it can also be used for many other things — for example, storing those same Parquet files as a database backing store.

## Hosting a website

If a site only has static content — HTML, JavaScript, CSS, images, all pre-rendered content sent to the browser with no server-side computation — it can be hosted via blob. At the storage account level, you can enable a static website, which shows you a URL and creates a `$web` folder to put your content in — that's basically it.

In the demo, under `Data management` → `Static website`, you enable it, and it shows the URL (plus a secondary endpoint if GRS is used), and lets you set the index name and error page. In `Containers`, a `$web` folder appears where content is uploaded — there are no true folders, but you can create subfolders (again just part of the blob name), and then you can browse to the URL and see the resulting website.

This is a very basic option, and you can put a vanity domain in front of it via a CNAME pointing at that DNS name. There's also **Azure Static Web Apps**, typically free or very cheap, which integrates with a content delivery network (CDN) — meaning that instead of the storage account living in a single region, the CDN has points of presence copying content all around the world, giving your client a better experience by talking to someone closer to them. Azure Static Web Apps is also pre-rendered content only, but it hooks in easily with managed Azure Functions if you want some server-side processing — realistically, using Azure Static Web Apps directly would be the better option.

## Access control options

There are different options for controlling access to talk to the data — not the control plane (regular Azure role-based access control), but the actual data plane in the storage account.

### Account keys

The first option is the two all-powerful storage account keys, which ideally you should not use. Under `Security + networking` → `Access keys`, you'll see two keys, both all-powerful for talking to the data plane — you can do anything and everything in the storage account with them. There are two keys, and there's also the option to rotate a key.

The reason there are two keys: if an application uses a key to talk to the storage account, rotating that key would break the application. Having two keys means that if you plan to rotate key 1, the app switches to key 2 first; once key 1 is rotated, you update the app to use key 1 again, and then in the future you can rotate key 2 without any interruption to clients. That gives you the ability to rotate keys without breaking anything, but there's no granularity here — you can do everything, and because it's just an access key, there's no good audit trail: audit logs just show that an access key was used, not which actual process or person did the thing.

> **Exam tip:** Because storage account keys have no granularity and provide no meaningful audit trail, they're not a good idea to use. You can set a rotation reminder if you must use one, ideally store it in Key Vault, and definitely never put it in code — that applies to everything discussed here. You can also use role-based access control on the secret storing the account key, but again, ideally you avoid using the key at all. You can also disable their use entirely.

In the demo, there's a rotation reminder option, and under the account's configuration you can actually disable storage account key access altogether, so those keys can't be used at all — this has implications for shared access signatures, discussed below.

### Blob anonymous access

There's also **Anonymous blob access** — you can disable read-anonymous access to your blob storage account. Even a read is an operation, so if someone spammed your storage account with reads, it would cost you money — you can block this from being possible. If it's enabled, at the container level you can additionally enable particular blob or list-content access; generally, the recommendation is no anonymous access, and you can enforce that with Azure Policy at scale.

### Entra ID integrated data plane RBAC

The better option is Entra ID integration for data plane role-based access control, now available for pretty much all of blob, queue, table, and files. For blob, you can even add the additional conditions discussed in the identity module. This is much better, since you can be far more granular about permissions.

In the demo, on `Access control (IAM)` for a container, you can set a role assignment scoped just to that container — roles like `Storage Blob Data Owner`, `Storage Blob Data Contributor`, `Storage Blob Data Reader` (equivalents exist for queue and table). Viewing a role like `Storage Blob Data Reader` shows both regular `actions` (control plane) and `dataActions` (data plane) — for example, the read blob permission on the data plane. Authenticating with an actual Entra ID account (rather than the access key) means permissions are based on that Entra-integrated authentication.

An additional benefit: if you're using managed identities — which are themselves Entra identities — a compute service can be granted data plane permissions for blob, queue, or table directly through its managed identity.

> **Exam tip:** The recommendation is to use Entra ID-integrated data plane RBAC instead of account keys wherever it's available (this applies broadly to many other database offerings too). For Azure Files specifically, you can use Kerberos via your own Active Directory Domain Services, Entra ID's Kerberos service, or, if you have no AD DS at all, Entra Domain Services — avoiding the powerful account key.

### Shared Access Signatures

If the all-powerful account key is too powerful, you can create shared access signatures (SAS) — a more granular set of permissions on a smaller scope.

- **Account SAS** — ad hoc, meaning there's no policy driving the configuration; you just set what you want, and it's at the account level, so it can apply to any of the service types that exist in the account.
- **Service SAS** — issued against a specific storage account service (blob, queue, or table), and can either be ad hoc or based on a policy; if you update the policy, the already-issued signature picks up those updates automatically.

> **Exam tip:** A key point is that all shared access signatures are signed by one of the account keys. If you disable account keys, SAS tokens can't be used. If you rotate the account key that signed one, the SAS is broken — this is effectively a way to revoke it, since it's signed and validated using that key.

In the demo, at the account level, under `Shared access signature`, you create a signature (account-level): choose the services, resource types, permissions, start and expiry times, allowed IPs, routing preference, and which key it's signed by (the key you'd have to rotate to revoke it), then generate it. At a specific container's `Shared access tokens`, there's also an option to use a delegation key for Entra ID-delegated access, though it's still signed by a certain key. You set permissions scoped to the container, possibly based on a policy — you can create multiple policies for different configurations, and can update a policy later to change what an already-issued signature effectively grants. You also set start/expiry and IP ranges, generate the token, and it's usable for that period of time. The generated URL includes the service, the specific container, and the signature itself, which carries all the access control detail. Because it's signed with key 1, rotating key 1 immediately invalidates that signature.

If you're using an ad hoc SAS, you can't revoke it except by rotating the key, so people tend to keep them short-lived, similar to entry tokens, because they're hard to revoke. A common architecture when you do need to use a SAS is the "valet key" pattern: a separate process creates the SAS, puts it, for example, in an Azure Key Vault secret, gives the requesting process permission to that secret, and something else periodically rotates or updates it — rather than giving a process direct ability to create signatures itself.

> **Exam tip:** The general recommendation is to avoid shared access signatures altogether where possible — data plane RBAC is a much better option. A signature tied to an account key can't easily be revoked without regenerating the key itself.

### Don't worry about key over TLS

One clarification worth making: you might look at a shared access signature and worry it's sent in plain text over the internet and anyone could grab it. That's not how TLS works — it's sent over `https`.

In reality, the client first makes a DNS request to work out the IP address of the storage account's host name, then establishes a TCP session with that IP address, then establishes a TLS session — still with just the IP address, no URL sent yet. Only after the TLS session is established, over that encrypted channel, does the actual URL (which contains the signature) get sent. So it's always sent over TLS encryption and is never sent in plain text over the internet. The recommendation is still to prefer data plane RBAC over SAS, but from a data-in-transit perspective, there's nothing to worry about here.

## Encryption

Data is always encrypted at rest — there's no option to turn this off. When creating a storage account, you can enable double encryption, which additionally uses infrastructure-level encryption. Normally it's 256-bit AES, FIPS 140-2 compliant, and infrastructure encryption adds another layer using a different algorithm and a different key, configurable when creating the storage account.

In the demo, under `Security + networking` → `Encryption`, `Infrastructure encryption` is shown disabled — enabling it adds a second layer, but only at creation time. When creating a storage account, you choose `Microsoft managed` or `Customer managed` for the encryption key. Customer managed means the key lives in your Key Vault, and you're responsible for its rotation; you can also turn on infrastructure encryption, adding another layer using a Microsoft-managed key — giving you double encryption.

You can also change the encryption key later, switching from Microsoft-managed to customer-managed by pointing to a key in your own Key Vault, with appropriate permissions configured. The key can even live in a Key Vault in a different tenant — useful, for example, in a scenario where you're hosting services for a customer: the customer keeps the key in their own Key Vault and could revoke it if they wanted to, while you use it to encrypt the storage account — a nice option in a SaaS-type scenario. With a customer-managed key, you also control how often to rotate it and can revoke it whenever you like.

> **Exam tip:** By default, a customer-managed key doesn't apply to queue and table — if you want it applied to those as well, you have to set that at account creation time.

### Encryption scopes

The encryption key discussed above applies to the entire account. But in a scenario with different customers in the same storage account, you can use encryption scopes — different configurations of encryption, applied per container or even per uploaded blob. It's not just one big key for everything.

In the demo, under `Encryption`, there's a list of `Encryption scopes` — one using a Microsoft-managed key (a different specific key), and another can be added using a customer-managed key with infrastructure encryption. When creating a new container, in advanced settings, you can specify that it should use a specific encryption scope (a different key) instead of the default. You can even do this at the blob level, at time of upload — specifying an encryption scope for that particular blob. This is really useful if a single storage account, at the container level or even the individual blob level, has different encryption requirements.

Encryption in transit can be enforced at the storage account level — while data is always encrypted at rest, you can require TLS as part of the storage account's configuration. In the demo, under a storage account's `Configuration`, there's an option `Secure transfer required`, which requires TLS when talking through the APIs and SMB3 when using Azure Files.

## Network protection

Nearly every Azure resource providing a service has a native firewall capability, and storage accounts are no exception. Standard features like private endpoint, service endpoint, and IP-based firewalls are all available.

The way this works: picture a storage account (Storage account 1) with its own built-in firewall. A virtual network is a private, non-internet-routable IP space, so IP-based rules in the firewall aren't directly useful for addresses inside the v-net.

- **Service endpoint** — enabled on a subnet (for example, subnet 2) for the `storage` service, optionally scoped to a certain region. This does two things: gives resources in that subnet a more optimal routing path to the storage account, and makes that subnet a known entity to storage accounts (within the region or globally). You can then add that subnet (v-net 1 / subnet 2) to a storage account's access configuration — everything in that subnet is then allowed through the firewall, while anything in other subnets or networks is blocked.
- **Private endpoint** — an IP address configured to talk to a specific instance of a service (for example, private endpoint 1 for storage account 1). The difference here is that anything able to resolve and route to that IP address can talk to the storage account — other things in the v-net, peered v-nets, v-net peering, ExpressRoute, or site-to-site VPN — as long as the right DNS name can be resolved (covered in the networking module) and certificate requirements for encryption are met.
- **Resource instance rules** — allow you to restrict access, at the storage account's firewall level, to specific instances of a supported resource type. For example, if a SQL database instance uses storage as a backup mechanism, you can allow just that SQL instance to talk to the storage account — a resource instance rule.

> **Exam tip:** Service endpoints also have service endpoint policies — not really a security feature of the storage account itself, but a way to prevent data exfiltration: you configure the subnet so it's only allowed to talk to storage account one, and can't talk to storage account two, preventing someone from copying data to a rogue storage account they shouldn't.

In the demo, under `Networking`, choosing `Enabled from specific networks` lets you pick a particular resource type and then a particular instance of it — a nice way to lock things down.

## Lifecycle management

Over time, especially in data lakes, massive amounts of data can accumulate from many systems, and you pay for storing it. Access tiers help optimize cost — pay less for capacity, more for interactions — but manually moving files between tiers is a lot of work, usually based on some period of time (creation time, last modified time, or last access time, which requires enabling access tracking).

Life cycle management is a capability to automate this, but it can only do what the type of object supports — for example, premium storage has no tiers, so all you can do is delete after a certain number of days; append blobs also don't support tiering, so only deletion is possible.

Life cycle management lets you define rules that automatically move data between tiers or delete it after a period of time, with filters to limit scope by name, based on creation date, modified date, or access date.

In the demo, under `Data management` → `Lifecycle management`, a rule is created: move to `cool` if the data hasn't been accessed for 15 days, move to `cold` after 45 days, move to `archive` after 135 days. No deletion rule is included in this example, but it's also possible to add one, based on modified, created, or access date. These rules operate within a given storage account.

## Azure Storage Actions

**Azure Storage Actions** takes this a step further than classic life cycle management. It supports block, page, and append blobs, for both flat namespace and hierarchical (Data Lake) namespace, has everything life cycle management has and more — you can set immutability, blob tags, undelete objects, and much more. If you try to use a feature that isn't supported for that type of blob, it just doesn't take effect. You can build much more complex conditions, including group clauses and wildcards (`*` for multiple characters, `?` for a single character).

The model: you centrally create tasks, each made up of a set of "if... then" conditions, and then you assign the task to storage accounts — for example, storage account 1 and storage account 2.

> **Exam tip:** A key advantage is that assignments can be applied far beyond a single subscription, across an entire tenant — making this significantly more powerful than classic life cycle management at the individual storage account level.

In the demo, under `Access control (IAM)`, a role assignment such as `Storage Blob Data Owner` is granted to a task (in the example, called `image task`) — the task has its own identity that needs data plane RBAC permissions on any storage account it's meant to work on. Under `Storage account actions`, a task called `image task` is shown, defining conditions — for example, blob name ending with a certain string, starting with, empty, equals, matches, creation time, and many other clauses and available actions (setting tiers, legal holds, immutability policies, undelete, and more). The task is then assigned wherever needed within the same tenant.

There's a separate charge for storage tasks (once out of preview/GA), but the potential savings from managing these things at scale can be significant.

## Native protection constructs

The first one is snapshots — a point-in-time, read-only view of a blob or file share, using incremental storage (you only pay for the changes, not the entire content). It's covered here mostly because it's been largely superseded by a better combination of features.

### Blob versioning

Recall that a blob is essentially a chain of blocks joined together. If a block blob consists of a chain of blocks of various sizes, and then one of the blocks (say the middle one) is changed, the versioning capability treats the original chain as V1 and the new chain as V2 — you get versioning, and it only stores the changed blocks (again of various sizes).

In the demo, under `Data management` → `Data protection`, `Versioning` is shown enabled, along with `Blob change feed` and `Soft delete` for blobs and containers. Since blob versions accumulate over time, life cycle management can be used to clean them up.

### Change feed

Change feed, as the name suggests, is a log of changes — a record of what changed, as it happens. It's implemented using a special append blob (visible as `$blobchangefeed`) in Apache Avro format. You can configure deletion after a certain amount of time. It lets you track at any point in time when a change happened.

### Soft delete

Soft delete means that when something is deleted, it's actually kept for a configured number of days (between 1 and 365), settable at the blob and container level. You do pay for this — it still has to be stored — but you can undelete it. If blob versioning is enabled, you get a soft-deleted version of the blob, retaining all the associated capabilities.

### Point-in-time restore

Combining versioning, change feed, and soft delete gives you point-in-time restore, which is superior to snapshots. Since you know all the versions of something and when each change happened, and deleted items are retained rather than removed, you can restore to essentially any point in time within your retention window (limited by whichever of these three settings has the shortest window). Snapshots are no longer needed — you can go back to any point in time you want.

## Azure File Sync

Most organizations still use Windows file shares — regular SMB running on a file server, and possibly multiple file servers around the organization. There's a Windows feature called Distributed File System Replication to keep instances in sync, but you can also use Azure File Sync, which uses an Azure file share as a cloud endpoint as part of what's called a sync group.

> **Exam tip:** Up to 100 servers, like Windows File servers with a share, can become part of a single sync group.

The order of operations: create a file share in Azure (the cloud side); install an agent on each file server for Azure File Sync, which registers the server so it shows up as an object the Azure File Sync service knows about; put those into a sync group. After that, the servers replicate data via the file share (they don't talk directly to each other), in both directions. This model isn't designed to handle concurrent changes on all servers at once (similar to a merge conflict in git) — it works well when one server is the primary being written to, and it can keep up to 100 servers in sync. Because it integrates with Kerberos authentication, you can also talk to the file share directly (subject to your permissions) — useful as a backup access point if there's a problem.

Because these servers usually have finite local storage, cloud tiering is available: on the local server, a policy can be set to tier off the least-used data, based on a percentage of free space remaining (e.g., once down to 20% free) or based on a number of days, or a combination. It keeps a "thumbnail" of the data locally, so a client sees a shortcut to it, but the content isn't actually stored locally — accessing it pulls it down from the file share and repopulates it locally.

> **Exam tip:** An important nuance: Windows has a convenient USN journaling capability, making it easy to know when something changes. There's no such concept in Azure Files, nor is there a change notification capability. Instead, there's a periodically run change detection job that lists the share to find changes and sends them down. That means originating a change locally could introduce a bit of delay before it's replicated to Azure — worth bearing in mind when planning.

## Azure Elastic SAN

A newer service, **Azure Elastic SAN**, is a block storage solution — "block" meaning the connecting client sees a disk and is responsible for managing the file system on top of it, compared to file-based storage where the client just sees files and folders through an API. It uses iSCSI, which communicates over the network, so it doesn't require special virtual hardware the way Fibre Channel would, and iSCSI is very widely supported by clients. A key use case for Azure Elastic SAN is Azure VMware Solution, where you want external storage rather than just the virtual SAN.

> **Exam tip:** Azure Container Storage is built on regular Azure Storage — it's not a separate storage appliance Microsoft acquired, but a service running on top of Azure Storage; when creating an instance, you can pick `LRS` or `ZRS`.

In the demo, when creating an elastic SAN, you pick a region like anything else in Azure, and choose redundancy (`LRS` or `ZRS`). You see the idea of base units, which come with an associated amount of IOPS and throughput, and capacity units, which don't have associated IOPS but add capacity — giving you two building blocks to work with. You create volume groups and volumes, and connect to them via iSCSI.

The structure: your Azure Elastic SAN is built on Azure Storage, redundancy is `LRS` or `ZRS`. You buy a certain number of base units (which give capacity plus an associated amount of IOPS and throughput) and optionally capacity units (just more capacity, no additional performance). You create one or more volume groups, which define things like network connectivity (which v-net it integrates with, whether private endpoints are needed) and security (platform-managed or customer-managed key) — essentially a network and security boundary. Within a volume group you create as many volumes as needed — each with a chosen capacity, with associated IOPS and throughput driven by that capacity. Clients connect to these volumes via iSCSI.

Because it's iSCSI, multiple servers can talk to the same volume — for Windows, this could support cluster shared volumes, for example. It's a very cost-efficient type of storage that can be leveraged by many different workloads.

> **Exam tip:** Because traffic runs over the network (iSCSI), it counts against the VM's network performance, not its storage performance — an important planning consideration.

## Azure NetApp Files

As mentioned, Azure generally doesn't use SANs — with one exception. **Azure NetApp Files** actually has NetApp filers sitting in Azure data centers, as a result of a partnership; it's available via a resource provider through ARM. This is a good option when you need management consistency with an on-premises NetApp deployment, or need very high performance that's hard to achieve with regular Azure Storage, with different performance tiers depending on your needs.

The structure: you create an Azure NetApp Files account, which exists in a certain region. Within it you create one or more capacity pools, each with a certain amount of capacity and a service level:

- `Standard` — 16 MB/s per TB.
- `Premium` — 64 MB/s per TB.
- `Ultra` — 128 MB/s per TB.

There's no particular IOPS limit — performance is tied to the capacity provisioned.

There's also a **cool access** feature: cold data (based on a configurable number of days since last access, roughly 2–183 days) is tracked, then packaged into 4 MB objects and moved into an Azure storage account, because it's cheaper there — this is called cool access, and it's controllable, including whether it's pulled back on access.

You pay for the capacity provisioned. Volumes can be dynamically increased and decreased; the smallest volume is 50 GiB. From the capacity pool you create volumes, each with a size and protocol (SMB, NFS, or dual protocol). A regular volume ranges from 50 GiB up to 100 TiB, and there's also a "large volume" option, up to around 1 or 2 PiB under special conditions. Because these numbers change frequently (minimums going down, maximums going up), it's worth checking current documentation. Each volume connects through a delegated subnet in a virtual network. Encryption in transit is supported for NFS via Kerberos integration, and data is always encrypted at rest.

Cross-region replication is supported, with more pairing flexibility than the standard Azure regional pairings — beyond the standard pairings, there are non-standard pairings you can choose based on your particular functional needs.

## Managed Disks

When talking about virtual machines and other services, the concept of managed disks matters a lot — this abstracts away the storage account. In the old days, a virtual machine would actually use a storage account (with its own IOPS and throughput limits) and create a page blob, which is what the VM used. Managed disks abstract that away — there's still a storage account and a page blob under the covers, but you don't see it (except in an import/export scenario using a shared access signature, where you'd see the storage account name) — it's completely managed for you.

What you see now is a managed disk, which is used by virtual machines, and then AKS uses virtual machine scale sets and other services that build on top of it, so many things end up using this concept without you having to worry about the storage account at all.

Disks and snapshots (point-in-time, delta-based storage) are first-class ARM resources. There are many SKUs, with performance increasing and latency decreasing as you move up the offerings, from standard hard disk drive upward:

- **Standard HDD**
- **Standard SSD**
- **Premium SSD v1**

> **Exam tip:** For these three, you always pay for the provisioned capacity, never the capacity used — the capacity is driven by the size of the disk, which is what determines the associated IOPS and throughput. You can increase disk size, but you can't shrink a disk.

Another capability with Premium SSD v1 is that you can change the performance tier separately from the capacity — for example, if you need higher performance for a shorter period of time, you can increase the performance tier without growing the disk itself: you'll pay for the higher size, but the disk stays the same size, while getting performance characteristics associated with a bigger SKU.

In the demo, on the Premium SSD pricing page, you pay for a disk size and get associated performance characteristics that go up as the disk gets bigger — so you might want a 512 GB disk but need the performance of a P40, in which case you can request P40 performance characteristics without the disk actually growing to 2 TB; it stays at 512 GB but you get the higher performance.

You also get bursting — the ability to burst up to higher performance for a limited amount of time.

> **Exam tip:** For some tiers (e.g., P20), you get a free bucket of burst capability; for P30 and above, you have to pay for burst capacity. Standard SSD at 1 TB and below also has free bursting.

Then there's:

- **Premium SSD v2**
- **Ultra Disk**

For both of these, you pay separately for capacity, the amount of IOPS you want, and the amount of throughput you want, and these can all be changed dynamically — giving much more flexibility to turn the dials as needed.

> **Exam tip:** Latency drops as you move through these tiers — Premium SSD is sub-millisecond, Ultra Disk is roughly sub-half-millisecond. None of these managed disk types support geo-redundancy (`GRS`) at all. Specifically for Premium SSD v2 and Ultra Disk, only `LRS` is available — not even `ZRS`, because the latency between availability zones would start to jeopardize the disk's performance.

Pricing is based on provisioned, not consumed, capacity. You can dynamically increase data disk size (except for Ultra and Premium V2 — always double-check current capabilities, since these change). If you increase disk size, you'd also need to grow the corresponding volume within the operating system. There's no GRS option; Premium SSD v2 and Ultra only support LRS. If you're using availability sets (hopefully you're moving toward availability zones instead), you can align managed disks to different storage clusters based on the VM's fault domain, as covered in the resiliency module.

> **Exam tip:** All the SSD and Ultra disk types have a Max Shares property, enabling multiple VMs to connect to the same managed disk — useful for SCSI persistent-reservation cluster scenarios in Windows or Linux clustering, where multiple machines share the same disk.

You can use your own encryption key via a disk encryption set: with a key in your Azure Key Vault, you create a disk encryption set that uses that customer-managed key, and place managed disks into that set so they use the key to unwrap the encryption material used to access the bits on the disk. There's also a host encryption capability, which encrypts local cache, temp disks, and data in transit between the host the VM is on and the underlying storage. You can also use a cross-tenant customer-managed key — the key can live in a Key Vault in a different tenant from where the managed disk itself lives, useful in that same SaaS-provider scenario.

## VM Storage

When a virtual machine starts, it's placed on a certain host in a certain data center, using a hypervisor, and gets associated CPU and memory. The VM has an operating system disk that's normally durable — a managed disk on a hidden storage account.

For some workloads, you don't actually care about the OS disk — maybe VMs are created and deleted as needed, such as a virtual machine scale set where instances are stood up and torn down as the workload changes and there's nothing special about them — they're expendable. In that case, you can use ephemeral storage, provided the VM has enough cache or temp space, for its OS disk instead of a managed disk. That saves you the cost of the managed disk and gives you very high performance.

The host has some local storage of its own (SSD); when a VM starts on a given host, there's essentially no latency between them, so using this for the OS disk is very high performance and very low latency. This is especially valuable for workloads like VM scale set-based clusters (for example, an AKS auto-scaling node), where you don't care about OS state — ephemeral storage saves money on the managed disk, creates faster, and gets you higher performance.

Most VMs also have a temporary drive, typically `D:` on Windows or `/dev/sdb1` on Linux by default, which is non-persistent. Windows puts the page file on it by default; you can also use it as a scratch drive, but don't put anything you care about on it — it can disappear at any time, since it's created and deleted every time you start or stop the VM. If you need a specific structure on it (e.g., SQL using it for temp DB), you'd need a script running to recreate that structure. Some VM SKUs have local NVMe storage as well, super high performance but again not durable — the L-series VMs, for example, and some newer VMs can use NVMe directly instead of SCSI for even higher performance, sometimes usable for the temp drive too.

> **Exam tip:** An important point about storage planning: the disk has its own performance characteristics — a certain amount of IOPS and throughput. You could attach many disks to a virtual machine to increase performance, but the VM itself also has its own storage performance characteristics — the number of IOPS and throughput it supports — so simply throwing more disks at it can hit a point where the VM itself becomes the bottleneck, not the disks.

In the demo, VM naming conventions are shown (a lowercase "d" indicating local disk), and a particular VM's characteristics show local storage (or lack thereof) and remote storage performance limits. If you keep adding disks, at some point the VM becomes the bottleneck, not the disk itself, so you need to balance both.

VMs also support a certain number of NICs and a certain amount of network bandwidth. This matters for storage too — talking to an SMB or NFS file share, or to iSCSI, goes over the network connection, not the storage performance path. So make sure to account for network performance if you're doing a lot of SMB, NFS, or iSCSI traffic. There are also VM throughput limits and some VMs have boost capabilities that let them burst beyond their normal limits.

## Handling big volumes

This is much less likely today given the capabilities of SSD V2 and Ultra Disk, but some of the biggest VMs can have up to 64 disks attached.

> **Exam tip:** Within the guest OS (e.g., Windows), use Storage Spaces to combine disks rather than classic RAID sets, which can be unreliable — you generally don't need mirroring or parity in this scenario, since there are always three copies of the underlying disk anyway. Some systems, like SQL Server, already use file groups to manage multiple attached disks, and multiple disks can give higher IOPS and throughput than a single disk — but with Premium SSD V2 and Ultra Disk, there's rarely a compelling use case for this level of complexity anymore.

## Storage tools

**Azure Storage Explorer** is a free tool you can download — supports authentication via account, account key, or shared access signature, and works with all the different capabilities.

In the demo, Storage Explorer shows a subscription with all its storage accounts and disks, allowing you to browse around and perform many operations, including right-clicking to create shared access signatures — very similar to what's shown in the portal.

In the portal itself, `Storage browser` is a web version of similar functionality — browsing containers, viewing data, working with file shares and queues — a nice way to interact with these capabilities without going directly to an API.

One of the really great things it can do is server-side copying: normally, if you're a client copying data, you download it from Azure and then upload it back to Azure (hairpinning through your client). With server-side copying, that doesn't happen — the copy happens directly between storage accounts, giving very high performance without your client being in the path.

### AzCopy

**AzCopy** is a free command-line utility that also uses a server-to-server API, avoiding that hairpinning. Depending on the number of CPUs on your machine, you can adjust concurrency — how many operations happen at once — to speed things up. Jobs let you track the status of operations. It's great for migrating data — not only from on-premises into Azure, but also from AWS S3 buckets or Google Cloud Storage buckets into Azure Storage (not the other way around). There's also a sync mode that detects changes in local storage and keeps things in sync.

There are also other data-copying solutions like Azure Data Factory, which is very powerful, but overall AzCopy is a great way to move a lot of data into Azure from another cloud and keep things in sync.

### Azure Storage Mover

**Azure Storage Mover** helps you migrate from on-premises shares — SMB or NFS — into Azure. For SMB 2.0 and above, it can migrate into an Azure file share; for NFS 3 and 4, it migrates into an Azure blob container. It's a fully managed migration: an agent (essentially a VM appliance) is deployed on your network with line of sight to the source, and it only needs to be able to talk to Azure over port 443.

It preserves as much as possible about the source data: for SMB, full fidelity — timestamps, permissions, everything; for NFS, an empty folder is represented as an empty blob, with the metadata of that empty folder stored as custom metadata on the blob.

You can also use **Azure Data Box**, an offline tool for cases where your network connection isn't good enough to move huge amounts of data into Azure. Since Data Box takes a couple of days to process and data may have changed since then, Storage Mover can then replicate whatever changed since that time.

## Import and Export

If you have a lot of data to get into (or out of) Azure, there are a few options:

- **Azure Import/Export** — uses your own 2.5-inch SSD disks or USB drives. A single job can include up to 10 disks; you put your data on them, ship them to Azure, and Microsoft copies the data into your storage account. Data is encrypted with BitLocker.
- **Azure Data Box Disk** — Microsoft provides the disks: between one and five 8 TB encrypted SSDs, connected via USB 3, that you copy your data onto.
- **Azure Data Box** — for much bigger jobs, this is an appliance that comes in various sizes. The original solution was a 4U appliance with about 80 TB of usable space; there was also a Data Box Heavy, a roughly 500-pound unit shipped as freight, with about 770 TB of usable space. There's now a new generation with 120 TB and 525 TB versions. This is an offline solution delivered to your data center — you copy your data onto it, ship it back, and Microsoft brings it into your storage account. The exact capabilities depend on the source and target.

## Data Governance

Not an Azure Storage feature specifically, but data governance is critically important — for most organizations, data is the single most valuable asset they have. As companies grow, they accumulate more systems and more data, often without a good handle on where sensitive data (like PII) is, how it got into the systems, or where it's gone since. That raises a lot of questions: how do you find your data, how do you classify it (potentially with your own custom classifications), how do you apply labels, how do you use data loss prevention, how do you use rights management, how do you delete things you don't need to keep past a certain age, and much more.

**Rights Management** is about controlling who can access sensitive data and how they can use it — through encryption, permissions, expiration, and so on. **Data Loss Prevention** is about monitoring and preventing sensitive data from being shared in ways you don't want — restricting emails and content, alerting, and potentially blocking, based on classifying and controlling data by its sensitivity.

**Microsoft Purview** is the data governance solution for all of this — across your entire data estate, not just Azure. Purview is available at purview.microsoft.com; from there you can run discovery, identify sensitive data, and apply sensitivity labels, which can then drive encryption, rights management, visual markings, and more. There are hundreds of built-in classifications, or you can create your own custom classifications, for example based on a regular expression. There's a unified catalog to make data easy to discover and access, and policy to help control access at scale. Given how much focus AI is putting on organizational data today, Purview is very much the solution for this space.

## Close

That's a wrap for this module — lots covered here. Hope it's useful; until next time, take care.
