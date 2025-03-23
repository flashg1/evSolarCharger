# EV Solar Charger
Home Assistant Blueprint using OCPP and/or EV specific API to charge EV from excess solar and weather forecast.

###############################################################################
# Disclaimer:
#
# Even though this automation has been created with care, the author cannot be responsible for any damage caused by this automation.  Use at your own risk.
#
###############################################################################

![Screenshot_20230702-094232_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/58d1df89-905b-422c-8542-0081b9fa342f)

![Screenshot_20230630-135925_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/2f04b1e2-b56d-493c-977f-82d5dd04cbe5)


Features
========

-   Charge from excess solar adjusting car charging current according to feedback loop value "Main Power Net".  The "Main Power Net" sensor expresses negative value in Watts for available power for charging car, or positive value for consumed power.
-   Support multi-day solar charging using sun elevation triggers to start and stop.
-   Compatible with off-peak night time charging.
-   Configurable daily car charge limit for 7 days.  Default is to use existing charge limit already set in car.
-   Automatically adjust to the highest charge limit set within a rainy forecast period.  The highest charge limit is selected from the 7 days charge limit settings that are within the forecast period taking into account the charge limit on bad weather setting.  The objective is to charge more before a rainy period.  Default disabled.
-   Might be possible to prolong car battery life by setting daily charge limit to 70%, and only charge more before a rainy period by enabling option to adjust daily car charge limit based on weather.
-   Allow top up from secondary power source (eg. grid, battery) if there is not enough solar during the day, or if required to charge during the night. Just need to set the power offset to specify the maximum power to draw from secondary power source. Also need to toggle on secondary power source if required to charge during the night.
-   Support setting minimum charge current if there is not enough solar electricity.
-   Support charging multiple cars at the same time based on power allocation weighting for each car.
-   Support skew to shift the power export/import curve left or right to achieve your minimal power import.
-   Beta feature: Use EV specific API to control a EV for charging, and/or use OCPP to control an OCPP compliant charger to charge a EV.  Only tested with [OCPP simulator](https://github.com/lewei50/iammeter-simulator) and Tesla car, hence this Blueprint might not work with your OCPP charger or EV.


**💡 Tip:** Please :star: this project if you like my work, and may be also buy me coffee!

<a href="https://www.buymeacoffee.com/flashg1" target="_blank">
  <img src="https://cdn.buymeacoffee.com/buttons/default-black.png" alt="Buy Me A Coffee" width="150px">
</a>


My setup
========

-	Home Assistant, https://www.home-assistant.io/
-	Enphase Envoy Integration configured for 30 seconds update interval, https://www.home-assistant.io/integrations/enphase_envoy
-	Tesla Custom Integration v3.20.4 (this is for people who want to control their Tesla via Tesla cloud), https://github.com/alandtse/tesla
- Tesla BLE MQTT docker v0.5.0 (this is for people who want to control their Tesla locally via Bluetooth without cloud), https://github.com/tesla-local-control/tesla_ble_mqtt_docker
- OCPP v0.84 (this is for people who want to use OCPP to control an OCPP compliant charger to charge their EV), https://github.com/lbbrhzn/ocpp
-	Tesla UMC charger, 230V, max 15A.
-	Tesla Model 3.


Installation
============

-	Set up "Main Power Net" sensor in Home Assistant (HA) config.  For example, for Enphase, sensor main_power_net expresses negative value in Watts for available power for charging car or positive value for consumed power.  For other inverter brands, adjust the formula to conform with above requirement according to your setup.
```
Settings > Devices & services > Helpers > Create helper > Template > Template a sensor >

Name: Main power net
State template: {{ states('sensor.envoy_[YourEnvoyId]_current_power_consumption')|int - states('sensor.envoy_[YourEnvoyId]_current_power_production')|int }}
Unit of measurement: W
Device class: Power
State class: Measurement
Device: Envoy [YourEnvoyId]
```

- If using OCPP charger, configure your charger to point to your HA OCPP central server, eg. ws://homeassistant.local:9000

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fflashg1%2FevSolarCharger%2Fblob%2Fmain%2Fev_solar_charger_automation.yaml)

-	Import the Blueprint automatically by clicking above, or manually copy the Blueprint file to following location and reload HA config,
\\HOMEASSISTANT\config\blueprints\automation\flashg1\ev_solar_charger_automation.yaml

-	Optionally create following helpers, or create them later for finer control, eg.
Settings > Devices & Services > Helpers > Create Helper >
1.	Toggle: MyEV set daily car charge limit
2.  Number: MyEV publish car charge limit
3.	Toggle: MyEV secondary power source (for night time charging)
4.  Number or template sensor: MyEV charger minimum current
5.	Number or template sensor: MyEV power offset
6.	Toggle: MyEV stop charging

-	Config the Blueprint automation specifying the power feedback loop sensor, charger voltage, maximum current, charge control API selection and the optional helper entities created above, ie.
Settings > Automations & Scenes > Blueprints > EV solar charger automation


How to use
==========

