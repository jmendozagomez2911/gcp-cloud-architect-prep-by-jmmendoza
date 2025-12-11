# 🤖☁️ **README — Module: Introducing Google Cloud**

**Goal:** Build the foundation the **Professional Cloud Architect** exam assumes you already understand:
what cloud computing *actually is*, why Google’s platform model matters, how to choose **service models**, how to design with **regions/zones**, and how to justify decisions using **security, reliability, and cost governance**.

**Read me like this:**

1. Cloud definition → 2) Evolution → 3) Service models → 4) Global infrastructure → 5) Security story → 6) Open source & multi-cloud → 7) Pricing + cost controls → 8) Architect-style decision rules.

---

## 1) ☁️ What “cloud computing” really means

Cloud is not “someone else’s server” — it’s a way of consuming IT with **five behaviors** that change your operating model:

### 1) **On-demand self-service**

Instead of filing tickets and waiting for hardware, you can provision resources yourself in minutes.
This matters architecturally because it enables faster experimentation, faster releases, and easier scaling of teams. It also changes governance: you need IAM and policies to control who can create what.

### 2) **Broad network access**

Resources are reachable over the internet from anywhere.
Architect implication: you must think about **latency, identity, and network security** because your systems are no longer bound to a single physical office or data center.

### 3) **Resource pooling**

Google maintains a massive shared pool of infrastructure and allocates what you need from it.
You don’t need to know the physical machine location (and usually cannot control it precisely).
Architect implication: you focus on **logical design** (projects, regions, IAM, service boundaries) instead of hardware-level planning.

### 4) **Elasticity**

You can scale up quickly for spikes and shrink during quiet periods.
Architect implication: design for variable demand. This is why autoscaling and managed platforms are so emphasized in GCP.

### 5) **Measured service / pay-as-you-go**

You are billed for usage rather than ownership.
Architect implication: cost becomes part of architecture. You must design with **budgets, alerts, autoscaling, right-sizing, and quotas** in mind.

> **Architect exam tip:**
> If a scenario asks “why cloud is better than on-prem,” don’t answer just “cheaper.”
> The stronger answer is: **speed of delivery + elasticity + operational efficiency + pay-for-use**.

---

## 2) 🧭 The three-wave evolution (why Google pushes managed platforms)

This section is not history trivia — it explains Google’s architectural philosophy.

### Wave 1 — **Colocation**

Companies reduced capital spend by renting physical space in data centers instead of building them.
But they still purchased and managed their own hardware.

### Wave 2 — **Virtualized data centers**

You get the same data center building blocks (compute, storage, network), but virtualized.
You still control configuration and operations, just with better flexibility.
Architect implication: this still requires meaningful infrastructure management and slows innovation.

### Wave 3 — **Container-based, fully automated cloud**

Google moved beyond virtualization because their business needed a faster model.
This wave emphasizes:

* **automation**
* **elastic platforms**
* **managed services**
* **scalable data systems**

Architect implication:
Google expects you to design using **higher abstraction levels** whenever possible.

> **Architect exam tip:**
> If two services can meet requirements, the exam often prefers the one that reduces operational burden.

---

## 3) 🧩 Service models (with real architectural meaning)

The exam tests not just definitions, but whether you understand **who manages what** and why that matters.

### **IaaS**

You receive raw virtual compute, storage, and networking.
You choose VM types, manage OS patching, scaling logic, and a lot of reliability engineering yourself.

* **GCP example:** Compute Engine
* When it’s the right answer:

    * You need specific OS control
    * You’re migrating legacy systems
    * You require unusual drivers or low-level networking behavior

### **PaaS**

The platform provides runtime and infrastructure integration so that your code can focus on business logic.
You trade some control for faster delivery and lower ops.

* **GCP example:** App Engine
* When it’s the right answer:

    * You want standardized deployment
    * You want the platform to handle scaling and much of the maintenance

### **Serverless**

This is more than a buzzword. It means:

* You stop thinking about servers entirely.

* You pay for execution/usage.

* Scaling is inherent.

* **GCP examples:**

    * **Cloud Run**: run containerized services with full management
    * **Cloud Run functions**: lightweight event-driven code

Architect value:
Serverless is a strong default when you need **rapid delivery and elasticity** with minimal ops.

### **SaaS**

You consume a complete application.
This sits outside infrastructure design but is relevant in enterprise architecture because SaaS often becomes part of workflows.

* **Examples:** Gmail, Docs, Drive

---

### A more “architect-readable” selection guide

