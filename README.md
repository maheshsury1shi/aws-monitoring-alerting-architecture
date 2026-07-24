# AWS Monitoring & Alerting Architecture

[![AWS EC2](https://img.shields.io/badge/AWS-EC2-232F3E?logo=amazonaws&logoColor=white)](https://aws.amazon.com/ec2/) [![Amazon Linux](https://img.shields.io/badge/Amazon%20Linux-2023-FF9900?logo=amazonaws&logoColor=white)](https://aws.amazon.com/linux/) [![Prometheus](https://img.shields.io/badge/Prometheus-3.5.3-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/) [![Grafana Enterprise](https://img.shields.io/badge/Grafana%20Enterprise-12.1.1-F46800?logo=grafana&logoColor=white)](https://grafana.com/) [![Node Exporter](https://img.shields.io/badge/Node%20Exporter-1.10.2-4E9A06?logo=linux&logoColor=white)](https://github.com/prometheus/node_exporter) [![systemd](https://img.shields.io/badge/systemd-Enabled-2F4F4F?logo=linux&logoColor=white)](https://www.freedesktop.org/wiki/Software/systemd/) [![Bash](https://img.shields.io/badge/Bash-Scripts-4EAA25?logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/) [![SSH](https://img.shields.io/badge/SSH-Access-000000?logo=ssh&logoColor=white)](https://www.openssh.com/)

## Overview

This project demonstrates a complete AWS monitoring stack deployed on a single EC2 instance running Amazon Linux 2023. It includes:

- Prometheus for metrics collection and storage
- Grafana Enterprise for visualization and dashboards
- Node Exporter for host-level system metrics
- Systemd service management for reliability
- AWS Security Group configuration for secure network access



## Tech Stack

- AWS EC2 (`c7i-flex.large`)
- Amazon Linux 2023
- Prometheus 3.5.3
- Grafana Enterprise 12.1.1
- Node Exporter v1.10.2
- systemd
- SSH / SCP / WinSCP
- AWS Security Group

## Services Used

| Service       | Port | Purpose |
|---------------|------|---------|
| Grafana Enterprise | 3000 | Visualization and dashboard UI |
| Prometheus    | 9090 | Metrics collection and query engine |
| Node Exporter | 9100 | System-level metrics exporter |
| SSH           | 22   | Remote access |

## Architecture Overview

This repository deploys a self-contained observability stack on a single AWS EC2 instance. The stack includes:

- `Node Exporter` for Linux host metrics
- `Prometheus` for scraping and storing time-series metrics
- `Grafana Enterprise` for dashboard visualization
- `systemd` for service lifecycle management

The components communicate locally on the EC2 host, with Grafana serving dashboards on port `3000`, Prometheus on port `9090`, and Node Exporter on port `9100`.

```text
                       +-------------------+
                       |   User Browser    |
                       | (remote client)   |
                       +---------+---------+
                                 |
                                 | HTTPS / HTTP
                                 |
                       +---------v----------+
                       | AWS EC2 Instance   |
                       | (Amazon Linux 2023)|
                       +---------+----------+
                                 |
               +-----------------+-----------------+
               |                                   |
        +------v------+                    +-------v-------+
        |   Grafana   |                    |  Prometheus   |
        | port 3000   |<-------------------| port 9090     |
        | Dashboards  |  queries metrics   | Scrapes       |
        +------+------+                    +-------+-------+
               |                                   |
               |                                   |
               |                                   |
      +--------v---------+                 +--------v--------+
      |  User browser /  |                 |  Node Exporter  |
      |  external access |                 |  port 9100      |
      +------------------+                 |  Host metrics   |
                                           +-----------------+
```

[View the full architecture details in ARCHITECTURE.md](ARCHITECTURE.md)

## Project Structure

- [ARCHITECTURE.md](ARCHITECTURE.md) — architecture design, diagram, and data flow
- [COMMANDS.md](COMMANDS.md) — command reference and operational commands
- [INSTALLATION.md](INSTALLATION.md) — setup and configuration instructions
- [README.md](README.md) — project overview and main landing page
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — troubleshooting, recovery, and validation
- [dashboard-codes.md](dashboard-codes.md) — dashboard query snippets and Grafana code
- [Linux-Commands-Bash-Scripts/](Linux-Commands-Bash-Scripts/) — shell scripts, commands, and install steps
- [EC2-images/](EC2-images/) — EC2 deployment proof screenshots
- [Grafana-images/](Grafana-images/) — Grafana dashboard evidence screenshots
- [Prometheus-images/](Prometheus-images/) — Prometheus validation screenshots

Each linked file or folder corresponds to the current project structure and is clickable on GitHub.

## Project Goals

- Install and configure a monitoring stack on AWS EC2
- Collect Linux host metrics using Node Exporter
- Store and query metrics with Prometheus
- Visualize metrics in Grafana Enterprise
- Run services as systemd-managed daemons
- Demonstrate a working architecture with screenshots and deployed evidence

## Deployed Environment

- Public IP: `3.95.147.178`
- Private IP: `172.31.91.85`
- Public DNS: `ec2-3-95-147-178.compute-1.amazonaws.com`
- AMI: `al2023-ami-2023.12.20260611.0-kernel-6.1-x86_64`

## How It Works

1. **Node Exporter** runs on the EC2 host and exposes system metrics at `http://<EC2-PUBLIC-IP>:9100/metrics`.
2. **Prometheus** scrapes Node Exporter and itself using a configuration file that includes the `node_exporter` and `prometheus` scrape jobs.
3. **Grafana** connects to Prometheus as a data source and displays dashboards from the scraped metrics.
4. All services are configured to start automatically using `systemctl enable` and managed by `systemd`.

## Project Showcase

### About This Project

This repository contains an AWS-based monitoring and alerting architecture built on a single EC2 instance. It demonstrates how to deploy and operate an observability stack using open-source tooling and AWS infrastructure.

### What Was Built

- `Node Exporter` for Linux host metrics
- `Prometheus` for metric collection and storage
- `Grafana Enterprise` for dashboards and visualization
- `systemd` service units for automated uptime
- AWS Security Group rules for secure access

### Why This Project Matters

- Provides visibility into EC2 host performance
- Demonstrates end-to-end monitoring setup on AWS
- Illustrates real-world use of Prometheus and Grafana
- Serves as a strong portfolio project for cloud and DevOps

### Proof of Work

The `EC2-images/`, `Grafana-images/`, `Prometheus-images/`, and `Linux-Commands-Bash-Scripts/` folders contain screenshots from the completed implementation and actual deployed environment:

- `Screenshot 2026-07-03 175005.png` — `node_exporter` service status and logs
- `Screenshot 2026-07-03 175037.png` — `prometheus` service status and logs
- `Screenshot 2026-07-03 175239.png` — AWS EC2 instance summary and instance metadata
- `Screenshot 2026-07-03 175804.png` — `node_exporter` systemd service configuration in `/etc/systemd/system/node_exporter.service`
- `Screenshot 2026-07-03 175822.png` — `grafana-server` service status and logs
- `Screenshot 2026-07-03 180911.png` — Grafana dashboard showing Node Exporter metrics
- `Screenshot 2026-07-03 181137.png` — Grafana Node Exporter dashboard graphs for CPU, memory, network, and disk
- `Screenshot 2026-07-03 181152.png` — Grafana dashboard details with Prometheus data source and node_exporter targets
- `Screenshot 2026-07-03 181300.png` — AWS EC2 instance summary including public IP and instance type
- `Screenshot 2026-07-03 181336.png` — Bash command history showing Node Exporter and Grafana install commands
- `Screenshot 2026-07-03 181419.png` — Prometheus query page showing healthy `up` metrics for `node_exporter` and `prometheus`
- `Screenshot 2026-07-03 181654.png` — Grafana dashboard graphs capturing real system metrics
- `Screenshot 2026-07-03 181944.png` — Additional Grafana dashboard details and metric graphs
- `Screenshot 2026-07-03 182104.png` — Final proof of dashboard and monitoring setup

#### Actual deployed target details from proof images

- Grafana dashboard: `Node Exporter Full`
- Prometheus query results: `up{instance="localhost:9100", job="node_exporter"}` and `up{instance="localhost:9090", job="prometheus"}`
- Node Exporter version: `v1.10.2`
- Grafana Enterprise version: `12.1.1`

### How to Present This on GitHub

1. Use the `README.md` as the primary project landing page.
2. Include the architecture details from `ARCHITECTURE.md`.
3. Provide setup and installation instructions from `INSTALLATION.md`.
4. Reference the `EC2-images/`, `Grafana-images/`, `Prometheus-images/`, and `Linux-Commands-Bash-Scripts/` folders for screenshots and validation.
5. Use this repository's documentation files as the hands-on implementation guide.

### Suggested GitHub Sections

- Project summary and goals
- Architecture and component flow
- Installation and verification steps
- Screenshot gallery or proof reference
- Future improvements
- Contact or author details

### Future Improvements

- Add Alertmanager for alerting workflows
- Create Grafana dashboards for CPU, memory, disk, and network
- Automate deployment with Terraform or Ansible
- Enable HTTPS and Grafana authentication hardening
- Add monitoring for additional AWS resources

## Architecture

- EC2 host runs Grafana, Prometheus, and Node Exporter
- Prometheus scrapes metrics from Node Exporter and its own endpoint
- Grafana reads metrics from Prometheus
- AWS Security Group allows inbound traffic on ports 3000, 9090, 9100, and 22 only

## Documentation Files

This project includes the following documentation files:

- `README.md` — Project overview, showcase content, and proof details
- `ARCHITECTURE.md` — Architecture details and component flow
- `INSTALLATION.md` — Installation and setup instructions

## Proof and Evidence

The `EC2-images/`, `Grafana-images/`, `Prometheus-images/`, and `Linux-Commands-Bash-Scripts/` folders contain screenshots from the completed setup. These images can be used as proof for GitHub and project presentations.

# Author

Mahesh Suryawanshi

maheshsury1shi@gmail.com
