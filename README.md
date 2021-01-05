# Raspberry Pi Supervision Dashboard
<u>_*RasPi_Supervision.json*_</u> is using Node Exporter and Cadvisor with Prometheus to display the load of a Raspberry Pi or Synology NAS or possibly any other Server/NAS.   
<img src="https://github.com/Froschie/raspi-supervision/raw/main/RasPi_Supervision.png" width="910" height="1114" alt="Grafana Dashboard">

This Dashboard is based on:  
[Prometheus Node Exporter Full](https://grafana.com/grafana/dashboards/1860)  
[Docker Container](https://grafana.com/grafana/dashboards/11600)  
[SMART disk data](https://grafana.com/grafana/dashboards/10664)

Just import the *RasPi_Supervision.json* file into your Grafana Instance with the configured Prometheus Datasource.  


# Example Setup with Docker Compose File on Raspberry Pi

The *docker-compose.yml* example consists of 3 containers:  
- [prom/node-exporter](https://hub.docker.com/r/prom/node-exporter)
- [zcube/cadvisor](https://hub.docker.com/r/zcube/cadvisor)
- [prom/prometheus](https://hub.docker.com/r/prom/prometheus)

Node Exporter and Cadvisor do not require and further configuration.  
Prometheus need to be properly setup to include a valid configuration file containing the data import job and storage location for the database.  

Example Scrape Job in *prometheus.yml* file:
```
  - job_name: 'RasPi4'
    scrape_interval: 60s
    static_configs:
    - targets: ['192.168.0.10:8009']
    - targets: ['192.168.0.10:9100']
```

Each Job should be correspond to one Server and can contain Node Exporter and/or Cadvisor target.


To use the example Docker Compose file do:
```bash
mkdir raspi-supervision
cd raspi-supervision
curl -O https://raw.githubusercontent.com/Froschie/raspi-supervision/main/docker-compose.yml
vi docker-compose.yaml
docker-compose up -d
```
*Note: please adapt the parameters as needed!*


# Node Exporter with smartmontools on Synology NAS.

As *root* user on your Synology NAS download [Node Exporter](https://github.com/prometheus/node_exporter/releases/) and [smartmon.sh](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts/blob/master/smartmon.sh).

Example command, use the latest _and_ correct architecture SW versions:
```bash
cd ~
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
curl -O  https://raw.githubusercontent.com/prometheus-community/node-exporter-textfile-collector-scripts/master/smartmon.sh
tar -xvf node_exporter-1.0.1.linux-amd64.tar.gz
sed -i "s/device_list=.*/device_list=\"\$(cat smartmon.disks)\"/" smartmon.sh
vi smartmon.disks
vi /etc/crontab
synoservicecfg --restart crond
/root/node_exporter-1.0.1.linux-amd64/node_exporter --collector.textfile.directory=/root &
```
*Note: please adapt the directory names if necessary!*  

Inside of *smartmon.disks*, please specify the harddrives of your NAS like:
```
/dev/sda|sat
/dev/sdb|sat
```
*Note: the smartctl scan will detect the drives as scsi which will not report the SMART values properly so they need to be forced to be "__sat__"!*

For */etc/crontab* enter the lines:  
```
*/5 *   *   *   *   root    cd /root && ./smartmon.sh > /root/smartmon.prom.$$ && mv /root/smartmon.prom.$$ /root/smartmon.prom
@reboot   root    /root/node_exporter-1.0.1.linux-amd64/node_exporter --collector.textfile.directory=/root
```
*Note: please adapt the directory names if necessary!*

This will start the SMART attribues export every 5min and the Node Exporter itself after reboot.
