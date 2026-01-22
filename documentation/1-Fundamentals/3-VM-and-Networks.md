# 🤖🌐 **VM and Networks in the Cloud (Cloud Architect focus)**

**Goal:** Understand how Google Cloud designs **compute + networking** so you can make correct architecture choices about:
VPC design (global vs regional), subnetting, routing/firewalls, VM sizing & scaling, load balancing types, DNS/CDN, and hybrid/multi-cloud connectivity (VPN/Interconnect/Peering).

**Read me like this:**

1. VPC fundamentals → 2) Compute Engine VM options & pricing models → 3) Scaling + quotas (scale out vs up) → 4) VPC routing/firewalls/peering/shared access → 5) Load Balancing types → 6) Cloud DNS + CDN → 7) Hybrid connectivity decision tree → 8) Lab takeaways → 9) Exam cheats + quiz answers.

---

## 1) 🧱 VPC fundamentals (what a VPC is and why GCP is “different”)

### What a VPC is (in practical terms)

A **Virtual Private Cloud (VPC)** is your private network **inside** Google Cloud.
It gives you the same capabilities you expect in a private data centre network—running workloads, isolating data, controlling traffic—while being hosted on Google’s public cloud infrastructure.

What you get architecturally:

* **Isolation:** your network is logically separated from other customers.
* **Control:** you decide IP ranges, subnet layout, firewall rules, and routes.
* **Internet connectivity:** you can expose services publicly or keep them private.
* **Enterprise patterns:** segment workloads (prod vs dev, public vs private tiers).

### The part that surprises people: **VPC networks are global**

In Google Cloud, a **VPC network is a global resource**.
That means you create one VPC and it can have subnets in **any region worldwide**.

But **subnets are regional** (more on that below). The combination matters:

* Global VPC → consistent network identity across the world.
* Regional subnets → you control IP ranges and placement per region.

### Subnets and zones (how placement actually works)

* A **subnet is regional**, and it **spans all zones inside that region**.
* VMs in different zones can still be “neighbors” on the same subnet (same IP range, same routing domain).

Architect benefit:

* You can design multi-zone architectures for resilience **without** creating separate subnets per zone.

### Expanding subnet size without breaking VMs

You can **expand** a subnet’s IP range without impacting existing VM configurations.
This is important for real-world growth: you can start smaller and scale address space later.

> **Architect exam tip:**
> If the question mentions “global network layout, but resources in multiple regions,” remember:
> **VPC is global; subnets are regional**.

---

## 2) 🖥️ Compute Engine (IaaS) — what you control and how you deploy it

### What Compute Engine gives you

Compute Engine lets you run **virtual machines** on Google infrastructure:

* Choose CPU + memory (predefined types or **custom machine types**).
* Choose OS images (Linux/Windows provided by Google or custom images).
* Manage storage, networking, and instance lifecycle.

How you create VMs:

* Cloud Console (GUI)
* `gcloud` CLI
* Compute Engine API

### Marketplace as a VM “accelerator”

Cloud Marketplace can deploy a full solution (VM + software stack + configuration) quickly.
Architectural note: Marketplace reduces setup time, but you still own VM operations like patching/hardening depending on what you deployed.

---

## 3) 💸 VM pricing models (this is architecture, not accounting)

### Standard billing behaviour

* **Per-second billing** (with a one-minute minimum).
* **Sustained-use discounts** apply automatically for VMs running a significant part of the month (after a threshold, discounts apply incrementally).

### Committed-use discounts (predictable workloads)

If you know workloads will be stable, you can commit to a certain amount of vCPU/memory for **1 or 3 years** for large discounts.
Architect implication: this is a cost optimisation tool for steady baselines.

### Preemptible / Spot VMs (batch-friendly, interruption-tolerant)

These VMs are cheaper because Google is allowed to reclaim capacity:

* Best for workloads that can be **stopped and restarted** (batch, distributed processing, CI jobs).
* Preemptible: historically limited runtime (the transcript states up to **24 hours**).
* Spot: more features and **no maximum runtime**, but can still be terminated; pricing described as the same as preemptible in the transcript.

Architect implication:

* Only choose these if your system design tolerates interruption (checkpointing, retries, idempotency).

> **Architect exam tip:**
> If you see “huge savings, but may be terminated” → **Preemptible/Spot** is the intended answer, *and* you must mention “workload must handle restarts.”

---

## 4) 📈 Scale out vs scale up (autoscaling + quotas)

### Most customers scale out first

You *can* create very large VMs (useful for in-memory DBs or CPU-heavy analytics), but typical cloud-native architecture starts with:

* many smaller instances (**scale out**)
* behind a load balancer
* managed by autoscaling

### Autoscaling (what it really means)

