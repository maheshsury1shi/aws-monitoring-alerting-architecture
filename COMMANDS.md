# Command Reference

This file contains the full set of commands used to build the AWS Monitoring & Alerting Architecture project from start to finish.

## 1. Update the EC2 Instance

```bash
sudo yum update -y
```

## 2. Install Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz

tar xvfz node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64
sudo cp node_exporter /usr/local/bin
sudo useradd node_exporter --no-create-home --shell /bin/false
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

## 3. Create Node Exporter systemd Service

Create `/etc/systemd/system/node_exporter.service` with the following content:

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

Start and enable service:

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

## 4. Install Grafana Enterprise

```bash
sudo yum install wget tar -y
sudo yum install make -y
sudo yum install -y https://dl.grafana.com/enterprise/release/grafana-enterprise-12.1.1-1.x86_64.rpm
```

Verify Grafana version:

```bash
grafana-server --version
```

Start and enable Grafana:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

## 5. Install Prometheus

Upload the Prometheus tarball to the EC2 instance, then run:

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

Copy binaries and configure directories:

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

## 6. Create Prometheus systemd Service

Create `/etc/systemd/system/prometheus.service` with the following content:

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

## 7. Update Prometheus Configuration

Edit `/etc/prometheus/prometheus.yml` to use the following scrape configuration:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

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

## 8. Start Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

## 9. Configure Grafana Data Source

In Grafana UI:

- Add Prometheus as a data source
- Use `http://localhost:9090` as the Prometheus URL

## 10. Verification Commands

```bash
# Verify Node Exporter metrics endpoint
curl http://localhost:9100/metrics

# Verify Prometheus web UI is reachable
curl http://localhost:9090

# Check Grafana service status
sudo systemctl status grafana-server
```

## 11. Access URLs

- Grafana: `http://<EC2-PUBLIC-IP>:3000`
- Prometheus: `http://<EC2-PUBLIC-IP>:9090`
- Node Exporter: `http://<EC2-PUBLIC-IP>:9100/metrics`

## Notes

- Replace `<EC2-PUBLIC-IP>` with your instance's actual public IP.
- Use the `EC2-images/`, `Grafana-images/`, `Prometheus-images/`, and `Linux-Commands-Bash-Scripts/` folders for proof and validation of the deployed environment.
