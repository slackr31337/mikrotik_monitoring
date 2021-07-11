# Monitor your Mikrotik router with Prometheus and Grafana

Over the past years I replaced all my networking gear with Mikrotik devices. I absolutely love WinBox for management, but sometimes I miss some fancy charts and graphs. Luckily, RouterOS comes with a REST-API that can be used to query arbitrary data from the device. After some research I found some useful tools and blog posts that solved similar problems. Namely:

- [A blog post by Devin Smith that first got me interested](https://blog.devinsmith.co.za/home-internet-grafana-lockdown/)
- [A somewhat useable Grafana Dashboard](https://grafana.com/grafana/dashboards/10950)
- [A Prometheus exporter for Mikrotik devices](https://github.com/nshttpd/mikrotik-exporter)

NOTE: I recently stumbled across [mktxp](https://github.com/akpw/mktxp). This is an alternative for `mikrotik-exporter` and seems to be more actively maintained. I am currently thinking about switching to this exporter.

## Setup

- Router running RouterOS 7.x.x
- Raspberry Pi 4 with 2 gb RAM (other PIs may also work, but I wanted ARM 64 bit)

## Demo pictures

![General system stats](https://github.com/M0r13n/mikrotik_monitoring/blob/main/doc/pic1.png)
![Wifi stats](https://github.com/M0r13n/mikrotik_monitoring/blob/main/doc/pic2.png)

## Installation

## Mikrotik Router
At first you need to prepare your router. 

Create a group on the device that has API and read-only access:

`/user group add name=prometheus policy=api,read,winbox,test`

Create a user that is part of the group:

`/user add name=prometheus group=prometheus password=TOP_SECRET`

Because the library makes a new connection for every API request, your logs are getting cluttered:

`system logging set 0 topics=info,!account`

## Prepare Raspi
You need Ubuntu Server for ARM 64 bit in order to use this setup. You may also use Raspian, but then you are limited to 32bit ARM executables. This would mean, that you need to compile the `mikrotik-exporter` by hand, because there are no predefined 32-bit Docker images.

Install Python and pip:

`sudo apt install python3-dev python3 python3-pip -y`

Install Docker + Docker-compose

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu
sudo pip3 install docker-compose
sudo systemctl enable docker
```

Spin up Grafana and Prometheus:

```bash
# Clone this repo
git clone https://github.com/M0r13n/mikrotik_monitoring.git

# Go into the cloned directory
cd mikrotik_monitoring

# Let docker-compose do it's job
sudo docker-compose up -d
```

Done. You should now be able to open the Grafana dashboard on Port 3000 of your Raspberry Pi.
