# AAP Configuration as Code (CaC)

This repository serves POC for  **Ansible Automation Platform (AAP) Configuration as Code** best practices. Utilizing a **"Monorepo"** model, it leverages the `infra.aap_configuration` collection to manage platform objects (Organizations, Projects, Job Templates, etc.) as version-controlled code.

---

## Proof of Concept (PoC) Overview
Ansible Best Practices

---

## Repository Structure
We follow a hierarchical data model to adhere to the **DRY (Don't Repeat Yourself)** principle:

```text
.
├── configs/
│   ├── all/            # GLOBAL: Universal baselines (Orgs, Projects, generic Roles)
│   ├── dev/            # DEV: Overrides for sandboxes (SCM branches, low-tier sizing)
│   └── prod/           # PROD: Production-grade limits (SCM main, vaulted credentials)
├── execution-environment/ # Definitions for custom EE container images
├── inventory/          # Connection parameters for AAP API endpoints
├── playbooks/
│   └── deploy_cac.yml  # Master orchestration playbook
└── .github/workflows/  # CI/CD pipeline (GitHub Actions)
```

---

## Deployment Logic
The core of the deployment is the `playbooks/deploy_cac.yml` file. It follows a **4-phase execution**:

1.  **Load All:** Imports universal configurations from `configs/all/`.
2.  **Load Environment:** Overlays specific variables from the target environment (`dev` or `prod`).
3.  **Dispatch:** Invokes the `infra.aap_configuration.dispatch` role, which intelligently handles the dependency order of API calls.
4.  **Audit:** Outputs execution logs and drift reports.

---

## CI/CD Pipeline (GitHub Actions)
The pipeline defined in `.github/workflows/deploy.yml` includes:

* **Linting:** `yamllint` and `ansible-lint` to enforce style and best practices.
* **Security:** `ggshield` scanning to prevent secret leakage.
* **Dry Run:** Validates the API payload against the AAP instance without making changes (`--check` mode).
* **Environment Gates:** Automatic deployment to development on push; manual approval required for production.

---

## Setup & Prerequisites

### 1. GitHub Secrets
To run the pipeline, you must configure the following Secrets in your GitHub repository (**Settings > Secrets and variables > Actions**):

| Secret Name | Description |
| :--- | :--- |
| `DEV_CONTROLLER_HOST` | API URL for the Dev AAP Instance |
| `DEV_CONTROLLER_USERNAME` | Admin username for Dev |
| `DEV_CONTROLLER_PASSWORD` | Admin password for Dev |
| `PROD_CONTROLLER_HOST` | API URL for the Prod AAP Instance

### 2. Local Development
Before committing, install the pre-commit hooks to ensure code quality:

```bash
pip install pre-commit ansible-lint yamllint
pre-commit install
```

---

## Usage
To trigger a manual deployment from your local machine (ensure you have the `infra.aap_configuration` collection installed):

**Deploy to Development**
```bash
ansible-playbook -i inventory/inventory_dev.yml playbooks/deploy_cac.yml -e "target_env=dev"
```

**Deploy to Production**
```bash
ansible-playbook -i inventory/inventory_prod.yml playbooks/deploy_cac.yml -e "target_env=prod"
```