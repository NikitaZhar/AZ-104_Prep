2026-07-03 17:44
Tags: #

## Introduction

This is the networking module of the Azure masterclass. The goal is to look at the basics of what a virtual network is, its capabilities in Azure, connectivity options — including connecting virtual networks together, as well as connecting virtual networks to on-premises — and then controlling and viewing traffic.

## Virtual network basics

A virtual network exists within a specific subscription and within a specific region, so it cannot span subscriptions and it cannot span regions.

Think of it this way: you create a subscription — the fundamental initial building block and boundary for various security and billing capabilities. Within a specific subscription, you can then use one or more regions. The virtual network (v-net) you create lives within a specific region you've selected, and within a specific subscription — it can't span either.

A virtual network is made up of at least one IP range. Initially this range is IPv4 — a CIDR block, a contiguous block of IP addresses. You can add additional ranges, and remove them too, as long as they're empty (no subnets in them).

A virtual network is broken up into subnets — you take a portion of a range and create one or more subnets within it. Subnets can't use the same IP range multiple times; you're just breaking up the IP space into subnets that will then be used by various resources.

In the demo, creating a virtual network in the portal requires picking a resource group, a subscription, and a specific region — those are the boundaries discussed above. By default, the portal picks the range `10.0.0.0/16` and creates a default subnet as a `/24` of that (using 8 bits for the subnet), giving up to 256 addresses within it. You can add additional subnets, delete the default subnet, and have complete control over the address spaces — both IPv4 and IPv6 — and this can all be modified after creation time too.

Typically, you'll use RFC 1918 — a set of IPv4 address spaces that isn't routable on the internet. The goal is to avoid wasting IP space, and because these addresses aren't routable, different companies can all use the same addresses internally without conflicts. These are `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16` — you'll typically see one of these on your home network from your internet provider.

When devices in an organization need to talk to the internet, some form of network address translation (NAT) is used — services sitting at the edge of the network have internet-routable addresses and perform translation on behalf of internal clients.

You don't have to use RFC 1918 — you could bring a block of public IP space into Azure. But even if you use internet-addressable ranges for your virtual network, they're still treated as private IP space in Azure — not contactable or routable out to the internet. There are also some IP ranges you can't use, reserved for various Azure functions and outlined in the documentation.

When you break up your IP space into subnets, you lose a number of IP addresses, because Azure and the IP protocol in general use some of them.

