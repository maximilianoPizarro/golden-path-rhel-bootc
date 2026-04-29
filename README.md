# Golden Path — RHEL Virtual Machine (bootc / OpenShift Virtualization)

Backstage Software Template for [Red Hat Developer Hub](https://developers.redhat.com/rhdh) that provisions RHEL virtual machines on **OpenShift Virtualization** with configurable compute, storage, and networking. Supports **bootc** (image-mode RHEL) for GitOps-driven OS lifecycle management. Optionally exposes the VM through **Connectivity Link** (Gateway API + AuthPolicy + RateLimitPolicy). Creates a Gitea repo synced by **ArgoCD**.

**Documentation:** [https://maximilianopizarro.github.io/golden-path-rhel-bootc/](https://maximilianopizarro.github.io/golden-path-rhel-bootc/)

## Key Features

- **Self-service VM provisioning** — 5-page wizard in Developer Hub with 30+ configurable parameters
- **bootc (image-mode RHEL)** — Deliver the OS as an OCI container image with atomic updates and automatic rollbacks
- **GitOps with ArgoCD** — Every VM backed by a Git repo; push a change, ArgoCD reconciles it
- **Connectivity Link** — Expose VMs through Gateway API with Kuadrant-powered authentication (OIDC / API Key) and rate limiting
- **Backstage Catalog** — Automatic registration with TechDocs, ArgoCD status, and Kubernetes topology

## Architecture

```
Developer Hub ──► Gitea ──► ArgoCD ──► OpenShift Cluster
                                            │
                                       VirtualMachine
                                       Service
                                       Gateway + Policies (optional)
```

## Requirements

| Product | Version | Status |
|---------|---------|--------|
| Red Hat OpenShift Container Platform | 4.14+ | Required |
| Red Hat OpenShift Virtualization | 4.14+ | Required |
| Red Hat Developer Hub | 1.x | Required |
| Red Hat OpenShift GitOps (ArgoCD) | 1.10+ | Required |
| Gitea (or compatible Git server) | 1.21+ | Required |
| Red Hat Connectivity Link / Kuadrant | 1.x | Optional |
| Red Hat OpenShift Service Mesh / Istio | 2.4+ | Optional |
| Red Hat Build of Keycloak | 22+ | Optional |

For full details see the [Requirements](https://maximilianopizarro.github.io/golden-path-rhel-bootc/requirements.html) page.

## Repository Structure

```
├── template.yaml              # Backstage scaffolder template (5 wizard pages)
├── skeleton/                  # Template skeleton (rendered per VM instance)
│   ├── README.md              # Per-instance README
│   ├── catalog-info.yaml      # Backstage catalog entity + optional API definition
│   ├── mkdocs.yml             # TechDocs configuration
│   ├── docs/
│   │   └── index.md           # TechDocs landing page
│   └── manifests/             # Kubernetes manifest templates
│       ├── virtualmachine.yaml
│       ├── cloudinit-secret.yaml
│       ├── ssh-secret.yaml
│       ├── service.yaml
│       ├── route.yaml
│       ├── gateway.yaml
│       ├── httproute.yaml
│       ├── authpolicy.yaml
│       ├── ratelimitpolicy.yaml
│       ├── planpolicy.yaml
│       ├── apiproduct.yaml
│       └── apikey-secret.yaml
└── docs/                      # GitHub Pages documentation site
    ├── index.html
    ├── architecture.html
    ├── requirements.html
    ├── usage.html
    ├── parameters.html
    ├── bootc.html
    ├── connectivity-link.html
    ├── manifests.html
    ├── css/style.css
    └── js/main.js
```

## How to Use

### 1. Register the template in Developer Hub

Add the template to your Developer Hub catalog by referencing the `template.yaml`:

```yaml
# app-config.yaml
catalog:
  locations:
    - type: url
      target: https://github.com/maximilianoPizarro/golden-path-rhel-bootc/blob/main/template.yaml
      rules:
        - allow: [Template]
```

### 2. Provision a VM

1. Open Developer Hub and navigate to **Create**
2. Select **"RHEL Virtual Machine (bootc / OpenShift Virtualization)"**
3. Fill out the wizard:
   - **VM Identity** — Name, owner, namespace
   - **Compute** — CPU cores/sockets/threads, memory, run strategy
   - **Storage** — Disk size, boot source (registry or bootc image)
   - **Network** — SSH key, password, Apache httpd, Connectivity Link toggle
   - **Connectivity Link** — Route path, auth model (None/OIDC/API Key), rate limit
4. Click **Create**

### 3. Access the VM

```bash
# SSH via virtctl
virtctl ssh cloud-user@<vm-name> -n <namespace>

# VNC console
virtctl console <vm-name> -n <namespace>
```

### 4. Day-2 Operations

All changes go through Git — edit manifests, commit, push, and ArgoCD syncs automatically:

```bash
git clone https://gitea-gitea.<cluster-domain>/ws-<owner>/<vm-name>-vm
cd <vm-name>-vm
# Edit manifests/virtualmachine.yaml (CPU, memory, image, etc.)
git add -A && git commit -m "scale VM to 4 cores" && git push
```

## Documentation

Full documentation is available at **[maximilianopizarro.github.io/golden-path-rhel-bootc](https://maximilianopizarro.github.io/golden-path-rhel-bootc/)**:

- [Architecture](https://maximilianopizarro.github.io/golden-path-rhel-bootc/architecture.html) — Provisioning flow and network topology
- [Requirements](https://maximilianopizarro.github.io/golden-path-rhel-bootc/requirements.html) — Red Hat product prerequisites
- [How to Use](https://maximilianopizarro.github.io/golden-path-rhel-bootc/usage.html) — Step-by-step guide
- [Parameters](https://maximilianopizarro.github.io/golden-path-rhel-bootc/parameters.html) — Full parameter reference
- [bootc Deep Dive](https://maximilianopizarro.github.io/golden-path-rhel-bootc/bootc.html) — Image-mode RHEL lifecycle
- [Connectivity Link](https://maximilianopizarro.github.io/golden-path-rhel-bootc/connectivity-link.html) — Gateway API, Auth, Rate Limiting
- [Manifest Reference](https://maximilianopizarro.github.io/golden-path-rhel-bootc/manifests.html) — All 12 Kubernetes resources

## Author

[maximilianoPizarro](https://maximilianopizarro.github.io)
