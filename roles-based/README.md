# Roles-Based Ansible — Demo

This directory demonstrates the **Ansible Roles approach**: the same outcome as `flat/` but organised into structured, reusable, independently testable roles.

## Directory Structure

```
roles-based/
├── inventory/
│   └── hosts                     ← Target hosts (identical to flat/)
├── group_vars/
│   └── all.yml                   ← Shared vars (users list + apps list)
├── site.yml                      ← Thin orchestration playbook (8 lines!)
└── roles/
    ├── common/                   ← OS baseline: pre-flight + groups
    │   ├── tasks/main.yml
    │   ├── handlers/main.yml
    │   ├── defaults/main.yml     ← Low-precedence defaults (overridable)
    │   ├── vars/main.yml         ← Internal constants (not overridden)
    │   └── meta/main.yml         ← Galaxy metadata + dependencies
    ├── users/                    ← 20 Linux users
    │   ├── tasks/main.yml
    │   ├── defaults/main.yml
    │   ├── meta/main.yml         ← Declares dependency on 'common'
    │   └── README.md
    └── apps/                     ← 5 applications + nginx service
        ├── tasks/main.yml
        ├── handlers/main.yml
        ├── defaults/main.yml
        ├── meta/main.yml         ← Declares dependency on 'common'
        └── README.md
```

## What Makes This Different from flat/

| Concern | `flat/` | `roles-based/` |
|---------|---------|----------------|
| Entrypoint | `users.yml` + `apps.yml` (200+ lines total) | `site.yml` (8 lines) |
| Logic location | Inline in playbook files | `roles/{name}/tasks/main.yml` |
| Variable scoping | Everything in `group_vars/all.yml` | Role defaults in `defaults/`, internal constants in `vars/` |
| Reusability | Copy-paste to another project | `ansible-galaxy install` or git submodule |
| Dependencies | Manually ordered tasks | Declared in `meta/main.yml` — Ansible enforces order |
| Documentation | README only | Per-role README + `meta/` with Galaxy-compatible metadata |
| Testability | `--check` mode only | Molecule + Testinfra per role |
| Handlers | Inline or none | `handlers/main.yml` per role |

## Prerequisites

```bash
ansible --version    # 2.12+
pip install passlib  # required for password_hash filter
```

## How to Run

```bash
# From the workspace root:

# 1. Syntax check
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts --syntax-check

# 2. Dry run (no changes made)
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts --check

# 3. Full run
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts

# Or use the Makefile shortcuts:
make syntax-roles
make check-roles
make run-roles
```

## Selective Execution with Tags

```bash
# Run only the common role tasks:
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts --tags common

# Run only user creation:
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts --tags user_create

# Run only package installation:
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts --tags packages

# Run only service management:
ansible-playbook roles-based/site.yml -i roles-based/inventory/hosts --tags services
```

## Verify Results

```bash
# Verify groups:
getent group developers devops qa managers readonly

# Verify users:
id alice_smith
getent passwd | grep -c ":/home/"

# Verify applications:
which nginx git curl htop vim

# Verify nginx service:
systemctl is-active nginx
systemctl is-enabled nginx
```

## Why Roles Are Better for Production

1. **Reusability** — the `users` role can be published to Ansible Galaxy and used in any project
2. **Scoped variables** — `defaults/main.yml` has the lowest precedence; operators can safely override without editing role code
3. **Dependency declaration** — `meta/main.yml` ensures `common` always runs before `users` or `apps`, regardless of play order
4. **Isolation** — a bug in the `apps` role cannot affect the `users` role
5. **Testability** — each role can be tested independently with Molecule
