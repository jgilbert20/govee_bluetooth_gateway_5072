
# What is this fork

Jeremy: Adds support for 5072 sensors

Install: sudo cp govee_gw.service /etc/systemd/system/


# Full Install Instructions

```

sudo apt-get install emacs-nox git

sudo apt-get install python3-pip libglib2.0-dev
sudo pip3 install bluepy
sudo apt install -y mosquitto mosquitto-clients
sudo pip3 install paho-mqtt


mkdir src
cd src
git clone https://github.com/jgilbert20/govee_bluetooth_gateway_5072.git

cd govee_bluetooth_gateway_5072

# verify working
sudo python3 govee_ble_mqtt_pi.py

# install service
sudo systemctl enable govee_gw

# start service
sudo service govee_gw start

# check mqtt
mosquitto_sub -h localhost -t '#' -v

```


# govee_bluetooth_gateway
**Bluetooth to MQTT gateway for Govee brand bluetooth sensors.**

<img src="https://raw.githubusercontent.com/tsaitsai/govee_bluetooth_gateway/main/images/chest_freezer_test.jpg" width="359" height="300">

I got a chest freezer recently, and planned to put it in the basement.  Since it's not used frequently, I wanted to keep track of the temperature and send myself a notification in case the freezer breaks down and I need to save the food.

I have zigbee temperature sensors that are small and use CR2032 coin cells.  They'll work for a little while, but freezing temperatures may not be best for coin cell battery performance and might necessitate frequent battery changes.  I came across these [Govee brand](https://www.amazon.com/Govee-Temperature-Notification-Hygrometer-Thermometer/dp/B0872X4H4J) Bluetooth temperature sensors that use a pair of AAA batteries.

If I swap the alkaline AAA batteries out with Energizer Lithium AAA batteries, the sensor should work better in a chest freezer.  The lithium-iron disulfide chemistry in these Energizer AAA batteries perform better in freezing conditions than the Lithium Manganese dioxide chemistry used in typical CR2032 coin cells.  The added capacity should also reduce battery changes.

<img src="https://raw.githubusercontent.com/tsaitsai/govee_bluetooth_gateway/main/images/chest_freezer_temp_sensor.jpg" width="302" height="403"> <img src="https://raw.githubusercontent.com/tsaitsai/govee_bluetooth_gateway/main/images/battery.jpg" width="403" height="302">


Like a lot of these gateway projects, I wanted to use a Raspberry Pi to read the bluetooth sensor data and act as a dedicated gateway.  I came across this [GoveeWatcher project](https://github.com/Thrilleratplay/GoveeWatcher/issues/2) that explains the how the Govee bluetooth advertisement formats the temperature, humidity, and battery level.  The sensor data is encoded in the advertisement, so no GATT connections needed.  The manufacturer provides mobile apps for typical usage, but it has creepy permissions and it's not useful for my purpose.  Instead, I used the Bluepy library to read the BLE advertisement and parse it out per the description from GoveeWatcher project.  As I'm already using MQTT, Influx, Grafana, and telegraph, it was convenient to publish the sensor data as MQTT messages and visualize it in Grafana.  Then send email notifications using Node-Red.

<img src="https://raw.githubusercontent.com/tsaitsai/govee_bluetooth_gateway/main/images/chest_freezer-cooldown_warmup.jpg" width="472" height="480">

I'm also interested in how the freezer behaves.  How the temperature fluctuates, how many watts it consumes, etc.  It's interesting how long it takes for it to go from -13F back put up to 32F when the power is shut off.  To monitor energy consumption, I used a [wifi outlet](https://www.amazon.com/BN-LINK-Monitoring-Function-Compatible-Assistant/dp/B07VDGM6QR) with built-in current sensor and flashed it with Tasmota via [Tuya Convert](https://github.com/ct-Open-Source/tuya-convert).  This 7 cu-ft Hisense chest freezer uses about 0.7 kWh per day when the unit is totally empty and door kept closed.

<img src="https://raw.githubusercontent.com/tsaitsai/govee_bluetooth_gateway/main/images/energy_consumption_monitoring.jpg" width="317" height="423">

  I'll update the energy usage once the freezer is more full of food.  I'll also update when I get battery life data.  The Govee bluetooth sensor unfortunately sends out updates way more often than necessary (every couple of seconds).  So the added battery capacity from the AAA batteries may be somewhat negated by the excess data transmissions.
  
The chest freezer isn't blocking the signal too much.  The Pi gateway is one level up and one room over from the chest freezer location, and I'm able to receive 3 out of 5 advertisements on average.
