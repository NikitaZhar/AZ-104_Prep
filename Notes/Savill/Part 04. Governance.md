2026-07-02 16:42
Tags: #

## Introduction

Welcome to the governance module of the Azure Masterclass V3. In the last module we talked about identity as one of the first things you ever need to think about when adopting the cloud. We're now going to build on that by looking at governance. We'll explore the importance of having guardrails because of changes in the process of how application owners actually go and create resources, understand some of the core constructs — specifically Management Groups, subscriptions and resource groups — which are where we apply the various types of guardrails available, specifically policy, role-based access control and budgets. Then we'll look at some more general best practices, standards and architectural guidance. That's the goal of this module.

## Governance 101

Governance in the cloud is absolutely critical because the behavior in the cloud is very different compared to what we have on-premises.

Consider an application owner. On-premises, we have a certain amount of resource available — some backend servers — and for most organizations, if the app owner wants to create some resource, there is an operations team. The app owner sends a request outlining what they want — a VM, a container, a specific operating system — maybe via a form, maybe via email, or the old-school way of picking up the phone or knocking on a door. Whatever the mechanism, there are organizational guidelines and requirements that must be adhered to, and it's the operations person's responsibility to look at the request, consult those rules, possibly modify the request, drive the type of resource created, and then provision that specific resource. The point at which the requirements of the organization are enforced is that operations person standing between the app owner and the ultimate provisioning of the resource, making sure things are done the right way.

Now consider the cloud scenario — this applies to any cloud, not just Azure. The app owner now has direct access and provisions resources directly against the cloud. There is no operations person standing in between. Provisioning might happen through the portal, through a CLI, through infrastructure as code, through a pipeline as part of DevOps — whatever the mechanism, the app owner creates it directly, with no human in between that part of the chain.

This isn't necessarily a good thing, because not every app owner is going to know all the particular requirements they have to adhere to — company policies, or regulatory requirements by industry or country. Those requirements still need to be enforced, so we need the ability to have governance to control and make sure app owners are operating within the standards the organization has. Governance needs to protect no matter how someone interacts with the cloud — that's the key point. This covers the amount of resources created, the types of resources created, and the configurations on those resources.

By default, the cloud offers a huge number of different services, and a lot of them could be misused in negative ways — for example, creating public-facing resources where people don't know what they're doing and might expose things that shouldn't be exposed, leading to data leaks or creating a large attack surface. Governance is about enforcing the rules and ensuring that all the standards the organization needs to adhere to are actually being enforced, no matter how the app owner goes about creating resources.

## Understanding requirements

Before designing any solution, you first have to understand your requirements — what you're actually trying to meet. As an organization, you will have standards, which could be based on internal policies, or driven by outside standards such as regulatory compliance — regulatory requirements tied to a country or an industry.

For Azure, as discussed in the foundational module, there are shared responsibilities: some things the provider (Microsoft) is responsible for, and some things the customer is responsible for. Generally, the more you move from infrastructure as a service to platform as a service to software as a service, the fewer responsibilities you have as a customer — but you still have some responsibilities.

Microsoft holds a massive number of compliance certifications and provides a lot of detail about those compliance standards through the Microsoft Trust Center. From the Trust Center, you can navigate to **Compliance**, and from there to **Compliance offerings**, which shows the different standards that apply to various services. This can be broken down by country and by particular types of service, such as Microsoft Azure or Dynamics offerings. On the main compliance page you can also see accessibility reports, regional compliance reports, Azure solution reports, and audit reports. The **Service Trust Portal** provides the full list of compliance attestations from the various inspections Microsoft has undergone.

## Compliance manager in Purview

To help track compliance status, there's the Compliance Manager, which is now part of Microsoft Purview. This requires an Office or Microsoft 365 license, or, for USG or DoD cloud, there are some special assessments that provide additional regulatory comparisons.

In the Compliance Manager portal, you get a compliance score along with key improvement actions. From **Assessments**, you can see your current assessments (and add new ones) — for example, a Data Protection Baseline assessment shows your progress, split between points that are the customer's responsibility and points that are Microsoft's responsibility under the shared responsibility model (Microsoft typically scores much better on their portion).

From there you can look at all the various controls being assessed, broken down into different areas, with detail on Microsoft's actions and their pass/fail scores, as well as your own improvement actions and the points you've achieved. You can select these items, assign an owner to track who is responsible, and edit implementation details — for example, marking whether something is implemented, out of scope, tested, or passed, and adding notes. This makes Purview a great mechanism for tracking exactly how you're doing against your compliance requirements.

