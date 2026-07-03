# Troubleshooting

This document provides troubleshooting steps and commands for the AWS Monitoring & Alerting Architecture project.

## 1. Service Startup Issues

### Check service status

```bash
sudo systemctl status node_exporter
sudo systemctl status prometheus
sudo systemctl status grafana-server
```

### Review logs

```bash
sudo journalctl -u node_exporter --no-pager | tail -n 50
sudo journalctl -u prometheus --no-pager | tail -n 50
sudo journalctl -u grafana-server --no-pager | tail -n 50
```

### Reload systemd if unit files changed

```bash
sudo systemctl daemon-reload
sudo systemctl restart node_exporter
sudo systemctl restart prometheus
sudo systemctl restart grafana-server
```

## 2. Node Exporter Issues

### Verify binary exists and permissions

```bash
ls -l /usr/local/bin/node_exporter
```

Expected owner/group:

```bash
node_exporter node_exporter
```

### Confirm service unit file

`/etc/systemd/system/node_exporter.service` should contain:

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

### Test metrics endpoint locally

```bash
curl http://localhost:9100/metrics | head
```

If the endpoint is not reachable:
- ensure `node_exporter` is running
- check for port conflicts
- verify the service unit path is correct

## 3. Prometheus Issues

### Verify Prometheus binary and permissions

```bash
ls -l /usr/local/bin/prometheus
ls -l /usr/local/bin/promtool
```

Expected owner/group:

```bash
prometheus prometheus
```

### Check Prometheus config

```bash
cat /etc/prometheus/prometheus.yml
```

Scrape config should include:

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

### Verify Prometheus service unit

`/etc/systemd/system/prometheus.service` should contain:

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

### Test Prometheus local access

```bash
curl http://localhost:9090/-/ready
curl http://localhost:9090/metrics | head
```

If Prometheus is not scraping Node Exporter:
- confirm `node_exporter` is reachable at `localhost:9100`
- validate `prometheus.yml` syntax
- restart Prometheus after config changes

## 4. Grafana Issues

### Check Grafana service

```bash
sudo systemctl status grafana-server
```

### Verify Grafana listens on port 3000

```bash
sudo ss -tuln | grep 3000
```

### Troubleshoot login and data source

- Access `http://<EC2-PUBLIC-IP>:3000`
- Default Grafana credentials: `admin` / `admin`
- In Grafana, verify Prometheus data source URL is `http://localhost:9090`
- Test the data source connection inside Grafana

### Common Grafana issues

- If dashboards fail to load, verify Prometheus is up and scrape targets are healthy.
- If login fails, clear browser cache or restart Grafana.

## 5. Network and Security Group Issues

### Verify open ports

The AWS Security Group should allow:

- `22/TCP` for SSH
- `3000/TCP` for Grafana
- `9090/TCP` for Prometheus
- `9100/TCP` for Node Exporter

### Check firewall rules on EC2 instance

```bash
sudo firewall-cmd --list-all
```

If firewall is not used, ensure the operating system does not block required ports.

## 6. File and Directory Permissions

### Prometheus directories

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chmod -R 755 /etc/prometheus
sudo chmod -R 755 /var/lib/prometheus
```

### Node Exporter binary

```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

## 7. Validation and Evidence

### Verify each service endpoint

```bash
curl http://localhost:9100/metrics
curl http://localhost:9090
curl http://localhost:3000
```

### Prometheus target health

Query in Prometheus:

```promql
up
```

Healthy targets should show `1` for both `node_exporter` and `prometheus`.

## 8. Recovery Steps

### Restart services

```bash
sudo systemctl restart node_exporter
sudo systemctl restart prometheus
sudo systemctl restart grafana-server
```

### Check service status again

```bash
sudo systemctl status node_exporter prometheus grafana-server
```

### Reload systemd units if needed

```bash
sudo systemctl daemon-reload
```

## 9. Notes

- Keep screenshots in the `EC2-images/`, `Grafana-images/`, `Prometheus-images/`, and `Linux-Commands-Bash-Scripts/` folders as proof of working install and configuration.
- Use the `INSTALLATION.md` and `COMMANDS.md` files for exact command references.
- If a service fails repeatedly, inspect the corresponding journal logs for detailed errors.
