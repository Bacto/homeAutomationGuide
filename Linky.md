# Linky

Enedis, anciennement ERDF (Électricité réseau distribution France).


## TIC (télé-information client)

Informations techniques : http://www.enedis.fr/sites/default/files/ERDF-NOI-CPT_02E.pdf
Informations techniques plus claires : https://www.srd-energies.fr/wp-content/uploads/sites/7/2022/02/sorties_de_tele-information_linky_client.pdf

Sources :
- https://faire-ca-soi-meme.fr/domotique/2016/09/12/module-teleinformation-tic/
- https://miniprojets.net/index.php/2019/06/28/recuperer-les-donnees-de-son-compteur-linky/
- http://hallard.me/demystifier-la-teleinfo/
- https://tasmota.github.io/docs/Teleinfo/
- Schéma supposé interne de la TIC : https://forums.futura-sciences.com/attachments/electronique/334949d1487188300-compteur-linky-informations-client-tic-bornes-i1-i2-a-lunki_i1_i2_a.jpg

2 modes :
- Historique, 1200 bauds.
- Standard, apparu avec les compteurs Linky, 9600 bauds. Ce mode est plus complet.

Alimentation alternative, 12v crête à crête. 6 Vrms ± 10% à 50 kHz, 130 MW minimum. Pas assez puissante pour alimenter un ESP8266 :(
Données modulées sur une porteuse à 50kHz (modulation d’amplitude).


## ESP8266

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

```bash
# Clone Tasmota. Set the version to the last one available on https://github.com/arendst/Tasmota/releases
export TASMOTA_VERSION=v12.3.1
git clone --depth=1 https://github.com/arendst/Tasmota.git -b ${TASMOTA_VERSION} /tmp/tasmota
cd /tmp/tasmota

# Add Teleinfo feature
echo '#define USE_TELEINFO' >> user_config_override.h

# Compile using Docker
docker run -it -v /tmp/tasmota:/tasmota blakadder/docker-tasmota

# The firmware is here. Get it and update the ESP8266 with it using OTA (connect to the web interface, the "Firmware Upgrade" and put the firmware file).
ls -al /tmp/tasmota/build_output/firmware/tasmota.bin.gz
```


### Tasmota manual configuration (not recommanded)

- In "Console":
  - "Baudrate 1200"
  - "SerialConfig 7e1"
  - "SerialDelimiter 128"
  - "SerialSend1 ok": not required finally?
