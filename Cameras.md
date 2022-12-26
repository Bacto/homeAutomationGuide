# Cameras

> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰

## Devices


### Xiaomi YHS-113

Serial prefixed with "12CN".

Follow instructions from https://github.com/fritz-smh/yi-hack

### Orange LED blinking fast and logs showing "Could not read interface ra0 flags: No such device"

See https://github.com/fritz-smh/yi-hack/issues/31

It is because the WiFi SoC is a bcm43143 and no driver loaded to support it.

With this firmware version it works: https://github.com/fritz-smh/yi-hack/issues/31#issuecomment-315832133



### Xiaomi iSC5

> ⚠️ NOT TESTED YET

SoC SONiX SN98661 (ARM).

Use this hack: https://github.com/samtap/fang-hacks

Download the latest release here: https://github.com/samtap/fang-hacks/releases

Use balenaEtcher to create the SD card.



## Homebridge

https://github.com/seydx/homebridge-camera-ui