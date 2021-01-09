# Raspberry Pi Supervision Dashboard
<u>_*RasPi_Supervision.json*_</u> is using [Node Exporter](https://github.com/prometheus/node_exporter), [Cadvisor](https://github.com/google/cadvisor), [smartmon.sh](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts/blob/master/smartmon.sh) and [Infux Stats Exporter](https://github.com/carlpett/influxdb_stats_exporter) with Prometheus to display the load of a Raspberry Pi or Synology NAS or possibly any other Server/NAS.   
<img src="https://github.com/Froschie/raspi-supervision/raw/main/RasPi_Supervision.png" width="910" height="1114" alt="Grafana Dashboard">

This Dashboard is based on:  
[Prometheus Node Exporter Full](https://grafana.com/grafana/dashboards/1860)  
[Docker Container](https://grafana.com/grafana/dashboards/11600)  
[SMART disk data](https://grafana.com/grafana/dashboards/10664)
[InfluxDB stats](https://grafana.com/grafana/dashboards/5448)


# Grafana  

To use the Dashboard you need a Grafana Instance.  
To be described here...
```
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - GF_SERVER_ROOT_URL=http://192.168.0.10:3000
      - GF_INSTALL_PLUGINS=yesoreyeram-boomtable-panel
      - GF_DATE_FORMATS_FULL_DATE=DD.MM.YYYY HH:mm:ss
      - GF_DATE_FORMATS_INTERVAL_HOUR=DD.MM HH:mm
      - GF_DATE_FORMATS_INTERVAL_MONTH=MM.YYYY
      - GF_DATE_FORMATS_INTERVAL_DAY=DD.MM
    volumes:
      - /<path to grafana files>:/var/lib/grafana:rw
```

Pre-requisite:  
[Boom Table](https://github.com/yesoreyeram/yesoreyeram-boomtable-panel) Plugin

Then configure Prometheus unter __*Configuration -> Data Sources*Datasource*__.  

Finally import the *RasPi_Supervision.json* file.

But actually this is the last step, first setup Prometheus and the data collectors:

# Node Exporter  

Node Exporter will collect hardware and OS counters and make them available as text file available via web interface (default: port 9100). E.g. starting Node Exporter on a Host with IP 192.168.0.10, the counters will be presented via *http://192.168.0.10:9100/metrics*.  
The base address __*192.168.0.10:9100*__ must be entered as target in Prometheus.  

# Cadvisor  

Cadvisor collects counters from Docker or Kubernetes and same as Node Exporter makes them available via web interface.  
In the provided example *docker-compose.yml* exposed on port 8009.  
The base address __*192.168.0.10:8009*__ must be entered as target in Prometheus.  

# Prometheus  

Is a monitoring system based on a time series database collecting data via defineable jobs.  

Example Scrape Job in *prometheus.yml* file:
```
  - job_name: 'RasPi4'
    scrape_interval: 60s
    static_configs:
    - targets: ['192.168.0.10:8009']
    - targets: ['192.168.0.10:9100']
```


# Example Setup with Docker Compose File on Raspberry Pi

The *docker-compose.yml* example consists of 3 containers:  
- [prom/node-exporter](https://hub.docker.com/r/prom/node-exporter)
- [zcube/cadvisor](https://hub.docker.com/r/zcube/cadvisor)
- [prom/prometheus](https://hub.docker.com/r/prom/prometheus)

Node Exporter and Cadvisor do not require and further configuration.  
Prometheus need to be properly setup to include a valid configuration file containing the data import job and storage location for the database. See above for example description.  

Each Job should be correspond to one host and can contain Node Exporter, Cadvisor and InfluxDB targets.

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

Node Exporter does not load SMART informations from hard drives.  
For this the extension script *smartmon.sh* can be used which stores the SMART data as text file in the file system where it is read by Node Exporter.  

As *root* user on your Synology NAS download [Node Exporter](https://github.com/prometheus/node_exporter/releases/) and [smartmon.sh](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts/blob/master/smartmon.sh).

*Note: smartmon.sh uses the command "smartctl --scan" to detect the disk drives, however this will result in the disks to be detected as scsi which will not report the SMART values properly. The type must be forced to "__sat__"! Therefor the smartmon.sh script will be modified via sed command.*.  

Example commands, exchange the node_exporter file with the correct file for your NAS device (correct architecture):
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

To run Node Exporter and smartmon.sh enter cron jobs in file */etc/crontab*:  
```
*/5 *   *   *   *   root    cd /root && ./smartmon.sh > /root/smartmon.prom.$$ && mv /root/smartmon.prom.$$ /root/smartmon.prom
@reboot   root    /root/node_exporter-1.0.1.linux-amd64/node_exporter --collector.textfile.directory=/root
```
*Note: please adapt the directory names if necessary!*

This will start the SMART attribues export every 5min and the Node Exporter itself after reboot.

# InfluxDB Stats Exporter

InfluxDB Stats Exporter uses the `SHOW STATS` query to present metrics about the database.  
To be described...
