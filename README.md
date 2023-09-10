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

We will first get ESP up and running. These are the following steps, in order, to start building.

- Change `secrets.yml` and add your wifi credentials.
- Flash the device with the cable, barebone, without any sensor attached to it.
- See the logs with the cable, hoping that it's connected to wifi.
- Re-build the firmware and attempt OTA update. Make sure OTA is working (at this point you do not
  need a cable anymore).
- Attach the sensors. See sensor values are printed in the logs.
- Setup the server. Install MQTT, create credentials.
- Change MQTT configuration in YML. Re-build, OTA update. Check logs and make sure values are
  pushed.
- Install InfluxDB, Telegraf, Grafana and copy configuration files.
- Grow the hell out of those plants!

Once it's connected to wifi and OTA updates are working, the development workflow is as easy as
running 1 command and seeing it uploaded to the device.

Let's dive into the details.

## Python Virtual Environment

This is a template repository. You will need to change configuration files as you see fit. Clone
this repository and create Python 3.11+ virtual environment. If you do not have 3.11, install it
first. You can use your favorite virtualenv manager but for simplicity, we are using `venv/`
directory. Every time you enter this repository, you need to activate the virtual environment first with `source` command below. Otherwise, `esphome` command will not be found.

```shell
python3.11 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
```

## Installing Drivers

You need to get serial drivers installed first. How you do it depends on the OS and the device you
are using. For NodeMCU v3 (ESP8266), get [CP210x][cp210x] drivers. For other devices, this section will be updated as we have more information.

[cp210x]: https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers

## Flashing ESP

Once you connect your device to your computer and can see the serial port, you are ready to flash.
Change wifi values `secrets.yml` first then run the following command:

```shell
esphome run src/cannamon_tent.yaml
```

Got an error? Don't worry, you needed to see it. Once you get the device out of the factory, you
need to press/hold the little flash button for the first attempt. Now hold that button and try the
command again. This time it should succeed.

`run` command will show you logs after flashing. If you do not want to flash but see the logs, you can use the following command:

```bash
esphome logs src/cannamon_tent.yaml
```

## Wifi Connection and OTA

You should see in the logs that the device establishes a wifi connection. You will see IP address,
DNS servers, MAC address, etc. If you have problem with wifi, get it working first. Make sure you
can ping the device with the IP address.

We are now ready to test OTA update. Uncomment `use_address` configuration in `cannamon_tent.yaml` file with the IP addres the device got and issue the following command again:

```bash
esphome run src/cannamon_tent.yaml
```

On the selection screen, you should now see USB serial port and IP address of the device. Select the IP address (OTA) and push the image. Once succeeded, you no longer need the cable unless there is a problem with OTA updates. You may unplug it from your computer and use regular USB phone charger to power the device.

Congratulations! Hard part is over.

## Connecting the Sensor

This depends on the sensor you have. DHT11/DHT22 is analog and SHT31 is IC2. Search for how you can
connect the sensors for your device and take a look at esphome component documentation. There are
configuration examples on documentation page.

After pins are set, you just need to edit `cannamon_tent.yaml` accoding to the sensor and start
using it. Once you are done, re-build the image with `esphome run` and see the logs over the air. If
the sensor is working, you should see the values in the logs.

## Setting Up Server

Everything is ready so far. We can connect the device to the wifi, update it over the air, get sensor readings and logs. All we need is to push those metrics to MQTT and write it to a database.

We use Ubuntu 22.04. For now, I will not be able to go into the detail of which commands you need to run, it will be added later. To set up, install InfluxDB, Telegraf and Grafana.

Use the configuration files in `config/` directory and restart the services. For MQTT, you need to
create credentials. You can find how to create them with a simple Google search. After the credentials, edit the yaml file and run `esphome run` as usual.

InfluxDB and Grafana uses default configuration file. You only need to change MQTT configuration and
list the topics you want to get into InfluxDB. If you have more than one sensor, you need to add it
to the topic list so that Telegraf can consume and write it.

For grafana, it's a bit tricy to set InfluxDB v1 as a data source. It will be added later.
