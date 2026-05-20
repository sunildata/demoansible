# Role: apps

Installs a list of OS packages and manages service state for those that run as daemons.

## Requirements

- The `common` role must run first (declared as a dependency in `meta/main.yml`)

## Role Variables

Defined in `defaults/main.yml`:

| Variable | Default | Description |
|----------|---------|-------------|
| `apps_package_state` | `present` | `present` keeps installed version; `latest` upgrades |

## Data Variables (provided via group_vars)

| Variable | Type | Description |
|----------|------|-------------|
| `applications` | list | List of application definitions (see schema below) |

### Application definition schema

```yaml
applications:
  - name: nginx           # Human-readable name (used in task labels)
    package: nginx        # Package name as known to apt/yum
    service: nginx        # Systemd service name — use ~ (null) for CLI-only tools
    service_state: started
    service_enabled: true

  - name: git
    package: git
    service: ~            # No service to manage
    service_state: ~
    service_enabled: ~
```

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: common
    - role: apps
```

## Tags

| Tag | Description |
|-----|-------------|
| `apps` | All tasks in this role |
| `packages` | Package installation tasks only |
| `services` | Service management tasks only |
