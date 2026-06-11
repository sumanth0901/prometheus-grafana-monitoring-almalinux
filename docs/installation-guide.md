# Installation Guide: Prometheus + Node Exporter + Grafana on AlmaLinux

## Objective

Deploy a monitoring stack on AlmaLinux using:

* Prometheus
* Node Exporter
* Grafana

Server IP:

```text
192.168.1.24
```

---

# Step 1: Install Prometheus

Create Prometheus user:

```bash
sudo useradd --no-create-home --shell /sbin/nologin prometheus
```

Download Prometheus:

```bash
cd /tmp

wget https://github.com/prometheus/prometheus/releases/download/v3.12.0/prometheus-3.12.0.linux-amd64.tar.gz
```

Extract:

```bash
tar -xvf prometheus-3.12.0.linux-amd64.tar.gz

cd prometheus-3.12.0.linux-amd64
```

Create directories:

```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

Copy files:

```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo cp prometheus.yml /etc/prometheus/
```

Set permissions:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

---

# Step 2: Configure Prometheus Service

Create service file:

```bash
sudo vi /etc/systemd/system/prometheus.service
```

Contents:

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus

ExecStart=/usr/local/bin/prometheus \
 --config.file=/etc/prometheus/prometheus.yml \
 --storage.tsdb.path=/var/lib/prometheus \
 --web.listen-address=:9091

Restart=always

[Install]
WantedBy=multi-user.target
```

Enable service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

Verify:

```bash
systemctl status prometheus
```

---

# Step 3: Install Node Exporter

Create user:

```bash
sudo useradd --no-create-home --shell /sbin/nologin node_exporter
```

Download:

```bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.11.1/node_exporter-1.11.1.linux-amd64.tar.gz
```

Extract:

```bash
tar -xvf node_exporter-1.11.1.linux-amd64.tar.gz

cd node_exporter-1.11.1.linux-amd64
```

Copy binary:

```bash
sudo cp node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Create service:

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

Contents:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Verify:

```bash
systemctl status node_exporter
```

---

# Step 4: Configure Prometheus Targets

Edit:

```bash
sudo vi /etc/prometheus/prometheus.yml
```

Configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9091"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

Validate:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

Restart:

```bash
sudo systemctl restart prometheus
```

Verify:

```text
http://192.168.1.24:9091/targets
```

Expected:

```text
prometheus      UP
node_exporter   UP
```

---

# Step 5: Install Grafana

Create repository:

```bash
sudo tee /etc/yum.repos.d/grafana.repo <<EOF
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
EOF
```

Install:

```bash
sudo dnf install grafana -y
```

Enable service:

```bash
sudo systemctl enable --now grafana-server
```

Verify:

```bash
systemctl status grafana-server
```

---

# Step 6: Firewall Configuration

```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --permanent --add-port=9091/tcp
sudo firewall-cmd --permanent --add-port=9100/tcp

sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --list-ports
```

---

# Step 7: Configure Grafana

Access:

```text
http://192.168.1.24:3000
```

Default credentials:

```text
Username: admin
Password: admin
```

Change password after first login.

---

# Step 8: Add Prometheus Data Source

Navigate:

```text
Connections
→ Data Sources
→ Add Data Source
→ Prometheus
```

URL:

```text
http://localhost:9091
```

Save and Test.

---

# Step 9: Import Dashboard

Navigate:

```text
Dashboards
→ Import
```

Dashboard ID:

```text
1860
```

Node Exporter Full Dashboard

---

# Troubleshooting

## Port Conflict

Issue:

```text
Unable to start web listener
listen tcp 0.0.0.0:9090: bind: address already in use
```

Cause:

```text
Cockpit was already using port 9090
```

Resolution:

```text
Prometheus was moved to port 9091
```

---

# Validation

Successful monitoring of:

* CPU Usage
* Memory Usage
* Disk Usage
* Filesystem Usage
* Network Usage
* Load Average

using Prometheus, Node Exporter, and Grafana on AlmaLinux.

