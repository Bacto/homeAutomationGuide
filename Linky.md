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
This power supply is not powerful at all and can't power an ESP8266. Don't loose time on it like I did.

It's an alternative courant supply, 12v peak to peak.
6 Vrms ± 10% at 50 kHz with 130 MW minimum.


### Data

Data is modulated on a 50kHz carrier (amplitude modulation) on `I1` and `I2` connectors.
It's basically a serial port that can't be connected directly to an ESP8266. You'll have to create a small electronic "converter".

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


#### Compile Tasmota with Teleinfo

```bash
# Clone Tasmota. Set the version to the last one available on https://github.com/arendst/Tasmota/releases
export TASMOTA_VERSION=v12.3.1
git clone --depth=1 https://github.com/arendst/Tasmota.git -b ${TASMOTA_VERSION} /tmp/tasmota
cd /tmp/tasmota

# Add Teleinfo feature
echo '#define USE_TELEINFO' >> tasmota/user_config_override.h

# Compile using Docker
docker run -it -v /tmp/tasmota:/tasmota blakadder/docker-tasmota -e tasmota

# The firmware is here. Get it and update the ESP8266 with it using OTA (connect to the web interface, the "Firmware Upgrade" and put the firmware file).
ls -al /tmp/tasmota/build_output/firmware/tasmota.bin.gz
```

> Handle error "Not enough space":
> If the binary is larger than 500kb (which should be the case), the ESP will not have enough space to upgrade its firmware.
> In that case, flash it using the "Upgrade by web server" part and put this URL: `http://ota.tasmota.com/tasmota/release/tasmota-minimal.bin.gz`.
> Then flash it again with firmware that includes "Teleinfo".


#### Configure Teleinfo

Connect to your ESP8266, go to "Configuration"/"Configure Module" and:
- In "Module type", select "Generic (0)".
- In "RX GPIO3", select "TInfo RX".

Then go to the main page and click on "Console" and configure Teleinfo to use the "Standard" mode: `EnergyConfig standard`


### Tasmota manual configuration (not recommanded)

- In "Console":
  - "Baudrate 1200"
  - "SerialConfig 7e1"
  - "SerialDelimiter 128"
  - "SerialSend1 ok": not required finally?
