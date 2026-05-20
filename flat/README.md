# Flat Ansible Files — Demo

This directory demonstrates the **flat Ansible approach**: all logic lives in two standalone playbook files. Compare this with the `roles-based/` directory to see the difference.

## What This Does

| File | Responsibility |
|------|---------------|
| `users.yml` | Creates 20 Linux users across 5 groups |
| `apps.yml` | Installs 5 applications (nginx, git, curl, htop, vim) |
| `group_vars/all.yml` | All variables — users list + apps list |
| `inventory/hosts` | Target hosts (localhost for demo) |

## Prerequisites

```bash
# Ansible 2.12+ required
ansible --version

# Python passlib required for password_hash filter
pip install passlib
```

## How to Run

```bash
# From the workspace root:

# 1. Syntax check
ansible-playbook flat/users.yml -i flat/inventory/hosts --syntax-check
ansible-playbook flat/apps.yml  -i flat/inventory/hosts --syntax-check

# 2. Dry run (no changes made)
ansible-playbook flat/users.yml -i flat/inventory/hosts --check
ansible-playbook flat/apps.yml  -i flat/inventory/hosts --check

# 3. Full run
ansible-playbook flat/users.yml -i flat/inventory/hosts
ansible-playbook flat/apps.yml  -i flat/inventory/hosts

# Or use the Makefile shortcuts:
make syntax-flat
make check-flat
make run-flat
```

## Verify Results

```bash
# Check all 5 groups were created:
getent group developers devops qa managers readonly

# Check a user was created:
id alice_smith
id bob_jones

# Check all 20 users exist:
getent passwd | grep -E 'alice|bob|carol|dave|eve|frank|grace|henry|iris|jack|karen|liam|mia|noah|olivia|peter|quinn|rachel|sam|tara'

# Check applications installed:
which nginx git curl htop vim

# Check nginx is running:
systemctl is-active nginx
systemctl is-enabled nginx
```

## The Problem with This Approach (for Presentation)

- `group_vars/all.yml` contains **everything** — grows unmanageable as the project scales
- No separation of concerns — user logic and app logic are "close" but not isolated
- To reuse this in another project: copy-paste the files manually
- No test framework — only `--check` mode available
- All variables are in the same scope — naming collisions become a risk

**See `roles-based/` to see how Ansible Roles solve these problems.**
