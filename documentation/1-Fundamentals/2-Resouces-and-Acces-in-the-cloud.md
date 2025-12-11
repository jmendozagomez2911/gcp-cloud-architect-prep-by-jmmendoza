# 🤖🔐 **README — Module: Resources and Access in the Cloud**

**Goal:** Think like a **Cloud Architect** about *who owns what* in Google Cloud and *who can do what* to it. This module is about:
resource hierarchy → projects/folders/org → IAM (principals, roles, policies) → service accounts → Cloud Identity → access tools → Marketplace.

---

## 1) 🏗️ Google Cloud Resource Hierarchy (the backbone of governance)

Google Cloud organises everything into a **4-level hierarchy** (bottom → top):

1. **Resources**
   Concrete things you use:

    * VM instances
    * Cloud Storage buckets
    * BigQuery datasets/tables
    * Pub/Sub topics, etc.
      **Every resource belongs to exactly one project.**

2. **Projects**
   Logical containers for:

    * Resources
    * APIs/services enablement
    * Billing linkage
    * IAM policies and collaborators
      Architecturally, projects are your main **isolation and billing unit**.

3. **Folders (optional but important in enterprises)**

    * Group projects and subfolders.
    * Attach policies at a **department / environment / business unit** level.
    * Let you delegate admin responsibility (e.g. “Analytics Folder Admins”).

4. **Organization node (org)**

    * Top of the tree for a company domain.
    * Contains *all* folders, projects, and resources.
    * Where you define **company-wide** policies and constraints.

### Why this matters

* **Policies (IAM, org policies, constraints)** are attached to **org → folder → project → resource**.
* Policies are **inherited downwards**:

    * A role/policy on a folder applies to all projects and resources under it.
* Architecturally, you design:

    * **Org-level**: global constraints and security baselines.
    * **Folder-level**: per-department or per-environment rules.
    * **Project-level**: workload-specific permissions.

> **Architect exam tip:**
> When you see “we want to manage permissions for multiple projects at once” → **use folders + inheritance**, not copy-paste IAM to each project.

---

## 2) 📦 Projects in depth (identity + billing + API boundary)

Projects are the **main working unit** in Google Cloud.

### What projects are used for

* Enabling and managing **APIs and services**.
* Linking **billing accounts**.
* Adding/removing collaborators (IAM).
* Isolating workloads, environments (dev / test / prod), or customers (multi-tenant).

### Project identifiers (know the differences)

Each project has **three identifiers**:

1. **Project ID**

    * Chosen at creation (by you or auto-generated).
    * **Globally unique.**
    * **Immutable** after creation.
    * Used in **APIs, gcloud commands, resource names**.
    * This is what you normally see in URLs and commands.

2. **Project Name**

    * Human-friendly label.
    * **Not unique** and **changeable** at any time.
    * Used for display in the console.

3. **Project Number**

    * Numeric ID created by Google.
    * Unique, immutable.
    * Used mostly **internally and in some IAM/infra contexts**.

> **Exam quiz link:**
> “Globally unique, permanent, and unchangeable, but can be modified by the customer during creation?” → **Project ID**.

### Resource Manager API

* **Cloud Resource Manager** lets you manage projects programmatically:

    * List, create, update, delete, and even **recover** recently deleted projects.
* Accessible via **RPC** and **REST**.
* Important for **larger orgs** that manage many projects automatically (e.g. per team, per environment).

> **Architect exam tip:**
> Services and APIs are **enabled per project**, not per org or folder.

---

## 3) 🗂️ Folders & Organization Node: structuring an enterprise

### Folders

* Used to group projects (and subfolders) in a hierarchy.
* Typical patterns:

    * By **department**: `/Org/Finance`, `/Org/Analytics`, `/Org/Marketing`.
    * By **environment**: `/Org/Prod`, `/Org/NonProd`.
* Folder-level IAM and org policies are **inherited down** to all children.

Benefits:

* Centralised policy management for many projects at once.
* Simplified delegation: e.g. “Analytics folder admins can manage all analytics projects”.

### Organization node

