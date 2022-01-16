# Home Assistant - Configuration

## Table of content

- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Packages and Integrations](https://github.com/slittorin/home-assistant-configuration#packages-and-integrations)
  - [Packages - Home Assistant](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system)
  - [Package - Weather](https://github.com/slittorin/home-assistant-configuration#package---weather)
  - [Package - Unifi](https://github.com/slittorin/home-assistant-configuration#package---unifi)

## Generic information

For all changes to Home Assistant configuration files, you usually need to restart:
-  Goto `Configuration` -> `Settings` -> `Server Controls` and press `Check Configuration`.
   - The output should state 'Configuration valid'. If not, change the recorder config above.
   - On the same page press `Restart` under `Server management`.
- Any warnings or errors can be found in the file `/config/home-assistant.log`.
- Configuration-files/yaml:
  - Follow the [Style guide](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/).
  - Utilize [Packages](https://www.home-assistant.io/docs/configuration/packages/) to bundle together services/entities in packages.
    - Utilize [Style 1](https://www.home-assistant.io/docs/configuration/devices#style-2-list-each-device-separately) convention.
  - Naming convention:
    - Entity ID: type, area, location, device, lower-case with `_` as delimiter.
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
# This file includes all the items for Home Assistant system.

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
```



5. Through the `File Editor` add-on, edit the file `/config/sensors.yaml` and add after `  - platform: template` (if there is not template-row, add this row (mind the spaces)):
```
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
        value_template: "{{ state_attr('weather.smhi_home', 'wind_speed') / 3.6 }}"
        
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
        value_template: "{{ state_attr('weather.smhi_home', 'wind_gust_speed') / 3.6 }}"
        
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

## Package - Unifi

We want to have gather information from our Unifi network.

1. Add a read-only local user to the Unify controller.
2. Add the integration `Unifi`.
   - Host is the IP of the Cloud Key.
   - User and password for the read-only local user created above.
