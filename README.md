# chia-monitoring

This repo is meant to provide a basic monitoring for your chia farming/plotting hardware. You should be able to find everything you need here

## Features
* Near realtime monitoring for used disk space, disk utilization, RAM and CPU usage
* Health overview over your HDDs and SSDs
* Historical data for as long as you want (Provided you have space to store them)

## Requirements
For all of this to work, you will need some software installed on your system. This can be the plotter, the farm system or basically any system you want to use *as long as it can run docker*

## Installation
### docker
For ubuntu follow the instructions from [docker.com](https://docs.docker.com/engine/install/ubuntu/). For other operating systems please see the [docs](https://docs.docker.com/engine/install/).


### docker-compose
Follow the instructions from [docker.com](https://docs.docker.com/compose/install/)

### prometheus & grafana
Clone this repository
```
git clone https://github.com/speedmann/chia-monitoring
cd chia-monitoring
```

Edit the `prometheus/prometheus.yml`

#### everything on one system
If you only have one system which you like to monitor, you can use this config snippet
```yaml
global:
  scrape_interval:     10s
  evaluation_interval: 10s
  scrape_timeout:      10s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          host_name: "mysystem"

```

#### multiple systems
If you have more than one system you'd like to monitor, simply repeat the `-targets` part as often as you have systems to monitor e.g.:
```yaml
global:
  scrape_interval:     10s
  evaluation_interval: 10s
  scrape_timeout:      10s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          host_name: "plotter"
      - targets: ['10.0.0.2:9100']
        labels:
          host_name: "harvester"

```

Prometheus is now configured. To run prometheus and grafana do 
`sudo docker-compose up -d`

### node_exporter
This has to be done on every system which should be monitored.

Get the download url from [their github](https://github.com/prometheus/node_exporter/releases) and download it to the system you want to monitor

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
tar xfvz node_exporter-1.1.2.linux-amd64.tar.gz
```

Then setup a systemd service to automatically start the exporter
```bash
sudo cp node_exporter.service /etc/systemd/system/
sudo useradd -s /sbin/nologin node_exporter
sudo mkdir -p /var/lib/node_exporter/textfile_collector
sudo chown node_exporter: /var/lib/node_exporter/textfile_collector
sudo cp node_exporter-1.1.2.linux-amd64/node_exporter /usr/sbin/node_exporter
sudo mkdir /etc/sysconfig
sudo cp sysconfig.node_exporter /etc/sysconfig/node_exporter
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

For S.M.A.R.T data to be available for monitoring you need to create a new cronjob which runs every minute. To do so edit your `/etc/crontab` file and add this
```
* * * * * root /usr/bin/python3 /home/<yourusername>/chia-monitoring/smartmon.py | sponge /var/lib/node_exporter/textfile_collector/smart.prom
```
Also you need to install moreutils `sudo apt install moreutils`

After it is done you will be able to access your grafana dashboard at `<host ip>:3000`. Login with the default username/password `admin/admin`. Please change the password after first login!

## TODOs

 - [ ] Make installation easier
 - [ ] Add chia farm monitoring (Plots, Proofs, Challenge response time, etc.)
 - [ ] Alerting (Telegram?)
 - [ ] Make the dashboards nicer

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License
[GNU General Public License v3.0](LICENSE)
