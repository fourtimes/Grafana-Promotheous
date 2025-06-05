

---

# Prometheus Setup Guide

## Overview

This guide covers setting up Prometheus along with Node Exporter and Alertmanager. It includes configuration details, steps to set up services, and troubleshooting tips.

## Prerequisites

- A Linux-based system (e.g., Ubuntu)
- Access to terminal with sudo privileges
- Basic knowledge of Linux commands and configuration files

## 1. **Installing Prometheus**

1. **Download Prometheus**:
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.54.0-rc.1/prometheus-2.54.0-rc.1.linux-amd64.tar.gz
   ```

2. **Extract the tarball**:
   ```bash
   tar -xzvf prometheus-2.54.0-rc.1.linux-amd64.tar.gz
   ```

3. **Create a Prometheus user**:
   ```bash
   sudo useradd --no-create-home --shell /bin/false prometheus
   ```

4. **Move Prometheus files and set permissions**:
   ```bash
   sudo mv prometheus-2.54.0-rc.1.linux-amd64/prometheus /usr/local/bin/
   sudo mv prometheus-2.54.0-rc.1.linux-amd64/promtool /usr/local/bin/
   sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
   sudo mkdir /etc/prometheus
   sudo mkdir /var/lib/prometheus
   sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
   ```

5. **Create the Prometheus systemd service file**:
   ```bash
   sudo nano /etc/systemd/system/prometheus.service
   ```

   ```ini
    [Unit]
   Description=Prometheus
   Wants=network-online.target
   After=network-online.target

   [Service]
   User=prometheus
   Group=prometheus
   ExecStart=/usr/local/bin/prometheus \
     --config.file=/etc/prometheus/prometheus.yml \
     --storage.tsdb.path=/var/lib/prometheus/data \
     --storage.tsdb.retention.time=90d \
     --web.console.templates=/usr/local/share/prometheus/consoles \   
     --web.console.libraries=/usr/local/share/prometheus/console_libraries \
     --web.listen-address=0.0.0.0:19091 \
     --log.level=info
   StandardOutput=syslog
   StandardError=syslog
   SyslogIdentifier=prometheus

   [Install]
   WantedBy=multi-user.target


   ```

6. **Start and enable Prometheus**:
   ```bash
   sudo systemctl stop prometheus
   sudo systemctl daemon-reload
   sudo systemctl start prometheus
   sudo systemctl enable prometheus
   ```
7.   **Check the Logs**:

Now, Prometheus should be logging to syslog. You can check the logs using:

```bash
sudo tail -f /var/log/syslog | grep prometheus
```


###  Expected outcome 
![Image](https://github.com/devopsflash/test/blob/main/Screenshot%20from%202024-08-16%2014-05-58.png)



After completing [Step 2](https://github.com/rcms-org/proof-of-concepts/blob/main/docs/monitor/node%20exporter.md), [Step 3](https://github.com/rcms-org/proof-of-concepts/blob/main/docs/monitor/Alertmanager.md), and [Step 4](https://github.com/rcms-org/proof-of-concepts/blob/main/docs/monitor/Integration%20with%20Slack.md), you should proceed to the next section to configure Prometheus to monitor these services.

## 5. **Configuring Prometheus**

Edit the Prometheus configuration file `/etc/prometheus/prometheus.yml` to include your Node Exporter and Alertmanager targets:

```yaml
   global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:19097']

rule_files:
  - /etc/prometheus/linux.yml
  - /etc/prometheus/linux1.yml
  - /etc/prometheus/linux2.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:19091']
        labels:
          instance: 'prometheus'

  - job_name: 'local_node_exporter'
    static_configs:
      - targets: ['localhost:19095']
        labels:
          instance: 'erp'

  - job_name: 'remote_node_exporter'
    static_configs:
      - targets: ['192.168.1.181:19095']
        labels:
          instance: 'node1'
```
