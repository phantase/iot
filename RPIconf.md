Can be useful: https://medium.com/@petey5000/monitoring-your-home-network-with-influxdb-on-raspberry-pi-with-docker-78a23559ffea

## Install Docker

```bash
# Retrieve installation script and execute it
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# add current user to docker group to be able to launch docker command w/o sudo
sudo usermod -aG docker pi

# reboot to apply changes
sudo reboot

# test if installation is good
docker ps
```

## Docker containers

```bash
# Create a folder to put all the stuff
mkdir /home/pi/iot

# Container for InfluxDB
docker run --rm -d --name=influxdb --net=host --volume=/home/pi/iot:/data hypriot/rpi-influxdb

# Only before first run - Retrieve conf file for Telegraf
docker run --rm arm32v7/telegraf telegraf config > /home/pi/iot/telegraf.conf

# Container for Telegraf
docker run --rm -d --net=host --name telegraf -v /home/pi/iot/telegraf.conf:/etc/telegraf/telegraf.conf:ro arm32v7/telegraf

# Volume for Grafana
docker run -d -v /var/lib/grafana --name grafana-storage busybox:latest

# Container for Grafana
docker run --rm -d --net=host --name grafana --volumes-from grafana-storage fg2it/grafana-armhf:v4.1.2

# Create folders for MQTT
mkdir -p /home/pi/iot/mqtt/config/
mkdir -p /home/pi/iot/mqtt/data/
mkdir -p /home/pi/iot/mqtt/log/

# Container for MQTT
docker run --rm -d -p 1883:1883 -p 9001:9001 -v /home/pi/iot/mqtt/config:/mqtt/config:ro -v /home/pi/iot/mqtt/log:/mqtt/log -v /home/pi/iot/mqtt/data/:/mqtt/data/ --name mqtt fstehle/rpi-mosquitto
```