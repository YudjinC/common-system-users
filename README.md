# common_system_users

 [🇷🇺 Читать на русском](README.ru.md)

Configuration and CI for deploying users (and removing them) via an Ansible playbook. Interaction mechanisms:

## Pipeline

A pipeline (`stage: lint` via `ansible-lint` and `stage: deploy` via `ansible-playbook`) is created after pushing a commit to the master branch (`lint` is triggered from any branch).  
Such a deployment mechanism is not limited — all inventories are rolled out.

To use the limiting mechanism, you should create a pipeline from **"Run pipeline"** in the GitLab web interface.  
In this case, you must set the `LIMITS` variable and specify the required limits in its value.

## Inventory

`hosts.yaml` — an inventory file containing the required variables and a list of all hosts.

## User lists

Users with `state: present` are represented in the following lists:

- `ansible_deploy_user` — the ansible user itself (needs to be deployed manually if it is not present on the host)
- `default_gp_ssh_users`
- `inf_ops_ssh_users`

Users with `state: absent` are listed separately in `absent_ssh_users`.  
This separation is important because `create` and `delete` user tasks are performed on these respective groups.

## The `users` list for the `create users` tasks

By default, the `ssh_users` list is used, which contains all user groups described above.  
In addition, there is a `default_with_ansible` list (the same users + the ansible user, if it needs to be deployed manually).

These groups can be specified in `vars` for host groups if necessary.

## Principles of creating host groups

The main variables that may be needed:

- `ansible_host` — IP address for SSH connection
- `ansible_port` — SSH connection port
- `proxy_host` — host for proxy jump

When creating the inventory, the main goal was to minimize host duplication in definitions. For this reason, the inventory is organized as follows:

1. At the top `vars` level (`all`), hosts that act as proxy hosts for other hosts are defined (e.g., `moscow_integration`, `belgorod_integration`, etc.).
2. If `ansible_host`, `proxy_host`, or `ansible_port` is required for multiple hosts in a group, it is set in `vars` at the group level.  
   Example for `project2`:
   ```yaml
   project2:
     vars:
       ansible_host: 54.190.136.210
     hosts:
       bel_k8s_node1:
         ansible_port: 2201
       bel_k8s_node2:
         ansible_port: 2202
       bel_k8s_node3:
         ansible_port: 2203
       bel_k8s_node4:
         ansible_port: 2204