## Mitigating risk

The key point about all these governance capabilities is that they are primarily about mitigating risk — risk of data loss, risk of infiltration, risk of service interruption, risk of reputation damage, and risk of being fired for a poorly implemented environment. They're all about mitigating risk.

## Key organizational components

There are key organizational components that play a large role in the overall governance approach.

### Management groups

Management groups help form a hierarchy under which Azure subscriptions — the real building block for creating everything — eventually sit.

Every Azure subscription trusts a specific Entra tenant, which is the directory that houses the various security principals used within that subscription. Under an organization's Entra tenant, the first key building block is the management groups. Every Entra tenant has a Root Management Group — one and only one. You cannot move the Root Management Group anywhere else. Under the Root Management Group, you create a hierarchy of however many management groups you want, to organize your subscriptions and apply the types of governance you need. This hierarchy can go up to six levels deep.

> **Exam tip:** Every Entra tenant has exactly one Root Management Group, which cannot be moved or relocated. The management group hierarchy can go up to six levels deep, and a tenant can have up to 10,000 management groups. A management group can only have one parent, but a parent can have many children.

### Entra GA Azure resource elevation

Entra permissions have a say over Azure permissions. A Global Administrator in Entra has a property under **Overview → Properties** called **Access management for Azure resources**. This is normally set to "No," but if set to "Yes," it grants the account a special permission: it can manage all Azure subscriptions and management groups in the tenant. Looking at the access control (IAM) of any subscription that trusts the tenant, you'll see the role **User Access Administrator** inherited from the Root Management Group.

> **Exam tip:** This elevation isn't something you should normally leave enabled, but it's a valuable "break glass" capability. For example, if a subscription owner leaves the company and everyone is locked out, a Global Administrator for the Entra tenant can temporarily elevate themselves to grant access and resolve the problem.

### Organizing management groups

You cannot rename the ID of the Root Management Group, but you can change its display name. All subscriptions in the tenant must trust that Entra tenant to be part of the hierarchy.

Management group design is intended to make governance easier at scale. Broad policies or permissions that must apply everywhere should be applied at the root or high up in the hierarchy; more specific requirements, tailored to a particular environment or geography, should be applied lower down. It's common to organize management groups by geography (if policies differ by region), by business unit (if different business units require different permissions), or by environment (for example, different policies for dev/test versus production).

Only the subscription owner can change a subscription's place in the management group hierarchy — subscriptions can be moved around, but that action requires subscription owner permissions.

For example, in the portal's **Management groups** view, a tenant root exists, and by default a subscription that trusts the tenant rolls up automatically to the tenant root. From there it can be moved into child management groups — for instance, a group containing all of an organization's subscriptions, with further child groups for Development and Production, each containing relevant subscriptions.

## Subscriptions

A subscription is the starting unit for creating anything in Azure — it's the basic unit for billing and management, and it has certain security elements as well. A subscription trusts a specific Entra tenant, which places it under that tenant's Root Management Group, from where it can be moved.

You *can* change the Entra tenant a subscription trusts for identity, but doing so removes all role-based access control assignments and managed identities on that subscription — it will not change the billing status, but it's a very disruptive operation, since security and permissions are core to the resources within it. This is generally something you never want to do.

As the Entra administrator, you can control whether subscriptions are allowed to join your tenant, or whether subscriptions that trust your tenant are allowed to leave it.

### Controlling subscription policies

In the Azure portal, under **Subscriptions → Advanced options**, there is a **Manage policies** menu, where you can control whether subscriptions can leave the Entra tenant, and who can add a subscription to the tenant. Options range from "permit everyone" to "permit no one," with the ability to add exempted users who are still permitted even under a "permit no one" setting.

Once a subscription's place in the hierarchy is established, you can have many subscriptions under a particular management group — not just one. Good naming standards for subscriptions matter, and will be covered later.

### Azure limits

There are limits on everything in Azure, documented on the Azure subscription limits page. This includes management group limits (up to 10,000 management groups per tenant, up to six levels below the root), subscription limits (an unlimited number of subscriptions per Entra tenant, up to 980 resource groups per subscription, tag limits, subscription-level deployment limits), resource group limits, template limits, and Entra ID limits.

