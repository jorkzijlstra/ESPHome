# ESPHome

## ABB EV-1 kWh meter


```
substitutions:
    abb2_device_address: 2
    abb3_device_address: 3

# In order to use other language or a different ESP chip, fix include labels file name below:
# Currently supported languages are en, nl. 
# ESP32 is a mh-et-live (v1 version of my pcb) and esp32s3 is a lilygo ESP32S3-T7 (v2 version of my pcb)
# Be carefull not to upload the wrong code to the wrong chip. This could brick your ESP chip.
packages:
  remote_package:
    url: https://github.com/jorkzijlstra/ESPHome
    ref: main
    refresh: 0s
    files: [ esphome/labels/ABB2.yaml,
             esphome/labels/ABB3.yaml,
           ]

```