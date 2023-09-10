# Cannamon: Cannabis Monitoring using ESP8266/ESP32

Cannabis plants need stable temperature and humidity. Although it is possible to grow using
conventional temperature/humidity sensors, we can do much more with affordable WiFi connected
devices. This allows us to get complete picture of our setup, see changes/trends with beautiful
metric interfaces, and create alarms if anything is out of ordinary. This is how it looks like:

![main screen](https://www.thcfarmer.com/attachments/software-engineer-first-growing-experience-jpeg.2035285/)
![detail screen](https://www.thcfarmer.com/attachments/software-engineer-first-growing-experience-2-jpeg.2035286/)

This project aims to provide you with working configuration files and guides so that you can build
one for yourself. You can also see the discussion on [THC Farmers][thc-farmer-first-topic].

[thc-farmer-first-topic]: https://www.thcfarmer.com/threads/software-engineer-first-growing-experience.152810/

## Getting Started

Building the system requires some technical knowledge but it's not rocket science.
Before starting, get yourself familiar with the following topics:

- [ESPHome][esphome]: We are using this project to create the firmware for ESP32/8266. You do not
  need to look at HomeAssistant integration. We don't use it and we only care about pushing values
  to a remote server. Learn how it works and try to make sense of its configuration file. Find
  your sensor component and look at its configuration parameters.

- [MQTT][mqtt-wiki]: Get yourself familiar with what it is. It's basically a message queue where the
  device pushes sensor readings. We get the reading from MQTT server and write it to TSDB (Time
  Series DataBase. In our case, [InfluxDB][influxdb])

- [Telegraf][telegraf]: A software which gets sensor values (subscribes to sensor topics) from MQTT
  server and writes it to InfluxDB.

- [Grafana][grafana]: This is the main screen. What it does is to query the database (InfluxDB) and
  create beautiful charts.

- Cloud provider: We need a server to install the services above. You can use local RaspberryPI but
  I prefer a running instance in DigitalOcean, AWS, Hetzner, etc. We're using Ubuntu 22.04.

[esphome]: https://esphome.io/
[mqtt-wiki]: https://en.wikipedia.org/wiki/MQTT
[influxdb]: https://www.influxdata.com/
[telegraf]: https://www.influxdata.com/time-series-platform/telegraf/
[grafana]: https://grafana.com/