> **Exam tip:** If you're ever curious about what you can do with a particular service, check the Azure subscription and service limits page.

### How many subscriptions?

There used to be guidance to keep the number of subscriptions very small, adding one only when necessary. That guidance has loosened considerably: create subscriptions as makes sense for the business — for example, one subscription per workload, where a workload is a complete unit of related resources. A subscription is a boundary for certain things (a virtual network cannot span subscriptions, for example), and other factors like Service Health alerts can be hard to interpret if many unrelated apps share a subscription, since the alert doesn't always give clear specifics about which resource it relates to. There are also certain limits at the subscription level that get consumed faster when many unrelated workloads share a subscription.

With management groups available, it becomes much easier to be looser about how many subscriptions you have, as long as the management group structure is logical and subscriptions are placed correctly in the hierarchy — they will still receive the right policies and RBAC through inheritance.

## Resource groups

A resource group is what's actually used to deploy a resource into. Every Azure resource lives in one and only one resource group, and a resource group lives in a specific subscription. Resource groups are not nested — you cannot put a resource group inside another resource group; it's all flat within a subscription.

Resources that share a common lifecycle — created together, run together, and likely decommissioned together — should be grouped into the same resource group. This also allows policy and RBAC to be applied at that resource group level, which is convenient because resources with a common lifecycle often need similar permissions and similar policies applied.

> **Exam tip:** A resource group is not a boundary for resources that need to work together. Resources in different resource groups can still interact — for example, an AKS cluster's nodes can connect to a virtual network that lives in a completely different resource group. It's very common for networking resources to sit in a separate resource group, managed by a separate networking team with different permission requirements.

A resource group has a region specified when created, but this is just where its metadata lives — resources from any region can exist within the same resource group.

### Moving resources

Resources can be moved between resource groups, and even between subscriptions or to another region, via the **Move** option in the portal. Moving to another region triggers a validation to check whether the move is practical. Moving resources between resource groups within a subscription is generally the easiest operation; moving between subscriptions or regions varies depending on the specific resource.

A resource group and everything within it can be deleted via the portal. A resource group cannot be renamed, since the name forms a core part of the identity of the resources within it — to rename effectively, you would need to create a new resource group and move the resources into it.

Management groups, subscriptions and resource groups are the primary places to apply governance. Broad governance should be applied at the root and top-level management groups; more specific governance at the subscription level; and very specific governance at the resource group level. You *can* apply governance directly to individual resources, but this is generally avoided due to limits on how many role assignments and similar constructs can exist, and because it becomes impractical to manage at scale. If resources have been grouped well into resource groups, there's typically no need to apply governance at the individual resource level.

## Naming standards

Having a naming standard is important. The Azure Cloud Adoption Framework has good recommendations on developing a naming and tagging strategy for Azure resources, including a recommended naming convention broken down into: resource type, workload/application, environment (e.g., dev or non-prod), Azure region (or on-premises data center), and instance number.

> **Exam tip:** Aim for a naming convention that is consistent across on-premises and cloud environments, and extend it even to guest OS names, so that you can quickly and intuitively identify a resource's region, workload, function, type, and instance from its name alone.

## Tags

There's only so much information that can fit in a resource's name. Tagging provides a name/value pair mechanism available for almost every type of resource in Azure — including subscriptions and resource groups, but *not* management groups.

Tags can be applied at various levels: resource group, subscription, or an individual resource. There are recommended minimum tags for an environment, such as business criticality, business unit, operations team, app owner, cost center, environment, Azure region, and owner name.

> **Exam tip:** A good tagging strategy — including a tag for the owner — helps identify who is using a resource, avoiding the risk of deleting a resource that appears abandoned but is actually in active use.

It's also important to standardize tag *values*, not just tag keys — for example, consistently using "prod" rather than a mix of "prod," "production," and "live" — otherwise the tags become hard to use effectively.

> **Exam tip:** You can enforce tag usage through policy, but be careful — enforcing mandatory tags via a deny policy can block automation that doesn't have the ability to add tags, which can be painful.

Tags are heavily used for filtering views and for cost/billing purposes. A tag's value can even be a JSON document, allowing richer content beyond the number-of-tags limit — for example, malware or antimalware version details, or firewall configuration information, which code can then inspect.

> **Exam tip:** Tags are not inherited by default. If you apply a tag at a subscription or resource group, it is not automatically applied to the resources within it — unlike policy or role assignments, which are inherited.

