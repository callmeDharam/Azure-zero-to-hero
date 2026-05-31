# AGENTS.md

Guidance for AI agents working in this repository.

## Repository overview

**Azure Zero to Hero** is a training curriculum (markdown lessons, ARM templates, sample Terraform, Azure DevOps pipeline YAML, and shell lab scripts). It is not a single deployable application. Hands-on projects (voting app, three-tier e-commerce, etc.) use **external** repos and **Azure cloud** resources.

## Cursor Cloud specific instructions

### What runs locally vs in Azure

| Scope | Where it runs |
|-------|----------------|
| Read/edit course content, validate ARM/TF/YAML/shell | Local VM (this environment) |
| `az deployment`, AKS, Functions, full CI/CD labs | Azure subscription + `az login` |

No `npm`, `pip`, or `docker compose` lives in this repo. There is no dev server or in-repo test/lint CI.

### Toolchain (expected on the VM)

Install once if missing (Ubuntu):

- **Azure CLI** — [InstallAzureCLIDeb](https://aka.ms/InstallAzureCLIDeb)
- **Terraform** — HashiCorp apt repo (`terraform`)
- **kubectl** — Kubernetes apt repo (AKS days)
- **jq**, **shellcheck** — `apt install jq shellcheck`
- **PyYAML** (optional) — `pip3 install pyyaml` for pipeline YAML checks

Day-23 Terraform references `file("~/.ssh/id_rsa.pub")`. Ensure a key exists before `terraform validate` / `plan` (the VM update script can generate one).

### Local validation (no Azure login)

From repo root:

```bash
# ARM JSON
jq empty Day-11/01-storage-account/01-storage-account.json
jq empty Day-11/02-virtual-machine/01-create-vm.json

# Shell
bash -n Day-22/01-create-function-app.sh
bash -n Day-15/updateK8sManifests.sh

# Azure Pipelines YAML
python3 -c "import yaml; yaml.safe_load(open('Day-15/vote-pipeline.yaml'))"

# Terraform (Day-23) — local provider only, skip remote backend
cd Day-23
terraform init -backend=false -input=false
terraform validate
```

`terraform plan` / `apply` need **`az login`** and a configured remote backend in `Day-23/backend.tf` (or re-init with your own backend). Do not commit `.terraform/` or local state.

### Azure-authenticated labs

1. `az login` (or service principal).
2. Follow the relevant `Day-*/` markdown (e.g. Day-11 deploy docs, Day-20 Key Vault, Day-22 function script).
3. Day-15 CD script needs real values for `<ACCESS-TOKEN>`, `<AZURE-DEVOPS-ORG-NAME>`, `<ACR-REGISTRY-NAME>`.

### Lint / test / build

Not defined in-repo. Use the validation commands above plus `shellcheck` on `*.sh` if you change scripts.

### Gotchas

- **Terraform 1.x** with pinned `azurerm = 3.0.0` in `Day-23/main.tf`; `backend.tf` points at Azure Storage — empty `access_key` until you configure it.
- **Pipeline YAML** (`Day-15/vote-pipeline.yaml`) references org-specific ACR and agent pool names; it documents a sample DevOps setup, not this repo’s layout.
- **Course apps** (voting stack, e-commerce) are not vendored here; clone/deploy per video/README, not from this tree.
