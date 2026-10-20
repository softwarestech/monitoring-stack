# Monitoring stack

A ready-to-run **Prometheus + Grafana + Alertmanager** stack for Docker hosts, maintained by [Softwares Tech](https://www.softwarestech.com).

One `docker compose up` gives you host and container metrics, a provisioned Grafana dashboard, and alert rules for the problems that most often take services down: unreachable targets, CPU and memory pressure, disks filling up, and restart loops.

## What's included

| Component | Version | Role |
| --- | --- | --- |
| Prometheus | 2.55 | Scrapes and stores metrics (15-day retention), evaluates alert rules |
| Alertmanager | 0.27 | Groups alerts and sends them to Slack (or email, PagerDuty, Opsgenie) |
| Grafana | 11.3 | Dashboards; Prometheus data source and dashboard provisioned on start |
| Node Exporter | 1.8 | Host metrics: CPU, memory, disk, network |
| cAdvisor | 0.49 | Per-container CPU and memory |

```
monitoring-stack/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml      # scrape targets
│   └── alerts.yml          # alert rules
├── alertmanager/
│   └── alertmanager.yml    # routing and receivers
└── grafana/
    ├── provisioning/       # data source + dashboard provider
    └── dashboards/
        └── host-and-containers.json
```

## Quick start

Requires Docker with the Compose plugin on a Linux host.

```bash
git clone https://github.com/softwarestech/monitoring-stack.git
cd monitoring-stack
cp .env.example .env        # then set a strong GRAFANA_ADMIN_PASSWORD
docker compose up -d
```

| UI | URL |
| --- | --- |
| Grafana | http://localhost:3000 |
| Prometheus | http://localhost:9090 |
| Alertmanager | http://localhost:9093 |

The ports are bound to `127.0.0.1` only. To reach them remotely, put them behind a reverse proxy with TLS and authentication, or use an SSH tunnel:

```bash
ssh -L 3000:localhost:3000 user@your-server
```

## Alert rules

| Alert | Fires when | Severity |
| --- | --- | --- |
| `TargetDown` | A scrape target has been unreachable for 2 minutes | critical |
| `HighCpuUsage` | CPU above 85% for 10 minutes | warning |
| `HighMemoryUsage` | Memory above 90% for 10 minutes | warning |
| `DiskSpaceLow` | Less than 15% free on a filesystem | warning |
| `DiskWillFillIn24Hours` | The last 6 hours' trend says the disk fills within a day | critical |
| `ContainerRestarting` | A container started more than twice in 15 minutes | warning |

Critical alerts silence the matching warning for the same host, so one problem sends one notification.

## Customising

- **Send alerts somewhere:** replace the Slack webhook in `alertmanager/alertmanager.yml`, or swap in another [receiver](https://prometheus.io/docs/alerting/latest/configuration/#receiver).
- **Monitor your apps:** add a job under `scrape_configs` in `prometheus/prometheus.yml`, then reload with `curl -X POST http://localhost:9090/-/reload`.
- **More hosts:** run Node Exporter on each one and add it to the `node` job's targets.
- **Dashboards:** drop more JSON files into `grafana/dashboards/`; Grafana picks them up automatically.

## Validation

Every push runs [`validate.yml`](.github/workflows/validate.yml): `promtool` checks the Prometheus config and rules, `amtool` checks Alertmanager, `jq` checks the dashboard JSON, and `docker compose config` checks the Compose file.

## Need help?

Softwares Tech sets up and maintains monitoring, CI/CD, Kubernetes and cloud infrastructure on AWS and Azure. Write to **sales@softwarestech.com** or visit [softwarestech.com](https://www.softwarestech.com).

## License

[MIT](LICENSE)