However, inheritance for tags can be achieved. Azure Policy includes built-in policies (searchable under category "Tags" with the keyword "inherit") that let you inherit a tag from the resource group if it's missing on a resource, or inherit it from the subscription if missing, with the option to overwrite existing values or leave them if already present.

Separately, for cost management purposes specifically, you can enable tag inheritance so that billing reports inherit a tag's value from the resource group or subscription — this does not actually copy the tag onto the resource itself, but it is a powerful cost management capability, useful for grouping billing by application, for example.

## Types of governance

Across management groups, subscriptions, and resource groups, a number of governance mechanisms can be applied.

### Inheritance

> **Exam tip:** Inheritance is fundamental to everything in Azure governance. Whatever you apply — policy, permissions, or otherwise — at a given scope is automatically inherited by every child scope: if applied at the root, every management group, subscription, resource group, and resource inherits it; if applied at a subscription, everything within that subscription and its resource groups inherits it. Be careful where you apply things, since it affects not just the chosen scope but all child scopes as well.

If a budget is set at a given scope, spending rolls up to that scope — for example, a budget set at the subscription level aggregates the spend of all resources in all resource groups under it.

### Who, what and how much

Role-based access control (RBAC) governs *who* can do certain actions at a specific scope. Policy governs *what* can be done. Budget governs *how much* — which can be a financial spend limit, or limits on the number of a certain type of resource, and can also use tags and other dimensions for showback/chargeback purposes.

### Locks

Locking can be applied at the subscription, resource group, or resource level.

> **Exam tip:** A lock operates at the control plane of Azure only — it does not stop data-plane actions against a resource. For example, locking a storage account can prevent deleting or changing the attributes of the storage account itself, but you could still delete data (such as a blob) within it.

When creating a lock, you choose between:
- **Read-only** — attributes of the resource cannot be changed, and it cannot be deleted.
- **Cannot delete** — attributes can still be changed, but the resource cannot be deleted.

Locks are inherited: applying a lock at the subscription level applies it to everything within the subscription; the same applies at the resource group or individual resource level. You need owner permission at the scope the lock is applied to in order to remove it. Some services (such as Azure Backup) add locks automatically. Locks are useful for protecting against accidental deletions, but they don't provide immutability of the content within a resource.

## ARM and resource structure

Understanding the structure of resources is important, particularly because Azure Policy is built entirely around resource attributes. Azure is made up of many different resource providers, and within a resource provider there are different resource types (for example, the Compute resource provider includes virtual machines, virtual machine scale sets, and managed disks). Each resource type has different properties and different actions that can apply to it.

This structure can be explored via the Azure Resource Explorer, which lists resources and resource providers, or via a resource's **JSON view** in the portal, which shows the underlying resource type — for example, `Microsoft.Storage/storageAccounts`. This is exactly the same resource type identifier used in infrastructure-as-code templates such as Bicep files, along with a specific API version.

### Actions available on resources

You can query, for example via Azure CLI (`az role`), all of the applicable actions that exist for a given resource type in a given resource provider — for a virtual machine this might include actions like start, power off, and reapply. This is useful when building custom roles, to understand exactly what actions are available to leverage. The exact set of actions varies widely depending on the resource type.

## Role Based Access Control

RBAC is supported at all scopes — management groups, subscriptions, resource groups, and resources — controlling the actions various identities can take at that scope, with inheritance down to all child scopes.

> **Exam tip:** RBAC at the individual resource level is generally avoided in practice; stick to resource group level or above where possible. There is a limit of around 4,000 role assignments per subscription, so creating many per-resource assignments risks hitting that limit. Grouping resources logically into resource groups allows RBAC to be applied at that minimum practical level.

RBAC is always inherited, and there is no way to block inheritance — you cannot prevent a resource group from inheriting a permission set at the subscription level, and there is no explicit "deny" permission in native RBAC (outside of deployment stacks, discussed later). The rationale is that if a permission is set at a higher scope, it's deliberately intended to apply to everything beneath it; if a lower-scope owner could override it, that would undermine the purpose of the higher-scope assignment.

### Role assignments

A **role** is a grouping of actions that can be taken on resources. Rather than assigning individual permissions one at a time (which becomes unmanageable at scale), common sets of actions are grouped into roles, and roles are granted to security principals — ideally groups rather than individual identities (users, managed identities, or service principals), since individual assignments become messy to manage over time.

