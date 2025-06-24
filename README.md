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
    name: "Solax Inverter"
    resource: !secret solax_api_url
    method: POST
    payload: !secret solax_api_payload
    headers:
      Content-Type: application/json
      tokenId: !secret solax_api_token
    value_template: "{{ value_json.result.acpower }}"  # just a placeholder
    json_attributes_path: "$.result"
    json_attributes:
      - acpower
      - yieldtoday
      - yieldtotal
      - powerdc1
      - powerdc2
    scan_interval: 60

template:
  - sensor:
      - name: "Solax AC Power"
        unique_id: solax_ac_power
        unit_of_measurement: "W"
        device_class: power
        state_class: measurement
        state: "{{ state_attr('sensor.solax_inverter', 'acpower') | float(0) }}"
      - name: "Solax Yield Today"
        unique_id: solax_yield_today
        unit_of_measurement: "kWh"
        device_class: energy
        state_class: total_increasing
        state: "{{ state_attr('sensor.solax_inverter', 'yieldtoday') | float(0) }}"
      - name: "Solax Yield Total"
        unique_id: solax_yield_total
        unit_of_measurement: "kWh"
        device_class: energy
        state_class: total_increasing
        state: "{{ state_attr('sensor.solax_inverter', 'yieldtotal') | float(0) }}"
      - name: "Solax DC Power Total"
        unique_id: solax_dc_power_total
        unit_of_measurement: "W"
        device_class: power
        state_class: measurement
        state: >
          {% set dc1 = state_attr('sensor.solax_inverter', 'powerdc1') | float(0) %}
          {% set dc2 = state_attr('sensor.solax_inverter', 'powerdc2') | float(0) %}
          {{ dc1 + dc2 }}
```
