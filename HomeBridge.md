# HomeBridge


## Mosquitto

### Create Mosquitto configuration

```bash
mkdir -p /mosquitto/config
nano /mosquitto/config/mosquitto.conf
persistence true
persistence_location /mosquitto/data/
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
    logging:
      driver: json-file
      options:
        max-size: "10mb"
        max-file: "1"
    extra_hosts:
      - "mosquitto:192.168.1.12"
  # -e TZ - for timezone information e.g. -e TZ=Australia/Canberra

  mosquitto:
    image: "eclipse-mosquitto:2.0.15"
    container_name: "mosquitto"
    hostname: "mosquitto"
    restart: unless-stopped
    ports:
      - "1883:1883"
    volumes:
      - /mosquitto:/mosquitto
    logging:
      driver: json-file
      options:
        max-size: "10mb"
        max-file: "1"

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


Got to http://192.168.1.12:8581/ for HomeBridge.
Got to http://192.168.1.12:8080/ for Zigbee2MQTT.


## Debug MQTT

docker exec -it mosquitto /bin/sh

Subscribe: `mosquitto_sub -h 127.0.0.1 -u homebridge -P <password> -t '#' -v`
Publish: `mosquitto_pub -h 127.0.0.1 -u homebridge -P <password> -t '#' -m 'message'`

Example of messages that can be send to openshw:
https://github.com/openshwprojects/OpenBK7231T_App#console-commands

## Configuration example

```yaml
{
    "bridge": {
        "name": "Homebridge 6D36",
        "username": "0E:17:BB:A9:6D:36",
        "port": 51961,
        "pin": "784-94-320",
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
            "noWhiteMix": true,
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

Clear a retained message: `mosquitto_pub -h 127.0.0.1 -u <username> -P <password> -t '<topic>' -n -r`
