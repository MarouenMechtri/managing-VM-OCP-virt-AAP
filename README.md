# Managing VMs on OpenShift Virtualization via AAP

Playbooks and AAP job templates to:

1. **Provision a VM** on OpenShift Virtualization in the requester's project (`user-<username>`), creating the namespace / OpenShift User / admin RoleBinding when missing.
2. **Show deployment cost** from Cost Management for that project (tag `instance-type` = S/M/L/XL).

**Repo:** https://github.com/MarouenMechtri/managing-VM-OCP-virt-AAP

## Size catalog (Cost Management)

Aligned with cost model `workshop-cloud-vm-rates` (`vm_cost_per_month` + `instance-type` tag):

| Size | Instantype | Disk (default) | Rate (USD / VM / month) |
|------|------------|----------------|-------------------------|
| S    | u1.small   | 32 Gi          | 29.99                   |
| M    | u1.medium  | 48 Gi          | 59.99                   |
| L    | u1.large   | 64 Gi          | 119.99                  |
| XL   | u1.xlarge  | 96 Gi          | 239.99                  |

Storage: ~$0.10 / GiB-month (`storage_gb_request_per_month`).

## Playbooks

| Playbook | Job template name | Purpose |
|----------|-------------------|---------|
| `playbooks/provision_vm.yml` | **Provision VM on OCP Virtualization** | Create ns + RBAC + VirtualMachine |
| `playbooks/show_deployment_cost.yml` | **Show VM deployment cost** | Query Cost Management for `user-<username>` |

Survey JSON samples: `surveys/`.

## AAP wiring

1. Project SCM → this repository (branch `main`).
2. Credential **OpenShift or Kubernetes API Bearer Token** (cluster API + token) on both job templates.
3. Inventory: localhost (or Demo Inventory with `ansible_connection=local`).
4. Enable surveys from `surveys/*.json`.
5. For cost job: Cost Management SSO user/password via survey (workshop Keycloak).

## Namespace / user behaviour

- Namespace: `user-{{ username }}` (e.g. `user1` → `user-user1`).
- If the namespace does not exist → create it.
- If the OpenShift `User` object does not exist → create it.
- RoleBinding `{{ username }}-admin` → `ClusterRole/admin` for that User.
- VM labels include `instance-type: S|M|L|XL` for chargeback.

## Cost Management UI

https://cost-onprem-ui-cost-onprem.apps.cluster-xwgs6.dyn.redhatworkshops.io

Filter by project `user-<username>` after metrics have been collected.