Autoscaling adds/removes VMs based on load metrics.
Architecturally, autoscaling is only half the story—the other half is **load balancing**, otherwise traffic won’t be distributed correctly as instance count changes.

### Limits: machine family + zone quotas

* Max CPU per VM depends on **machine family**.
* Your ability to scale is also constrained by **quotas**, often **zone-dependent**.
  Architect implication: in real designs, quotas can block scale-out if not planned.

---

## 5) 🧭 VPC routing, firewalls, and connecting VPCs

### Routing (built-in)

GCP VPC has built-in routing tables:

* You don’t deploy routers for basic routing.
* VPC routes allow traffic:

    * between instances in the same network
    * across subnets
    * across zones
    * **without needing external IPs** for internal traffic

Architect implication:

* Internal multi-zone communication is a first-class feature.

### Firewall (global distributed)

GCP provides a **global distributed firewall**:

* You define rules once; enforcement is distributed.
* Supports both **ingress and egress** control.

A very exam-relevant concept: **firewall rules can target instances by network tags**.
Example pattern:

* Tag all web servers as `WEB`
* Create one firewall rule: allow ingress on 80/443 to tag `WEB`
  This scales better than chasing IP addresses.

### VPC-to-VPC connectivity

If different projects have different VPCs, you have options:

* **VPC Peering**: connects two VPCs so they can exchange traffic.
* The transcript hints at the more enterprise approach: using IAM-powered sharing so one project can use a VPC in another (this is the “shared VPC” idea, even though the transcript cuts off mid-sentence).
  Architect point: in enterprises, you often centralise networking in one “host” project and attach workloads from “service projects.”

> **Architect exam tip:**
> If the question says “multiple projects need to use the same centrally managed network,” think **Shared VPC** patterns rather than ad-hoc peering everywhere.

---

## 6) ⚖️ Cloud Load Balancing (how users reach autoscaled apps)

### What it does

A load balancer distributes client traffic across multiple backends, reducing overload risk and improving performance.

### Why GCP load balancing is architecturally attractive

* Fully managed (you don’t run load balancers on VMs).
* Handles multiple traffic types: HTTP(S), TCP/SSL, UDP.
* **Cross-region load balancing** with **automatic multi-region failover**.
* No “pre-warming”: you don’t file tickets for traffic spikes.
* Responds quickly to backend health and traffic changes.

### Load balancer families (OSI-layer mental model)

#### Application Load Balancers (Layer 7)

* For HTTP/HTTPS.
* Provide advanced features:

    * content-based routing
    * SSL/TLS termination
* Operate as reverse proxies.
* Can be external (internet-facing) or internal.

Use when:

* Web apps, APIs, microservices
* You need routing logic based on URL/host/path

#### Network Load Balancers (Layer 4)

Handle TCP/UDP and other IP protocols.

Two types described:

* **Proxy Network Load Balancer**

    * Terminates the client connection and creates a new connection to the backend.
    * More advanced traffic management.
    * Can support backends across on-prem and cloud environments.

* **Passthrough Network Load Balancer**

    * Does not terminate connections.
    * Preserves the original source IP.
    * Useful when apps need the client IP or direct server return patterns.

> **Architect exam tip:**
>
> * Need HTTP routing / TLS termination / content-based routing → **Application LB**.
> * Need TCP/UDP performance, potentially preserve source IP → **Passthrough NLB**.
> * Need L4 but with proxy features / advanced control → **Proxy NLB**.

---

## 7) 🌍 Cloud DNS + Cloud CDN (global presence for naming + performance)

### Cloud DNS

Cloud DNS is a managed DNS service running on Google infrastructure:

* Low latency, high availability.
* DNS records served from redundant locations worldwide.
* Programmatic management via console, CLI, API.
* Supports massive scale (millions of zones/records).

Architect implication:

* Use Cloud DNS for reliable global name resolution for GCP-hosted services.

### Edge caching and Cloud CDN

Google has a global edge cache system. Cloud CDN lets you use it:

* Cache content closer to users → lower latency.
* Reduce load on your origin servers.
* Can reduce costs by serving cached content at the edge.
* Enablement is simple once you have an **Application Load Balancer** (single checkbox concept).

Architect implication:

* For static/streaming assets and global user bases, CDN is a standard performance pattern.

---

## 8) 🔌 Hybrid & multi-cloud connectivity (choose the right link type)

Many orgs need to connect:

* on-prem networks ↔ GCP VPC
* other clouds ↔ GCP VPC

### Option A: Cloud VPN (tunnel over the internet)

* Fast to start.
* Uses the public internet.
* If you need **dynamic routes**, use **Cloud Router** with **BGP**.

    * Benefit: when you add a subnet in GCP, on-prem learns routes automatically.

Best when:

* You need private-to-private connectivity
* Your internet performance is acceptable
* You want a simpler, cheaper setup