| You need…                              | Likely best fit           | Reasoning the exam wants                                        |
| -------------------------------------- | ------------------------- | --------------------------------------------------------------- |
| Full control, lift-and-shift           | **Compute Engine (IaaS)** | You must manage OS/runtime; more control but more ops           |
| Managed runtime for apps               | **App Engine (PaaS)**     | You focus on code; platform handles much of scaling/maintenance |
| Container portability with minimal ops | **Cloud Run**             | Good balance: standard containers + fully managed               |
| Event-driven glue logic                | **Cloud Run functions**   | Small units of code triggered by events, pay-per-use            |
| End-user productivity tools            | **SaaS**                  | You don’t build/maintain the app stack                          |

> **Architect exam tip:**
> The correct answer is often the **highest managed level** that still satisfies constraints.

---

## 4) 🌍 Global infrastructure: regions, zones, and architectural tradeoffs

Google’s network and global presence exist to support three design outcomes:

* **low latency**
* **high availability**
* **disaster resilience**

### **Regions**

A region is an independent geographic area.
Architecturally, it is your primary unit of:

* data residency
* risk separation
* service placement

### **Zones**

Zones are deployment areas within a region.
They are designed so that a failure in one zone doesn’t imply a failure in another.

### Why use **multiple zones** in one region?

To design **high availability** without moving too far from your users.
This is the common pattern for production apps that need resilience but don’t require global distribution.

### Why use **multiple regions**?

Two big reasons:

1. **Proximity to users worldwide**
2. **Protection against region-wide outages**

### Multi-region services

Some services provide built-in multi-region replication strategies.
The transcript mentions Spanner multi-region configurations to improve availability and low-latency reads across locations.

> **Architect exam tip:**
>
> * **HA inside a geography** → multi-zone architecture.
> * **Global DR + worldwide user base** → multi-region architecture.

---

## 5) 🔐 Security: what you should *take away* as an architect

This section is teaching you to trust the platform’s **defence-in-depth** approach — but also to know how to talk about it.

### The key architectural message

Security is not one feature — it is layered:

* physical
* hardware
* network
* identity
* data
* operations

### Highlights worth remembering

* **Custom hardware + secure boot**
  Protects against tampering and ensures only trusted software runs.
* **Encryption in transit by default**
  Especially important for inter-service traffic and global workloads.
* **Strong identity controls**
  Risk-based authentication and multi-factor support.
* **Encryption at rest**
  Applied at storage service layer with centrally managed keys.
* **Internet edge protections**
  TLS termination and built-in DoS mitigation.
* **Operational security discipline**
  Red teams, monitoring, least privilege for admins, strong code review and secure libraries.

> **Architect exam tip:**
> If a question contrasts cloud vs on-prem security,
> answer with **layered security + default encryption + identity-first model**.

---

## 6) 🔓 Lock-in concerns: Google’s portability narrative

Google tries to reduce “fear of being trapped” by emphasizing:

* **open source** ecosystems
* **interoperability**

### What matters for architects

* **Kubernetes / GKE** = portability of microservices across clouds
* **Observability across providers** = operational visibility even in hybrid/multi-cloud strategies

> **Architect exam tip:**
> If a scenario worries about vendor lock-in,
> your design justification should mention:
> **containers + Kubernetes + open standards**.

---

## 7) 💸 Pricing + cost governance (more than “cheaper cloud”)

The transcript’s cost section isn’t about exact numbers — it’s about **architectural responsibility**.

### Pricing behaviors

* **Per-second billing** reduces waste for bursty workloads.
* **Sustained-use discounts** reward consistent VM usage.
* **Custom machine types** enable right-sizing.

### Guardrails you should design into architectures

* **Budgets** at billing account or project level
* **Alerts** at thresholds to catch abnormalities early
* **Quotas** to prevent runaway consumption from bugs or attacks

    * Rate quotas (time-window limits)
    * Allocation quotas (resource count limits)

> **Architect exam tip:**
> “Prevent accidental huge bills” is a **governance architecture** question.
> The correct trio is usually: **budgets + alerts + quotas**.

---

## 8) 🧠 Cloud Architect exam mindset (the real point of this module)

You are being trained to answer like a platform strategist:

### The default design bias

* Choose **managed services** first.
* Use **serverless/container-managed** when feasible.
* Use **multi-zone** for critical availability.
* Use **multi-region** when business requires global resilience or global users.

### The justification chain the exam likes

When explaining your choice, connect:

* **Requirements** → **service model** → **reliability** → **security** → **cost controls**

Example reasoning style:

> “We chose a managed container platform to reduce operational burden,
> run across multiple zones for high availability,
> implement IAM least privilege and encryption defaults,
> and control costs with budgets, alerts, and quotas.”