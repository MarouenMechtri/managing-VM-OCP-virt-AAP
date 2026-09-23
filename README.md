# Managing VMs on OpenShift Virtualization via AAP + SSAP

Self-Service Automation Portal template that surveys **VM count** and **size** (small / medium / large), shows a **monthly cost estimate**, then launches the AAP **Provision VMs** job against OpenShift Virtualization.

Built to fit the [rhpds/ocp-virt](https://github.com/rhpds/ocp-virt) Day-1 demo (`provision_vms.yml`, `linuxvm` namespace, RHEL9 DataSource).

**SSAP import URL:** https://github.com/MarouenMechtri/managing-VM-OCP-virt-AAP

## Size catalog (matches OCP RHEL9 server templates)

| Size   | Template              | vCPU | Memory | Disk  | Rate (XOF / VM / month) |
|--------|-----------------------|------|--------|-------|-------------------------|
| small  | rhel9-server-small    | 1    | 2 GiB  | 30 GiB | 25 000                 |
| medium | rhel9-server-medium   | 1    | 4 GiB  | 30 GiB | 45 000                 |
| large  | rhel9-server-large    | 2    | 8 GiB  | 30 GiB | 85 000                 |

**Estimate:** `unit rate × vm_count`

## Files

| Path | Role |
|------|------|
| `customtemplate/ocp_virt_vm_deploy.yml` | SSAP form (import this repo) |
| `playbooks/provision_vms.yml` | Parameterized provisioner (drop-in for demo playbook) |
| `requirements.yml` | `kubernetes.core` (+ related) collections |

## Wire it to your demo

The stock [rhpds/ocp-virt `provision_vms.yml`](https://github.com/rhpds/ocp-virt/blob/main/provision_vms.yml) hardcodes a fixed list of **small** VMs. To use count/size from SSAP:

### Option A — Point the AAP project at this repo (simplest)

1. In AAP, edit the project that currently uses `https://github.com/rhpds/ocp-virt`  
   → change SCM URL to `https://github.com/MarouenMechtri/managing-VM-OCP-virt-AAP` (or keep rhpds and use Option B).  
2. Job template **Provision VMs** → playbook `playbooks/provision_vms.yml`.  
3. Keep existing OpenShift kube credential on the job template.  
4. Ensure survey / extra vars still provide `ssh_public_key` and `vm_password` (as in the demo).

### Option B — Keep rhpds/ocp-virt as the project

Copy `playbooks/provision_vms.yml` from this repo over `provision_vms.yml` in your fork of `rhpds/ocp-virt`, then sync the AAP project.

### Option C — Job template name

The SSAP template launches a job named exactly:

```text
Provision VMs
```

Rename your AAP job template to match, or edit `values.template` in `customtemplate/ocp_virt_vm_deploy.yml`.

## Import into Self-Service Automation Portal

1. Platform admin → **Templates** → **Add template**  
2. SCM URL: `https://github.com/MarouenMechtri/managing-VM-OCP-virt-AAP`  
3. **Analyze** → **Import** → set RBAC  

Auth uses `${{ secrets.aapToken }}` (Backstage Plugins ≥ 2.2.0).

## OpenShift cluster (workshop)

Use your workshop console and the kube credential already configured in AAP. Do **not** store cluster admin passwords in Git or in the SSAP YAML.

VMs are created in the `linuxvm` namespace by default (same as the demo).

## Customize pricing

Search for `25000`, `45000`, and `85000` in:

- `customtemplate/ocp_virt_vm_deploy.yml` (form + Nunjucks)  
- `playbooks/provision_vms.yml` (`size_catalog`)  

Keep both files in sync.