Granting a role to a group at a particular scope (subscription, management group, or resource group) is called a **role assignment**.

There are a large number of built-in roles, and the roles shown when browsing IAM for a resource are filtered to those relevant to that resource type.

### Permissions in a role

**Owner** has essentially every permission across all resource providers, including the ability to change permissions on the resource. **Contributor** can do almost everything Owner can do, except change permissions at that scope.

> **Exam tip:** Apply the principle of least privilege — grant the minimum role necessary for someone to perform their function.

### Data plane roles

Control-plane roles (such as Contributor) may have no data actions associated with them. Some roles — for example, Storage Blob Data Contributor, Storage Queue Data Contributor, Storage Table Data Contributor — include both control-plane actions and data-plane actions. This allows access to be granted via Entra identity-integrated RBAC rather than using access keys, which are not identity-specific and are harder to audit.

> **Exam tip:** Use data-plane RBAC integrated with Entra wherever possible, for better granularity and auditability compared to access keys. This also allows managed identities to interact with data-plane capabilities.

### Sum of role assignments

> **Exam tip:** If multiple roles are assigned to an identity (directly or via inheritance from parent scopes), the effective permissions are the **sum** of all those roles — not the least, and not the most restrictive. You can review this using "Check access" on a resource, which shows every assignment contributing to your effective access.

### Custom roles

Where no built-in role matches the exact permissions an identity needs, you can create a **custom role**, made up of `actions`, `notActions`, `dataActions`, and `notDataActions`, along with specific assignable scopes.

> **Exam tip:** You can have up to 5,000 custom roles per Entra tenant (2,000 for the China cloud), so avoid creating excessive numbers of them.

Custom roles can be created from scratch or cloned from an existing role, with permissions added or excluded as needed. Unlike built-in roles, you must explicitly specify assignable scopes.

> **Exam tip:** If a custom role is likely to be useful across many subscriptions or resource groups, set its assignable scope at a higher level (such as a management group), rather than limiting it to a single resource group — this avoids "wasting" entries against the 5,000 custom role limit.

### PIM usage

> **Exam tip:** Privileged Identity Management (PIM) provides just-in-time and just-enough access by making role assignments **eligible** rather than permanently **active**. Users activate the role for a limited period when they need it, following configured rules. In the portal, when adding a role assignment, the **Assignment type** option lets you choose eligible (the recommended default) versus active, and whether the assignment is permanent or time-bound.

