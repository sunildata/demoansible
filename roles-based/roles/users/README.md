# Role: users

Creates Linux user accounts with hashed passwords and group memberships.

## Requirements

- The `common` role must run first (declared as a dependency in `meta/main.yml`)
- `passlib` Python library on the control node (`pip install passlib`) — required for `password_hash` filter

## Role Variables

Defined in `defaults/main.yml` — all overridable:

| Variable | Default | Description |
|----------|---------|-------------|
| `users_default_shell` | `/bin/bash` | Default login shell if not specified per user |
| `users_create_home` | `true` | Whether to create a home directory |

## Data Variables (provided via group_vars)

| Variable | Type | Description |
|----------|------|-------------|
| `users` | list | List of user definitions (see schema below) |

### User definition schema

```yaml
users:
  - name: alice_smith                                         # Linux username
    password: "{{ 'PlainPass' | password_hash('sha512') }}"  # Pre-hashed password
    primary_group: developers                                 # Primary (login) group
    groups: [developers, devops]                              # Supplementary groups
    shell: /bin/bash                                          # Login shell (optional)
    state: present                                            # present | absent
```

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: common
    - role: users
```

## Tags

| Tag | Description |
|-----|-------------|
| `users` | All tasks in this role |
| `user_create` | User creation tasks only |
