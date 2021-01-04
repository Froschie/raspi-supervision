# Raspberry Pi Supervision Dashboard
<u>_*RasPi_Supervision.json*_</u> is using Node Exporter and Cadvisor with Prometheus to display the load of a Raspberry Pi or any other Server/NAS.   
<img src="https://github.com/Froschie/raspi-supervision/raw/main/RasPi_Supervision.png" width="910" height="1114" alt="Grafana Dashboard">

This Dashboard is based on:  
[Prometheus Node Exporter Full](https://grafana.com/grafana/dashboards/1860)  
[Docker Container](https://grafana.com/grafana/dashboards/11600)  

Just import the *RasPi_Supervision.json* file into your Grafana Instance with the configured Prmetheus Datasouse.  


# Example Setup with Docker Compose File

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