> **Exam tip:** PIM is a [[Glossary#Microsoft Entra ID|Microsoft Entra ID]] P2 feature and has an associated licensing cost. Access reviews (also part of Entra ID Governance) can be used to periodically review and fine-tune group memberships and access.

## Attribute Based Access Control

RBAC alone can become bloated when very granular permissions are needed — for example, different containers within a single storage account requiring different access for different groups of users. Managing this purely through many storage accounts and many role assignments would be impractical.

Attribute-Based Access Control (ABAC) allows conditions to be added on top of a role assignment, comparing an attribute of the requesting identity against an attribute of the resource being accessed — for example, requiring a custom security attribute on the user's account to match a blob index tag on a blob, or requiring the connection to come through a private endpoint, a specific subnet, or within a certain time window (compared against UTC).

> **Exam tip:** At the time of recording, ABAC conditions are limited to certain built-in or custom roles related to Blob or Queue data actions, though this scope may expand over time.

Conditions are configured as an additional, optional layer on top of a role assignment. For example, a role assignment for **Storage Blob Data Reader** granted to a group can include a condition requiring the blob container name to match a specific value, and requiring a custom security attribute on the user's Entra profile (defined via a custom attribute set) to match a corresponding blob index tag value on the blob itself.

Custom security attributes are configured in Entra under **Protection → Custom security attributes**, where an attribute set and specific attributes (with predefined allowed values) can be defined and then populated on individual user accounts. When a user authenticates via their Entra identity (not an access key) against blob storage, ABAC evaluates these conditions dynamically — so a user might see and access only the containers or blobs whose tag values match their assigned attribute, transforming how fine-grained data permissions can be managed without needing a separate container or storage account per project.

## Azure Policy

Azure Policy sits on top of everything — like RBAC, there is no way to bypass it, regardless of whether access is via the portal or infrastructure as code; every action must go through the Azure control plane, and therefore through policy.

Policy can be used for both **audit** (assessing compliance) and **enforcement** (restricting what can actually be done) — for example, Microsoft Defender for Cloud (Security Center) is powered by policy.

> **Exam tip:** Always start with audit rather than enforcement/deny. Starting with enforcement risks accidentally blocking legitimate activity across the environment. Starting with audit lets you assess impact, understand how many resources are out of compliance, and plan communication to users before enforcing. When a policy does deny an action, it provides an explanation of which policy caused the denial — write good, clear policy descriptions so users understand why they're seeing what they're seeing.

Policy conditions are built around resource attributes, which are exposed via **aliases** — not every property is exposed, but many are. A policy definition specifies an `if` condition based on these aliases (for example, resource type is a storage account and its replication type is LRS) and an associated **effect**.

Policies can be browsed and created under **Policy → Definitions**, filtered by category (for example, "Storage"). A definition includes a description, optional parameters that can be customized at assignment time, the rule logic, and the effect (commonly recommended as **audit** or **deny**, with **disabled** also available). There is also the ability to perform a deny action specifically on **delete**, preventing deletion of a resource via policy.

### Initiatives

Grouping multiple policy definitions into an **initiative** makes it much easier to track compliance and enforcement — assigning and monitoring hundreds of individual policy definitions separately would be impractical. An initiative is assigned to a scope (management group, subscription, or resource group) just like an individual policy, and provides an overall compliance view that rolls up all the individual policies within it. Built-in initiatives exist for regulatory frameworks such as FedRAMP High, NIST, and PCI DSS v4, some of which comprise hundreds of individual policy definitions (for example, PCI DSS v4 includes around 719 individual policies).

> **Exam tip:** For anything made up of multiple related policies, create and assign an initiative rather than assigning individual policies separately, especially at a higher management group level so it applies broadly through inheritance.

### Policy effects and features

Custom policy definitions can use many different effects, including:
- **Deny** — returns a 403 Forbidden response, preventing the action (deny-on-delete is one specific, commonly used pattern).
- **Audit** — tracks compliance without blocking the action.
- **Disabled** — the policy is defined but not evaluated.
- **Append / Modify** — append adds an entry to an array field; modify can add, replace, or remove properties or tags on a resource as it's deployed.
- **Audit if not exists** — tracks an existence condition for compliance purposes.
- **Deploy if not exists** — evaluates a template and can remediate by deploying additional configuration (using a defined managed identity) after a resource is created, such as adding a Log Analytics extension or another agent/extension.

When assigning a policy or initiative to a scope, you can specify **exclusions** ("not scopes") for child scopes that shouldn't be included in the assignment — up to 400 per assignment. If you have permission at the scope of the assignment itself, you can also create a **policy exemption** on a specific resource or resource group, marking it as mitigated/waived without changing the underlying assignment — commonly used for built-in Defender for Cloud policies where a particular resource is intentionally exempt.

Common practical uses of policy include restricting allowed regions, restricting allowed VM SKUs (for example, disallowing expensive SKUs in development while allowing them in production), enforcing storage redundancy types (for example, requiring ZRS/GRS/GZRS instead of LRS in production), requiring certain tags to be present, and restricting the use of public IP addresses in certain scenarios.

## Cost management and budgets

Cost Management provides insight into and control over Azure spending, and also AWS spending via a connector. It provides **cost analysis**, giving insight into spending patterns, along with **smart views** that use machine learning to highlight deviations in spend.

In **Cost Management → Cost analysis**, smart views cover areas such as resource groups, resources, and services, surfacing insights like an unusually high daily run rate on a particular day, with the ability to drill into specific resources. Customizable views (such as cost by resource or cost by service) allow filtering — including by tag — and grouping by different dimensions. Data can also be downloaded or exported for further analysis, and this same data is available through billing APIs. If Azure reservations are in use, a monetized view shows the reservation cost spread over the purchase period.

An **automatic export** can be configured to regularly export cost data on a chosen frequency for consumption elsewhere. **Cost anomaly alerts** can also be configured, using machine learning to detect spending anomalies and email designated recipients.

### Budgets

Budgets let you define a target spend amount at various scopes — management group, subscription, or resource group — with optional filters (for example, by tag or resource type) to scope the budget more precisely. Actions can be triggered at defined spend thresholds, and importantly, thresholds can be based either on **actual** spend or on **forecasted** spend (based on current trend), allowing proactive action before a budget is actually exceeded.

Actions can include email notifications, or calling an **action group**, which can send an SMS, call a webhook or secure webhook, trigger a function or logic app, or create a ticket in an ITSM system.

### Tag inheritance for billing

Under **Cost analysis → Configure subscription**, there is a **tag inheritance** setting. When enabled, resources inherit a specified tag from the parent resource group or subscription for billing purposes only — this is useful when granular per-resource tagging isn't consistently applied, but billing rollups at the resource group or subscription level are still needed. It does not write the tag onto the resource itself.

### Cost allocation

Cost allocation addresses the scenario of shared resources — for example, a shared ExpressRoute circuit or shared infrastructure such as domain controllers, located in a certain subscription, resource group, or tagged with a specific value (the **source**). These costs can be split among a set of **targets** (subscriptions, resource groups, or tagged resources) that actually benefit from and consume the shared resource.

The split can be even, based on a custom percentage, or **proportional** — based on the relative spend of the targets, either their overall spend or, more granularly, their spend within a specific resource provider (for example, network spend, which may be more relevant than overall spend when allocating the cost of a shared ExpressRoute circuit). Targets see this reflected in their bill as an **allocated cost**, alongside their own accumulated cost, so they understand where the additional charge came from.

### API and PowerBI

An Azure Cost Management reporting API is available via the Microsoft Billing resource provider, usable at a subscription or Enterprise Agreement level. There is also a Microsoft Cost Management connector for Power BI Desktop.

> **Exam tip:** Microsoft Cost Management covers more than just Azure — it also includes Microsoft 365, Dynamics 365, and Power Platform costs.

### Pricing calculator

The Azure pricing calculator allows estimating costs in advance — for VMs, storage, databases, AI services, and other resource types — by specifying expected usage such as running hours and resource type/quantity.

### Optimizing costs

General cost optimization practices include right-sizing resources and choosing the appropriate SKU, using autoscale to adjust instance counts based on demand, using serverless where appropriate so cost is only incurred when work is triggered, shutting down or deleting resources when not in use (particularly for dev/test environments, recreated later via infrastructure as code), and taking advantage of free tiers where available. It's also important to think through all the ways a solution could be used — for example, a service that lets users upload content needs cost controls to prevent unexpected usage-driven costs.

### Azure reservations

Azure reservations provide a discount for a one- or three-year commitment to a very specific resource — a specific VM family, in a specific region, for example — or a specific amount of storage in a specific region for a specific duration.

> **Exam tip:** Reservations are extremely specific (resource type/family, region, and term). The billing engine applies the discount hourly to matching resources; if the committed capacity isn't used in a given hour, you still pay for it, since it's a financial commitment, not a usage-based charge.

Reserved instances can be exchanged for a different reservation if requirements change (Microsoft had planned to remove this exchange capability starting in 2024, but at the time of recording it has been extended indefinitely). Reservations can be created at the scope of an Enterprise Agreement, management group, or subscription, allowing child resources to consume against the shared commitment.

### Azure Compute Savings Plan

The Azure Compute Savings Plan offers a more flexible alternative to reservations, covering a broad range of compute services — including VMs, dedicated hosts, Azure Container Instances, Azure Container Apps, the Functions Premium plan, and App Service Premium v3/Isolated v2 (bare-metal VMs and the original Isolated v1 are not included) — across **any region**, for a one- or three-year term, based on a committed hourly spend amount rather than a specific resource/region combination.

> **Exam tip:** Compute Savings Plan discounts are generally lower than Azure reservation discounts (since reservations are far more specific), but the savings plan is far more flexible since it applies broadly across eligible compute services and regions. If both a reservation and a savings plan apply, the reservation is applied first (larger discount), and the savings plan applies to any remaining eligible spend.

### Azure Hybrid Benefit

Azure Hybrid Benefit lets you bring existing on-premises licenses (for Windows Server, SQL Server, and some Linux distributions) to use against Azure resources, rather than paying for a new license as part of the Azure resource cost. VM billing normally shows separate compute and OS license components; enabling Azure Hybrid Benefit removes the license charge, leaving only the raw compute charge.

### On-demand capacity reservations

> **Exam tip:** Despite the name, an Azure "reservation" does **not** guarantee capacity availability — it is purely a financial/billing commitment to pay for a certain resource type in a certain region for a discount. It does not guarantee that Azure will have the physical capacity available when you try to create the resource, particularly in high-demand scenarios.

Being flexible about region and SKU generally maximizes the chance of successfully provisioning a resource, even during high-demand periods. If a guaranteed, SLA-backed capacity guarantee is required for a very specific resource type in a very specific region, an **on-demand capacity reservation** provides that — you begin paying for it as soon as it's created, and then deploy resources into the reservation. It can be deleted at any time, but recreating it later is subject to the same capacity constraints. It pairs well with Azure reservations, since both are tied to the same very specific resource type, region, and term, but they solve different problems: one is a financial discount, and the other is a capacity guarantee.

## Deployment stacks

Deployment stacks are the modern replacement for the now-deprecated **Blueprints** feature. Blueprints combined creation of resource groups, RBAC assignment, resource deployment via ARM templates, and policy assignment, along with the ability to protect resources via read-only or "cannot delete" style deny assignments.

A **deployment stack** is a collection of resources on which lifecycle operations are performed together, functioning as a lifecycle boundary similar in spirit to a resource group — but able to span multiple resource groups, or even multiple subscriptions, while still being treated as one logical unit. When a stack is deleted, its resources can either be deleted along with it, or set to "unmanaged," leaving them as regular standalone resources. Resources can be added to or removed from a deployment stack at any time.

> **Exam tip:** Whatever scope a deployment stack is deployed to (resource group, subscription, or management group), all resources in the stack must be children of that scope.

A deployment stack uses the same template (for example, the same Bicep file) as a normal deployment — the difference is in how it's deployed (`az stack group` rather than `az deployment group`, for example). The name of the deployment stack is significant, since it identifies the stack for future operations, unlike a one-off deployment name.

Deployment stacks support **deny settings**, which are richer than what Blueprints offered — for example, denying modification or deletion of resources in the stack, excluding certain principals (such as a break-glass account) from the deny setting, excluding certain action types, and controlling how deny settings cascade to child resources.

> **Exam tip:** There is generally no downside to using deployment stacks over a standard deployment command when deploying infrastructure as code.

## Resource graph

Azure Resource Graph lets you query the control plane efficiently, using the same Kusto Query Language (KQL) used for Log Analytics. Standard ARM interactions have a limit (around 12,000 reads per hour), which can be exhausted; Resource Graph is a separately maintained, high-performance database that operates at massive scale, is read-only, and still respects the caller's actual access permissions.

Resource Graph is accessible through the portal (**Resource Graph Explorer**), Azure CLI, and PowerShell, and comes with example queries such as listing all public IP addresses.

> **Exam tip:** Resource Graph queries are rate-limited (around 15 requests every 5 seconds), though this is rarely a practical bottleneck given the tool's performance. Resource Graph is not well suited to querying permissions on resources, but is excellent for everything else related to resource configuration.

## Resource configuration change

Resource configuration change tracking (powered by Resource Graph) tracks what has changed in the environment, including the **change actor** (who made the change), the client used (for example, the Azure portal), and the operation performed. Selecting a change shows a before/after comparison of the affected properties — for example, revealing that a tag was added or a property was updated.

> **Exam tip:** This capability is enabled by default on the ARM control plane and, at the time of recording, provides a 14-day lookback window. For longer retention, changes would need to be captured (for example, via a function or logic app) into a Log Analytics workspace or similar store.

## Azure Advisor

Azure needs, capabilities, and available SKUs are constantly changing, so it's worth periodically re-evaluating optimization opportunities. Azure Advisor provides recommendations across key areas — cost, security, reliability, operational excellence, and performance — along with an overall **Advisor score**.

> **Exam tip:** Review Azure Advisor recommendations on a regular (e.g., weekly) basis. Alerts can be configured to notify on new recommendations of a certain category (such as security) and impact level, via an action group.

Advisor also supports creating recommendation digests (periodic email summaries) and assessments that walk through questions and provide actionable steps.

## Great resources

Additional resources worth reviewing include the Azure governance documentation, landing zone planning guidance, environment preparation guidance covering identity, governance, HR, DR, security, monitoring, and networking, the Cloud Adoption Framework, and the Well-Architected Framework — covering both overall Azure architecture thinking and individual workload configuration, including the Well-Architected Review and the Azure Architecture Center for reference architectures.

## Close

That concludes the governance module.
