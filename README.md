# OpenShift Virtualization — SSAP VM catalog

Custom [Self-Service Automation Portal](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7/html/using_the_self-service_automation_portal/develop-con_self_service_customize_template) template for deploying VMs on OpenShift Virtualization, with size tiers and a monthly cost preview.

## What the user sees

1. **VM configuration** — name prefix, count (1–20), size, OS, namespace, environment, optional data disk, AAP inventory/credentials  
2. **Cost estimate & confirmation** — pricing rules + mandatory acceptance checkbox  
3. **Review** — portal review screen, then a computed estimate in the run log / job output  

Size → resources → base rate (edit in the YAML to match your catalog):

| Size   | vCPU | Memory | OS disk | Base rate (XOF / VM / month) |
|--------|------|--------|---------|------------------------------|
| small  | 2    | 4 GiB  | 40 GiB  | 45 000                       |
| medium | 4    | 8 GiB  | 80 GiB  | 85 000                       |
| large  | 8    | 16 GiB | 160 GiB | 160 000                      |
| xlarge | 16   | 32 GiB | 320 GiB | 300 000                      |

**Formula:** `(base rate + data_disk_GiB × 150) × vm_count × env_multiplier`  
Multipliers: development `1.0`, staging `1.1`, production `1.25`.

## Files

| Path | Role |
|------|------|
| `customtemplate/ocp_virt_vm_deploy.yml` | SSAP template to import |
| `playbooks/deploy_ocp_virt_vms.yml` | Example playbook for the AAP job template |

## Setup

### 1. AAP job template

Create a job template named **exactly**:

```text
OpenShift Virt / Deploy VMs
```

Point it at `playbooks/deploy_ocp_virt_vms.yml` (or your real kubevirt playbook). Enable **Prompt on launch** for inventory/credentials if the SSAP form should choose them.

### 2. Publish the Git repo

Push this repository to GitHub or GitLab. The portal imports from the SCM URL.

### 3. Import into Self-Service Automation Portal

1. Sign in as a platform administrator  
2. **Templates** → **Add template**  
3. Paste the Git SCM URL → **Analyze** → **Import**  
4. Configure [RBAC for custom templates](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7/html/using_the_self-service_automation_portal/develop-con_self_service_customize_template) so end users can see and launch it  

### 4. Align auth with your portal version

This template uses `${{ secrets.aapToken }}` (Ansible Backstage Plugins ≥ 2.2.0).  
If your portal still uses the older form token pattern (like the [ssap-lab rhel_dynamic example](https://raw.githubusercontent.com/ansible-tmm/ssap-lab/main/customtemplate/rhel_dynamic.yml)), switch the launch step to `token: ${{ parameters.token }}` and add the hidden `AAPTokenField` from that sample.

## Customize pricing

Search for `45000`, `85000`, `160000`, `300000`, and `150` in `customtemplate/ocp_virt_vm_deploy.yml` and update:

- size dependency descriptions (`sizeSummary`)  
- Nunjucks expressions in `cost-preview`, `extraVariables`, and `output`  

Keep the three places in sync so the form, the job `extra_vars`, and the final message all agree.