* **Top-level** container for the whole company in Google Cloud.
* All folders and projects are its “children”.
* Special roles at org level:

    * **Organization Policy Administrator**: can define and manage org policies.
    * **Project Creator**: controls who can create projects (and therefore who can spend money).

### How the org node appears

* If you are a **Google Workspace** customer:

    * Org node is automatically linked to your Workspace domain.
* If not, you can create one via **Cloud Identity**:

    * Cloud Identity gives you a managed identity domain; the org node is then associated with that domain.

> **Architect exam tip:**
> You **must** have an organization node to use **folders**.
> For serious/mature environments, expect: **Org → Folders → Projects → Resources**.

---

## 4) 👥 IAM Basics: who can do what, on which resource

**Identity and Access Management (IAM)** controls access in Google Cloud.

An IAM policy answers three questions:

1. **Who** → the *principal*

    * Google account (user)
    * Google group
    * Service account
    * Cloud Identity / Workspace domain

2. **Can do what** → the *role*

    * A role is a bundle of **permissions**.
    * Example permissions: `compute.instances.start`, `storage.objects.get`, etc.

3. **On which resource** → scope in the hierarchy

    * Org, folder, project, or specific resource.

### Inheritance & deny

* If you grant a role at the **folder level**, that role applies to:

    * The folder **and all projects and resources** under it.
* IAM now supports **deny policies**:

    * Deny rules can say “this principal cannot perform these permissions”.
    * Deny is evaluated **before** allow; if deny matches, access is blocked even if a role allows it.
    * Deny is also **inherited down** the hierarchy.

> **Architect exam tip:**
> When you want to block specific actions globally or for entire branches, think about **deny policies at higher levels** in the hierarchy.

---

## 5) 🎭 IAM Role Types: basic, predefined, custom

IAM provides three role categories:

### 1) Basic roles (broad, project-wide)

* **Owner, Editor, Viewer, Billing Admin**
* Applied at project level and affect all resources in that project.

Behaviours:

* **Viewer**: read-only access; cannot modify resources.
* **Editor**: read and write on most resources.
* **Owner**:

    * Same as Editor
    * Plus ability to manage IAM and billing.

Billing Admin:

* Can manage **billing** but not necessarily modify resources.

**Architect implication:**

* Basic roles are **very broad** and often **too permissive** for production environments.
* Use them carefully, usually only in small/test setups.

### 2) Predefined roles (service-specific, least privilege-friendly)

* Provided **per service**, e.g.:

    * Compute Engine `roles/compute.instanceAdmin`
    * Storage `roles/storage.objectViewer`
* Define *specific* sets of permissions tuned to common job functions.
* Can be applied at org, folder, or project (depending on role type).

**Architect usage:**

* Recommended default for real environments.
* Allow you to implement **least privilege** and align with job roles.

### 3) Custom roles (most fine-grained)

* You define the exact permission set.
* Used when predefined roles are either:

    * too broad, or
    * don’t exactly match your needs.

**Constraints:**

* You must maintain them (permissions may change over time).
* **Can be created at project or organization level only**
  (not at the folder level).

> **Exam order question:**
> From broadest to finest-grained:
> **Basic roles → Predefined roles → Custom roles**

> **Architect exam tip:**
> “We want least privilege and fine-grained permissions” → favour **predefined roles** first.
> Only go to **custom roles** when you have a well-understood requirement that predefined roles can’t meet.

---

## 6) 🤖 Service Accounts: identities for workloads

### What is a service account?

* A **special Google account** used by applications and services, not humans.
* Named like an email (e.g. `my-app@project-id.iam.gserviceaccount.com`).
* Authenticates using **keys/credentials**, not passwords.

### Why they matter architecturally

* You don’t want to embed user credentials in apps or scripts.
* You give the **VM, container, or Cloud Run function** a service account with specific roles.
* Application calls other GCP services using that service account identity.

Example:

* A VM hosting an app needs to write to Cloud Storage.
* You create a **service account** and grant it the necessary **Storage roles**.
* The VM runs with that service account, and only that VM/app can access the bucket.