-	Set your car charge limit.
-	Connect charger to car.  Normal charging at constant current should begin immediately if schedule charging is disabled.  After a little while, the script will take over and manage the charging current during daylight hours.  Please see work-arounds below if automation cannot be triggered.
-	There are 2 options on how to charge the car (see below).
-	The script will stop if charger is turned off manually or automatically by car when reaching charge limit.
-	To abort charging, turn on "MyEV stop charging".  The script will take about a minute to terminate if using default values.

2 options on how to charge the car:

Option 1
--------
To charge from excess solar, just plug in the charger.  The initial charge current is 6A.  After about 1 minute it will adjust the current according to amount of excess power exported to grid.

Option 2
--------
To charge from secondary power source and solar, set amount of power offset to draw from secondary power source.  Also need to toggle on secondary power source if charging at night.


Notes
=====

Please also check out the [wiki](https://github.com/flashg1/TeslaSolarCharger/wiki) pages.

Automation cannot be triggered
------------------------------
The Tesla triggers and conditions are slow to update unless car is polled often.  Polling too often can drain the car battery.  So might have to wait a minute or two for the conditions to refresh and the triggers to work.  Please see below for possible work-arounds.

Work-arounds:
1. Run the automation manually by selecting the automation and then select "Run Actions".
2. Turn polling off, then on.
3. Press the "Force data update" button before and after plugging in the charger.

Daily car charge limit settings
-------------------------------
- If set daily car charge limit is toggled off, charge limit will be set according to the Tesla app.
- If set daily car charge limit is toggled on and charge car based on weather is disabled, charge limit will be set according to the limit configured for the day.
- If set daily car charge limit is toggled on and charge car based on weather is enabled, charge limit will be adjusted to the highest limit set within the rainy forecast period taking into account the car charge limit on bad weather setting.
- If charge car based on weather is enabled, daily car charge limit and weather provider settings must be configured.

Special note for 3-phase chargers
---------------------------------
Please see [discussion](https://github.com/flashg1/TeslaSolarCharger/issues/18) on voltage to set for charger with 3-phase power.

Charge mutiple EVs at the same time based on power allocation weighting for each car
------------------------------------------------------------------------------------
Note: This is theoretical only since I don't have 2 EVs to test this, but happy for any feedback.  To ensure power is allocated according to weighting, the "Main power net" update cycle should be the same as the script looping cycle, ie. 1 minute.

- Create power allocation weighting for each car.  For example, to create for car1,
```
Settings > Devices & services > Helpers > Create helper > Number >
Name: Car1 power allocation weight
Minimum value: 1
Maximum value: 10
```
- Create power allocation sensor for each car.  For example, to create power allocation sensor for car1 assuming we have car1 and car2,
```
Settings > Devices & services > Helpers > Create helper > Template > Template a sensor >
Name: Car1 power allocation
State template: {{ states('sensor.main_power_net')|int * states('input_number.car1_power_allocation_weight')|int / (state_attr('automation.car1_solar_charger_automation', 'current') * states('input_number.car1_power_allocation_weight')|int + state_attr('automation.car2_solar_charger_automation', 'current') * states('input_number.car2_power_allocation_weight')|int) }}
Unit of measurement: W
Device class: Power
State class: Measurement
```
- Use the power allocation sensors defined above as input to "Main power net" in each car's Blueprint.


GUI display examples
====================

Dashboard Tesla style power card
--------------------------------
https://github.com/reptilex/tesla-style-solar-power-card

```
type: custom:tesla-style-solar-power-card
name: Power Usage
show_w_not_kw: 1

# 3 flows between bubbles
grid_to_house_entity: sensor.grid_power_import
generation_to_grid_entity: sensor.grid_power_export
generation_to_house_entity: sensor.solar_power_consumption

# optional appliances with consumption and extra values
appliance1_consumption_entity: sensor.charger_power
appliance1_extra_entity: sensor.battery

# optional 3 main bubble icons for clickable entities
grid_entity: sensor.main_power_net
house_entity: sensor.envoy_[YourEnvoyId]_current_power_consumption
# Watch this for car charge limit change
house_extra_entity: number.charge_limit
generation_entity: sensor.solar_power_production

```

Dashboard EV solar charger control
----------------------------------
```
type: entities
entities:
  - entity: automation.[YourEvName]_solar_charger_automation
  - type: attribute
    entity: automation.[YourEvName]_solar_charger_automation
    attribute: current
    name: Running instance count
  - type: attribute
    entity: automation.[YourEvName]_solar_charger_automation
    attribute: last_triggered
    name: Last triggered
  - entity: input_boolean.[YourEvName]_set_daily_car_charge_limit
  - entity: input_boolean.[YourEvName]_publish_car_charge_limit
  - entity: input_boolean.[YourEvName]_secondary_power_source
  - entity: input_number.[YourEvName]_charger_minimum_current
  - entity: input_number.[YourEvName]_power_offset
  - entity: input_boolean.[YourEvName]_stop_charging
  - entity: button.wake_up
  - entity: button.force_data_update
  - entity: device_tracker.location_tracker
  - entity: binary_sensor.charger
  - entity: binary_sensor.charging
  - entity: number.charging_amps
  - entity: sensor.range
  - entity: sensor.battery
  - entity: number.charge_limit
  - entity: sensor.time_charge_complete
  - entity: lock.charge_port_latch
```
