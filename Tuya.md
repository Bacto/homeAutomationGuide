# Tuya


> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰


## Flash devices remotely with Tuya Cloudcuter

Don't loose time, do it on a Raspberry with Raspberry Pi OS Lite installed.
See https://github.com/tuya-cloudcutter/tuya-cloudcutter/blob/main/HOST_SPECIFIC_INSTRUCTIONS.md

```bash
git clone https://github.com/tuya-cloudcutter/tuya-cloudcutter/
cd tuya-cloudcutter
./run_detach.sh <SSID> <SSID password> wlan0
```

> Note: for lights, it should blink slow. If it blinks fast, power on and off for 6 times during 1 second each time.


Then flash with the custom firmware. Here is an example with OpenBK7231T:
```bash
wget https://github.com/openshwprojects/OpenBK7231T_App/releases/download/1.15.93/OpenBK7231T_UG_1.15.93.bin
mv OpenBK7231T_UG_1.15.93.bin custom-firmware/
./run_flash.sh wlan0 <deviceProfile> OpenBK7231T_UG_1.15.93.bin
```
Replace `<deviceProfile>` with profile in `device-profiles` (example: `LSC/3001702-970727`).
