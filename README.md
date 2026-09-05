# ansible-role-victoriametrics

Ansible role for installing and configuring
[VictoriaMetrics](https://victoriametrics.com/) — a fast,
cost-effective, Prometheus-compatible time series database, running
as a single-node binary managed via systemd.

> **Release cadence:** VictoriaMetrics ships new releases roughly
> weekly. The pinned `victoriametrics_version` default will go stale
> quickly — check the
> [releases page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)
> and bump it periodically, or override it explicitly in your own
> playbook.

## Requirements

- Ansible >= 2.14
- Target OS: Ubuntu 22.04/24.04, Debian 11/12, RHEL/Rocky/Alma 8/9 (systemd)
- Internet access on the target host (to download the release archive from GitHub)

## Role Variables

See the full list with defaults in [`defaults/main.yml`](defaults/main.yml).

| Variable                              | Default                     | Description                                        |
|------------------------------------------|--------------------------------|--------------------------------------------------------|
| `victoriametrics_version`                | (pinned, see note above)       | Release version to install                          |
| `victoriametrics_http_address`           | `""` (all interfaces)          | Bind address; empty means listen on all interfaces  |
| `victoriametrics_http_port`              | `"8428"`                       | HTTP port                                            |
| `victoriametrics_storage_dir`            | `/var/lib/victoriametrics`     | Where time series data is stored                    |
| `victoriametrics_retention_period`       | `12`                           | Retention in months (bare number = months, per VictoriaMetrics' own `-retentionPeriod` flag semantics) |
| `victoriametrics_selfscrape_interval`    | `"10s"`                        | How often VictoriaMetrics scrapes its own metrics   |
| `victoriametrics_memory_allowed_percent` | `5`                            | `-memory.allowedPercent` flag                       |

## Example Playbook

```yaml
- hosts: monitoring_servers
  become: true
  roles:
    - role: halif.victoriametrics
      vars:
        victoriametrics_version: "1.151.0"
        victoriametrics_retention_period: 6
```

After the role runs, VictoriaMetrics is reachable at
`http://<server-ip>:8428`. Point Prometheus (or vmagent) at it with:

```yaml
remote_write:
  - url: "http://<server-ip>:8428/api/v1/write"
```

## Testing

CI runs a **full smoke test** on Ubuntu 22.04 and Rocky Linux 9:
install, start, and verify both the open port and the `/health`
endpoint respond. Unlike some of the other exporter roles in this
author's collection, VictoriaMetrics doesn't depend on reaching any
external system to start correctly, so this test is a genuinely
complete functional check, not just an automation smoke test.

```bash
pip install ansible molecule "molecule-plugins[docker]" docker
molecule test
```

## License

MIT

## Author

[halif](https://github.com/halif)
