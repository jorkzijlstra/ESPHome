# ABB EV-1 Modbus

This allow ESPHome to read the ABB EV-1 kWh meters via modbus. I created an ABB2 and an ABB3 which allows users to import 1 or 2 meters.

I personally have 2 meters installed, once device ID 2 is the Misubishi Ecodan Heatpump itself and device ID 3 is the booster inside the healtpomp inside unit

## Update ABB EV-1 kWh meter modbus settings. 
See [EV1_Modbus_Map_RevB.pdf](documentation/EV1_Modbus_Map_RevB.pdf) for the registers

## Device Slave ID
 Set 0x8900 to your wanted Slave ID (f.i. 2)

## set parity to none
 Set 0x0414 to 1 

# ESPHome
## Add this to your esphome yaml file, either ABB2 or ABB3 or both.

I assumed the procon would be modbus address 1 and hence named the kWh meter ABB2 and ABB3.

```
substitutions:
    abb2_device_address: 2
    abb3_device_address: 3

packages:
  remote_package:
    url: https://github.com/jorkzijlstra/ESPHome
    ref: main
    refresh: 0s
    files: [ esphome/labels/ABB2.yaml,
             esphome/labels/ABB3.yaml
           ]

```


# Home Assistant

Now we can use these values in home assistant

```
template:
  - sensor:
      - name: deltat
        state: >
          {% set q = states('sensor.mitsubishi_flow_temperature') | float %} 
          {% set w = states('sensor.mitsubishi_return_temperature') | float %}
          {% if q >= 0 and w > 0 %}
          {{ (q - w) | round(2) }}
          {% else %}
            0
          {% endif %}
        unit_of_measurement: "°C"
        unique_id: "DeltaT"
  - sensor:
      - name: realtime_afgifte
        unique_id: "Realtime_Afgifte"
        state: >
          {% set deltat = states('sensor.deltat') | float %}
          {% set flow = states('sensor.mitsubishi_flow_rate') | float %}
          {% if (deltat > 0)%} 
          {{(deltat * 4.2 * (flow / 60)) | round(1, default=0)}}
          {% else %} 0
          {% endif %}
  - sensor:
      - name: realtime_cop
        unique_id: "Realtime_COP"
        state: >
          {% set deltat = states('sensor.deltat') | float %}
          {% set flow = states('sensor.mitsubishi_flow_rate') | float %}
          {% set pump_raw = states('sensor.abb2_power_total') | float %}
          {% set booster_raw = states('sensor.abb3_power_total') | float %}
          {% set pump = pump_raw / 1000 | float %}
          {% set booster = booster_raw / 1000 | float %}
          {% if (pump_raw > 0) and (deltat > 0)%} 
          {{(deltat * 4.2 * (flow / 60)) / (pump + booster) | round(1, default=0)}}
          {% else %} 0
          {% endif %}
```

After which we can add something visual on a dashboard

```
cards:
      - type: custom:mini-graph-card
        entities:
          - sensor.mitsubishi_outdoor_ambient_temperature
          - sensor.realtime_afgifte
          - sensor.realtime_cop
        name: Buitentemperatuur
        show:
          labels: true
        line_width: 0.5
        hours_to_show: 24
```
