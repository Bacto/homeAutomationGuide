# Xiaomi Mijia (LYWSD03MMC)

> **If it helps you, please star it 🙏**
>
> It's the only way to me to know that it's useful to other people 🥰

It is a low cost (~5€) temperature and humidity sensor with a screen.
It communicates with bluetooth.

By default data exchanged in bluetooth are encrypted.
We can simply disable this by flashing it with a custom firmware.


## Flash firmware

See https://github.com/pvvx/ATC_MiThermometer

Use a browser that supports bluetooth (like Chrome).

- Go to https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html
- Click on "Connect"
- Wait for a device like "LYWS..."
- Click on "Do Activation"
- Click on "Custom Firmware ver ..."
- Click "Start flashing"
- Wait for "Update done after ..." in logs
- Click on "Connect"
- Select the "ATC_...". Restart the browser if it doesn't work.


## Configure firmware

Configure it like this:
- In "Send settings to custom firmware"
  - Set "advertising type" to "MIJIA (MiHome)"
  - To reduce battery consumption:
    - Set "Advertising interval" to 20000
    - Set "Measure interval" to 1
  - Click "Send config"
- Set name to something like "temp_1" and click "Set new name"
- Set temperature low to "20.99" and click "Set comfort parameters"
- Get the MAC address: click on "Show all mi keys", copy the "Device MAC" and remove the 4 last characters. It will be needed for Homebridge configuration.



## Use it with Homebridge

In Homebridge, add the plugin `Homebridge Mi Hygrothermograph`.

Then, in settings:
- Set "Accessory Name" to the name previously defined in the firmware configuration (like `temp_1`)
- Set "Device MAC Address"
- Configure MQTT:
  - Broker URL: mqtt://homebridge:<password>@mosquitto:1883
  - Temperature Topic: temp_1/temperature
  - Humidity Topic: temp_1/humidity
  - Battery Topic: temp_1/battery


## Stop working after some time

In logs we can see this:
```
ERROR (MainThread) [homeassistant.components.bluetooth.scanner] hci0 (xxx): Error stopping scanner: [org.bluez.Error.InProgress] Operation already in progress
WARNING (MainThread) [bluetooth_auto_recovery.recover] Bluetooth adapter hci0 [xxx] could not be reset due to timeout
WARNING (MainThread) [bluetooth_auto_recovery.recover] Bluetooth management socket connection lost: [Errno 9] Bad file descriptor
```

See https://github.com/home-assistant/core/issues/76186
And https://github.com/home-assistant/core/issues/76186#issuecomment-1217784437

bluetoothctl
power off
power on
scan on
quit

reload HA bluetooth integration

=> Then it works again

TODO
- Do a script that start scans
while true ; do bluetoothctl power off ; /etc/init.d/bluetooth restart ; sleep 30 ; done
while true ; do bluetoothctl --timeout 5 power off ; killall -9 bluetoothd ; sleep 5 ; /usr/lib/bluetooth/bluetoothd ; sleep 120 ; done

/etc/init.d/bluetoothd stop
while true ; do killall -9 bluetoothd ; sleep 5 ; (/usr/lib/bluetooth/bluetoothd &) ; sleep 120 ; done