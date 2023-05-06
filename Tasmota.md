# Tasmota

> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰

Tasmota is an alternative firmware for devices based on ESP8266 or ESP32 chips.

Development firmware is available at https://github.com/arendst/Tasmota-firmware/tree/main/release-firmware/tasmota


## Compile Tasmota with custom options

```bash
# Prepare container to build the firmware
# We need to build the image as it is only for AMD64 on hub.docker.com
cd /tmp
git clone https://github.com/tasmota/docker-tasmota /tmp/docker-tasmota
cd /tmp/docker-tasmota
docker build -t docker-tasmota .


# Clone Tasmota. Set the version to the last one available on https://github.com/arendst/Tasmota/releases
export TASMOTA_VERSION=v12.4.0
git clone --depth=1 https://github.com/arendst/Tasmota.git -b ${TASMOTA_VERSION} /tmp/tasmota
cd /tmp/tasmota

# Add Teleinfo feature
# echo '#define USE_TELEINFO' >> tasmota/user_config_override.h

# MCP230xx I2C expander support

# # [I2cDriver77] Enable MCP23xxx support as virtual switch/button/relay (+3k(I2C)/+5k(SPI) code)
# echo '#define USE_MCP23XXX_DRV' >> tasmota/user_config_override.h
# See https://github.com/arendst/Tasmota/blob/ccdab295e738fe53a0335719aa9c41f3be43ee5d/tasmota/tasmota_xdrv_driver/xdrv_67_mcp23xxx.ino
# And https://github.com/arendst/Tasmota/blob/2e5501e9abb727694f728e2f1681c7c7176dc8ed/tasmota/tasmota_xdrv_driver/xdrv_88_esp32_shelly_pro.ino
# Unfortunately I can't make it work.
# Tried with MultiUsages board and `{"NAME":"Bacto Multiusages Board","GPIO":[1376,0,0,0,640,608,0,0,0,0,0,0,0,0],"FLAG":0,"BASE":1,"CMND":"rule3 on file#mcp23x.dat do {\"NAME\":\"Expander\",\"GPIO\":[224,225,226,227,64,65,66,67]} endon"}`
# See https://github.com/arendst/Tasmota/discussions/18124

# Enable INPUT mode (pinmode 1 through 4)
echo '#define USE_MCP230xx' >> tasmota/user_config_override.h

# Enable OUTPUT mode (pinmode 5)
echo '#define USE_MCP230xx_OUTPUT' >> tasmota/user_config_override.h

# Display state of OUTPUT pins on main Tasmota web interface
echo '#define USE_MCP230xx_DISPLAYOUTPUT' >> tasmota/user_config_override.h

# Set address to 0x20
echo '#define USE_MCP230xx_ADDR 0x20' >> tasmota/user_config_override.h


# On Wemos D1 Mini with 4MB:
# - `cp platformio_override_sample.ini platformio_override.ini`
# - Edit `platformio_override.ini` and replace "board" with "esp8266_4M2M"

# Activate UFS (universal file system)
# See https://tasmota.github.io/docs/UFS/
# On WEMOS D1 Mini we have 4MB.
# - `cp platformio_override_sample.ini platformio_override.ini`
# - Edit `platformio_override.ini`
#   - Comment "board ..."
#   - Uncomment "board = esp8266_2M1M"


# Compiling using Docker.
#
# For an ESP8266:
docker run -it -v /tmp/tasmota:/tasmota docker-tasmota -e tasmota
#
# For an ESP32:
# docker run -it -v /tmp/tasmota:/tasmota docker-tasmota -e tasmota32

# The firmware is here:
ls -al /tmp/tasmota/build_output/firmware/*.bin

# tasmota.bin: firmware
# tasmota32.factory.bin: Factory binaries to be used for inital flashing using esptool
# tasmota32.bin: OTA firmware
```

Flash it with OTA or with serial connection.


## Flash an ESP with Tasmota over a serial connection

Example using serial connection:
```bash
docker run -it -v /dev:/dev -v $PWD:/mnt --privileged ubuntu:22.04

apt-get update \
  && apt-get install -y git python3 python3-setuptools wget \
  && git clone https://github.com/espressif/esptool \
  && cd esptool \
  && python3 setup.py install

# For an ESP8266
esptool.py --baud 921600 write_flash --erase-all -fm dout 0x0 /mnt/build_output/tasmota.bin

# For an ESP32
esptool.py --chip esp32 --baud 921600 --before default_reset --after hard_reset write_flash -z --flash_mode dout --flash_size detect 0x0 /mnt/build_output/tasmota32.factory.bin
```


> Handle error "Not enough space":
> If the binary is larger than 500kb (which should be the case), the ESP will not have enough space to upgrade its firmware.
> In that case, flash it using the "Upgrade by web server" part and put this URL: `http://ota.tasmota.com/tasmota/release/tasmota-minimal.bin.gz`.
> Then flash it again with firmware that includes "Teleinfo".
