# Home Automation

HomeAssistant: permits to controls home automation devices.
HomeBridge: permits to control home automation devices with HomeKit (Apple native system).

Here I install both of them, so I can control my devices with HomeKit from my iPhone, Macbook, Apple TV and Siri easily.
And HomeAssistant to try it and maybe use it for advanced scenarios, because Apple HomeKit is limited.

All of this is running on Linux with Docker, on a Raspberry or any computer.

## MQTT server

We use Mosquitto as MQTT server.
MQTT is a lightweight protocol ideal for realtime communications between devices.

In our case, each device will connect to MQTT.
Then HomeAssistant and HomeBridge will connect to it to communicate with devices.

### Create Mosquitto configuration

```bash
mkdir -p /mosquitto/config
nano /mosquitto/config/mosquitto.conf
persistence true
persistence_location /mosquitto/data
log_dest stdout

allow_anonymous false
password_file /mosquitto/passwords
listener 1883
```

### Generate Mosquitto passwords

```bash
docker run -it -v /mosquitto:/mosquitto eclipse-mosquitto:2.0.15 /bin/sh

apk add pwgen
touch /mosquitto/passwords
chmod 600 /mosquitto/passwords

# Generate secured password with `pwgen -s 64 1`

mosquitto_passwd /mosquitto/passwords devices
mosquitto_passwd /mosquitto/passwords homebridge
```


## Zigbee2MQTT

Zigbee is a great radio protocol.
A lot of devices use Zigbee, like some Ikea devices.
It is power efficient so devices can run on battery, like remotes, doors sensors etc...
Devices are cheap: a door sensor is for example ~5€.

Zigbee2MQTT handles Zigbee data and send them to MQTT.
It needs a USB key (SONOFF ZBDongle in my case) which is connected to the Raspberry Pi.
It has a nice web UI that gives the ability to associate devices easily.


### Create Zigbee2MQTT configuration

See https://www.zigbee2mqtt.io/guide/configuration/

```bash
mkdir -p /zigbee2mqtt/data
nano /zigbee2mqtt/data/configuration.yaml

permit_join: true

mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://mosquitto
  user: homebridge
  password: <password>

serial:
  port: /dev/ttyUSB0

frontend:
  port: 8080
  auth_token: <password>

advanced:
  network_key: GENERATE
  # It is recommended to disable cache state. See https://z2m.dev/install.html
  cache_state: true

advanced:
  log_level: warn
```



## Start containers

```yaml
version: "3.8"
services:
  homebridge:
    image: "oznu/homebridge:2022-10-14"
    container_name: "homebridge"
    hostname: "homebridge"
    restart: unless-stopped
    network_mode: "host"
    volumes:
      - /homebridge:/homebridge
    environment:
      - TZ=Europe/Paris
    extra_hosts:
      - "mosquitto:192.168.1.12"

  homeassistant:
    image: "ghcr.io/home-assistant/home-assistant:stable"
    container_name: "homeassistant"
    hostname: "homeassistant"
    restart: unless-stopped
    network_mode: "host"
    volumes:
      - /homeassistant:/config
    environment:
      - TZ=Europe/Paris
    extra_hosts:
      - "mosquitto:192.168.1.12"

  mosquitto:
    image: "eclipse-mosquitto:2.0.15"
    container_name: "mosquitto"
    hostname: "mosquitto"
    restart: unless-stopped
    ports:
      - "1883:1883"
    volumes:
      - /mosquitto:/mosquitto

  zigbee2mqtt:
    image: koenkk/zigbee2mqtt:1.28.4
    container_name: zigbee2mqtt
    restart: unless-stopped
    volumes:
      - /zigbee2mqtt/data:/app/data
      - /run/udev:/run/udev:ro
    ports:
      - 8080:8080
    environment:
      - TZ=Europe/Paris
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
```


docker-compose up -d


Go to http://192.168.1.12:8581 for HomeBridge.
Go to http://192.168.1.12:8123 for HomeAutomation.
Go to http://192.168.1.12:8080 for Zigbee2MQTT.


## Configuration examples

### HomeAssistant

MQTT integration has to be added in "Settings".

### HomeBridge
```yaml
{
    "bridge": {
        "name": "Homebridge",
        "port": 51961,
        "advertiser": "bonjour-hap"
    },
    "accessories": [
        {
            "type": "lightbulb-RGBWW",
            "name": "Bulb 1",
            "url": "mqtt://mosquitto:1883",
            "username": "homebridge",
            "password": "xxxxxxxxxxxxxx",
            "mqttOptions": {
                "keepalive": 5
            },
            "mqttPubOptions": {
                "retain": false
            },
            "logMqtt": true,
            "topics": {
                "getOnline": "bulb_1/connected",
                "getRGBWW": "bulb_1/led_finalcolor_rgb/get",
                "setRGBWW": "cmnd/bulb_1/led_basecolor_rgbcw",
                "getOn": "bulb_1/led_enableAll/get",
                "setOn": "cmnd/bulb_1/led_enableAll"
            },
            "confirmationIndicateOffline": true,
            "integerValue": true,
            "hex": true,
            "noWhiteMix": false,
            "switchWhites": true,
            "accessory": "mqttthing"
        },
        {
            "type": "lightbulb-RGBWW",
            "name": "Mood Light 1",
            "url": "mqtt://mosquitto:1883",
            "username": "homebridge",
            "password": "xxxxxxxxxxxxxx",
            "mqttOptions": {
                "keepalive": 5
            },
            "logMqtt": true,
            "optimizePublishing": false,
            "topics": {
                "getOnline": "mood_light_1/connected",
                "getRGBWW": "mood_light_1/led_basecolor_rgbcw/get",
                "setRGBWW": "cmnd/mood_light_1/led_basecolor_rgbcw",
                "getOn": "mood_light_1/led_enableAll/get",
                "setOn": "cmnd/mood_light_1/led_enableAll"
            },
            "confirmationIndicateOffline": true,
            "integerValue": true,
            "hex": true,
            "accessory": "mqttthing"
        }
    ],
    "platforms": [
        {
            "name": "Config",
            "port": 8581,
            "platform": "config"
        }
    ]
}
```


## MQTT

docker exec -it mosquitto /bin/sh

Clear a retained message: `mosquitto_pub -h 127.0.0.1 -u <username> -P <password> -t '<topic>' -n -r`
Subscribe: `mosquitto_sub -h 127.0.0.1 -u homebridge -P <password> -t '#' -v`
Publish: `mosquitto_pub -h 127.0.0.1 -u homebridge -P <password> -t '#' -m 'message'`

Example of messages that can be send to openshw:
https://github.com/openshwprojects/OpenBK7231T_App#console-commands

