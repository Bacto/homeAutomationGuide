# LSC hack

> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰


These devices run on Tuya.
They are cheap and can be found at "Action" stores in France (and Europe).


## Devices

### Smart Dimmer Switch

The WiFi module is a WB3S from Tuya and it supports Bluetooth too.
The SOC used is the BK7231T.
https://developer.tuya.com/en/docs/iot/wb3s-module-datasheet?id=K9dx20n6hz5n4


### Led Mood Light

The WiFi module is a CB3S or a WB3S from Tuya and it supports Bluetooth too.
The SOC used is the BK7231N for CB3S and BK7231T WB3S.

CB3S: https://developer.tuya.com/en/docs/iot/cb3s?id=Kai94mec0s076
WB3S: https://developer.tuya.com/en/docs/iot/wb3s-module-datasheet?id=K9dx20n6hz5n4

Pinout :
- P6: PWM, channel 5
- P7: PWM, channel 4
- P8: PWM, channel 1
- P14: Button, channel 5
- P24: PWM, channel 2
- P26: PWM, channel 3

- Activate flags 10, 16, 17, 18, 27
- Uptime seconds required to mark boot as ok to 5

- To disable automatic power on
  - Click on "Filesystem"/"Create File" and name it "early.bat".
  - Click on "/early.bat", and set `backlog ; led_dimmer 0 ; led_basecolor_rgbcw #0000000000 ; led_enableAll 0`
  - Click on "Save".



HomeBridge configuration:
```json
{
  "type": "lightbulb-RGBWW",
  "name": "Mood Light 1",
  "url": "mqtt://192.168.1.12:1883",
  "username": "homebridge",
  "password": "",
  "mqttOptions": {
    "keepalive": 5
  },
  "mqttPubOptions": {
    "retain": true
  },
  "logMqtt": false,
  "debounceRecvms": 1000,
  "topics": {
    "getOnline": "mood_light_1/connected",
    "getRGBWW": "mood_light_1/led_finalcolor_rgbcw/get",
    "setRGBWW": "cmnd/mood_light_1/led_basecolor_rgbcw",
    "getOn": "mood_light_1/led_enableAll/get",
    "setOn": "cmnd/mood_light_1/led_enableAll"
  },
  "onlineValue": "online",
  "offlineValue": "offline",
  "confirmationIndicateOffline": true,
  "integerValue": true,
  "hex": true,
  "noWhiteMix": false,
  "switchWhites": true,
  "accessory": "mqttthing"
}
```


### Smart LED E27 806 (RGBCW)

The SOC used is the BK7231T and a BP5758D.

Pinout :
- P24: clock (`BP5758D_CLK`)
- P26: data (`BP5758D_DAT`)

- Activate flags 4, 10, 16, 17, 18, 27
- Uptime seconds required to mark boot as ok to 5

Click on "Return menu", then "Launch Web Application".

Click on "Filesystem"/"Create File" and name it "early.bat".
Click on "/early.bat", and set `backlog ; led_dimmer 100 ; led_basecolor_rgbcw #000000FFFF ; led_enableAll 1`
Click on "Save".

Click on "Filesystem"/"Create File" and name it "autoexec.bat".
Click on "/autoexec.bat", and set `BP5758D_Map 1 0 2 3 4`
Click on "Save".



HomeBridge configuration:
```json
{
  "type": "lightbulb-RGBWW",
  "name": "Bulb 1",
  "url": "mqtt://192.168.1.12:1883",
  "username": "homebridge",
  "password": "",
  "mqttOptions": {
    "keepalive": 5
  },
  "mqttPubOptions": {
    "retain": false
  },
  "logMqtt": false,
  "debounceRecvms": 1000,
  "topics": {
    "getOnline": "bulb_1/connected",
    "getRGBWW": "bulb_1/led_finalcolor_rgbcw/get",
    "setRGBWW": "cmnd/bulb_1/led_basecolor_rgbcw",
    "getOn": "bulb_1/led_enableAll/get",
    "setOn": "cmnd/bulb_1/led_enableAll"
  },
  "onlineValue": "online",
  "offlineValue": "offline",
  "confirmationIndicateOffline": true,
  "integerValue": true,
  "hex": true,
  "noWhiteMix": false,
  "switchWhites": true,
  "accessory": "mqttthing"
}
```


### Smart Power Plug

The WiFi module is a CB2S from Tuya and supports Bluetooth too.
The SOC used is the BK7231N.
https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq

- Activate flags 10, 27
- In "OpenBK7231T" web application
  - In "Config", select "BK7231N" then "LSC Smart Connect Plug 2578677 970764"
  - Then set pin 8 from `WifiLED` to `WifiLED_n` and click "Save pins"

- To disable automatic power on
  - Click on "Filesystem"/"Create File" and name it "early.bat".
  - Click on "/early.bat", and set `power off`
  - Click on "Save".

mosquitto_pub -h 127.0.0.1 -u <username> -P <password> -t '<clientId>/1/set' -m 1
mosquitto_pub -h 127.0.0.1 -u <username> -P <password> -t '<clientId>/1/set' -m 0


HomeBridge configuration:
```json
{
  "type": "outlet",
  "name": "Plug 1",
  "url": "mqtt://192.168.1.12:1883",
  "username": "homebridge",
  "password": "",
  "mqttOptions": {
    "keepalive": 5
  },
  "mqttOptions": {
    "retain": true
  },
  "topics": {
    "getOnline": "plug_1/connected",
    "getOn": "plug_1/1/get",
    "setOn": "plug_1/1/set"
  },
  "onlineValue": "online",
  "offlineValue": "offline",
  "confirmationIndicateOffline": true,
  "integerValue": true,
  "accessory": "mqttthing"
}
```


