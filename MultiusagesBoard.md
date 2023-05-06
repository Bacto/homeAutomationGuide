# Multiusages board

- Pin out:
  - D1: I2C SCL
  - D2: I2C SDA
  - D3: WS2812

### Peripherals


## RGB LED (WS2812B)

A WS2812B LED is connected to D3.


## I2C expander support (MCP23008)

A MCP23008 expander is connected to I2C (SDA on D2, SCL on D1).

- D0 to D3 are relays.
- D4 to D7 are buttons terminals.

> ⚠️ An internal **weak** pull-up resistor is available (to configure).
> Put this resistor is so weak that it seems we can trigger inputs just by touching the wire.
> Adding an external pull-up resistor can be great.

See https://tasmota.github.io/docs/MCP230xx/
And https://tasmota.github.io/docs/I2CDEVICES/

Compile Tasmota with MCP230xx support:
```bash

# Enable INPUT mode (pinmode 1 through 4)
echo '#define USE_MCP230xx' >> tasmota/user_config_override.h

# Enable mode 2 to handle MCP23XXX as native I/Os (see https://tasmota.github.io/docs/MCP230xx/#mode-2)
echo '#define USE_MCP23XXX_DRV' >> tasmota/user_config_override.h

# Enable OUTPUT mode (pinmode 5)
echo '#define USE_MCP230xx_OUTPUT' >> tasmota/user_config_override.h

# Display state of OUTPUT pins on main Tasmota web interface
echo '#define USE_MCP230xx_DISPLAYOUTPUT' >> tasmota/user_config_override.h

# Set address to 0x20
echo '#define USE_MCP230xx_ADDR 0x20' >> tasmota/user_config_override.h
```

## Tasmota configuration

- In `Configure Other`
  - Add template: `{"NAME":"Bacto Multiusages Board","GPIO":[1376,0,0,0,640,608,0,0,0,0,0,0,0,0],"FLAG":0,"BASE":1}`
  - Activate template
  - Add password
  - Configure device name and friendly name

- In console
  - Do not keep power state after restart:
    - SetOption0 0
  - Improve Home Assistant integration:
    - SetOption59 1
- Configure MQTT


## Rules examples

> ⚠️ Don't forget to activate rule with `Rule1 1`

Plug a button on "Button-1" (D4 on MCP230xx), then have a look at the LED (controlled by `Power1`):
```
Rule1
  ON MCP230XX_INT#D4 DO Power1 %value% ENDON
```

```
Rule1
  ON MCP230XX_INT#D4=1 DO Power1 0 ENDON
  ON MCP230XX_INT#D4=0 DO Power1 1 ENDON
```

Control the relay:
```
Rule1
  ON MCP230XX_INT#D4=1 DO sensor29 0,OFF ENDON
  ON MCP230XX_INT#D4=0 DO sensor29 0,ON ENDON
```

On Button-1 press, power on the relay. After 2 seconds power off the relay:
```
Rule1
  ON MCP230XX_INT#D4=0 DO Backlog sensor29 0,ON; RuleTimer1 2 ENDON
  ON Rules#Timer=1 DO sensor29 0,OFF ENDON
```


- On Button-1 press, enable relay 1. On Button-1 long press, disable relay 1:
```
Rule1
  ON MCP230XX_INT#D4=0 DO backlog sensor29 0,ON ; RuleTimer1 2 ENDON
  ON MCP230XX_INT#D4=1 DO RuleTimer1 0 ENDON
  ON Rules#Timer=1 DO sensor29 0,OFF ENDON
```

- Send Button-1 value to topic "cmnd/example":
```
Rule1
  ON MCP230XX_INT#D4 DO publish cmnd/example %value% ENDON
```


## TODO

- Add 10k pull-up resistor to buttons