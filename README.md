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
-   Configurable 7 days charge limit schedule.  Default is to use existing charge limit already set in car.
-   Support just-in-time schedule charging to required charge limit using solar and grid if charge completion time is set for the day.
-   Automatically charge more today if today has no charge completion time and next 3 days have higher charge limit.
-   Automatically adjust to the highest charge limit set within a rainy forecast period.  The highest charge limit is selected from the 7 days charge limit settings that are within the forecast period taking into account the charge limit on bad weather setting.  The objective is to charge more before a rainy period.  Default disabled.
-   Might be possible to prolong car battery life by setting daily charge limit to 60%, and only charge more before a rainy period by enabling option to adjust daily car charge limit based on weather.
-   Allow manual top up from secondary power source (eg. grid, battery) if there is not enough solar during the day, or if required to charge during the night. Just need to set the power offset to specify the maximum power to draw from secondary power source. Also need to toggle on secondary power source if required to charge during the night.
-   Support manually setting or automatic programming of minimum charge current according to your requirement.
-   Support charging multiple cars at the same time based on power allocation weighting for each car.
-   Support skew to shift the power export/import curve left or right to achieve your minimal power import.
-   Configurable return codes for comparison with connect trigger states, connected states and charging states returned by your EV or charger specific API. These states are used to determine the stages of the charging process.
-   Use EV specific API to control a EV for charging, and/or use OCPP to control an OCPP compliant charger to charge a EV. Only tested with [OCPP simulator](https://github.com/lewei50/iammeter-simulator) and Tesla car. OCPP and Tesla Fleet API support in beta testing phase.


**💡 Tip:** Please :star: this project if you find it useful, and may be also buy me a coffee!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/flashg1)


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

-	Set up "Main Power Net" sensor in Home Assistant (HA) config.  For example, for Enphase, sensor main_power_net expresses negative value in Watts for available power for charging or positive value for consumed power.  For other inverter brands, adjust the formula to conform with above requirement according to your setup.
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

-	Create following non-optional helpers, eg.
Settings > Devices & Services > Helpers > Create Helper >
1.  Number or template sensor: MyEV charger effective voltage (for single-phase or 3-phase system)
1.  Date and time: MyEV next charge time trigger
1.  Number or template sensor: MyEV battery maximum charge speed

-	Optionally create following helpers, or create them later for finer control, eg.
Settings > Devices & Services > Helpers > Create Helper >
1.  Number or template sensor: MyEV charger minimum current
1.	Number or template sensor: MyEV power offset
1.	Toggle: MyEV secondary power source (for night time charging)
1.	Toggle: MyEV set daily charge limit
1.	Toggle: MyEV adjust charge limit based on weather
1.	Number: MyEV charge limit Monday .. Sunday
1.	Time: MyEV charge completion time Monday .. Sunday
1.  Number: MyEV publish charge limit (only required if setting charge limit is not supported by car specific API)
1.	Toggle: MyEV stop charging

-	Config the Blueprint automation specifying the power feedback loop sensor, charger effective voltage, maximum current, next charge time trigger, maximum charge speed, charge control API selection and the optional helper entities created above, ie.
Settings > Automations & Scenes > Blueprints > EV solar charger automation


How to use
==========

-	Set your car charge limit.
-	Connect charger to car.  Normal charging at constant current should begin immediately if schedule charging is disabled.  After a little while, the script will take over and manage the charging current during daylight hours.  Please see [wiki](https://github.com/flashg1/evSolarCharger/wiki/User-guide#automation-cannot-be-triggered) if automation cannot be triggered.
-	There are 2 options on how to charge the car (see below).
-	The script will stop if charger is turned off manually or automatically by car when reaching charge limit.
-	To abort charging, turn on "MyEV stop charging".  The script will take about a minute to terminate if using default values.

2 options on how to charge the car:

Option 1
--------
To charge from excess solar, just plug in the charger.  The initial charge current is 6A.  After about 1 minute it will adjust the current according to amount of excess power exported to grid.

Option 2
--------
To charge from secondary power source and solar, set minimum charge current or power offset to draw from secondary power source.  Also need to toggle on secondary power source if charging at night.

Please also check out the [wiki](https://github.com/flashg1/evSolarCharger/wiki) pages.