### Outdoor Party Lights

The WiFi module is a CB2L from Tuya and it supports Bluetooth too.
The SOC used is the BK7231N.
https://developer.tuya.com/en/docs/iot/cb2l-module-datasheet?id=Kai2eku1m3pyl

It supports RGB and CW but only RGB is used here.

The box has to be destroyed as it is super glued :-/

- Pin 6: blue, channel 2
- Pin 24: green, channel 1
- Pin 26: red, channel 0

Activate flags 10, 16, 17, 18, 27

HomeBridge configuration:
```json
{
  "type": "lightbulb-RGB",
  "name": "Garland 1",
  "url": "mqtt://192.168.1.12:1883",
  "username": "homebridge",
  "password": "",
  "mqttOptions": {
    "keepalive": 5
  },
  "mqttPubOptions": {
    "retain": true
  },
  "logMqtt": false,
  "debounceRecvms": 1000,
  "optimizePublishing": false,
  "topics": {
    "getOnline": "garland_1/connected",
    "getRGB": "garland_1/led_basecolor_rgb/get",
    "setRGB": "cmnd/garland_1/led_basecolor_rgb",
    "getOn": "garland_1/led_enableAll/get",
    "setOn": "cmnd/garland_1/led_enableAll"
  },
  "onlineValue": "online",
  "offlineValue": "offline",
  "confirmationIndicateOffline": true,
  "integerValue": true,
  "hex": true,
  "accessory": "mqttthing"
}
```

### Smart Filament LED (E14 and E27)

The WiFi module is a WBLC5 from Tuya and it supports Bluetooth too.
The SOC used is the BK7231T.
https://developer.tuya.com/en/docs/iot/wblc5-module-datasheet?id=K9duilns1f3gi

- Pin 24 (warm) => PWM, channel 1
- Pin 26 (cold) => PWM, channel 0

- Activate flags 10, 17, 18, 27
- Uptime seconds required to mark boot as ok to 5

Click on "Return menu", then "Launch Web Application".
Click on "Filesystem"/"Create File" and name it "early.bat".
Click on "/early.bat", and set one of this line:
  - To start with white: `backlog led_temperature 154 ; led_dimmer 100 ; led_enableAll 1`
  - To start with yellow: `backlog led_temperature 500 ; led_dimmer 100 ; led_enableAll 1`
Click on "Save".

HomeBridge configuration:
```json
{
  "type": "lightbulb-ColTemp",
  "name": "Filament 1",
  "url": "mqtt://192.168.1.12:1883",
  "username": "homebridge",
  "password": "",
  "mqttOptions": {
    "keepalive": 5
  },
  "mqttPubOptions": {
    "retain": false
  },
  "logMqtt": true,
  "debounceRecvms": 1000,
  "topics": {
    "getOnline": "filament_1/connected",
    "getOn": "filament_1/led_enableAll/get",
    "setOn": "cmnd/filament_1/led_enableAll",
    "getBrightness": "filament_1/led_dimmer/get",
    "setBrightness": "cmnd/filament_1/led_dimmer",
    "getColorTemperature": "filament_1/led_temperature/get",
    "setColorTemperature": "cmnd/filament_1/led_temperature"
  },
  "onlineValue": "online",
  "offlineValue": "offline",
  "confirmationIndicateOffline": true,
  "integerValue": true,
  "hex": true,
  "noWhiteMix": false,
  "switchWhites": true,
  "accessory": "mqttthing"
}
```


### Smart Digital Led Strip

There is 1 button, 1 IR receiver, 1 microphone, RGB, CC and CW leds.
The SOC used is the BK7231N.

LED cable:
  - Green: GND
  - White: VCC (24V) controled by Q6, Q5 and P6
  - Blue: digital output for RGB leds (SM16703P chipsets, single SPI)
  - Black: CC
  - Red: CW

IR cable:
  - Red: 3.3v
  - Black: GND
  - Green: IR (P26)

Pinout:
  - P6 => Q5 => Q6 => VCC 24V (White)
  - P7 => Button
  - P9 => Q1 => CW (red wire) = YELLOW
  - P16 => C255 chip (probably transitors just to convert 3.3V to 5V) => RGB LED controler on strip (SM16703P)
  - P23 => microphone
  - P24 => Q2 => CC (black wire) = WHITE
  - P26 => IR

On the ruban, there is 8 pins chipsets controlling 6 RGB LEDs each with reference SM16703P.

Waiting for SM16703P support:
https://github.com/openshwprojects/OpenBK7231T_App/issues/176#issuecomment-1321209563


## Soldering cables

Use a USB to TTL converter with 3.3V logic levels (like HW 597).

See pins here:
https://community.home-assistant.io/t/lsc-smart-connect-smart-led-mood-light-970743-tuya-cb3s-bk7231n-mqtt-conversion/463702
https://community-assets.home-assistant.io/original/4X/8/8/e/88e4319395e37a9d428c04c3c8c71f70ec07a74b.jpeg

And:
https://developer.tuya.com/en/docs/iot/cb3s?id=Kai94mec0s076#title-4-Dimensions%20and%20package

> For easier access, we can use the 5V and GND pins from USB module :)
> ⚠️  The SOC use a 3.3V voltage!

> Solder a wire on RESET (or CEN) pin not plugged at anything but closed to the GND pad.


