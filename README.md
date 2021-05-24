# Auto GreenHouse ![Version](https://img.shields.io/badge/Version-v0.0.1-orange?style=flat-square&url=https://github.com/DEADSEC-SECURITY/AutoGreenHouse) ![License](https://img.shields.io/badge/License-MIT-red?style=flat-square) ![Donate](https://img.shields.io/badge/Donate-Crypto-yellow?style=flat-square)

## 📝CONTRIBUTIONS

Before doing any contribution read <a href="https://github.com/DEADSEC-SECURITY/AutoGreenHouse/blob/main/CONTRIBUTING.md">CONTRIBUTING</a>.

If you want to buy this project online please refer to: [Tindie](https://www.tindie.com/products/deadsec-iot/greenhouse-controller-box-ha-esphome/)

## 📧 CONTACT

Email: amng835@gmail.com

General Discord: https://discord.gg/dFD5HHa

Developer Discord: https://discord.gg/rxNNHYN9EQ

## ⚙️ SETTING UP EVERYTHING

**Note:** To use this cards you need to install HACS and the following repositories:
- [Bar Card](https://github.com/custom-cards/bar-card)
- [Config Template Card](https://github.com/iantrich/config-template-card)

**Note:** Please use you're own entities if yours aren't equal to mine.

#### Dashboard Card Code:

*Weather:*
```yaml
type: weather-forecast
entity: weather.home
```
*Plant Monitor:*
```yaml
type: glance
entities:
  - entity: plant.plant_1
title: 'Green House #1'
```
*Plant (plant.yaml)*
```yaml
Plant#1:
  sensors:
    moisture: sensor.green_house_1_plant_1_moisture
    temperature: sensor.green_house_1_plant_1_temperature
    conductivity: sensor.green_house_1_plant_1_soil_conductivity
    brightness: sensor.green_house_1_plant_1_illuminance
  min_moisture: 15
  min_temperature: 15
  min_conductivity: 350
```
To add the plant.yaml to configuration add the following code to configuration.yaml:
```yaml
plant: !include plant.yaml
```
*GreenHouse Actions/Monitor:*
```yaml
type: 'custom:config-template-card'
entities:
  - sensor.green_house_1_temperature
  - sensor.green_house_1_humidity
  - sensor.green_house_1_intake_water_ph
  - sensor.green_house_1_intake_water_flow
  - switch.greenhouse_1_fan_1
  - switch.greenhouse_1_fan_2
  - sensor.green_house_1_intake_neutral_ph
variables:
  INTAKE_NEUTRAL_PH: 'states["sensor.green_house_1_intake_neutral_ph"].state'
  INTAKE_PH: 'states["sensor.green_house_1_intake_neutral_ph"].state'
  OFFSET: 0.5
  WATER_FLOW: 'states["sensor.green_house_1_intake_water_flow"].state'
card:
  type: entities
  entities:
    - entity: sensor.green_house_1_temperature
      name: Temperature
    - entity: sensor.green_house_1_humidity
      name: Humidity
    - entity: sensor.green_house_1_intake_water_ph
      name: Intake Water PH
      max: '${INTAKE_NEUTRAL_PH*2}'
      min: 0
      entity_row: true
      target: '${INTAKE_NEUTRAL_PH}'
      type: 'custom:bar-card'
      icon: >-
        ${INTAKE_PH >= INTAKE_NEUTRAL_PH - OFFSET && INTAKE_PH <=
        INTAKE_NEUTRAL_PH + OFFSET ? 'mdi:water-remove' : 'mdi:water-check'}
    - entity: sensor.green_house_1_intake_water_temperature
      name: Intake Water Temperature
    - entity: sensor.green_house_1_intake_water_flow
      name: Intake Water Flow
      icon: '${WATER_FLOW == 0 ? ''mdi:pipe-disconnected'' : ''mdi:pipe''}'
    - entity: switch.greenhouse_1_fan_1
      name: 'Fan #1'
      icon: 'mdi:fan'
    - entity: switch.greenhouse_1_fan_2
      name: 'Fan #2'
      icon: 'mdi:fan'
    - entity: switch.green_house_1_intake_water
      icon: 'mdi:engine'
  title: 'Green House #1'
```

#### Esphome Code:

**Note:** Feel free to change any values as long as they are the same in your dashboard so no errors occur.

**Note:** This code uses the [Xiaomi LYWSD03MMC](https://esphome.io/components/sensor/xiaomi_ble#lywsd03mmc) for mesuring temperature and humidity, the [Xiaomi MiFlora](http://esphome.io/components/sensor/xiaomi_ble#hhccjcy01) to measure plant temperature, brightness, condutivity, and moisture and used some other electronics like the following:
- [4-Channel Relay Module](https://amzn.to/3eKqenM)
- [Temperature Sensor Waterproof](https://amzn.to/3fievvB)
- [pH Sensor](https://amzn.to/3bpS7zr)
- [110V/220V -> 5V Transformer](https://amzn.to/3w4f1Ei)

*GreenHouse Box:*
```yaml
esphome:
  name: greenhouse_1_sensors
  platform: ESP32
  board: esp32dev

wifi:
  ssid: "<YOUR_SSID>"
  password: "<YOUR_PASSWORD>"

logger:

# Enable Home Assistant API
api:
  password: "<YOUR_PASSWORD>"

ota:
  password: "<YOUR_PASSWORD>"

binary_sensor:
  - platform: status
    name: "Green House 1 Hub Status"

esp32_ble_tracker:

switch:
  - platform: gpio
    pin: 25
    name: "Green House 1 Intake Water"

dallas:
  - pin: GPIO33

sensor:
  - platform: atc_mithermometer
    mac_address: "<YOUR_MAC_ADDR>"
    temperature:
      name: "Green House 1 Temperature"
    humidity:
      name: "Green House 1 Humidity"
    battery_level:
      name: "Green House 1 Battery Level"
      
  - platform: xiaomi_hhccjcy01
    mac_address: '<YOUR_MAC_ADDR>'
    temperature:
      name: "Green House 1 Plant 1 Temperature"
    moisture:
      name: "Green House 1 Plant 1 Moisture"
    illuminance:
      name: "Green House 1 Plant 1 Illuminance"
    conductivity:
      name: "Green House 1 Plant 1 Soil Conductivity"
    
  - platform: adc
    pin: GPIO32
    id: intake_water_ph
    name: "Green House 1 Intake Water PH"
    icon: "mdi:water"
    update_interval: 1s
    unit_of_measurement: pH
    attenuation: 11db
    filters:
      - median:
          window_size: 10
          send_every: 5
          send_first_at: 5
      - calibrate_linear:
          - 3.00 -> 6.0
          - 2.50 -> 9.5
    
  - platform: dallas
    address: <YOUR_ADDR>
    id: intake_water_temp
    name: 'Green House 1 Intake Water Temperature'
    icon: "mdi:thermometer"
    on_value:
      then:
        lambda: |-
          float temp = id(intake_water_temp).state;
      
          float in_min = 0;
          float in_max = 100;
          float out_min = 7.47;
          float out_max = 6.14;
          
          float neutral_pH = ((x - in_min) * (out_max - out_min) + out_min * (in_max - in_min)) / (in_max - in_min);
          
          id(intake_neutral_ph).publish_state(neutral_pH);
          char str[32];
          dtostrf(neutral_pH, 8, 2, str);
          ESP_LOGD("'Intake Neutral pH'", str);
  
  - platform: adc
    pin: GPIO35
    id: intake_water_flow
    name: "Green House 1 Intake Water Flow"
    icon: "mdi:pipe-leak"
    update_interval: 1s
    unit_of_measurement: L/min
    attenuation: 11db
    filters:
      - median:
          window_size: 10
          send_every: 5
          send_first_at: 5
  
  - platform: template
    id: intake_neutral_ph
    name: "Green House 1 Intake Neutral pH"
```

**Note:** Our fans where not wifi controllable by default so we used a [Sonoff Basic](https://amzn.to/2RRr9tK) to act as the middle man and made them wifi controllable.

*Fan control: (Sonoff)*
```yaml
esphome:
  name: greenhouse_1_fan_1
  platform: ESP8266
  board: esp01_1m

wifi:
  ssid: "<YOUR_SSID>"
  password: "<YOUR_PASSWORD>"

logger:

# Enable Home Assistant API
api:
  password: "<YOUR_PASSWORD>"

ota:
  password: "<YOUR_PASSWORD>"
  
binary_sensor:
  - platform: status
    name: "GreenHouse #1 Fan #1 Board"
  - platform: gpio
    pin:
      number: GPIO0
      mode: INPUT_PULLUP
      inverted: True
    name: "GreenHouse #1 Fan #1 Button"
    on_press:
      - switch.toggle: relay

switch:
  - platform: gpio
    name: "GreenHouse #1 Fan #1"
    pin: GPIO12
    id: relay

status_led:
  pin:
    number: GPIO13
    inverted: yes

```

**Note:** This projects is under development and new updates should be added with frequency.

**Note:** Automations will be added and created in the near future.

## 🖼️ SCREENSHOTS

#### Dashboard

![alt text](https://github.com/DEADSEC-SECURITY/AutoGreenHouse/blob/main/Images/Home%20Assistant%20Dashboard.PNG)

#### Fans with Sonoff Basic

![alt text](https://github.com/DEADSEC-SECURITY/AutoGreenHouse/blob/main/Images/Fans%20With%20Sonoff.jpeg)

#### GreenHouse Box Open/Closed

![alt text](https://github.com/DEADSEC-SECURITY/AutoGreenHouse/blob/main/Images/GreenHouse%20Box%20Open.jpg)

![alt text](https://github.com/DEADSEC-SECURITY/AutoGreenHouse/blob/main/Images/GreenHouse%20Box%20Closed.jpg)

**Note:** These picturesdo not include the Water Flow Sensor and Valve
