---
marp: true
theme: default
paginate: true
size: 16:9
header: 'Execution Environments & Configuration as Code in AAP'
footer: '© Internal — Ansible Automation Platform'
---

<!-- _class: lead -->
<!-- _paginate: false -->
<!-- _header: '' -->

# Scaling Automation
## Execution Environments & Configuration as Code

A look at how our GitHub Actions pipeline declaratively manages the
**Ansible Automation Platform (AAP)** using portable, versioned
Execution Environments.

<!--
**Presenter Notes:**
* **Analogy:** Think of Configuration as Code for AAP like a recipe book for a master chef. The recipe book (code) guarantees the dish (Ansible automation) turns out perfectly every single time, no matter who is cooking, instead of relying on memory or manual steps.
-->

---

## Agenda

1. What is an Execution Environment (EE)?
2. Why EEs matter in our workflow
3. Anatomy of our EE
4. CI/CD pipeline — from commit to AAP
5. Deep dive: `deploy_cac.yml`
6. Business benefits
7. Q&A

<!--
**Presenter Notes:**
* **Analogy:** If our Ansible Automation Platform (AAP) is a massive factory, today we are going to look at the blueprints (our code) and the specialized workers (Execution Environments) that keep the factory running smoothly and automatically.
-->

---

## What is an Execution Environment?

A **container image** that serves as the control node for Ansible automation.

**Components inside the image:**

- Base OS — *RHEL 9 Minimal*
- `ansible-core` & `ansible-runner`
- Python & system dependencies — `gcc`, `python3.12-devel`, …
- Ansible Collections — `ansible.controller`, `ansible.platform`, …

> **Purpose:** Eliminates the *"it works on my machine"* problem by
> packaging every automation dependency into one portable container.

<!--
**Presenter Notes:**
* **Analogy:** Think of an Execution Environment (EE) like a fully-equipped food truck. You don't just show up to an event and hope they have an oven, ingredients, and utensils; you bring the entire kitchen with you. The EE brings the OS, Ansible, Python, and Collections all packaged in one truck. No matter where you park (or run the automation), you have everything you need to cook the exact same meal.
-->

---

## Why EEs Matter in Our Workflow

### Consistency
Same dependencies in **local dev**, **GitHub Actions CI/CD**, and the **AAP cluster**.

### Security & Control
We explicitly declare every collection and package.
Built reproducibly via `ansible-builder`.

### Decoupling
The AAP **control plane** is decoupled from the **execution plane**.
EEs can be updated independently of the underlying AAP infrastructure.

<!--
**Presenter Notes:**
* **Analogy for Consistency:** Like shipping a fully assembled car instead of a box of loose parts. You know it will drive.
* **Analogy for Security & Control:** Like a detailed manifest on a cargo ship. We know exactly what's inside and nothing sneaks on board.
* **Analogy for Decoupling:** Like separating the conductor (AAP Control Plane) from the orchestra (Execution Plane). You can bring in new instruments or musicians without having to replace the conductor or the stage.
-->

---

## Anatomy of Our Execution Environment

**Source of truth:** `execution-environment/execution-environment.yml`
**Base image:** `ee-minimal-rhel9:latest` *(AAP 2.6)*

**Dependencies installed at build time:**

- Python: `ansible-core`, `ansible-runner`, `setuptools`, `wheel`
- Collections from `requirements.yml`
  — pulled from **Red Hat Automation Hub** and **Galaxy**

**Build tool:** `ansible-builder` compiles this YAML into a standard OCI image.

<!--
**Presenter Notes:**
* **Analogy:** Think of `execution-environment.yml` as the architectural blueprint, and `ansible-builder` as the construction crew. You tell the crew what foundation you want (base image) and what specific tools you need (Python dependencies, Ansible Collections). The crew then builds a standardized, modular pod (the container image) that can be safely dropped onto any site.
-->

---

## How This Code Pushes to AAP
### The CI/CD Pipeline — `.github/workflows/deploy.yml`

| Step | Stage | What happens |
|------|-------|--------------|
| 1 | **Lint & Validate** | `yamllint` + `ansible-lint` enforce standards |
| 2 | **Build & Publish EE** | Build image, push to **GHCR** and **Private Automation Hub** |
| 3 | **Dry-Run Validation** | Run `--syntax-check` inside the new EE against Dev |
| 4 | **Declarative Deploy** | Execute `deploy_cac.yml` inside the EE → AAP Controller API |

<!--
**Presenter Notes:**
* **Analogy:** Think of the CI/CD pipeline like an automated assembly and delivery line:
  * **Step 1 (Lint):** Quality Control checking the raw parts before assembly.
  * **Step 2 (Build/Publish):** Boxing up the final product and storing it safely in the warehouse.
  * **Step 3 (Dry-Run):** A test drive on a closed track to ensure it works.
  * **Step 4 (Deploy):** Safely delivering the final product to the showroom (AAP) exactly as ordered.
-->

---

## Deep Dive: `deploy_cac.yml`

**Declarative:** We define *what* AAP should look like — no UI clicks.

**Phased execution:**

- **Phases 1–3** — Load baseline (`configs/all`) + env overrides (Dev / Prod)
- **Phase 4** — Apply to AAP in strict dependency order:

  1. Organizations
  2. Execution Environments
  3. Projects & Inventories
  4. Hosts & Groups
  5. Job Templates & Workflows
  6. RBAC / Roles

**Idempotent** — changes are applied only when state drifts from code.

<!--
**Presenter Notes:**
* **Analogy:** Think of this phased execution like setting up a brand new restaurant. You don't just throw things in randomly.
  * First, you lease the building (Organizations).
  * Then, you bring the kitchen tools (Execution Environments).
  * Next, you stock the pantry (Projects/Inventories) and map the tables (Hosts).
  * Then, you create the menu (Job Templates).
  * Finally, you assign the staff their roles (RBAC).
* **Declarative & Idempotent:** You give the contractor a picture of the finished restaurant. They look at what's already there and *only* build what's missing, rather than tearing it all down and starting from scratch.
-->

---

## Business Benefits

### Auditability
Every AAP change is a **Git commit** with PR review and full history.

### Disaster Recovery
Lost a cluster? Rebuild the entire AAP configuration **from this repo** in minutes.

### Velocity
Teams propose changes via **Pull Requests**, not IT tickets.

<!--
**Presenter Notes:**
* **Analogy for Auditability:** Like an airplane's black box combined with a security camera. We know exactly who changed what, when, and why.
* **Analogy for Disaster Recovery:** Like having a magical 3D-printer blueprint for your entire office building. If the building burns down, you don't panic—you just click 'print' and rebuild it exactly as it was.
* **Analogy for Velocity:** Instead of waiting in a slow line at the DMV to file a paper form (a traditional IT ticket), you're ordering off a self-serve digital menu (Pull Requests) and letting the automated kitchen process your request.
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->

# Questions?

EEs · `ansible-builder` · `deploy_cac.yml`

*Thanks for your time.*

<!--
**Presenter Notes:**
* **Analogy Summary:** Remember, AAP is the factory, EEs are the food trucks bringing everything we need, and Configuration as Code is the blueprint guaranteeing it's built the exact same way every time!
-->
