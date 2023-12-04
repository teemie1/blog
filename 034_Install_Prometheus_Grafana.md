# Install Prometheus & Grafana

## Install Prometheus & Configuration
### Creating Prometheus System Users and Directory
~~~
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo mkdir /etc/prometheus
sudo mkdir /data/prometheus
sudo ln -s /data/prometheus /var/lib/prometheus
~~~

### Update Prometheus user
~~~
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /data/prometheus
~~~

### Download Prometheus Binary File
~~~
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/prometheus-2.48.0.linux-amd64.tar.gz
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/prometheus-web-ui-2.48.0.tar.gz
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/sha256sums.txt
~~~
### Install Prometheus and Grafana on Ubuntu
~~~
sha256sum prometheus-2.48.0.linux-amd64.tar.gz
sha256sum prometheus-web-ui-2.48.0.tar.gz
tar -xvf prometheus-2.48.0.linux-amd64.tar.gz
cd prometheus-2.48.0.linux-amd64.tar.gz
~~~
### Copy Prometheus Binary files
~~~
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
~~~
### Update Prometheus user ownership on Binaries
~~~
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
~~~

### Copy Prometheus Console Libraries
~~~
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp -r prometheus.yml /etc/prometheus
~~~
### Update Prometheus ownership on Directories
~~~
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
~~~
### Check Prometheus Version
~~~
prometheus --version
promtool --version
~~~
### Prometheus configuration file
~~~
cat /etc/prometheus/prometheus.yml
# my global config
global:
scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
# scrape_timeout is set to the global default (10s).
# Alertmanager configuration
alerting:
alertmanagers:
- static_configs:
- targets:
# - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
# - "first_rules.yml"
# - "second_rules.yml"
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
# The job name is added as a label `job=&lt;job_name&gt;` to any timeseries scraped from this config.
- job_name: 'prometheus'
# metrics_path defaults to '/metrics'
# scheme defaults to 'http'.
static_configs:
- targets: ['localhost:9090']
~~~
### Creating Prometheus Systemd file
~~~
sudo -u prometheus /usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
sudo nano /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
 

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
 

[Install]
WantedBy=multi-user.target
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
~~~
### Accessing Prometheus
~~~
sudo ufw allow 9090/tcp
http://sdc-bitwarden.duckdns.org:9090
~~~
## Install Grafana & Configuration
~~~
cd /tmp
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server.service
sudo ufw allow 3000/tcp
http://sdc-bitwarden.duckdns.org:3000
Username – admin
Password – admin

On browser
Settings Icon ->> Configuration ->> data sources
Add Data sources and select Prometheus
~~~
## Install Node Exporter
~~~
sudo useradd --no-create-home --shell /bin/false node_exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
sudo tar xvzf node_exporter-1.7.0.linux-amd64.tar.gz
cd node_exporter-1.7.0.linux-amd64
sudo cp node_exporter /usr/local/bin
sudo chown node_exporter.node_exporter /usr/local/bin/node_exporter
cd /lib/systemd/system
sudo nano node_exporter.service
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter \
--collector.mountstats \
--collector.logind \
--collector.processes \
--collector.ntp \
--collector.systemd \
--collector.tcpstat \
--collector.wifi
Restart=always
RestartSec=10s
[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
~~~

### Configure the Node Exporter as a Prometheus target
~~~
cd /etc/prometheus
sudo nano prometheus.yml
- targets: [‘localhost:9090’, ‘localhost:9100’]
sudo systemctl restart prometheus
sudo ufw allow 9100/tcp
https://localhost:9100/targets
~~~
### Creating Grafana Dashboard to Monitor Linux Server
~~~
Add dashboard
Go to Dashboards
Select "New" --> "New Dashboard"
Select "Import Dashboard"
Fill "1860" (Node Exporter Full) and click "Load"
Search more dashboard template at https://grafana.com/grafana/dashboards
~~~
