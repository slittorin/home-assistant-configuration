# Home Assistant - Configuration

## Table of content

- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Packages and Integrations](https://github.com/slittorin/home-assistant-configuration#packages-and-integrations)
  - [Package - Home Assistant](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system)
  - [Package - Weather](https://github.com/slittorin/home-assistant-configuration#package---weather)
  - [Package - Nordpool](https://github.com/slittorin/home-assistant-configuration#package---nordpool)
  - [Package - SMA](https://github.com/slittorin/home-assistant-configuration#package---sma)

## Generic information

For all changes to Home Assistant configuration files, you usually need to restart:
-  Goto `Configuration` -> `Settings` -> `Server Controls` and press `Check Configuration`.
   - The output should state 'Configuration valid'. If not, change the recorder config above.
   - On the same page press `Restart` under `Server management`.
- Any warnings or errors can be found in the file `/config/home-assistant.log`.
- Configuration-files/yaml:
  - Do not create more sensors than needed. Rely on the standard integration entities/attributes.
  - Follow the [Style guide](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/).
  - Utilize [Packages](https://www.home-assistant.io/docs/configuration/packages/) to bundle together services/entities in packages.
    - Utilize [Style 1](https://www.home-assistant.io/docs/configuration/devices#style-2-list-each-device-separately) convention.
  - Naming convention:
    - Entity ID: type, area, location, device, lower-case with `_` as delimiter.
    - If there are similar entities add `_unitofmeasurement`, such as `weather_sun_snapshot_elevation_below_0_degrees_seconds`.
    - When testing, add `test_` before the entity id.
  - Verify and set unit of measurement for all entities/sensors.
    - Standard units is found [here](https://github.com/home-assistant/core/blob/dev/homeassistant/const.py).

## Governing principles

- Setup HA with [Home Assistance setup](https://github.com/slittorin/home-assistant-setup).

# Packages and Integrations

## Package - Home Assistant system

We want to keep track of the following for the HA system:
- Database sizes.
- Number of objects managed by Home Assistant.

Perform the following:

1. Through the `File Editor` add-on, create the file `/config/packages/ha_system.yaml`
2. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and after `  packages:` (mind the spaces):
```
    ha_system: !include packages/ha_system.yaml
```
3. Through the `File Editor` add-on, edit the file `/config/packages/ha_system.yaml` and add:
```
# This file includes all the items for the Home Assistant system.

sensor:
  - platform: sql
    db_url: !secret recorder_db_url
    scan_interval: 600 # Scan every 10:t minute.
    queries:
      # We keep track of the MariaDB database size.
      - name: home_assistant_db_size
        query: 'SELECT table_schema "database", Round(Sum(data_length + index_length) / 1024 / 1024, 1) "value" FROM information_schema.tables WHERE table_schema="homeassistant" GROUP BY table_schema;'
        column: "value"
        unit_of_measurement: 'MB'
```

## Package - Weather

We want to have a more accurate weather integration for Sweden than the built in, so we utilize SMHI.\

We want to keep track of the following for the HA system:
- Sensor data from the integration.
- Sun states, including different time/elevation over the horizon to be able to utilize that for trending for the solar panels (SMA).

Perform the following:

1. Add and enable integration `SMHI`.
   - Utilize the GPS coordinates set in setup/onboarding stage.
3. Disable the default integration (in my setup, 'met.no').
4. Delete thereafter the default integration.
   - If we this at initial setup of the HA, we do not loose any valid data.
   - If you want to keep historical data, do not delete this integration.
5. Through the `File Editor` add-on, create the file `/config/packages/weather.yaml`
6. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and after `  packages:` (mind the spaces):
```
    ha_system: !include packages/weather.yaml
```
7. Through the `File Editor` add-on, edit the file `/config/packages/weather.yaml` and add:
```
# This file includes all the items for the Weather integration.

sensor:
  - platform: template
    sensors:
      # We want to keep track of the elevation of the sun.
      weather_sun_elevation:
        friendly_name: "Weather sun elevation"
        unit_of_measurement: 'degrees'
        value_template: "{{ state_attr('sun.sun', 'elevation') }}"
        
      # We want to keep track of the azimuth of the sun.
      weather_sun_azimuth:
        friendly_name: "Weather sun azimuth"
        unit_of_measurement: 'degrees'
        value_template: "{{ state_attr('sun.sun', 'azimuth') }}"
        
      # We want to keep track of weather temperature.
      weather_temperature:
        friendly_name: "Weather temperature"
        unit_of_measurement: '°C'
        value_template: "{{ state_attr('weather.smhi_home', 'temperature') }}"
        
      # We want to keep track of weather humidity.
      weather_humidity:
        friendly_name: "Weather humidity"
        unit_of_measurement: '%'
        value_template: "{{ state_attr('weather.smhi_home', 'humidity') }}"
        
      # We want to keep track of weather wind speed.
      weather_wind_speed_kmh:
        friendly_name: "Weather wind speed"
        unit_of_measurement: 'km/h' # SMHI integration gives km/h.
        value_template: "{{ state_attr('weather.smhi_home', 'wind_speed') }}"
      weather_wind_speed_ms:
        friendly_name: "Weather wind speed"
        unit_of_measurement: 'm/s' # SMHI integration gives km/h, so we convert.
        value_template: "{{ (state_attr('weather.smhi_home', 'wind_speed') / 3.6) | round(1) }}"
        
      # We want to keep track of weather wind bearing.
      weather_wind_bearing:
        friendly_name: "Weather wind bearing"
        unit_of_measurement: 'degrees'
        value_template: "{{ state_attr('weather.smhi_home', 'wind_bearing') }}"
        
      # We want to keep track of weather wind gust speed.
      weather_wind_gust_speed_kmh:
        friendly_name: "Weather wind gust speed"
        unit_of_measurement: 'km/h' # SMHI integration gives km/h.
        value_template: "{{ state_attr('weather.smhi_home', 'wind_gust_speed') }}"
      weather_wind_gust_speed_ms:
        friendly_name: "Weather wind gust speed"
        unit_of_measurement: 'm/s' # SMHI integration gives km/h, so we convert.
        value_template: "{{ (state_attr('weather.smhi_home', 'wind_gust_speed') / 3.6) | round(1) }}"
        
      # We want to keep track of weather pressure.
      weather_pressure:
        friendly_name: "Weather pressure"
        unit_of_measurement: 'hPa' # SMHI integration gives hectopascal.
        value_template: "{{ state_attr('weather.smhi_home', 'pressure') }}"
        
      # We want to keep track of weather visibility.
      weather_visibility:
        friendly_name: "Weather visibility"
        unit_of_measurement: 'km' # SMHI integration gives km
        value_template: "{{ state_attr('weather.smhi_home', 'visibility') }}"
        
      # We want to keep track of weather cloudiness.
      weather_cloudiness:
        friendly_name: "Weather cloudiness"
        unit_of_measurement: '%' # SMHI integration gives 0-100 (%).
        value_template: "{{ state_attr('weather.smhi_home', 'cloudiness') }}"
        
      # We want to keep track of weather thunder probability.
      weather_thunder_probability:
        friendly_name: "Weather thunder probability"
        unit_of_measurement: '%' # SMHI integration gives 0-100 (%).
        value_template: "{{ state_attr('weather.smhi_home', 'thunder_probability') }}"
```

## Package - Nordpool

We want to have gather the current cost for electricity in my region.

1. Go to [HA Customer component - Nordpool](https://github.com/slittorin/home-assistant-configuration#package---nordpool`.
2. Choose `Code` -> `Download as ZIP`.
3. Through the `File Editor` add-on, edit create the directory `nordpool` under `/config/custom_components`.
4. Through the `File Editor` add-on, upload the zip-file to the `nordpool` directory.
5. Logon with ssh to the Home Assistant server (user is  `root`):
   - In directory `/config/custom_components/nordpool` run the command `unzip nordpool-master.zip`.

## Package - SMA

We want to have gather information from our SMA Solar inverter and Home Manager 2.0.

1. Add integration 'SMA Solar' with the following parameters:
   - `Host`: IP of the SMA Inverter (that we have set fixed IP on).
   - Check `Uses an SSL certificate`.
   - Remove check for `Verify SSL certificate`.
   - `Group`: Set to `user`.
   - `Password`: Set to the user-password for the SMA Inverter.
   - Once connected, set `Area` to the area where the SMA Inverter is located.
2. Go to 'Energy':
   - Choose `Add consumption` (Configure grid consumption):
     - Choose `metering_total_absorbed`.
     - Since I have a static cost, I add this cost with `Use as static price`.
   - Choose `Add return` (Configure grid production):
     - Choose `metering_total_yield`.
   - Press `Next`.
   - Choose `Add solar production` (Configure solar panels):
     - Choose `total_yield`.
     - Choose `Don't forecast production`, as we at this point do not want any forecasting.
   - Press `Next`.
   - Since we do not have a battery system, Press `Next`.
   - Since we do not have a gas source, Press `Next`.
