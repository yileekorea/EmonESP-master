# EmonESP

[![Build Status](https://travis-ci.org/openenergymonitor/EmonESP.svg?branch=master)](https://travis-ci.org/openenergymonitor/EmonESP)

ESP8266 WIFI serial to emoncms link

For applications that only require basic posting of data from one emonTx to a remote server such as Emoncms.org an emonTx with this WiFi module provides a lower cost route than an emonBase or emonPi base-station installation.

![EmonEsp WiFi AP Setup Portal](docs/emonesp.png)

## Contents

<!-- toc -->

- [Requirements](#requirements)
- [User Guide](#user-guide)
  * [Hardware Setup](#hardware-setup)
  * [First Setup](#first-setup)
    + [1. WiFi Connection](#1-wifi-connection)
  * [2. Emoncms](#2-emoncms)
  * [3. MQTT](#3-mqtt)
  * [5. Admin (Authentication)](#5-admin-authentication)
  * [7. System](#7-system)
  * [OTA Firmware Update](#ota-firmware-update)
  * [HTTP API Examples](#http-api-examples)
    + [View units status:](#view-units-status)
    + [4. Data Input](#4-data-input)
      - [UART Input](#uart-input)
      - [HTTP API](#http-api)
    + [Save Emoncms server details](#save-emoncms-server-details)
    + [Save Emoncms MQTT server details](#save-emoncms-mqtt-server-details)
  * [Installation](#installation)
    + [Option 0: Flash using pre-compiled binaries](#option-0-flash-using-pre-compiled-binaries)
      - [Using EmonUpload](#using-emonupload)
      - [Using Esptool](#using-esptool)
    + [Option 1: Compile Using PlatformIO](#option-1-compile-using-platformio)
      - [1. Install PlatformIO](#1-install-platformio)
      - [2. Clone this repo](#2-clone-this-repo)
      - [3. Compile](#3-compile)
      - [3. Upload](#3-upload)
        * [a.) Upload main program:](#a-upload-main-program)
      - [4. Debugging ESP subsystems](#4-debugging-esp-subsystems)
    + [Troubleshooting Upload](#troubleshooting-upload)
      - [Erase Flash](#erase-flash)
    + [Development Forum Threads](#development-forum-threads)
    + [License](#license)

<!-- tocstop -->

# Requirements

- ESP-12E module with 4M Flash

***

# User Guide

## Hardware Setup

- [Purchase a pre-loaded ESP8266](https://shop.openenergymonitor.com/esp8266-wifi-adapter-for-emontx/)
- To connect an ESP to emonTx see [This User Guide Section](https://guide.openenergymonitor.org/setup/esp8266-adapter-emontx/)

## First Setup

On first boot, ESP should broadcast a WiFI AP `emonESP_XXX`. Connect to this AP and the [captive portal](https://en.wikipedia.org/wiki/Captive_portal) should forward you to the log-in page. If this does not happen navigate to `http://192.168.4.1`

*Note: You may need to disable mobile data if connecting via a Android 6 device*

### 1. WiFi Connection


- Select your WiFi network from list of available networks
- Enter WiFi PSK key then click `Connect`

![Wifi setup](docs/wifi-scan.png)

- emonESP should now connect to local wifi network and return local IP address.
- Browse to local IP address by clicking the hyperlink (assuming your computer is on the same WiFi network)
On future boots EmonESP will automatically connect to this network.

*Note: on some networks it's possible to browse to the EmonESP using hostname [http://emonesp](http://emonesp) or on windows [http://emonesp.local](http://emonesp.local)*

**If re-connection fails (e.g. network cannot be found) the EmonESP will automatically revert back to WiFi AP mode after a short while to allow a new network to be re-configued if required. Re-connection to existing network will be attempted every 5min.**

*Holding the `boot` button at startup (for about 10's) will force AP mode. This is useful when trying to connect the unit to a new WiFi network.*


![Wifi setup](docs/wifi.png)

## 2. Emoncms

![emoncms setup](docs/emoncms.png)

EmonESP can post data to [emoncms.org](https://emoncms.org) or any other  Emoncms server (e.g. emonPi) using [Emoncms API](https://emoncms.org/site/api#input).

In the *Emoncms Server* field, enter just the hostname or address without any path (e.g. emoncms.org), in the *Emoncms Path* field enter the path including the leading slash (e.g. /emoncms) or leave it empty if not required.

Data can be posted using HTTP or HTTPS. For HTTPS the Emoncms server must support HTTPS (emoncms.org does, emonPi does not). Due to the limited resources on the ESP the SSL SHA-1 fingerprint for the Emoncms server certificate must be manually entered and regularly updated.

*Note: the emoncms.org fingerprint will change every 90 days when the SSL certificate is renewed.*

To obtain the certificate fingerprint, you can use several methods, some examples:
* Chrome under Windows: click the secure icon next to the address bar and click on the certificate row to get the details, in the *Details* tab copy the hexadecimal digits from the box *Thumbprint* substituting spaces with colons and paying attention not to include any leading invisible character;
* Firefox under Linux: click the secure icon next to the address bar, Show connection details, More information, in the security tab click *View Certificate* and copy the *SHA1 Fingerprint*
* openssl under Linux: issue the following command substituting your host in place of www.example.com:

`echo | openssl s_client -connect www.example.com:443 -servername www.example.com |& openssl x509 -fingerprint -sha1 -noout`

## 3. MQTT


![mqtt setup](docs/mqtt.png)

EmonESP can post data to an MQTT server. Each data key:pair value will be published to a sub-topic of base topic.E.g data `CT1:346` will results in `346` being published to `<base-topic>/CT1`

- Enter MQTT server host and base-topic
- (Optional) Enter server authentication details if required
- Click connect
- After a few seconds `Connected: No` should change to `Connected: Yes` if connection is successful. Re-connection will be attempted every 10s.

*Note: `emon/xxxx` should be used as the base-topic if posting to emonPi MQTT server if you want the data to appear in emonPi Emoncms. See [emonPi MQTT docs](https://guide.openenergymonitor.org/technical/mqtt/).*


## 5. Admin (Authentication)

HTTP Authentication (highly recomended) can be enabled by saving admin config by default username and password

**HTTP authentication is required for all HTTP requests including input API**

![admin setup](docs/admin.png)

## 7. System

Displays free system memory and firmware version

![system](docs/system.png)

## OTA Firmware Update

TBC

## HTTP API Examples

### View units status:

`http://<IP-ADDRESS>/status`

Example return in JSON:

```
{"mode":"STA","networks":[],"rssi":[],"ssid":"OpenEnergyMonitor","srssi":"-58","ipaddress":"10.0.1.93","emoncms_server":"emoncms.org","emoncms_node":"emonesp","emoncms_apikey":"xxxxxxxx","emoncms_connected":"0","packets_sent":"0","packets_success":"0","mqtt_server":"emonpi","mqtt_topic":"emonesp","mqtt_user":"emonpi","mqtt_pass":"xxxxxx","mqtt_connected":"0","free_heap":"25040"}
```

### 4. Data Input

Data can be inputed to EmonESP via serial UART or HTTP API.

![input setup](docs/input.png)

#### UART Input

Data in serial:pairs string format can be inputed to EmonESP via serial UART **(115200 baud)** e.g:

`ct1:3935,ct2:325,t1:12.5,t2:16.9,t3:11.2,t4:34.7`

#### HTTP API

Data in string:pairs can be sent to EmonESP via HTTP API. This is useful to emulate the serial string data function while using the UART for code upload and debug. API example:

`http://<IP-ADDRESS>/input?string=ct1:3935,ct2:325,t1:12.5,t2:16.9,t3:11.2,t4:34.7`

### Save Emoncms server details

`http://<IP-ADDRESS>/saveemoncms?&server=emoncms.org&apikey=xxxxxxxxxxxxxxxxxx&node=emonesp&fingerprint=7D:82:15:BE:D7:BC:72:58:87:7D:8E:40:D4:80:BA:1A:9F:8B:8D:DA`

*SSL SHA-1 fingerprint is optional, HTTPS connection will be enabled if present*

### Save Emoncms MQTT server details

`http://<IP-ADDRESS>/savemqtt?&server=emonpi&topic=emonesp&user=emonpi&pass=emonpimqtt2016`

*MQTT user and pass are optional, leave blank for connection to un-authenticated MQTT servers*

***

## Installation

EmonESP uses [ESP8266 core, Arduino framework](https://github.com/esp8266/Arduino)

Firmware can be compiled and uploaded either using ([PlatfomIO,](https://blog.openenergymonitor.org/2016/06/platformio/)) Arduino IDE, or flashed with pre-compiles binaries found in Releases using esptool.py.

Note different flash sizes of ESP12 module (4Mb) verses Sonoff devices (1Mb). 

### Option 0: Flash using pre-compiled binaries

#### Using EmonUpload

Use our emonupload tool to download latest pre-compiled firmware release and upload to EmonESP: https://github.com/openenergymonitor/emonupload

#### Using Esptool 
Find and Install [esptool, python required.](https://github.com/espressif/esptool) 
Navigate to the Releases section of the github page and get the firmware.bin and spiffs.bin files. Use the command below to flash the ESP.

`esptool.py write_flash 0x0 ./firmware.bin`

If you're having issues uploading try a slower baudrate, `--baud 115200` is a failsafe option.

### Option 1: Compile Using PlatformIO

For more detailed ESP8266 Arduino core specific PlatfomIO notes see: https://github.com/esp8266/Arduino#using-platformio

#### 1. Install PlatformIO

The easiest way if running Linux is to install use the install script, this installs platformIO via python pip and installs pip if not present. See [PlatformIO installation docs](http://docs.platformio.org/en/latest/installation.html#installer-script).

```
$ sudo python -c "$(curl -fsSL https://raw.githubusercontent.com/platformio/platformio/master/scripts/get-platformio.py)"
```

#### 2. Clone this repo

```
$ git clone https://github.com/openenergymonitor/EmonESP
```

#### 3. Compile

```
$ cd EmonESP
$ pio run
```

The first time platformIO is run the espressif Arduino toolchain and all the required libs will be installed if required.

To compile EmonESP for sonoff s20 smartplugs, WIFI Relay and Heatpump Monitor specify the following environment:

```
$ pio run -e smartplug
$ pio run -e wifirelay
$ pio run -e hpmon
```

To compile for 4Mb esp12e modules use environment:

```
$ pio run -e esp12e
```

#### 3. Upload

Put ESP into bootloader mode:

- For Adafruit HUZZAH press and hold `GPIO0` button then press Reset, LED should light dimly to indicate bootloader mode.
- For WiFi relay press and hold the reset button for the duration of the upload 
- For Sonof S20 press the front button when connecting power
- On Heatpump monitor use jumper to pull `GPIO0` low then reset then connect power (simulates reset) or pull RST pin low.


##### a.) Upload main program:

```
$ pio run -t upload
```

To compile and upload EmonESP for sonoff s20 smartplugs, WIFI Relay and Heatpump Monitor specify the following environment:

The default option without specifying any enviroment will upload to the Huzzah

```
$ pio run -t upload
$ pio run -e smartplug -t upload
$ pio run -e wifirelay -t upload
$ pio run -e hpmon -t upload
```

#### 4. Debugging ESP subsystems

The ESP subsystems have a lot of logging that can be enabled via setting various build options.

Using Platform IO the easiest way to configure these is via the [PLATFORMIO_BUILD_FLAGS](http://docs.platformio.org/en/stable/envvars.html#envvar-PLATFORMIO_BUILD_FLAGS) environment variable.

First select the serial port to output debug;
```
-DDEBUG_ESP_PORT=Serial
-DDEBUG_ESP_PORT=Serial1
```

Then add one or more of the debug options;
```
-DDEBUG_ESP_CORE
-DDEBUG_ESP_WIFI
-DDEBUG_ESP_HTTP_CLIENT
-DDEBUG_ESP_HTTP_SERVER
-DDEBUG_ESP_HTTP_UPDATE
-DDEBUG_ESP_UPDATER
-DDEBUG_ESP_OTA
-DDEBUG_ESP_SSL
-DDEBUG_TLS_MEM
```

For example from the Windows Power shell you may do something like;
```
$env:PLATFORMIO_BUILD_FLAGS="-DDEBUG_ESP_PORT=Serial1 -DDEBUG_ESP_CORE -DDEBUG_ESP_WIFI"
pio run -t clean
pio run
pio run -t upload --upload-port 172.16.0.80
```
***


### Troubleshooting Upload

#### Erase Flash

If you are experiencing ESP hanging in a reboot loop after upload it may be that the ESP flash has remnants of previous code (which may have the used the ESP memory in a different way). The ESP flash can be fully erased using [esptool](https://github.com/themadinventor/esptool). With the unit in bootloder mode run:

```
$ esptool.py erase_flash
```

*`sudo` maybe be required*

Output:

```
esptool.py v1.2-dev
Connecting...
Running Cesanta flasher stub...
Erasing flash (this may take a while)...
Erase took 8.0 seconds
```

### Development Forum Threads

- https://community.openenergymonitor.org/t/emonesp-firmware-development/1191/43
- https://community.openenergymonitor.org/tags/emonesp

### License

GNU V3 General Public License