### Option B: Peering (Direct / Carrier)

* Exchange traffic at a Google **Point of Presence (PoP)**.
* **Direct Peering**: you have equipment in a PoP.
* **Carrier Peering**: a partner/carrier helps you connect if you don’t have presence.

Important limitation in the transcript:

* Peering is not covered by a Google SLA (architect risk factor).

Best when:

* You don’t need private addressing end-to-end (or you accept the model)
* You want better performance than plain internet routing
* SLA is not the top priority

### Option C: Dedicated Interconnect (private direct connection)

* One or more private physical connections to Google.
* If topology meets specs, SLA up to **99.99%**.
* Can be backed up by VPN.

Best when:

* You need high performance + private connectivity + SLA
* You can colocate/install routing equipment at Google’s locations

### Option D: Partner Interconnect (via service provider)

* Connectivity via supported provider.
* Useful when:

    * you can’t reach a Dedicated Interconnect colocation facility, or
    * you don’t need a full 10 Gbps connection
* SLA up to 99.99% possible for qualifying topologies, but provider outages outside Google network are not Google’s responsibility.

Best when:

* You need private connectivity + decent SLA
* You cannot install/manage equipment at a Google PoP

### Option E: Cross-Cloud Interconnect

* Dedicated connectivity between Google Cloud and another cloud provider.
* Supports multi-cloud strategies with reduced complexity and encryption.
* Sizes: **10 Gbps** or **100 Gbps**.

Best when:

* You’re integrating GCP with another major cloud and need high bandwidth, dedicated links.

---

### The transcript’s “3 questions” decision framework (very exam-like)

1. Do you need **private-to-private** connectivity (on-prem private IP ↔ GCP private IP)?
2. Does your current internet connectivity meet **bandwidth/performance** requirements?
3. Are you willing/able to install and manage routing equipment at a Google PoP?

Then:

* Need private + internet is fine → **Cloud VPN** (optionally + Cloud Router for dynamic routes)
* Don’t need private + internet is fine → use **public IP** access
* Don’t need private + internet not performing → **Peering**
* Need private + want high performance + can install equipment → **Dedicated Interconnect**
* Need private + can’t install equipment → **Partner Interconnect**
* Need cloud-to-cloud dedicated link → **Cross-Cloud Interconnect**

> **Architect exam tip:**
> SLA requirement is a key differentiator:
> **Dedicated/Partner Interconnect can have SLA; Peering is explicitly called out as not SLA-backed.**

---

## 9) 🧪 Lab takeaways (what the lab is actually teaching)

The lab reinforces three practical truths:

1. **You can’t create a VM without a VPC network**
   VPC is foundational: no network → no routes → no firewall rules → no VM networking.

2. Default networks exist, but you should understand them

    * Default VPC includes:

        * subnets in each region
        * default routes
        * default firewall rules (including common ingress like SSH/RDP/ICMP)
    * In production, default permissive rules are often tightened.

3. Firewall rules directly explain connectivity outcomes

    * Allow SSH rule → SSH works.
    * Remove allow-icmp → external ping fails.
    * Remove allow-custom (internal) → internal ping fails.
    * Remove allow-ssh → SSH fails.

Architect implication:

* Connectivity troubleshooting and secure design always come back to:
  **subnets + routes + firewall rules + tags**.

---

## ✅ Exam cheats (memorise these “selection moves”)

* **VPC is global; subnets are regional.**
* Subnets span **all zones** in their region.
* Use **tags** to apply firewall rules to groups of instances (scales better than IP-based rules).
* Autoscaling needs load balancing to distribute traffic as instance count changes.
* Pick LB type by need:

    * HTTP(S) routing/TLS/content rules → **Application LB**
    * TCP/UDP with preserved client IP → **Passthrough NLB**
    * L4 with proxy behaviour → **Proxy NLB**
* Connectivity choices:

    * Private + internet ok → **VPN (+ Cloud Router/BGP if dynamic routes)**
    * SLA + private high performance → **Dedicated/Partner Interconnect**
    * Better public routing but no SLA → **Peering**
    * Multi-cloud dedicated high bandwidth → **Cross-Cloud Interconnect**

---

## ✅ Quiz answers

1. **What is the main reason customers choose Preemptible VMs?**
   ✅ **To reduce cost.**

2. **For which interconnect option is an SLA available?**
   ✅ **Dedicated Interconnect.**

3. **In Google Cloud VPCs, what scope do subnets have?**
   ✅ **Regional.**

4. **Which describes a VPC?**
   ✅ **A secure, individual, private cloud-computing model hosted within a public cloud.**

5. **How does global Cloud Load Balancing allow you to balance HTTP-based traffic?**
   ✅ **Across multiple Compute Engine regions.**
