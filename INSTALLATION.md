# Installation and Setup

This guide summarizes the main installation steps for the AWS monitoring project. Use this as the project setup reference when sharing it on GitHub.

## Prerequisites

- An AWS EC2 instance running Amazon Linux 2023
- SSH access with the key pair `grafana.pem`
- A working Security Group with these inbound rules:
  - `22` TCP
  - `3000` TCP
  - `9090` TCP
  - `9100` TCP
- Internet access from the EC2 instance

## Step 1: Update the System

```bash
sudo yum update -y
```

## Step 2: Install Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz

tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
sudo cp node_exporter /usr/local/bin
sudo useradd node_exporter --no-create-home --shell /bin/false
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Create the systemd unit file `/etc/systemd/system/node_exporter.service` with the following content:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Start and enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

Verify Node Exporter in a browser:

```text
http://<EC2-PUBLIC-IP>:9100/metrics
```

## Step 3: Install Grafana Enterprise

```bash
sudo yum install wget tar -y
sudo yum install make -y
sudo yum install -y https://dl.grafana.com/enterprise/release/grafana-enterprise-12.1.1-1.x86_64.rpm
```

Verify installation:

```bash
grafana-server --version
```

Start Grafana:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

Access Grafana:

```text
http://<EC2-PUBLIC-IP>:3000
```

Default credentials:

- Username: `admin`
- Password: `admin`

## Step 4: Install Prometheus

Upload the Prometheus tarball to the EC2 instance, then extract and install it:

```bash
sudo mv prometheus-3.5.3.linux-amd64.tar.gz /opt
cd /opt
sudo tar -xvf prometheus-3.5.3.linux-amd64.tar.gz
sudo mv prometheus-3.5.3.linux-amd64 prometheus
sudo rm prometheus-3.5.3.linux-amd64.tar.gz
```

Create the Prometheus user:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Configure directories and permissions:

```bash
sudo cp /opt/prometheus/prometheus /usr/local/bin/
sudo cp /opt/prometheus/promtool /usr/local/bin/
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo cp /opt/prometheus/prometheus.yml /etc/prometheus/

sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chmod -R 755 /etc/prometheus
sudo chmod -R 755 /var/lib/prometheus
```

Create the Prometheus systemd unit file `/etc/systemd/system/prometheus.service`:

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

Edit `/etc/prometheus/prometheus.yml` to include the scrape jobs:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
n
alerting:
  alertmanagers:
    - static_configs:
        - targets: []

rule_files: []

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
        labels:
          app: "prometheus"

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

Start Prometheus:

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

Access Prometheus:

```text
http://<EC2-PUBLIC-IP>:9090
```

## Step 5: Configure Grafana Data Source

- Open Grafana in the browser
- Add Prometheus as a data source
- Use `http://localhost:9090` for Grafana server configuration on the same host

## Verification

- Node Exporter metrics: `http://<EC2-PUBLIC-IP>:9100/metrics`
- Prometheus UI: `http://<EC2-PUBLIC-IP>:9090`
- Grafana UI: `http://<EC2-PUBLIC-IP>:3000`

## Notes

This installation guide is based on the project documentation. Keep the `EC2-images/`, `Grafana-images/`, `Prometheus-images/`, and `Linux-Commands-Bash-Scripts/` folders as evidence of the completed setup and dashboards.