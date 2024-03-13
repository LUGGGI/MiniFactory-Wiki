# Hardware overview

## Controllers

Password for all Controller: `1234 `
SSH-Connect via `ssh pi@<<Hostname>>`

Code is found in directory: `/var/lib/revpiload/`

### MiniFactory

Hostname: `MiniFactory`
Password: `1234`

Runs:

#### MQTT-Broker

Local Broker for MQTT

Broker address: `MiniFactory/ `
Port: `1883`

All topics are under `MiniFactory/#`

#### MQTT-Controller

Listens to and logs all MQTT-Communication from the whole MiniFactory system

Converts request for Products to the format that the MiniFactory-PLC needs and sends it to a selected factory

#### WebShop

Webshop for MiniFactory

Link: [MiniFactory/](MiniFactory/)

### RightMiniFactory and LeftMiniFactory

PLC-Control for the factory.

Hostname: `RightMiniFactory`
Password: `1234`

Hostname: `LeftMiniFactory`
Password: `1234`


# Install new Image and configs

To install follow Tutorial: [https://revolutionpi.com/en/tutorials/images/install-jessie]()

* Install RevPi Bullseye 32-bit Lite

Configs on RevPi after installing:

Open config: `sudo raspi-config`

* Change Password to `1234`
* Change Hostname to desired
* set Timezone to Berlin
* exit with Finish and reboot

Add University NTP server to allow timesync in uni network

* `sudo nano /etc/systemd/timesyncd.conf`
* Add Line `NTP=rustime01.rus.uni-stuttgart.de rustime02.rus.uni-stuttgart.de`
* `systemctl restart systemd-timesyncd.service`

Upload config if (RevPiCommander doesn't work:

```bash
copy file to /tmp
move into /tmp
sudo cp _config.rsc /var/www/revpi/pictory/projects
```

Install packeges if needed:

```
sudo apt install git
sudo apt install pip
```

Disable not needed Services:

[https://revolutionpi.com/en/tutorials/software-2/activating-deactivating-services]()

## Setup MiniFactory-Controller

* Config hostname to `MiniFactory`
* Install and enable mqtt-Broker

  * Broker Address: `"MiniFactory"` if installt on MiniFactory Host
  * Port: `1883`

```bash
sudo apt-get install mosquitto
sudo systemctl enable mosquitto
sudo systemctl status mosquitto

sudo nano /etc/mosquitto/conf.d/local.conf
Add:
listener 1883
allow_anonymous true

sudo systemctl restart mosquitto

```

* Clone [MQTT](https://github.tik.uni-stuttgart.de/IAS-MiniFactory/MQTT) into `/var/lib/revpiload/`
* add `mqtt_receive.py` as PLC programm (with RevPiCommander)
  * create log folder in `/var/lib/revpiload/`

## Setup MiniFactory-PLC

* Config hostname to `LeftMiniFactory` or `RightMiniFactory`
* Clone **[PLC_Code](https://github.tik.uni-stuttgart.de/IAS-MiniFactory/PLC_Code)** into `/var/lib/revpiload/`
* add `rightline.py` or `leftline.py` as PLC programm (with RevPiCommander)
  * create log folder in `/var/lib/revpiload/`
