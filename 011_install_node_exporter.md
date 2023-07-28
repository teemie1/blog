# Install Prometheus node exporter on linux os

## Install node exporter
~~~
# Login CLN Node
$ sudo useradd --no-create-home --shell /bin/false node_exporter
$ cd /tmp
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
$ sudo tar xvzf node_exporter-1.6.1.linux-amd64.tar.gz
$ cd node_exporter-1.6.1.linux-amd64
$ sudo cp node_exporter /usr/local/bin
$ sudo chown node_exporter.node_exporter /usr/local/bin/node_exporter
$ cd /lib/systemd/system
$ sudo nano node_exporter.service
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

$ sudo systemctl daemon-reload
$ sudo systemctl enable node_exporter
$ sudo systemctl start node_exporter
$ sudo systemctl status node_exporter
~~~
## Configure the Node Exporter as a Prometheus target
~~~
# Login Prometheus Server
$ cd /etc/prometheus
$ sudo nano prometheus.yml
- targets: [‘localhost:9090’, ‘localhost:9100’]
$ sudo systemctl restart prometheus
$ sudo ufw allow 9100/tcp

# Go to https://localhost:9100/targets
~~~
