# Linky

Enedis (was ERDF - Électricité Réseau Distribution France) distribute the Linky meters in France.
They have a serial port where we can connect to them and retrieve some live informations. This is called the "TIC" (télé-information client).


## TIC (télé-information client)

Technicals informations: http://www.enedis.fr/sites/default/files/ERDF-NOI-CPT_02E.pdf
Technicals informations (simpler): https://www.srd-energies.fr/wp-content/uploads/sites/7/2022/02/sorties_de_tele-information_linky_client.pdf

Sources:
- https://faire-ca-soi-meme.fr/domotique/2016/09/12/module-teleinformation-tic/
- https://miniprojets.net/index.php/2019/06/28/recuperer-les-donnees-de-son-compteur-linky/
- http://hallard.me/demystifier-la-teleinfo/
- https://tasmota.github.io/docs/Teleinfo/
- Supposed schematic of the internal TIC system: https://forums.futura-sciences.com/attachments/electronique/334949d1487188300-compteur-linky-informations-client-tic-bornes-i1-i2-a-lunki_i1_i2_a.jpg


### Power supply

The TIC can supply power, between `A` and `I1` connectors.
This power supply is not powerful at all and can't power an ESP. Don't loose time on it like I did.

It's an alternative courant supply, 12v peak to peak.
6 Vrms ± 10% at 50 kHz with 130 MW minimum.


### Data

Data is modulated on a 50kHz carrier (amplitude modulation) on `I1` and `I2` connectors.
It's basically a serial port that can't be connected directly to an ESP. You'll have to create a small electronic "converter".

You'll find a simple example here (which works for me):
https://faire-ca-soi-meme.fr/domotique/2016/09/12/module-teleinformation-tic/

If it doesn't work for you, have a look here:
http://hallard.me/demystifier-la-teleinfo/


If you don't want to create the electronical board yourself, you can buy one of these ones:
- https://www.tindie.com/products/hallard/wemos-teleinfo/
- https://www.tindie.com/products/hallard/pitinfo/

> Buying one of these cards is a good idea to support "Charles" awesome work.
> He written some really good [documentation](http://hallard.me/demystifier-la-teleinfo/) and the [Tasmota "Teleinfo" plugin](https://tasmota.github.io/docs/Teleinfo/).


The TIC has 2 modes:
1. "Historique", with speed at 1200 bauds. This is the default mode.
2. "Standard", with speed at 9600 bauds. This is more complete.

To activate the "Standard" mode, which is recommended, send a message to your electricity provider and ask them to make a request to Enedis to activate the "Standard" TIC mode.
It takes around 24 hours to have it activated.

Here is an example of a message (in french) you can send:
```
Bonjour,

J'ai équipé mon compteur Linky d'un système de « télé-information client » (TIC).
Celui-ci requiert le réglage du compteur en mode "standard" et non pas "historique" comme il l'est actuellement.

D'après le document "Enedis-PRO-CF_55E V1", il appartient au fournisseur titulaire du point de connexion de demander à Enedis ce changement.

Pouvez-vous donc demander à Enedis de changer le mode TIC à "standard" svp ?

Merci d'avance,
```


## ESP

Connect serial to GPIO3 (RX).

Install Tasmota firmware.

Then configure Tasmota:
- In "Configure module":
  - Set "Module type" to "Generic (0)"
- Configure MQTT
- In "Configure other", set "Device name" and "Friendly name" to "linky"


### Tasmota Teleinfo (recommanded)

See https://tasmota.github.io/docs/Teleinfo/

Teleinfo feature is not available in Tasmota precompiled binaries.
We have to compile it manually. Thanks to Docker it will be easy :)

> ⚠️ With an ESP8266, when serial data is received, the MCU reboots for an unknown reason (memory issue?).
> We have to use and ESP32 to resolve this. Officially Teleinfo works with ESP8266 but not really tested, no support is offered for it and ESP32 is recommanded.


#### Compile Tasmota with Teleinfo

```bash
# Prepare container to build the firmware
# We need to build the image as it is only for AMD64 on hub.docker.com
git clone https://github.com/tasmota/docker-tasmota
cd docker-tasmota
docker build -t docker-tasmota .


# Clone Tasmota. Set the version to the last one available on https://github.com/arendst/Tasmota/releases
export TASMOTA_VERSION=v12.4.0
git clone --depth=1 https://github.com/arendst/Tasmota.git -b ${TASMOTA_VERSION} /tmp/tasmota
cd /tmp/tasmota

# Add Teleinfo feature
echo '#define USE_TELEINFO' >> tasmota/user_config_override.h

# Compiling using Docker.
#
# For an ESP8266:
# docker run -it -v /tmp/tasmota:/tasmota docker-tasmota -e tasmota
#
# For an ESP32:
docker run -it -v /tmp/tasmota:/tasmota docker-tasmota -e tasmota32

# The firmware is here:
ls -al /tmp/tasmota/build_output/firmware/*.bin

# tasmota32.factory.bin: Factory binaries to be used for inital flashing using esptool
# tasmota32.bin: OTA firmware
```

Flash it with OTA or with serial connection.

Example using serial connection:
```bash
docker run -it -v /dev:/dev -v $PWD:/mnt --privileged ubuntu:20.04

apt-get update \
  && apt-get install git python3 python3-setuptools wget \
  && git clone https://github.com/espressif/esptool \
  && cd esptool \
  && python3 setup.py install

# For an ESP8266
# esptool.py --baud 921600 write_flash --erase-all -fm dout 0x0 /mnt/build_output/firmware/tasmota.bin

# For an ESP32
esptool.py --chip esp32 --baud 921600 --before default_reset --after hard_reset write_flash -z --flash_mode dout --flash_size detect 0x0 /mnt/build_output/firmware/tasmota32.factory.bin
```


> Handle error "Not enough space":
> If the binary is larger than 500kb (which should be the case), the ESP will not have enough space to upgrade its firmware.
> In that case, flash it using the "Upgrade by web server" part and put this URL: `http://ota.tasmota.com/tasmota/release/tasmota-minimal.bin.gz`.
> Then flash it again with firmware that includes "Teleinfo".


#### Configure Teleinfo

Connect to your ESP:
- In "Configuration"/"Configure Module"
  - Configure pin out depending on the board: https://github.com/hallard/WeMos-TIC#detailed-description
- Configure MQTT
- Configure name and password in "Configure Other"
- In "Configure Logging", set "Telemetry period" to 10 (report every 10 seconds to MQTT)
- In "Console" and configure Teleinfo to use the "Standard" mode: `EnergyConfig standard`
- Restart


### Tasmota manual configuration (not recommanded)

- In "Console":
  - "Baudrate 1200"
  - "SerialConfig 7e1"
  - "SerialDelimiter 128"
  - "SerialSend1 ok": not required finally?
