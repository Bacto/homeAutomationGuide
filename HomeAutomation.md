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
mkdir -p /mosquitto/data
chown 1883:1883 /mosquitto/data

mkdir -p /mosquitto/config
nano /mosquitto/config/mosquitto.conf
persistence true
persistence_location /mosquittoVolume/data
log_dest stdout

allow_anonymous false
password_file /mosquittoVolume/passwords
listener 1883
```

### Generate Mosquitto passwords

```bash
docker run -it -v /mosquitto:/mosquittoVolume eclipse-mosquitto:2.0.15 /bin/sh

apk add pwgen
touch /mosquittoVolume/passwords
chmod 600 /mosquittoVolume/passwords

# Generate secured password with `pwgen -s 64 1`

mosquitto_passwd /mosquittoVolume/passwords devices
mosquitto_passwd /mosquittoVolume/passwords homebridge
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

```bash
mkdir -p /traefik
cat << 'EOF' > /traefik/configuration.yml
http:
  routers:
    traefik-dashboard:
      entryPoints:
        - dashboard
      priority: 1000
      tls:
        certresolver: letsencrypt
      middlewares:
        - traefik-dashboard-auth
      service: api@internal
#      rule: Host(`{{ domain }}`)

#  middlewares:
#    traefik-dashboard-auth:
#      basicAuth:
#        users:
#        - {{ dashboard.login }}:{{ dashboard.passwordHashed }}
EOF


# mkdir /node-red
# chown 1000:1000 /node-red
```

TODO:
- traefik to containers
- close ports from containers
- what about ports opened in homeassitant?

```yaml
version: "3.8"
services:
  # traefik:
  #   image: "traefik:v2.9.6"
  #   container_name: "traefik"
  #   hostname: "traefik"
  #   restart: unless-stopped
  #   volumes:
  #     - /traefik/configuration.yml:/etc/traefik/traefik.yml
  #   ports:
  #     - 80:80
  #     - 443:443

  # homebridge:
  #   image: "oznu/homebridge:2022-10-14"
  #   container_name: "homebridge"
  #   hostname: "homebridge"
  #   restart: unless-stopped
  #   network_mode: "host"
  #   volumes:
  #     - /homebridge:/homebridge
  #   extra_hosts:
  #     - "mosquitto:192.168.1.12"

  # https://hub.docker.com/r/homeassistant/home-assistant/tags
  homeassistant:
    image: "ghcr.io/home-assistant/home-assistant:2023.11"
    container_name: "homeassistant"
    hostname: "homeassistant"
    restart: unless-stopped
    network_mode: "host"
    volumes:
      - /homeassistant:/config
      # Useful for Bluetooth
      - /run/dbus:/run/dbus:ro
    extra_hosts:
      - "mosquitto:192.168.1.12"

  # node-red:
  #   # See https://github.com/node-red/node-red-docker#image-variations
  #   image: "nodered/node-red:3.0.2-18-minimal"
  #   container_name: "node-red"
  #   hostname: "node-red"
  #   restart: unless-stopped
  #   volumes:
  #     - /node-red:/data
  #   ports:
  #     - 1880:1880

  mosquitto:
    image: "eclipse-mosquitto:2.0.15"
    container_name: "mosquitto"
    hostname: "mosquitto"
    restart: unless-stopped
    ports:
      - "1883:1883"
    # Use "/mosquittoVolume" in place of "/mosquitto" because the "eclipse-mosquitto" image creates volumes in "/mosquitto" which interferes.
    volumes:
      - /mosquitto:/mosquittoVolume
    entrypoint: "/usr/sbin/mosquitto -c /mosquittoVolume/config/mosquitto.conf"

  # https://hub.docker.com/r/koenkk/zigbee2mqtt/tags
  zigbee2mqtt:
    image: "koenkk/zigbee2mqtt:1.33.2"
    container_name: zigbee2mqtt
    restart: unless-stopped
    volumes:
      - /zigbee2mqtt/data:/app/data
      - /run/udev:/run/udev:ro
    ports:
      - 8080:8080
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


## Issues

After a certain time, bluetooth is not working anymore:
```
ERROR (MainThread) [homeassistant.components.bluetooth.scanner] hci0 (43:45:C0:00:1F:AC): Error stopping scanner: [org.bluez.Error.InProgress] Operation already in progress#033[0m
ERROR (Bot:879454005:dispatcher) [homeassistant.components.telegram_bot.polling] Update "None" caused error: "Conflict: terminated by other getUpdates request; make sure that only one bot instance is running"#033[0m
WARNING (MainThread) [bluetooth_auto_recovery.recover] Bluetooth adapter hci0 [43:45:C0:00:1F:AC] could not be reset due to timeout#033[0m
WARNING (MainThread) [bluetooth_auto_recovery.recover] Bluetooth management socket connection lost: [Errno 9] Bad file descriptor#033[0m
```

Seen on a Raspberry 3 B Plus, kernel 5.15.93-0-rpi.
Not sure about the reason but there is lot of threads about this.
For now, the easy solution is to use an external USB Bluetooth key.