> **Exam tip:** You always lose 5 IP addresses in any subnet. This is because the first and last addresses are always reserved for the network address and broadcast address respectively (e.g., for a `/24`, that's `.0` and `.255`), and Azure additionally reserves the first address for the default gateway and the next two for mapping to Azure DNS. So out of a "standard" 256 addresses in a `/24`, only 251 are actually available.

For IPv6, subnets are always `/64` in size. IPv6 is 128 bits, and the address space is enormous — the goal is that there should be millions of addresses available for every square foot of the earth — but for compatibility with things like on-premises routers, and because the address space is so huge, a subnet in Azure is always `/64`.

> **Exam tip:** At time of recording, you can have up to 3,000 subnets per virtual network. Subnets can be resized, provided they're empty.

Sometimes you'll see a subnet marked as delegated to a certain service — this means the subnet is going to be used exclusively by that service. A number of Azure capabilities integrate or inject themselves into a subnet for tight integration, but want exclusive use of it. If you see a delegation on one of your subnets, it's because a certain Azure service is using it, and you can't put your own resources into it.

Resources connect to a subnet — you don't really put things "into" a subnet directly. When you create a resource — say, a virtual machine — one of the resources it has is a virtual NIC (network interface card), which is what actually connects to a particular subnet. That connection has a configuration recording that it's connected to a specific virtual network and a specific subnet.

Many resources can have multiple virtual NICs, and each NIC can have multiple IP configurations, but they all have to be within the same virtual network — you might have another NIC connecting to a different subnet, but you can't connect a NIC to a completely different virtual network. This can matter for something like a network virtual appliance, which might need hooks into different subnets — one could be a front-end DMZ, another more of an internal network, letting the appliance bridge the gap and route between them.

In the demo, looking at a network interface card for a virtual machine (visible as its own resource within the resource group) shows it's connected to a certain virtual network and subnet, with a private IP as part of an IP configuration, an optional public IP address, and even a private IPv6 address without a public one — so a resource can have several different IP configurations, but it's the NIC that connects to the subnet, and the NIC that connects to the virtual machine.

An important point relating to the resiliency model: regions have availability zones, and a subnet is a regional construct that actually spans availability zones. A subnet isn't linked to a particular availability zone — resources (e.g., VMs) living in AZ1, AZ2, and AZ3 can all be within the same subnet, with no hard linking between those constructs.

### VM NIC

The IP address for a resource always comes from the Azure fabric. From the guest operating system's point of view, it's essentially leveraging DHCP — the VM's NIC shouts out over the network asking for network information, normally via a broadcast. Broadcasts don't work in Azure — the virtual network is entirely software-defined networking. What actually happens is that shout-out is captured by Azure's software-defined networking components, which look at the configuration of the virtual NIC and return the required information: IP address (consistently, if you want), gateway, DNS config, subnet mask.

There's an exception: if you assign multiple IP addresses, you do have to manually configure them within the guest OS — on Windows you'd configure the primary and secondaries, on Linux only the secondaries. You can reserve addresses, but you still wouldn't typically configure that as a static setting inside the VM itself — instead, as part of the NIC configuration, you can mark it as static.

In the demo, under the network interface's `IP configurations`, two configurations are shown: the primary (always IPv4), currently `Dynamic` (meaning it could change if the VM is stopped and restarted), with a public IP address also assigned; and a second, IPv6 configuration (since IPv6 was enabled on the v-net and subnet). The private IP address setting can be switched from `Dynamic` to `Static`, letting you specify a fixed address — effectively a DHCP reservation, handled by the Azure fabric.

You can add multiple NICs; the number depends on the size of the VM, and they can be attached and detached through the portal or CLI.

> **Exam tip:** An important point to remember: this is all software-defined networking — there isn't actually a physical NIC on the VM physically attached to a unique piece of wire. The traditional reasons for multiple physical NICs in the real world (connecting to different physical cables, ports, VLAN segments) don't apply in Azure, since it's all software-defined, using network security groups and routing. So most of the usual arguments for multiple NICs don't work the same way — but it can still be useful, for example, for a network virtual appliance that needs to connect to a front-end subnet and a back-end subnet for routing purposes, as long as everything stays within the same virtual network.

Many virtual machine types support accelerated networking. Ordinarily, traffic passes through a virtual switch within the hypervisor, which looks at the rules to enforce and where packets should go — introducing some latency. With accelerated networking, that virtual switch is essentially bypassed, and instead hardware capabilities expose something called a virtual function — think of it like a virtual NIC exposed through the hardware directly into the VM, without going through the virtual switch. The hardware within the Azure host still handles routing and network security groups, but latency is reduced and you get enhanced performance.

In the demo, checking a virtual machine (e.g., a `Dsv6`) for `Accelerated networking` support is how you verify availability. Today, most modern VMs with more than two virtual CPUs (or hyper-threaded to four virtual CPUs) should support it.

> **Exam tip:** Accelerated networking is essentially free of downsides — it doesn't cost you any extra money, and if it's available for a VM, it's worth turning on. It does look a little different within the guest OS, so it's worth checking on that, but for the most part it's nothing but a benefit.

There's also **Azure Boost** — special hardware to accelerate various things, including storage performance, security, and networking. Azure Boost includes something called the Microsoft Azure Network Adapter (MANA), supporting up to 200 Gbit/s. At time of recording this is in preview and will likely roll out further over time; today it's only on the Dv6 and Ev6 series (both Windows and Linux). So accelerated networking is broadly available today, while Azure Boost, with its special hardware components (including FPGAs) in the hosts, is the next stage that will make things even better.

As mentioned, you can have multiple IP configurations per NIC — the primary configuration is always IPv4 (optionally with a public address), and you can add secondary IP configurations, which can be another private IPv4 (optionally with a public IPv4), plus a single additional private IPv6 secondary configuration (which itself could also have a public IPv6 address) — only one such IPv6 configuration is allowed per NIC.

One reason you might want multiple IP addresses: each IP address gives you a whole set of usable ports. If you're running a workload with many clients talking to it, each requiring a unique port combination, you could max out what's available on a single IP address — similar to network address translation. Having multiple IPs gives you multiple sets of ports, letting you scale the number of clients or services you can talk to.

## Supported types of traffic

The key thing to remember is layer 3 — IP and above run on a virtual network.

- **TCP and UDP** are the big ones — the majority of regular communication is one or the other. TCP checks whether a packet was received and resends if not; UDP is fire-and-forget — it sends the packet but doesn't check whether it was received (that responsibility would fall to something like the client). So you have a choice between a more stable, checked form of traffic and a faster one with the risk that packets get lost without anyone noticing.
- **ESP and AH** are part of IPsec, providing encryption and integrity capabilities.
- **ICMP** is used for diagnostic functions — the echo request used by `ping`, and used by `traceroute`.

Other layer 4 protocols on top of IP may technically work, but as soon as you interact with other components — a load balancer, network security groups — they only understand specific protocols, so while another type of layer 4 traffic might work on the raw network, you may get very different functionality, potentially breaking entirely if it hits a network security group, an Azure Firewall, or a load balancer.

Things that definitely won't work:

- **Multicast** — talking to multiple different things as part of the communication.
- **Broadcast** — shouting out to your local subnet ("I need an IP address," "I'm changing something").
- **Generic routing encapsulation (GRE)** — the network virtualization technology Azure itself uses — is not something you can use for your own purposes.
- **A promiscuous network sniffer** inside a VM won't see other traffic on the wire — your virtual NIC isn't really connected in a traditional sense to a shared wire; it's software-defined networking, with intelligence determining which traffic is bound for a particular virtual NIC and forwarding only that. A sniffer inside a VM can see traffic for its own NIC, but not traffic on the subnet that isn't actually bound for it.
- **Your own DHCP server** for Azure clients — because when a client does that broadcast, it's grabbed by the software-defined network and would never reach a regular DHCP server listening for those requests.

> **Exam tip:** There used to be rate limiting for DHCP relay traffic, which meant you couldn't even host a DHCP server to serve an on-prem set of clients. That restriction has been removed — you can now technically run a DHCP server in a VM to serve on-premises clients, by setting up a relay that sends traffic directly (unicast, not broadcast) to the DHCP service, so it isn't captured by the broadcast mechanism.

You cannot ping the Azure Gateway. When talking to something outside your subnet, traffic goes via the Gateway; talking to things within the same subnet is direct. In a physical world, gateways have wires or VLAN configurations into different subnets — that doesn't exist in software-defined networking, but there's still a Gateway component Azure provides: when you try to talk to something outside your subnet, it goes to the Gateway, which passes it on to the target resource in its target subnet. You can't ping it because it isn't a "real" thing in that sense. Pinging between services can work if the client OS firewall allows ICMP/echo requests, but you cannot ping the Gateway itself.

> **Exam tip:** Traditional layer 2 VLANs aren't supported on-premises in the sense we're used to — VLANs sit below layer 3, and Azure is entirely layer 3, software-defined networking.

## IPv6

Virtual networks in Azure are dual stack — you can have IPv4 and optionally IPv6, but IPv4 is required. You cannot be IPv6 only.

In the demo, under a virtual network's `Address space`, an IPv6 address space has been added. Ordinarily this would probably be a `/48`, but it's split further to reuse the same `/56` range across multiple virtual networks. Under a subnet's settings, IPv6 can optionally be enabled — the size is always `/64`, giving good compatibility, for example when talking to on-premises. A VM connected to that subnet will then get both an IPv4 and an IPv6 address — but again, it can never be IPv6 only, only dual stack.

Just like IPv4, there are different types of IPv6 addresses — globally unique (internet routable), link local (self-generated), and non-internet-routable addresses (like the `FD`-prefixed one shown in the demo). This is covered in a lot more detail in a dedicated IPv6 video.

IPv6 is supported today by a lot of Azure's capabilities:

- Virtual machines and their NICs.
- Network security groups — which control what traffic can flow.
- User defined routes — which help route traffic to particular next hops.
- Azure load balancers, distributing traffic through a single front end, and their health probes.
- Peering — connecting virtual networks.
- **Private Link** does not currently support IPv6 — worth checking exactly what's supported before committing to a full solution.

Because virtual machines support IPv6, virtual machine scale sets support it too, and because scale sets support it, AKS can as well — though it depends on the networking stack. For example, using `kubenet`, dual stack IPv6 is supported.

> **Exam tip:** Azure's control plane, which these resources have to talk to, is always IPv4 — so IPv4 is always required. If using the Azure CNI overlay for AKS, dual stack is supported starting around Kubernetes 1.126.3 or so — nodes and pods get both IPv4 and IPv6 in dual stack, but a given service within AKS using that stack can be configured as just IPv4 or just IPv6.

Enabling IPv6 on an existing resource is possible, whether it needs a reboot or not entirely depends on the OS and its current DHCPv6 configuration (different Linux distros respond differently). You would not, however, need to redeploy the resource entirely — you might need to reboot it, for example, if enabling IPv6 on the subnet.

ExpressRoute private peering also supports IPv6 (covered in more detail later) — all you need is a pair of `/126` addresses for the primary and secondary links.

Public IP addresses are IPv4 or IPv6 — never both. In the demo, when creating a public IP address, you're asked to choose one or the other.

## External (Internet) access

There's no special demilitarized zone (DMZ) in Azure that you have to put things in to be able to talk to the internet. There's an implicit internet access for resources in a virtual network — you don't have to do anything manual; today, being in a virtual network, you can talk to things on the internet without any special configuration.

Depending on configuration, Azure provides a public IP address, using port address translation to masquerade your private IP addresses, plus a set of ephemeral ports that let you talk to the internet. This is always stateful — even though you have access to the internet, that doesn't mean things from the internet can talk to your virtual network: you can get the response back from a request you made, but that doesn't mean someone from the internet can just talk to your resources. This flow can be controlled with network security groups, Azure Firewall, and similar tools.

> **Exam tip:** A key point is that this implicit access is actually going away. Per the documentation (accurate at time of recording — this may have already changed by the time you're watching this, so worth checking): as of September 30th, 2025, this default implicit access for new resources will be retired, and over time other aspects will be retired as well. The takeaway is you don't want to be relying on this implicit access.

There is something you can do today: on a subnet, there's an option to make it a private subnet — checking that box turns off implicit internet access. You should still be restricting things with network security groups and other capabilities, but this option is available now.

If you want an explicit method for outbound internet access, the best option is **NAT Gateway** — specifically designed to provide outbound internet access. You create a NAT Gateway instance, which has public IP addresses or prefixes (a contiguous set of public IP addresses), and you link it to one or more subnets. A route table on the subnet then sends internet-bound traffic to this next hop.

> **Exam tip:** Each public IP the NAT Gateway has provides around 64,500 ports, and you can have up to 16 public IPs on it — very high scalability, able to support a massive number of clients. You can link a NAT Gateway to up to 800 subnets, though it has to be within the same subscription.

This is nice because it's specifically designed for outbound connections, and you'll know exactly which IP addresses your traffic will come from when talking to some other internet-based service — unlike implicit access, where you don't know which public IP Azure would use. Of course, you shouldn't rely on IP address alone — you should also use authentication and think zero trust — but this can be a piece of that puzzle.

Another option is the Azure Load Balancer, distributing traffic to backend services. If that load balancer has a public IP address (as opposed to a private one), you can create outbound rules allowing resources behind it to talk to the internet via the load balancer.

> **Exam tip:** An important point today: putting a resource behind a private load balancer means it won't be able to talk to the internet — that breaks implicit access. You'd have to explicitly grant internet access some other way in that case.

You can give a resource a public IP address directly, as part of its IP configuration, and it would use that to talk to the internet — always an option, but generally we don't like resources having public IP addresses directly. For security reasons, you generally don't want things directly accessible from the internet. And even if you're providing a service to the internet, you probably don't want it provided by a single resource — what if it's undergoing maintenance, what if it fails? Generally you'd want multiple resources providing a service, spread across availability zones or even geo-distributed, sitting behind a load balancer, and possibly with an App Gateway or Azure Front Door with a Web Application Firewall in front for extra protection. So yes, direct internet access is possible, but generally it's not a great idea as part of your overall configuration.

You could also use Azure Firewall, which inspects traffic at layer 4 and layer 7 and can provide outbound source NAT, though it's not very efficient — roughly 2,496 ports per IP per backend instance, and while you can have lots of IP addresses, it's just not as IP-efficient as other solutions. It's fantastic for inspection at layer 4 and layer 7, including TLS, and can crack open packets to look at them. You could also use a third-party network virtual appliance with a public IP — there are many options available.

### External access warning

> **Exam tip:** An important warning: you shouldn't have RDP, SSH, or WS-Man directly available from the internet on your resources. If you enable that from the internet, you'll see attempts to hack it almost immediately — it's not a good practice. If you must RDP into a resource in Azure from the internet, use something like Defender Just-in-Time, which uses network security group controls to only allow access from your current IP address for a limited window of time. There are also solutions like Azure Bastion, which provide a managed jump-box experience using Entra authentication. Ideally, though, you want your virtual network connected to your on-prem network so resources have a direct IP path on private networking, rather than having to go via the internet — thinking of the virtual network as an extension of your existing network.

### Bring your own IP

When you create a public IP resource — its own separate resource — you'll get a public IP address from Microsoft-owned sets of internet-routable addresses, depending on the region; Microsoft owns those public IPs.

There are occasions where you might not want to use Microsoft's address ranges — for example, if you currently host a service on-premises using an IP address you own, one that's built up a certain reputation, known by parties that talk to it, with firewalls configured to allow communication from it. If you're migrating that service to Azure, you don't want to lose that trust associated with the IP address or range — so you can bring your own IP.

Internet routing is complex — you can't just bring an IP address and expect it to route from the internet, since the internet is really a whole set of networks that learn paths and connectivity between one another. Microsoft has enabled the ability to bring a portion of internet-routable IP space into Azure: you bring a specific portion of your IP space (either IPv4 or IPv6), which gets created as an Azure address prefix — a coherent set of IP addresses.

For IPv4, the global range you bring to Azure has to be between a `/21` and a `/24`; for IPv6, a `/48`. You then break that into regional child ranges used for regional services in a given region: for IPv4, regional child ranges are `/22`–`/26`, and for IPv6, `/64`.

Because public IP ranges are governed and allocated by global internet bodies, and are part of a much larger routing system, bringing that IP space to Azure and having Microsoft advertise it out to the internet — updating routing tables all over the world so that traffic destined for that IP goes via Microsoft's public points of presence — takes several steps:

1. You have to prove you own the address space — registered with a routing internet registry (e.g., ARIN) — and authorize Microsoft to advertise the range.
2. The range is provisioned, created as a custom IP prefix in your subscription.
3. From that range, public IP prefixes and public IPs can be derived and associated with Azure resources, supporting standard SKU public IPs — though at this point they're still not advertised or reachable.
4. You commission the range, at which point it starts being advertised by Azure, first regionally and then out to the internet via the Microsoft network.

That's a significant number of steps, but the upshot is you can bring your own IPs into Azure. Generally, people don't talk directly to IP addresses — they want to use DNS names, and you generally shouldn't trust just the IP address; you want authentication and multiple layers of protection. But if you have a service with established trust that's configured to talk to specific IPs, this capability enables that.

## Connecting virtual networks

As mentioned, a virtual network is bound to a subscription and a region — so any scenario involving more than one region or more than one subscription is going to involve more than one virtual network.

In the old days, you could connect virtual networks by connecting them to the same ExpressRoute circuit, or via a site-to-site VPN — both of which are pretty poor solutions.

**Using ExpressRoute to connect v-nets:** you buy a circuit — the Microsoft backbone network expands at certain points into carrier-neutral facilities, points of presence where different networks come together. Say you have `v-net 1` and `v-net 2`, and want them to talk to each other. Even if they're in the same region, physically close, normally able to talk to each other with very low latency because they're in the same data centers — if you connect them both to the same ExpressRoute circuit through the same gateway, say a meet-me facility hundreds of miles away in a different city, traffic between them would flow via that facility, adding a lot of unnecessary latency even though they're in the same data center.

**Using site-to-site VPN:** traffic is encrypted — each side has a gateway, and they establish a VPN over the internet (though Azure is smart enough not to bounce the traffic via the internet itself). But now the traffic has to be encrypted, so you're limited by the bandwidth of the gateways, since encryption uses a lot of compute.

Both approaches are less than ideal — one adds latency, the other throttles bandwidth due to encryption overhead. The better solution is **virtual network peering**.

### Peering

Virtual network peering lets networks connect using software-defined networking, purely over the Azure backbone — you get the regular performance the resource is capable of, with no new limits introduced. You can peer virtual networks in the same region (regional peering) or in different regions (global peering). The virtual networks can be in different subscriptions, and even different tenants.

If the virtual networks are in the same subscription, creating the peering can create both ends in one action. Otherwise it's a multi-step sequence — you need permission to request a peer to another virtual network, and then it has to be acknowledged and created in the other direction.

Picture `v-net 1` and `v-net 2`, each with their own IP address range, establishing a peering relationship — after which they know about each other.

> **Exam tip:** IP ranges of peered virtual networks cannot overlap, since traffic needs to be routable between them.

This is a fantastic, fairly simple-to-set-up solution, giving full IP traffic flow between networks, but you do pay for that traffic — there are both ingress and egress charges for virtual network peering.

> **Exam tip:** Within the same region, peering is roughly a penny per gigabyte in each direction. Global peering pricing depends on which zones the regions belong to (different regions are grouped into different pricing zones) — global peering costs more. Virtual network peering is not free, and this is worth accounting for when planning costs.

Since peering establishes routing, IP spaces can't overlap. There's an interesting nuance, though — a kind of "grandchild" peering scenario: imagine a network with range `cider 3` peered to another with range `cider 4` — these have to be unique. But technically, that second network could then be peered to a third network using, say, `cider 5`, which is in turn peered to yet another network also using `cider 5` — and this is fine, because routes don't propagate transitively: the first network never learns about, and never talks to, the "grandchild" network, so those ranges can overlap since those networks never actually communicate or exchange routes. This might be useful for something like a hub network peered to regional app v-nets that all use the same address space (e.g., `10.0.0.0/16`), which don't need to talk to each other directly.

> **Exam tip:** A key point about peering is that it's **not transitive**. If network A is peered with network B, and network B is peered with network C, that doesn't mean A can talk to C — you'd have to manually create a separate peering between A and C, since these connections aren't providing transitive communication.

Worth calling out separately: subnet-level peering is now supported — a fairly cool capability. Traditionally, peering covered the entire IP space of a virtual network. Now you can set the parameter `peer complete v-net` to `false` and specify the specific local and remote subnet names you want to peer — so you can peer just specific subnets rather than the whole address space. You can also do IPv6-only peering. This might help if, say, full ranges overlap but non-overlapping subnets can still be peered. If the virtual networks genuinely fully overlap and need to talk, you'll need network address translation — that's what Private Link is for (covered later).

### User Defined Routes and appliances

Because peering typically isn't transitive, connecting two networks through an intermediary would normally require manually adding a separate peering between them. To avoid building a huge full-mesh network of peerings, the hub-and-spoke pattern is common: a central hub network is peered with several spoke networks. Normally spokes don't need to talk to each other directly — they peer to the hub and can talk to it, but not to each other.

In the demo, a hub virtual network has peerings to two spokes (`hub spoke 1` and `hub spoke 2`) — those two spokes can't talk to each other, only to the hub, since the hub is what provides the services they actually need.

But if you do want spokes to talk to each other without a full mesh of peerings (which could be enormous with hundreds of spokes), you can add Azure Firewall (or a third-party network virtual appliance) to the hub. That firewall gets an IP address from the hub's address space. You then need to tell spoke 1 that if it wants to talk to spoke 2, it should talk to the firewall's IP address, which will forward the traffic. This is done by creating a **user defined route (UDR)**: a route saying that for the destination range `cider 2` (spoke 2), the next hop is the firewall's IP address. Because this is all software-defined networking, the next hop doesn't have to be in the same subnet or virtual network — the firewall has connections to both spokes and forwards traffic accordingly. You then link this route to the subnets that need that connectivity.

In the demo, under a route table's `Route table` (UDR), adding a route requires a name, a destination type (an IP address or a service tag — a set of IP addresses for certain Azure services), the range (e.g., `cider 2` for spoke 2), and the next hop (which could be a virtual appliance, with the Azure Firewall's IP address, a virtual network gateway, the internet, and so on) — there's a whole set of default routes plumbed into virtual networks, routes learned by gateways, and user defined routes you add yourself.

### Remote gateway use

Under a virtual network's `Peerings`, you'll see options like "allow gateway or route server in the hub v-net to allow traffic," "use remote gateway or route server," "allow forwarded traffic," and so on. What are these all about?

Typically, a hub not only has resources that spokes need, but also connectivity via a series of gateways — for example, an ExpressRoute gateway, connected to an ExpressRoute circuit, giving connectivity to on-premises locations via private peering (covered later), or a site-to-site VPN gateway connected to your on-premises networks over the internet. That gateway learns about other IP spaces and, via BGP (Border Gateway Protocol) — a way to learn and distribute routes — plumbs those routes into the virtual network. Resources in that network then know that to reach a given IP space, they should send traffic to the gateway, which forwards it on.

Those peering settings essentially let you say, on the hub side, "allow gateway transit" ("let others use my gateways"), and on the spoke's peering, "use remote gateway." The result is that the hub, through that peering connection, propagates the routes it's learned via its gateway into the spoke as well — so resources in the spoke now know how to reach that IP space through the gateway, and resources on those external networks can also now talk to things in the spokes. It's a way to expose and extend that routing.

### Route server

Another component of the Azure networking stack is **Route Server**. This matters when you're not using Azure's own ExpressRoute or site-to-site VPN gateway (which automatically plumb learned IP spaces into the virtual network's route tables), but instead something like a third-party network virtual appliance (e.g., an SD-WAN solution) that talks to other IP spaces on its own but can't plumb those routes directly into the virtual network — you'd otherwise have to create user defined routes manually pointing at the appliance's IP address, and keep maintaining them as those IP spaces change, which becomes messy.

Route Server solves this by becoming a conduit for plumbing routes into the virtual network: you deploy a Route Server instance, and the network virtual appliances establish BGP sessions with it, telling it which IP spaces they're talking to. Route Server then plumbs those routes into the virtual network's route table.

> **Exam tip:** An important point: when using Route Server, traffic does **not** flow through Route Server itself — it's purely there to establish BGP sessions that plumb routes into the virtual network, not to forward traffic.

The gateway transit settings discussed above also work with Route Server and the IP spaces it's learned from an NVA. When using Route Server, spokes can't have their own gateways (their own ExpressRoute gateway or site-to-site VPN gateway) and use remote gateways at the same time — connectivity is concentrated at that layer. This is a very common configuration: focus connectivity in the hub, and let spokes leverage it, possibly also talking to each other through something like Azure Firewall.

## Connecting to on-premises

Very few companies run entirely in Azure — most are in a hybrid scenario, with some services remaining on-premises and clients on both sides needing to reach resources in the other environment. Most Azure services, especially many PaaS offerings, have internet-facing public endpoints, so you could consume them that way. But often you don't want to — if you're on your internal corporate network, you don't want to bounce via the internet just to talk to, say, a Postgres database running in Azure. You want to think of that Azure virtual network as an extension of your on-premises network. Generally, you want the Azure virtual network, and any services connected into it (private endpoints and Private Link will make more sense once covered later), to be reachable from your on-premises network.

There are a few options for virtual network connectivity:

- **Point-to-site VPN** — a specific client, like your laptop in a coffee shop, talking directly to your Azure virtual network.
- **Site-to-site VPN** — connecting one network to another, e.g., your premises network to your Azure virtual network.

It's worth realizing that Azure regions connect into a massive, global Microsoft Wide Area Network — one of the biggest networks in the world. All Azure regions connect into that backbone. At the edge of the region (not part of your v-net, but Azure's physical infrastructure), there are Regional Network Gateways — highly redundant components that connect into that Microsoft backbone network. That backbone network, in turn, connects around the world to many points of presence — some for private connectivity, some that simply connect out to the internet, which is where you enter Microsoft's network when talking to an Azure service over the internet — distributed all around the world to give you the lowest possible latency onto the Microsoft network, and then into the region and your virtual network.

### S2S VPN

The first thing needed is a Gateway subnet — a subnet specifically for housing the various types of gateways.

> **Exam tip:** For headroom, it's recommended to make the Gateway subnet `/27` or larger. Technically it can be as small as `/29`, but that limits your ability to grow later (e.g., adding site-to-site VPN and ExpressRoute together), so `/27` is recommended.

For VPN, you deploy a pair of VPN gateways — think of these as managed virtual machines Azure creates for you, exposing the ability to create a VPN.

There's route based and policy based. Route based is preferred and is the default — it dynamically enables IP addresses. Policy based is static and is largely deprecated today; nearly everything supports route based now.

VPN gateways on the Azure side and on your side can be active-active or active-passive.

- A single active connection with no redundancy isn't great — an outage would require establishing a new connection, with downtime.
- A better option is **active-active** on both sides, establishing dual connections — giving resiliency to a failure on either side, forming a "bow tie" pattern with cross connections in both directions, resilient to any single component failing.

> **Exam tip:** Active-active is available for all Gateway SKUs except `Basic`. An existing active-standby gateway can be easily switched to active-active. With active-standby, you may see a brief interruption during an outage or even planned maintenance.

Since a VPN gateway goes over the internet, latency can vary — you don't know exactly which routers your traffic is taking at any given time — but you can still get very good resiliency, and traffic is encrypted. It's a reasonably good option, typically for smaller organizations — larger organizations typically use the various types of **ExpressRoute**.

### ExpressRoute

ExpressRoute, specifically private peering, is about connecting IP spaces — connecting a network to a virtual network via a peering location and a gateway.

With ExpressRoute you deploy ExpressRoute gateways. There are many points of presence around the world where the Microsoft backbone extends into carrier-neutral facilities with resilient connections. This is sometimes called a peering point or meet-me facility, since it's where networks meet. Within it, Microsoft has a pair (or many pairs) of routers, known as the Microsoft Enterprise Edge (MSEE).

For a particular ExpressRoute circuit you buy, your ExpressRoute gateways connect to that pair of Microsoft Enterprise Edge routers in a bow-tie pattern — both your gateways connect to both MSEE routers, which sit in Microsoft's own cages within that carrier-neutral facility.

On your side, you typically have your own location, usually going through a service provider (e.g., a major carrier). You'll have your own routers connecting into this facility — perhaps via a direct connection, perhaps via MPLS — establishing your ExpressRoute connections. These are always two connections, running active-active, used for both the data path and BGP route exchange, making it very resilient.

Because both connections are active, if you buy a circuit sized at, say, 100 Mbit/s, on a good day with no issues you'll actually get 200 Mbit/s (because both are running), but during maintenance or a problem you'd drop down to the size you paid for.

What you pay for: the circuit (to Microsoft), the provider's portion, the gateways, and egress. You don't typically pay for ingress into Azure (with a few exceptions for peering and private endpoints) — you typically pay for egress.

In the demo, a page shows locations, listing service providers offering ExpressRoute at each location, the physical meet-me address, and whether there's a "local region" — for example, Dallas has no local region because there's no Azure region there (the closest might be San Antonio), while another location might have South Central US as its local region.

There are also different Gateway SKUs with different levels of resiliency — today it's often worth looking at ones supporting availability zones, where the gateways run active-active across two different zones. There's support for FastPath (covered later), a maximum number of circuit connections (a single gateway can connect to multiple circuits), and ExpressRoute Gateway Scale, giving a configurable number of scale units for the throughput you need.

> **Exam tip:** Only the `Ultra` and `Gateway 3AZ` SKUs support FastPath. However, all SKUs support coexistence with a VPN gateway, so you can have both VPN and ExpressRoute at once.

There are metered circuits (where you pay for data egress) and non-metered ones (which cost more but don't charge separately for egress). If you're doing a massive amount of egress through ExpressRoute, non-metered might be worth it; most of the time it's better to pay for egress.

As mentioned, you buy a circuit connecting to a certain location, and multiple v-nets can talk to the same circuit — another v-net with its own ExpressRoute gateway could technically connect to the exact same circuit, and while they could then talk to each other, it would hairpin via that location, adding latency.

> **Exam tip:** Typically 10 different v-nets can talk to the same circuit if it's a local standard SKU; if premium, the number is between 20 and 100, depending on the size of the circuit.

There's also a **Premium** SKU, adding the ability to learn up to 10,000 routes from on-premises (vs. 4,000 for standard), the ability to talk to regions across the globe rather than just within the same geopolitical boundary, and the ability to connect to Microsoft 365 (covered next).

Additionally, a single gateway can talk to more than one circuit — important for resiliency.

### Resilient ExpressRoute

A meet-me facility is a physical building — while there's a pair of routers, they're in the same building, so if the building fails (this famously happened in Chicago), your circuit goes down despite having two routers.

For resiliency, you should design in another circuit, in a completely different physical facility, hundreds of miles apart. Your ExpressRoute gateways would also connect there, and that circuit would also connect to your MPLS network or your own network. Because of the significant distance, you now have resiliency against a problem with a specific physical location.

> **Exam tip:** Depending on various factors, the number of circuits a gateway can connect to ranges between 4 and 16 — the exact figure depends on the scalable gateway SKU and the number of scale units chosen. This gives higher-level resiliency against any given meet-me facility failing.

### ExpressRoute Metro

There's also **ExpressRoute Metro**. As discussed, a pair of routers usually sits in the same physical facility. ExpressRoute Metro splits that pair across two different physical facilities within the same city (metroplex). This gives resiliency against a building-level failure (barring a city-wide event like flooding), providing facility-level resiliency — a bit like using availability zones in Azure, but applied to the meet-me facility for your connectivity. At time of recording this is relatively new and not available everywhere, but worth using where it is.

### ExpressRoute Direct

Normally you buy a circuit. For very large companies, **ExpressRoute Direct** lets you buy the ports themselves rather than a circuit — you don't own the whole router, or even a whole port; it's normally shared across customers. With ExpressRoute Direct, you buy a pair of ports outright — for example, a pair of 10 Gbit/s or 100 Gbit/s ports — and then create circuits of whatever size you want on top of those ports. This is for really large customers used to dealing with fiber lines directly, who get letters of approval allowing their router to hook up to a particular port on the Microsoft Enterprise Edge.

### Local SKU

As mentioned regarding egress charges, there are three levels — Local, Standard, and Premium.

- **Local** — as seen in the location table, some meet-me facilities have an associated local region. With local, you get free egress, but can only connect gateways to that specific local region. This can be attractive if you need high throughput but only from a particular region, or if you set up a series of circuits in different meet-me facilities each tied to a different local region.
- **Standard** — in addition to the local (paired) region, you can use any region within the same geopolitical boundary (e.g., all of the Americas).
- **Premium** — global: you could create a circuit in Dallas and connect it to Azure resources in Europe (within the same commercial cloud). Premium also enables **Microsoft 365** connectivity — in addition to private peering (connecting IP spaces), there's Microsoft peering, which lets you create route filters so that certain Azure services can advertise into your on-prem network as well.

> **Exam tip:** With Standard, you can't connect to Microsoft 365 services via ExpressRoute; with Premium, you technically can, but generally it's **not recommended** — Microsoft 365 is designed to be accessed over the internet, and once you start plumbing it into ExpressRoute you tend to end up with traffic flowing over different paths, breaking firewalls (a response arrives via a route they aren't expecting, given the request went out another path) — most companies that try this end up breaking it. Premium also supports up to 10,000 on-prem routes instead of the 4,000 for Standard.

As mentioned, ExpressRoute gateways can coexist with VPN gateways — ExpressRoute is preferred, with site-to-site VPN as the backup. You can also run a site-to-site VPN over ExpressRoute itself, for example if you want additional encryption of the traffic.

### GlobalReach

As discussed, gateway transit settings (allow gateway transit, use remote gateways) let route propagation extend spoke connectivity through the hub. There's also **ExpressRoute Global Reach**: imagine a second location with its own separate ExpressRoute circuit. Normally your company has its own backbone network paying a provider for private connectivity, but with multiple ExpressRoute circuits, you can enable Global Reach: it lets two different locations, each with their own ExpressRoute circuit, establish connectivity to each other via the Microsoft backbone network (from one meet-me facility, over the backbone, to the other meet-me facility, and down to the other location). This is an additional configuration option available when you have multiple circuits and multiple locations of your own — maybe as a backup, in case of some failure on your regular network, letting you fail over to ExpressRoute Global Reach to keep services running.

### ExpressRoute FastPath

As discussed with private peering, gateways run within the virtual network and perform a couple of functions: BGP (learning and exchanging routes, then plumbing them into the virtual network so resources know where to send traffic destined for an on-premises IP), and they're also part of the data path.

Incoming data flows to a Microsoft Enterprise Edge router, which sends it to the gateway, which then sends it through — the gateway is part of the data path for data coming into the virtual network, because the MSEE needs a valid destination. It's not part of the path for data leaving the virtual network toward on-premises — that traffic flows directly to the MSEE and out. So the gateway isn't used for outbound traffic from your virtual network, but it is in the path for incoming traffic, which introduces some latency.

**ExpressRoute FastPath** removes the gateway from that data path, achieving higher throughput since the gateway is no longer in that flow at all. The gateways are still needed for BGP — that function remains — but that's now all they're doing.

> **Exam tip:** Enabling FastPath requires the `Ultra Performance` or `Gateway 3AZ` (zone-redundant) SKU. If there's ever a problem with FastPath, it fails back to using the gateway for the data path until the bypass can be re-established. There are limitations worth checking in the documentation — around load balancers, some gateway transit nuances, some ExpressRoute Direct specifics, and Private Link support when using 100 Gbit/s ExpressRoute Direct.

Overall FastPath is great, since you're no longer limited by the gateway's throughput and latency is reduced, but there are a few limitations worth being aware of.

## Controlling traffic flows

Within a virtual network, traffic can generally flow freely — if internet access is available, it can talk to the internet and get a response, but can't just receive from the internet; it can talk to any peered virtual networks and connected networks. But often you want to control that flow — segmenting so some resources can talk to each other, only a certain port is allowed from the internet, and so on. There are a few approaches.

### Azure Firewall

A firewall is great for deep packet inspection — layer 4 (network) and layer 7 (application), where it can look at fully qualified domain names, do TLS inspection (cracking open the packet to see the full URL and path, enabling categorization), and do destination NAT for a target based on port. It's a very rich, powerful solution for these kinds of inspection.

Azure Firewall lets you define rules at layer 4 (understanding things like port and protocol, but not fully qualified domain names or paths) and layer 7 (the application layer), where, among other things, TLS inspection lets you use categories based on the path, and there's an intrusion detection and prevention system.

Capabilities vary by SKU. There are three: `Standard`, `Premium`, and `Basic`.

> **Exam tip:** `Premium` adds capabilities enabled by TLS inspection — full URL as part of filtering, full URL as part of web categories, and an intrusion detection and prevention system (IDPS). `Basic` is designed for lower throughput, has a better price point, and more limited capabilities — for example, Threat Intel in Basic only alerts, it doesn't stop traffic.

In the demo, a comparison table lets you drill into the specific capabilities of each SKU to decide which is right for you, based on scale and functionality needed.

To use this, you need to route traffic through it — using UDRs, for example a `0.0.0.0/0` route sending everything through Azure Firewall. The appliance then has to process all of that traffic.

> **Exam tip:** Azure Firewall is deployed in a dedicated `AzureFirewallSubnet` sized `/26`. It's fully managed, built on VM Scale Sets, and auto-scales as needed.

But sometimes you don't need all that capability, and the alternative is network security groups.

### Network Security Groups

Network security groups (NSGs) can be applied at the subnet level or at an individual NIC — generally, applying at the subnet is easier to manage; applying at the NIC level gets complicated fast.

> **Exam tip:** An important point: enforcement always happens at the NIC level, regardless of where you applied the NSG. It is not an edge device — even applying it at the subnet level, it still inspects traffic even within the same subnet.

An NSG must be in the same region and subscription as the v-net it's used with. There are default rules, and rules are made up of IP ranges, tags, ports, and actions.

In the demo, a set of inbound and outbound default rules is shown — always at the lowest priority (higher number = lower priority). These defaults can be toggled to show or hide. They allow: traffic from the virtual network to the virtual network; traffic from the Azure load balancer to anything; everything else denied. For outbound: anything in the v-net can talk to each other; anything can talk out to the internet; everything else blocked. Note there's no "allow from internet" rule by default — nothing from the internet can talk in by default.

> **Exam tip:** An important nuance: the `VirtualNetwork` tag is a special tag meaning the known IP space — not literally "this virtual network." It includes all v-nets peered to it, and any IP space connected via ExpressRoute private peering or site-to-site VPN — think of it as all known, connected IP space. The `Internet` tag correspondingly means all IP ranges not part of that known space, reachable via the internet.

### Service tags

Beyond special tags like `VirtualNetwork` and `Internet`, there's a whole set of service tags for various Azure services.

The point of a service tag is to represent a broad and potentially changing range of IP addresses used by a particular Azure service (e.g., Azure Storage). Since tracking and updating all of those manually is impractical, a service tag represents all the IP addresses a given service uses in a given region (e.g., `Storage.WestUS3`).

In the demo, creating an outbound rule with source `VirtualNetwork` and destination set to a service tag (a specific Storage service in West US 3), plus a specific port — say, HTTPS only, to ensure TLS encryption to the API — shows how service tags represent Microsoft-maintained sets of IP addresses, simplifying rule creation.

> **Exam tip:** You can download current ranges behind service tags at a point in time, if you have your own firewall, for example. Network security groups are stateful — allowing outbound traffic to storage lets you automatically receive the response.

Combining source, destination (IP range or service tag), protocol, port, and action (allow/deny), you can build a sequence of rules controlling traffic flow across subnets (e.g., front-end can talk to the internet on port 443, front-end can talk to middle-tier on port 443, middle-tier can talk to back-end on port 1433 for SQL, while back-end and middle-tier can't reach the internet directly or on any other port).

> **Exam tip:** Network security groups are part of the Azure virtual switch and don't add significant overhead; they're free and not subject to throttling, unlike what Azure Firewall might potentially need to do.

NSGs are used behind the scenes by things like just-in-time VM access: requesting RDP access adds a temporary rule allowing port 3389 from your current IP address for 8 hours, and then removes it.

### Application Security Groups

Working with rules based on IP ranges is fine until things get more complicated — for example, if you have instances of a service in different subnets that don't form a contiguous IP space, your rules get messy. That's where Application Security Groups (ASGs) come in — basically a tag.

An Application Security Group is essentially just a value (tag) tied to a region.

> **Exam tip:** An Application Security Group must be in the same region as the resource it's applied to.

You create an ASG (essentially just a name, tied to a region) and then assign that tag to a resource — for example, to a VM's NIC, in its network settings, under `Application security groups`.

Once assigned, the source or destination of an NSG rule can be an Application Security Group instead of an IP range — for example, "SQL VMs can be talked to over 1433." This is far more flexible than IP-based restrictions, especially when resources are scattered across different subnets.

Another nice thing you can do with this: if you've detected that a VM is compromised, you can simply tag it with an ASG like "quarantined," and your NSGs can block everything for resources with that tag, except what's needed to remediate it. ASGs are really nice for both maintenance and simplifying rule logic in general.

## Azure Virtual WAN

**Azure Virtual WAN** is really powerful for more complicated environments, when you don't want to be managing all of the connections, gateways, transitivity, and firewalls yourself. It's a managed hub — it removes you, as the customer, from worrying about routing, VPN gateways, and ExpressRoute gateways.

The way it works: you deploy an Azure Virtual WAN instance (a larger top-level resource), and then in each region you want it used, you deploy a particular hub. For example, with three regions, you could deploy a hub in each. You then define what each hub connects to — for example, one hub connected to a certain on-premises network, another connected to `v-net 1` and `v-net 2` — essentially peering connections.

There are two different SKUs, `Basic` and `Standard`. You'll also sometimes see the term "secured hub" — a hub with Azure Firewall deployed as well.

> **Exam tip:** `Basic` supports only site-to-site VPN, and doesn't mesh different hubs together — no peering connections between hubs, no v-net transiting (though hubs can be in different subscriptions or tenants, there's no peering between the hubs themselves). `Standard` supports site-to-site VPN, point-to-site VPN, and ExpressRoute, supports inter-hub communication, makes v-nets transitive, and adds a whole set of additional capabilities — for example, you can deploy a partner NVA (like Barracuda) into the hub. You can upgrade from Basic to Standard.

The choice comes down to scale: a more basic location can use Basic with a simple site-to-site VPN, but for any larger environment you'll probably want Standard, especially for inter-hub and v-net-to-v-net transiting through the virtual hub, and secured capabilities using network virtual appliances.

## Azure Virtual Network Manager

A newer component, **Azure Virtual Network Manager**, ties together a lot of what's been discussed — network security groups (created in a region, linked to subnets), user defined routes (also linked to subnets), and peering connections. As an environment scales, it gets really hard to manually manage all this routing, security, connectivity, and IP address management. Azure Virtual Network Manager gives you central management of all of these aspects.

Billing is based on the number of virtual networks it's managing (historically it was based on the number of subscriptions under its scope, but this is moving to the actual v-nets managed). It also enhances functionality — for example, for peering, there's normally a finite number of peerings possible; connectivity through Azure Virtual Network Manager lets you link a much larger number together.

The building block is a **network group** — a group of virtual networks, which can be created statically or dynamically.

In the demo, in Azure Virtual Network Manager, a network group is created, with group members added either manually (source: `manually added`) or dynamically via Azure Policy — based on a tag, subscription name, subscription tag, resource group name, or other criteria. This lets, for example, any new virtual network created in a Dev subscription automatically be added to a Dev network group — since Dev networks tend to have similar communication rules, connectivity types, IP ranges, and routing — without manually thinking about each one.

Once you've defined network groups (statically and/or dynamically populated), you can do several things with them.

**Connectivity.** As discussed, at large scale, classic peering gets very complex, especially for full mesh connectivity. Azure Virtual Network Manager offers three connectivity models:

- **Mesh** — as the name suggests, everything can talk to everything. But this isn't implemented using regular peering — instead a "connected group" is created, a construct only available through Azure Virtual Network Manager. You won't see traditional peerings; you can fit far more networks into this model than regular peering would allow, and it's much cleaner to manage.
- **Hub and spoke** — uses classic peering: spokes connect to the hub via peering connections.
- **Direct connect** — an addition on top of hub and spoke: enabling it adds an overlay connected group — spokes still use peering to talk to the hub, but talk directly to each other when needed, again not via a mesh of peerings, but through the same connected group mechanism, applied here to spoke-to-spoke traffic.

> **Exam tip:** A single virtual network can be part of multiple connectivity configurations, but only **up to two** mesh configurations. When you create or remove virtual networks, the relevant connectivity configurations update very quickly — especially powerful combined with dynamic network group membership: e.g., creating a new Dev or Prod network automatically picks up the right connectivity.

**Security admin rules.** These are similar to network security groups, but centrally managed, and importantly, applied **before** network security groups — acting like a funnel ahead of regular NSGs. Unlike NSGs, they also support `ESP`/`AH` (for IPsec-type scenarios), and:

> **Exam tip:** In addition to `allow` and `deny`, security admin rules have a third option — **always allow** — meaning this rule cannot be overridden (bypassed) by local network security groups. This is useful for critical infrastructure (e.g., patching infrastructure or domain services) you don't want accidentally blocked by a local NSG. With regular `allow`, traffic that reaches this stage still goes on to be evaluated by NSGs; with `deny`, traffic is stopped immediately, before it ever reaches NSGs.

In the demo, under `Configurations`, you can create a connectivity configuration, choosing a topology — mesh (using a connected group) or hub and spoke, with an optional direct connectivity add-on (again via an overlay connected group) — and a security admin configuration, made up of rules similar to NSGs, but with additional protocols (`ESP`, `AH`, `Any`) and the `always allow` action.

Other capabilities include creating a routing configuration with multiple user defined routes, applied to a network group and, correspondingly, all v-nets in it. There's also **IP address management (IPAM)** — managing IP space can get complicated (avoiding overlap, tracking usage). Azure Virtual Network Manager lets you define pools of IP addresses, and when virtual networks are created, they're assigned addresses from the configured pool, ensuring no overlap and optimal use of your IP space. Think of this as centralized, at-scale management of the key network attributes you care about.

## Service endpoints

Network security groups focus on the flow of traffic within, into, and out of a virtual network — great, and definitely needed, but it doesn't help with the idea of a PaaS service that isn't part of your virtual network, where you want to control what can talk to it — that's not typically something an NSG helps with.

Picture a virtual network with a subnet containing a resource, and a `storage account 1`. Nearly all PaaS services in Azure have a firewall of their own, and in that firewall you can say, for example, "only allow talking to me from certain IP addresses" — which sounds great, right? But it doesn't actually work if you try to restrict it to your subnet's own IP range.

> **Exam tip:** The reason it doesn't work: your virtual network is made up of a non-routable IP address range, so when it talks to the storage account, it's not using its normal native IP address — it will use whatever the NAT address is for talking to the internet (even though the traffic doesn't actually bounce via the internet, this is still the address the storage account sees). So you can't just restrict access to your subnet's own IP range — the service won't recognize it.

That's exactly what **service endpoints** solve: they make a specific subnet known to a particular type of service (like storage) and establish a dedicated path. This does two key things: for things in that subnet only, you get enhanced routing (avoiding bouncing out to the internet or the regular route), and that subnet/v-net becomes a known quantity to the service, so you can say on the firewall, "yes, allow this v-net/subnet through."

It's a two-stage process — enable the service endpoint, then configure access restrictions on the service.

In the demo, on a storage account's `Networking`, you can enable `Enable from selected virtual networks and IP addresses`, add an existing virtual network, and it lists all the subnets — you'll be prompted that a service endpoint is required (and the portal can create it for you if it doesn't exist), enabling just that particular subnet. You can also manually enable service endpoints directly on a subnet.

### Service endpoint policies

What service endpoints don't help with is data exfiltration. Picture a resource in a subnet that's allowed to talk to `storage account 1` — great, restricted to only things in this subnet — but there's another "evil" storage account (say, `storage account 3`) created by an attacker who's compromised a resource in the subnet, intending to copy data from the legitimate storage account into it.

Once a service endpoint for storage is enabled, you can create a **service endpoint policy**, allowing only `storage account 1` and `storage account 2`, and associate that policy with a particular subnet. Trying to talk to `storage account 3` would then be blocked — preventing data exfiltration.

In the demo, a `Service endpoint policy` can specify particular services (only `Storage` in the example) and add particular allowed resource instances, then be associated with a subnet (or configured directly on the subnet) — a way to restrict and lock this down.

> **Exam tip:** It's worth noting that service endpoints aren't getting much investment today — private endpoints, covered next, are the more preferred, actively developed direction.

## Private link

The term "private link" has come up several times already. Most Azure PaaS services have a public endpoint — an internet-routable address, always sitting behind firewalls and authentication, rarely open by default. Talking to that public endpoint from an Azure resource doesn't hairpin via the internet; it stays on the Microsoft network. But sometimes you don't even want to talk to a public endpoint — you want it unavailable entirely.

**Private Link** enables PaaS services to have an IP address in your virtual network / subnet, representing a connection directly to that specific service instance.

Create, say, `storage account 5`, with a public endpoint by default that you want to disable entirely. What you create instead is a **private endpoint** — just an IP address from the subnet where you create it, but connecting directly to a particular instance of a service.

Because it's just an IP address, anything with a network path to it can talk to it — other subnets in the v-net, peered v-nets, on-premises via site-to-site VPN or ExpressRoute private peering. It's a very useful capability: anything with a path to the IP can use the service. Private Link is supported across storage, SQL, Cosmos DB, managed services, and much more — today, it's the primary direction of development for nearly everything.

### DNS considerations

However, when we talk to resources, we don't actually initially talk directly to an IP address — we use a name, and that name matters for encrypted communication, since the certificate is tied to it. For example, `storage account 5` would normally be `storagecount5.blob.core.windows.net`, resolving to the public IP (which you've now disabled).

In the demo, under a storage account's `Endpoints`, alongside the usual name, there's an unusual variant with `privatelink` inserted between the storage account name and `blob.core.windows.net` — this is because a private endpoint has been enabled for that storage account into one of the virtual networks.

You need to talk to the storage account using the correct name — you can't just talk directly to the private endpoint's IP address, because the FQDN wouldn't match, the certificate wouldn't validate, and encrypted communication would fail. Enabling Private Link creates an additional alias — the storage account name with `privatelink` inserted before `blob.core.windows.net` — and this alias resolves to that private IP address. So talking to the ordinary name `sa5.blob.core.windows.net` needs to actually resolve to that private IP address — which means DNS configuration needs to change.

This sounds complex, but the good news is **Azure Private DNS**. You create a private DNS zone named `privatelink.blob.core.windows.net`, and Private Link integrates with it: when creating the private endpoint and it gets that name, it can auto-register (if integration is enabled) — for example, `storage account 5` automatically gets a record in this zone. You then link that zone to the virtual network — resulting in name resolution working correctly: looking up `sa5.blob.core.windows.net` reveals the `privatelink` alias, finds the linked private zone that maps it to the private endpoint's IP address, and you get an encrypted connection using the correct name.

If a peered v-net also wants to talk to this resource, it has an IP path to the private endpoint, but it also needs the same name resolution, so the zone would need to be linked to that v-net as well. If access is needed from on-premises, you'd either need to create the record in an on-premises zone, or use something like Azure Private DNS Resolver to forward the request for resolution.

> **Exam tip:** A key point — the private endpoint itself is very simple to set up; the DNS configuration is what most often complicates things about Private Link.

In the demo, a configured private endpoint is shown, linked to a private DNS zone; inside that zone, a record for `storage account 10` maps to the expected IP address; the zone is linked (`Virtual network link`) to a specific virtual network. Looking at the virtual network's connected devices shows the private endpoint as a virtual NIC. Running `nslookup` from within a VM connected to that network resolves the correct alias to the internal IP address, allowing access via the private endpoint.

This can seem a bit convoluted — the complicated part of Private Link isn't the private endpoint itself, it's the DNS configuration you have to get right around it. It's also worth noting that because this is instance-specific, it helps prevent data exfiltration, since only the specific instances you allow can be communicated with.

There's also a feature that can add more complexity, called internet fallback. Imagine `storage account 4` has a private endpoint linked to `privatelink.blob.windows.net` in its virtual network, while `storage account 6` has a private endpoint in a completely different v-net, using a different copy of that same-named zone (you can have multiple copies with the same name). Trying to talk to `storage account 6` from the first network, the client discovers the `privatelink` alias, but the local copy of that private zone doesn't have a record for `storage account 6`, so it returns "record doesn't exist" (NXDOMAIN), and the request just fails — even though `storage account 6` might have deliberately left its public endpoint open for other things to use.

For that scenario, you can turn on internet fallback on the link: instead of just returning NXDOMAIN, it will use the public DNS resolver to try to resolve it, and give you that response instead. This is a fairly recent capability.

In the demo, on a private DNS zone's `Virtual network link`, `Fall back to internet` is shown enabled — when trying to access a storage account that has a private endpoint in a different v-net using a different copy of `privatelink.blob.windows.net`, instead of NXDOMAIN, the internet resolver is used and access still succeeds.

### Private link service

Another capability is exposing your own custom services via Private Link. As mentioned earlier, there are scenarios where a different virtual network might have overlapping IP space, or you simply don't want to peer them — maybe it's a customer, and you don't want full IP routing between you. In that case, you can put a set of resources behind an Azure Load Balancer for resiliency, create a **private link service** instance, and then create a private endpoint in a different subnet pointing to it.

> **Exam tip:** Private Link performs network address translation, so IP ranges on either side could actually overlap — it doesn't matter. This is especially useful when peering is undesirable — for example, for a customer you just want to expose a specific service to without full IP routing between networks. Service endpoints and private endpoints can be used in the same subnet at the same time, if needed.

> **Exam tip:** An important cost distinction: service endpoints are free, but only work for things located in the subnet where the service endpoint is enabled. Private endpoints cost money, but work for anything with a network path to the IP address (peered v-nets, ExpressRoute private peering, site-to-site VPN), provided consistent name resolution (DNS) is in place — adding the complexity discussed above.

Today, for most companies, Private Link is the preferred direction of development. As mentioned, it removes the need for peering when peering was only there for this purpose.

## DNS in Azure

DNS is critical to almost everything — most of the time when there's a network problem, DNS ends up being the culprit. That's because we're not good with IP addresses, and we don't like using them directly — we want to be able to change the IP, reconfigure services — so we prefer to talk to a name that then resolves to an IP address, and DNS is the hierarchical solution for resolving a name to an IP address. It's a very powerful capability, and Azure has its own DNS services.

Azure has native DNS resolution, the default for a virtual network, resolving various names needed as part of your IP configuration. You can change a virtual network's DNS config — pointing it, for example, at your own DNS service, such as domain controllers. Azure provides both public and private DNS zones.

### Public DNS services

Both types are global resources — they don't live in just one region. You can create a public DNS zone, create a private DNS zone, and link a private DNS zone to regions and v-nets all around the world.

For public zones, you can create the usual records — the host record (A and AAAA for IPv6), CNAME, and other types — for when you want name resolution via the internet-routable DNS system. You have to set up Azure DNS as authoritative for the zone — you can't just create a public DNS zone in Azure and expect it to work; it's a hierarchy, so if you created `ntffaq.com`, you'd need to tell the `.com` registry to send requests about that domain to the Azure DNS servers that are authoritative for that zone.

Beyond manually creating all these records, a number of services integrate with Azure DNS directly, addressing one of the challenges DNS faces — dangling DNS records. Imagine you have an Azure resource with a certain assigned IP address and name (e.g., `app.appservice.azure.com`), and on your own DNS servers you create an alias, like `www.mystuff.com`, pointing at it. If you later delete that resource and forget to remove the alias, it becomes a dangling record, pointing at something that no longer exists. An attacker could then create their own resource with that same original name (e.g., `app.appservice.azure.com`, if freed up after deletion), and your old record would end up pointing at their resource.

Many Azure services take steps to protect against this, and one of the really nice things Azure DNS does is that many services integrate directly with public DNS.

In the demo, a public DNS zone shows both manually added records and records that are alias records tied directly to an Azure resource — you don't have to manually configure these. Because it's tied to the Azure resource, if that resource is deleted, the record is automatically deleted too, so no one can hijack the name afterward.

> **Exam tip:** Where possible, avoid manually aliasing to another name when it's an Azure resource — use the service integration instead. It's a much safer, more preferred option.

### Private DNS zones

Private zones are also supported. For a private zone, you can use any name you want, since it's not publicly usable. You can manually create records of any type — A, CNAME, MX, PTR, SRV, SOA, TXT — since it's purely internal, you have full control to create whatever you need.

Records can be manually created, but can also be automatically created by different types of resources. Importantly, a virtual network is linked to private DNS zones, and one of those linked zones can be used specifically for resolution purposes.

> **Exam tip:** Important numeric limits for private DNS zones:
> - A virtual network can be linked (for resolution purposes) to **up to 1,000** private DNS zones (you might need different zones for private links to blob, Data Lake, queue, table, Postgres, and so on — there can be a lot of them).
> - A single private DNS zone can be linked to **up to 1,000** different virtual networks.
> - A virtual network can be linked to **only one** private DNS zone for automatic registration purposes — meaning that as resources are created in that virtual network, records are automatically created for them in that one zone (e.g., a resource named `Bob` gets a record `bob.<zone-name>`).
> - A single private DNS zone can support automatic registration for **up to 100** different virtual networks.

So many private DNS zones can be used for resolution from a single virtual network, but only one of them can be the auto-registration target for that network. It's a global resource — usable from any region, any subscription, any v-net, any tenant, with the right permissions.

There's one more element — the **Azure Private DNS Resolver**, which has two different benefits. There are scenarios where you have private DNS zones linked to your virtual network, but on-premises DNS services also need to resolve records in those private zones — you can't just link an on-prem DNS server to a private DNS zone directly (relevant, for example, to Private Link scenarios).

The Private DNS Resolver uses different subnets — essentially two different types of endpoints. One provides a target IP that other DNS services can use for resolution, which talks to Azure DNS resolution, and all the private DNS zones linked to it can return the records. It's also a way for Azure DNS itself to query other DNS servers for forwarded zones — if something talking to Azure DNS is configured to reach, say, `savtech.net`, and it has a path to a DNS server for that, it can go get the answer and hand it back to Azure DNS.

It can operate in both directions, and you can use one or both services — each needs its own subnet (around `/28`). You can link the Azure DNS Resolver to multiple different subnets, and they don't all need the same connectivity — as long as one of them has the endpoint to resolve something, it can do it on behalf of the other subnets it's linked to. It's a really nice built-in capability you can deploy when needed.

## Close

That's it — a lot of ground covered in networking. It's a fascinating area, always growing. Hope that was useful — until next video, take care.
