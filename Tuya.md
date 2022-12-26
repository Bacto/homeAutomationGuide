# Tuya


> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰


## Flash devices remotely with Tuya Cloudcuter


### Prepare a Raspeberry PI

Don't loose time, do it on a Raspberry with Raspberry Pi OS Lite installed.

1. Install Raspberry Pi OS on a SD card
1. Create a "ssh" file on the "boot" partition of this SD card
1. Create a "userconf.txt" file on SD card with "<username>:<encryptedPassword>". Password is encrypted with `openssl passwd -6`
1. Boot the Raspberry Pi, connected with a RJ-45 cable on your internet box
1. Connect with `ssh <username>@raspberrypi.local`
1. Follow the instructions here: https://github.com/tuya-cloudcutter/tuya-cloudcutter/blob/main/HOST_SPECIFIC_INSTRUCTIONS.md#raspberry-pi

> Note: Do not run "sudo ./run_detach.sh -r ..." or "sudo ./tuya-cloudcutter.sh -s <SSID> <SSID password>".


### Flash with the custom firmware

Select the latest version "CCtr Flash" version on https://github.com/openshwprojects/OpenBK7231T_App/releases.

Here is an example with OpenBK7231T.
```bash
wget https://github.com/openshwprojects/OpenBK7231T_App/releases/download/1.15.228/OpenBK7231T_UG_1.15.228.bin
mv OpenBK7231T_UG_1.15.228.bin custom-firmware/
sudo ./tuya-cloudcutter.sh -f OpenBK7231T_UG_1.15.228.bin
```

> Note: for lights, it should blink slow. If it blinks fast, power on and off for 3 times during 1 second each time.


Once flashed, you'll get a message like "Firmware update messages triggered. Device will download and reset. Exiting in 30 seconds".
Wait for some seconds and try to find a WiFi access point named "OpenBK7231xxxxx".
Connect to it, then go to http://192.168.4.1 and configure it.


## Flash

Use a Raspberry or any Linux station.

Start a container:
```bash
docker run -it -v /dev:/dev --privileged ubuntu:20.04
```

Install dependencies:
```bash
apt update
apt install -y git wget python3-hid python3-serial python3-tqdm python3-setuptools
```

Install the upload script:
```bash
cd /tmp
git clone https://github.com/OpenBekenIOT/hid_download_py
cd hid_download_py
python3 setup.py install --user
```

Then go to https://github.com/openshwprojects/OpenBK7231T_App/releases, select the latest firmware and download the "UART Flash" for BK7231T and BK7231N:
```bash
cd /tmp
wget https://github.com/openshwprojects/OpenBK7231T_App/releases/download/1.15.231/OpenBK7231T_UA_1.15.231.bin
wget https://github.com/openshwprojects/OpenBK7231T_App/releases/download/1.15.231/OpenBK7231N_QIO_1.15.231.bin
```

Plug the USB adapter.
Plug the RESET cable to GND.
Run this command and after pressing enter, when "Getting bus" is displayed, release the RESET wire.

For BK7231T:
```bash
cd /tmp/hid_download_py
python3 uartprogram ../OpenBK7231T_UA_1.15.231.bin -d /dev/ttyUSB0 -w
```

For BK7231N:
```bash
cd /tmp/hid_download_py
python3 uartprogram ../OpenBK7231N_QIO_1.15.231.bin -d /dev/ttyUSB0 -w --unprotect --startaddr 0x0
```

Once flashed the chip will reboot himself and a WiFi access point is created.
Connect to it then go to 192.168.4.1


## Configure OpenBekenIOT

Connect to the access point created by OpenBekenIOT.
Open http://192.168.4.1


- Configure general: set flag 4 for BP5758D controllers.
- Configure names: set the name of the device (same as MQTT login). Repeat the same in both inputs.
- Configure WiFi at the end as it shutdown the access point.


## TuyaMCU

It is a MCU that communicates via UART to the WiFi module, usually use in low energy circuits to shutdown the WiFi module.

Documentations:
- https://developer.tuya.com/en/docs/iot/tuyacloudlowpoweruniversalserialaccessprotocol?id=K95afs9h4tjjh
- https://images.tuyacn.com/smart/aircondition/Guide-to-Interworking-with-the-Tuya-MCU.pdf
- https://developer.tuya.com/en/docs/iot/mcu-protocol?id=K9hrdpyujeotg

To start it we have to execute `startDriver TuyaMCU`.
Then in logs (with the "Web Application") we should see the UART logs.

