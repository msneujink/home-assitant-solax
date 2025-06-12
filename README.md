# Solax Inverter integration for Home Assitant
This integration allows you to monitor your Solax solar inverter’s real-time data directly in Home Assistant using the official Solax Cloud API. It fetches key metrics such as AC power output, daily and total energy yield, and combined DC power from your inverter via REST sensors.

By following the instructions below, you will be able to add these sensors to your Home Assistant setup and use them for monitoring, automation, and integration with the Home Assistant Energy Dashboard.

## Add the secrets to secrets.yml
You can find your serial number and token by logging into your account at solaxcloud.com.

```yml
solax_api_url: https://global.solaxcloud.com/api/v2/dataAccess/realtimeInfo/get
solax_api_token: YOUR_TOKEN
solax_api_payload: '{"wifiSn": "YOUR_SERIAL"}'
```


## Add the following to the Home Assitant configuration.yml
If you encounter API limitation issues try adjusting the `scan_interval`.

```yml
sensor:
  - platform: rest
    name: Solax AC Power
    resource: !secret solax_api_url
    method: POST
    payload: !secret solax_api_payload
    headers:
      Content-Type: application/json
      tokenId: !secret solax_api_token
    value_template: "{{ value_json.result.acpower }}"
    unit_of_measurement: "W"
    scan_interval: 1
    device_class: power
    state_class: measurement

  - platform: rest
    name: Solax Yield Today
    resource: !secret solax_api_url
    method: POST
    payload: !secret solax_api_payload
    headers:
      Content-Type: application/json
      tokenId: !secret solax_api_token
    value_template: "{{ value_json.result.yieldtoday }}"
    unit_of_measurement: "kWh"
    scan_interval: 60
    device_class: energy
    state_class: total_increasing

  - platform: rest
    name: Solax Yield Total
    resource: !secret solax_api_url
    method: POST
    payload: !secret solax_api_payload
    headers:
      Content-Type: application/json
      tokenId: !secret solax_api_token
    value_template: "{{ value_json.result.yieldtotal }}"
    unit_of_measurement: "kWh"
    scan_interval: 60
    device_class: energy
    state_class: total_increasing

  - platform: rest
    name: Solax DC Power Total
    resource: !secret solax_api_url
    method: POST
    payload: !secret solax_api_payload
    headers:
      Content-Type: application/json
      tokenId: !secret solax_api_token
    value_template: >
      {{ (value_json.result.powerdc1 | float(0)) + (value_json.result.powerdc2 | float(0)) }}
    unit_of_measurement: "W"
    scan_interval: 1
    device_class: power
    state_class: measurement
```
