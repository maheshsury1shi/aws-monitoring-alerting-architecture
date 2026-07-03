# Architecture

## Overview

This architecture document describes a production-inspired AWS monitoring and observability stack deployed on a single EC2 instance using Amazon Linux 2023. The design focuses on reliability, simplicity, and end-to-end visibility for host-level infrastructure metrics.

## Architecture Components

- **AWS EC2 Instance**
  - Instance type: `c7i-flex.large`
  - Operating system: Amazon Linux 2023
  - AMI: `al2023-ami-2023.12.20260611.0-kernel-6.1-x86_64`
  - Public IP: `3.95.147.178`
  - Private IP: `172.31.91.85`
  - Public DNS: `ec2-3-95-147-178.compute-1.amazonaws.com`
  - Purpose: hosts the monitoring stack in a self-contained environment

- **Node Exporter**
  - Role: collects hardware and OS metrics from the EC2 host
  - Exposes metrics on port `9100`
  - Metrics include CPU, memory, disk usage, network, and filesystem statistics

##  Architecture Diagram

```text
                       +-------------------+
                       |   User Browser    |
                       | (remote client)   |
                       +---------+---------+
                                 |
                                 | HTTPS / HTTP
                                 |
                       +---------v---------+
                       | AWS EC2 Instance   |
                       | (Amazon Linux 2023)|
                       +---------+---------+
                                 |
               +-----------------+-----------------+
               |                                   |
        +------v------+                    +-------v-------+
        |   Grafana    |                    |  Prometheus   |
        | port 3000    |<-------------------| port 9090     |
        | Dashboards   |  queries metrics   | Scrapes       |
        +------+------+                    +-------+-------+
               |                                   |
               |                                   |
               |                                   |
      +--------v--------+                  +--------v--------+
      |  User browser /  |                  |  Node Exporter  |
      |  external access  |                  |  port 9100      |
      +------------------+                  |  Host metrics   |
                                             +----------------+
```

The current architecture diagram shows a single-host deployment where Grafana, Prometheus, and Node Exporter all run on the same EC2 instance. The user browser connects to Grafana on port `3000`, Grafana queries Prometheus on port `9090`, and Prometheus scrapes Node Exporter on port `9100`.

## ASCII Workflow Diagram

```text
[User Browser] --HTTPS--> [Grafana 3000]
      |                      |
      |                      v
      |               [Prometheus 9090]
      |                      |
      |                      v
      +---------------- [Node Exporter 9100]
                             |
                             v
                   [EC2 Host Metrics / Linux OS]
```

The workflow diagram describes the operational data flow explicitly: the browser requests dashboards from Grafana, Grafana issues Prometheus queries, and Prometheus scrapes host metrics from Node Exporter. Since all components live on the same EC2 instance, the service-to-service traffic is local, while external dashboard access happens through the instance public address.

- **Prometheus**
  - Role: scrapes metrics from Node Exporter and stores them as time-series data
  - Exposes its own metrics and query UI on port `9090`
  - Uses a static scrape configuration for reliability
  - Managed as a systemd service with `Restart=always` for improved availability

- **Grafana Enterprise**
  - Role: visualizes time-series metrics from Prometheus
  - Provides dashboards and analytics UI on port `3000`
  - Configured as an enterprise deployment for enhanced features

- **systemd**
  - Role: manages service lifecycle for Node Exporter, Prometheus, and Grafana
  - Ensures services auto-start after reboot and simplifies operational monitoring

## Logical Data Flow

1. **Node Exporter** collects host metrics and publishes them at:
   - `http://localhost:9100/metrics`
2. **Prometheus** scrapes Node Exporter at regular intervals and stores metrics in TSDB.
3. **Prometheus** also self-scrapes its own endpoint for internal monitoring.
4. **Grafana** queries Prometheus as its primary data source and renders dashboards.
5. End users access Grafana through the browser to inspect metrics and alerts.

## Network and Access Model

- **Inbound access** is controlled via AWS Security Group rules:
  - `22/TCP` — SSH administration
  - `3000/TCP` — Grafana UI
  - `9090/TCP` — Prometheus UI
  - `9100/TCP` — Node Exporter metrics

- **Internal communication** is localhost-based on the EC2 instance:
  - Grafana → Prometheus
  - Prometheus → Node Exporter

- **Browser access** is through the public IP of the EC2 host.

## Systemd Service Design

The following systemd units are used to manage the stack:

- `node_exporter.service`
- `prometheus.service`
- `grafana-server.service`

Benefits of this approach:

- deterministic service startup ordering
- consistent restart behavior
- centralized logging with `journalctl`
- simplified operational commands for status and health checks

## Deployment Summary

This architecture demonstrates a clean separation of concerns:

- **Metric collection**: Node Exporter
- **Metric storage and retrieval**: Prometheus
- **Metric visualization**: Grafana Enterprise
- **Service reliability**: systemd & AWS Security Group

## Validation and Scope

- This is a single-host architecture: all components run on the same EC2 instance.
- Prometheus scrapes metrics from localhost targets (`localhost:9100` for Node Exporter and `localhost:9090` for itself).
- Grafana queries Prometheus locally and serves dashboards on port `3000`.
- There is no separate remote exporter or distributed cluster in this setup.

This confirms the architecture is correct for the deployed project.

## Professional Highlights

- Uses AWS-hosted infrastructure with secure access controls
- Supports end-to-end observability and metrics-driven insights
- Employs industry-standard tooling for monitoring and dashboarding
- Designed for portability and future automation with Terraform or Ansible
