# Tasmota

> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰

Tasmota is an alternative firmware for devices based on ESP8266 or ESP32 chips.


## Preparation

```bash
docker run -it -v /dev:/dev --privileged ubuntu:20.04

apt-get update
apt-get install git python3 python3-setuptools wget

git clone https://github.com/espressif/esptool
cd esptool
python3 setup.py install
```


## Download firmware

Get last version on https://github.com/arendst/Tasmota/releases/.
Then `wget https://github.com/arendst/Tasmota/releases/download/v12.3.1/tasmota.bin`


## Flash

Simply use:
`esptool.py --baud 921600 write_flash --erase-all -fm dout 0x0 tasmota.bin`


## Configure

Reboot the device and connect to the WiFi access point "tasmota-xxxxx" created by the device.
Then connect to http://192.168.1.4