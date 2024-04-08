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

Upload config if RevPiCommander doesn't work:

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

[https://revolutionpi.com/en/tutorials/software-2/activating-deactivating-services](https://revolutionpi.com/en/tutorials/software-2/activating-deactivating-services)

Using PiCtory to configer the modules on the controller

* Got the the website found under the hostname of the controller (for example [minifactory/]())
* log in login should be set to `admin` with pw `hiwi1234`
  * if not std login can be found on the right hand side of the controller

## Setup MiniFactory-Controller

* Config hostname to `MiniFactory`

### Install and enable mqtt-Broker

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

### Install and deploy Webshop

```
sudo apt install pip
sudo pip install django
sudo pip install pillow
```

* run with `sudo python manage.py runserver 0.0.0.0:80 `find site at [minifactory/](minifactory/)

### Install and set up dev webserver (should be replaced with apache))

Disable apache2

```
sudo systemctl stop apache2
sudo systemctl disable apache2
```

Create service
`sudo nano webshop.service`

copy into service file

```
[Unit]
Description=Hosting webshop
After=network.target

[Service]
WorkingDirectory=/var/lib/revpipyload/WebShop
ExecStart=/usr/bin/python /var/lib/revpipyload/WebShop/manage.py runserver 0.0.0.0:80
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

execute

```
sudo systemctl daemon-reload
sudo systemctl enable "webshop"
sudo systemctl start "webshop"
sudo systemctl status "webshop"
```

enter [minifactory/](minifactory/) or [http://minifactory/](http://minifactory/)

Doc for webshop found at [WebShop/webshop_doc.md](https://github.tik.uni-stuttgart.de/IAS-MiniFactory/WebShop/blob/main/webshop_doc.md "webshop_doc.md")

## Setup MiniFactory-PLC

* Config hostname to `LeftMiniFactory` or `RightMiniFactory`
* Clone **[PLC_Code](https://github.tik.uni-stuttgart.de/IAS-MiniFactory/PLC_Code)** into `/var/lib/revpiload/`
* add `rightline.py` or `leftline.py` as PLC programm (with RevPiCommander)
  * create log folder in `/var/lib/revpiload/`