### Service accounts are both identity and resource

* As an identity: they are **principals** that can be granted roles.
* As a resource: they can **have IAM policies themselves**.

    * Example: Alice is service account admin (can manage keys & roles).
    * Bob is viewer (can list but not modify).

> **Architect exam tip:**
> If a question describes a VM or service that needs to access other GCP services **without human involvement**, the correct solution is **service accounts with appropriate roles**, not hard-coded credentials.

---

## 7) 🧑‍💼 Cloud Identity: centralising user and group management

### Problem with ad-hoc Gmail + groups

* Teams often start by:

    * Logging in with individual **Gmail accounts**.
    * Using **Google Groups** for collaboration.
* Problems:

    * No central control by the organisation.
    * Hard to revoke access quickly when someone leaves.

### Cloud Identity solution

* Provides a managed identity platform for the organisation.
* Admins use the **Google Admin Console** to:

    * Manage users and groups.
    * Integrate with existing **AD/LDAP**.
    * Apply organisation-wide policies.

When someone leaves:

* Admin disables their account in the Admin Console.
* This immediately cuts their access to Google Cloud and other services.

Editions:

* **Free** Cloud Identity.
* **Premium** edition adds extra device management features.
* If you have **Google Workspace**, you already have this identity management capability.

> **Architect exam tip:**
> Cloud Identity / Workspace is how you **bridge enterprise identity** (AD/LDAP) with Google Cloud IAM.

---

## 8) 🛠️ Ways to access and operate Google Cloud

You’ll see four main ways to interact with GCP:

### 1) Google Cloud Console (web GUI)

* Browser-based graphical interface.
* Use cases:

    * Explore services and resources.
    * View health, logs, and metrics.
    * Set budgets and monitor spend.
    * Quick SSH connections to VMs.

### 2) Cloud SDK & Cloud Shell

* **Cloud SDK**:

    * Includes `gcloud`, `bq`, and other CLI tools.
    * Installed on your machine when you manage GCP from local environments.
* **Cloud Shell**:

    * Temporary Debian-based VM in your browser with:

        * 5 GB persistent home directory
        * SDK pre-installed
        * Already authenticated
    * Great for doing tasks from anywhere without local setup.

### 3) APIs & client libraries

* Each GCP service exposes **APIs**.
* Google provides:

    * **Cloud Client Libraries** and **Google API Client Libraries** for many languages (Java, Python, Go, Node.js, C#, PHP, Ruby, C++, etc.).
* Use cases:

    * Custom apps that control infrastructure.
    * Automation beyond what scripts and the CLI can easily handle.

### 4) Google Cloud mobile app

* Allows:

    * Start/stop VMs.
    * View logs and metrics.
    * Manage App Engine apps and Cloud SQL instances.
    * Monitor billing and alerts.
* Useful for **on-call** and quick operations.

> **Architect exam tip:**
> These access methods aren’t usually trick questions; they set context: “How would an admin quickly diagnose or stop something?” → console / app / Cloud Shell.

---

## 9) 🧱 Cloud Marketplace (LAMP Stack lab context)

The LAMP lab is really teaching **one concept**:

* **Cloud Marketplace** provides pre-packaged solutions (e.g. LAMP, databases, dev stacks) that you can **deploy with minimal configuration**.
* For the lab, you deploy a:

    * Linux + Apache + MySQL + PHP stack on a Compute Engine VM.
* Marketplace handles:

    * Creating the VM.
    * Installing and configuring the stack.
    * Wiring things so the site works out of the box.

Architectural takeaway:

* Use Marketplace when you want **standard solutions quickly**, but remember:

    * You still own **security hardening, lifecycle, patching** for the deployed resources.

---

## ✅ Quiz answers (for this module)

1. **Which of these values is globally unique, permanent, and unchangeable, but can be modified by the customer during creation?**
   ✅ **The project ID**

2. **Services and APIs are enabled on a per-__________ basis.**
   ✅ **Project**

3. **Order these IAM role types from broadest to finest-grained.**
   ✅ **Basic roles, predefined roles, custom roles**