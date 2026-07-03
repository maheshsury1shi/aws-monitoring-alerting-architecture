# AWS Monitoring & Alerting Architecture

## Overview

This project demonstrates a complete AWS monitoring stack deployed on a single EC2 instance running Amazon Linux 2023. It includes:

- Prometheus for metrics collection and storage
- Grafana Enterprise for visualization and dashboards
- Node Exporter for host-level system metrics
- Systemd service management for reliability
- AWS Security Group configuration for secure network access

The goal is to build a production-like observability architecture that can be showcased on GitHub with accompanying proof images.

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

## Project Structure

```text
AWS Monitoring & Alerting Architecture/
├── ARCHITECTURE.md
├── COMMANDS.md
├── INSTALLATION.md
├── README.md
├── TROUBLESHOOTING.md
├── dashboard-codes.md
├── Linux-Commands-Bash-Scripts/
├── EC2-images/
├── Grafana-images/
└── Prometheus-images/
```

- `ARCHITECTURE.md` explains the monitoring stack design, data flow, and validation details.
- `INSTALLATION.md` contains the exact setup steps for Node Exporter, Prometheus, and Grafana.
- `README.md` is the main project landing page, documentation summary, and showcase.
- `EC2-images/`, `Grafana-images/`, and `Prometheus-images/` store proof screenshots from deployment.

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

## Next Steps

- Add Grafana dashboards and alerting rules
- Convert setup into Terraform or Ansible automation
- Add TLS, authentication, and production security hardening
- Integrate Alertmanager for notification workflows
