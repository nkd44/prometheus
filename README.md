# Authentication (self-signed Certificate) between prometheus Server and target.

1.1 Prometheus Server & Configuration

Steps:

1-Configure the Prometheus server with a static IP address

First, update your package list to ensure you access the most recent versions available

$ sudo apt update

2-Next, install the Prometheus package:

$ sudo apt install prometheus prometheus-node-exporter prometheus-pushgateway prometheus-alertmanager

3-Check the main configuration file for Prometheus is located at /etc/prometheus/prometheus.yml.

 $ sudo vi /etc/prometheus/prometheus.yml

4-Enable Prometheus to start on boot and start the service

$ sudo systemctl enable prometheus

$ sudo systemctl start prometheus

5-You can also access Prometheus in a browser using the server’s IP address:

http://<PROMETHEUS_SERVER_IP>:9090

1.2 Prometheus Configuration:
Steps:

1- Login to the server where Prometheus is running then edit the config file

$ sudo vim /etc/prometheus/prometheus.yml


2- Locate the global.scrape_interval setting and change it to 10s:

global:

  scrape_interval: 10s
	
3- Reload the Prometheus configuration:

$ sudo systemctl restart Prometheus

2.1 Install & Configure Exporter:

1- Login to the machine that we will use for monitoring by Prometheus then create a user for Node Exporter:

  $ sudo useradd -M -r -s /bin/false node_exporter

2- Download and extract the Node Exporter binary:

  $ wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz


  $ tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz

3- Copy the Node Exporter binary to the appropriate location and set ownership:

  $ sudo cp node_exporter-1.5.0.linux-amd64/node_exporter  /usr/local/bin/


  $ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

4- Create a systemd unit file for Node Exporter:

  $ sudo vim /etc/systemd/system/node_exporter.service

***** Define the Node Exporter service in the unit file:

  [Unit]
	
  Description=Prometheus Node Exporter
	
  Wants=network-online.target
	
  After=network-online.target


  [Service]
	
  User=node_exporter
	
  Group=node_exporter
	
  Type=simple
	
  ExecStart=/usr/local/bin/node_exporter


  [Install]
	
  WantedBy=multi-user.target
 
  Start and enable the node_exporter service:
	
  $ sudo systemctl daemon-reload
	
  $ sudo systemctl start node_exporter
	
  $ sudo systemctl enable node_exporter


2.2 You can retrieve the metrics served by Node Exporter like so:

  $ curl localhost:9100/metrics

3.1 Configure Prometheus to Scrape Metrics

 1- Edit the Prometheus config file:
 $ sudo vim /etc/prometheus/prometheus.yml

******** Locate the scrape_configs section and add a new entry under that section. You will need to supply the private IP address of your new server for targets.
...
- job_name: 'Linux Server'
  
  static_configs:
  
  - targets: ['localhost:9100']
...

Validate config file:

$ promtool check config /etc/prometheus/prometheus.yml

Reload the Prometheus config:

sudo systemctl restart Prometheus


