# Home Assistant - Configuration

## Table of content

- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Packages and Integrations](https://github.com/slittorin/home-assistant-configuration#packages-and-integrations)
  - [Package - Home Assistant](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system)
  - [Package - Weather](https://github.com/slittorin/home-assistant-configuration#package---weather)
  - [Package - Nordpool](https://github.com/slittorin/home-assistant-configuration#package---nordpool)
  - [Package - SMA](https://github.com/slittorin/home-assistant-configuration#package---sma)
  - [Package - Balboa Spa](https://github.com/slittorin/home-assistant-configuration#package---balboa-spa)

## Generic information

For changes to Home Assistant configuration files:
1. Goto `Configuration` -> `Settings` -> `Server Controls` and press `Check Configuration`.
   - The output should state 'Configuration valid'. If not, change the recorder config above.
2. Try not to restart the full Home Manager through `Server Configuration`.
   - All states are lost, and sensors/data are lost that relates to states.
3. Restart only necessary part of Home Assistant at `YAML configuration reloading`.
   - Check logs.
   - It can take up to 1 minute before updates are made.

Logs:
- Any warnings or errors can be found in the file `/config/home-assistant.log`.

Styles, naming convention, and unit of measurement:
- Configuration-files/yaml:
  - Do not create more sensors than needed. Rely on the standard integration entities/attributes.
  - Follow the [Style guide](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/).
  - Utilize [Packages](https://www.home-assistant.io/docs/configuration/packages/) to bundle together services/entities in packages.
    - Utilize [Style 1](https://www.home-assistant.io/docs/configuration/devices#style-2-list-each-device-separately) convention.
  - Naming convention:
    - Entity ID: type, area, location, device, lower-case with `_` as delimiter.
    - If there are similar entities add `_unitofmeasurement`, such as `weather_sun_snapshot_elevation_below_0_degrees_seconds`.
    - When testing, add `test_` before the entity id.
      - However remember that these sensors are not written to history, so for instance history_stats will not work with `test_` entites.
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
# This file includes all the entities for the Home Assistant system.

sensor:
    # Get the MariaDB database size.
  - platform: sql
    db_url: !secret recorder_db_url
    # Scan every 10:t minute.
    scan_interval: 600
    queries:
      # We keep track of the MariaDB database size.
      - name: home_assistant_db_size
        query: 'SELECT table_schema "database", Round(Sum(data_length + index_length) / 1024 / 1024, 1) "value" FROM information_schema.tables WHERE table_schema="homeassistant" GROUP BY table_schema;'
        column: "value"
        unit_of_measurement: 'MB'


template:
  - trigger:
      - platform: time
        at: "00:00:00"
    sensor:
      # Get the number of domains, and entities within each domain.
      # https://community.home-assistant.io/t/where-can-i-find-a-list-of-domains/62654/20
      - name: test_home_assistant_domains_in_use
        state: >
          {%- for d in states | groupby('domain') %}
          {% if loop.first %}{{loop.length}} Domains:
          {% endif %}- {{ d[0] }}: {{d[0]|count}}
          {%- endfor %}
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

# We want to track different elevation of the sun, and time between the elevations.
template:
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        above: 0
    sensor:
      # Set a snapshot in seconds since 00:00 when sun is above 0 degrees.
      - name: test_weather_sun_snapshot_elevation_above_0_degrees_seconds
        unit_of_measurement: 'time'
        state: "{{ (now().hour * 3600) + (now().minute * 60) + now().second }}"
      - name: test_weather_sun_snapshot_elevation_above_0_degrees_hhmmss
        unit_of_measurement: 'time'
        state: >
          {{ "%02d"|format(now().hour) }}:{{ "%02d"|format(now().minute) }}:{{ "%02d"|format(now().second) }}
          
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        below: 0
    sensor:
      # Set a snapshot in seconds since 00:00 when sun is below 0 degrees.
      - name: test_weather_sun_snapshot_elevation_below_0_degrees_seconds
        unit_of_measurement: 'time'
        state: "{{ (now().hour * 3600) + (now().minute * 60) + now().second }}"
      - name: test_weather_sun_snapshot_elevation_below_0_degrees_hhmmss
        state: >
          {{ "%02d"|format(now().hour) }}:{{ "%02d"|format(now().minute) }}:{{ "%02d"|format(now().second) }}
      # Time in seconds when sun is above horizon.
      - name: test_weather_sun_time_above_horizon_seconds
        unit_of_measurement: 'time'
        # We cannot trust that states() gives the value set above on the same trigger, therefore we utilize the same formula.
        state: >
          {% set start = states('test_weather_sun_snapshot_elevation_above_0_degrees_seconds') %}
          {% set stop = (now().hour * 3600) + (now().minute * 60) + now().second %}
          {% set time  = stop - start %}
          {{ time }}
      - name: test_weather_sun_time_above_horizon_hhmmss
        # We cannot trust that states() gives the value set above on the same trigger, therefore we utilize the same formula.
        state: >
          {% set start = states('test_weather_sun_snapshot_elevation_above_0_degrees_seconds') %}
          {% set stop = (now().hour * 3600) + (now().minute * 60) + now().second %}
          {% set time  = stop - start %}
          {% set hours = (time/3600) | int %}
          {% set time = (time - (hours*3600)) %}
          {% set minutes = (time/60) | int %}
          {% set time = (time - (minutes*60)) %}
          {% set seconds = time | int %}
          {{ "%02d"|format(hours) }}:{{ "%02d"|format(minutes) }}:{{ "%02d"|format(seconds) }}

sensor:
  - platform: template
    sensors:
      # We want to keep track of weather wind speed in m/s, as SMHI integration gives km/h,
      weather_wind_speed_ms:
        friendly_name: "Weather wind speed"
        unit_of_measurement: 'm/s' # SMHI integration gives km/h, so we convert.
        value_template: "{{ (state_attr('weather.smhi_home', 'wind_speed') / 3.6) | round(1) }}"
        
      # We want to keep track of weather wind gust speed in m/s, as SMHI integration gives km/h,
      weather_wind_gust_speed_ms:
        friendly_name: "Weather wind gust speed"
        unit_of_measurement: 'm/s' # SMHI integration gives km/h, so we convert.
        value_template: "{{ (state_attr('weather.smhi_home', 'wind_gust_speed') / 3.6) | round(1) }}"
```

## Package - Nordpool

We want to have gather the current cost for electricity in my region.

1. Download the custom component:
   - Go to [HA Customer component - Nordpool](https://github.com/custom-components/nordpool).
   - Choose `Code` -> `Download as ZIP` to download the code.
   - Through the `File Editor` add-on, under `/config/custom_components` create the following directories:
     - `nordpool`.
     - `nordpool/download`.
   - Through the `File Editor` add-on, upload the zip-file to the `nordpool/download` directory.
   - Logon with ssh to the Home Assistant server (user is  `root`):
     - In directory `/config/custom_components/nordpool/download` run the command `unzip nordpool-master.zip`.
     - Copy the files in directory `/config/custom_components/nordpool/download/nordpool-master/custom_components/nordpool` to `/config/custom_components/nordpool`.
2. Restart the Home Assistant server under `Server management`.
   - This may take a while as the custom component is installed.
3. Through the `File Editor` add-on, create the file `/config/packages/tariff_electrical.yaml`
4. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and after `  packages:` (mind the spaces):
```
    tariff_electrical: !include packages/tariff_electrical.yaml
```
5. Through the `File Editor` add-on, edit the file `/config/packages/tariff_electrical.yaml` and add:
```
# This file includes all the items for the electrical tariffs.

sensor:
  # We get all the data from Nordpool, for my region and currency,
  - platform: nordpool
    region: "SE3"
    VAT: True
    currency: "EUR"
    precision: 3
    price_type: kWh
```
6. To get the custom component running restart the Home Assistant server under `Server management`.
   - Verify any warnings or errors in the log-file.

## Package - SMA

We want to have gather information from our SMA Solar inverter (that also gets information from Home Manager 2.0), to gather information about load, loads and yields.

1. Add integration `SMA Solar` with the following parameters:
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
     - As any grid production goes towards Nordpool hourly-rate we add `sensor.nordpool_kwh_se3_eur_3_10_025` with  `Use an entity with current rate`.
   - Press `Next`.
   - Choose `Add solar production` (Configure solar panels):
     - Choose `total_yield`.
     - Choose `Don't forecast production`, as we at this point do not want any forecasting.
   - Press `Next`.
   - Since we do not have a battery system, Press `Next`.
   - Since we do not have a gas source, Press `Next`.

## Package - Balboa Spa

We want to gather information about our Jacuzzi that has a Balboa Spa WiFi Module installed.

1. Add integration `Balboa Spa client` with the following parameters:
   - `Host`: IP of the SMA Inverter (that we have set fixed IP on).
   - Once connected, set `Area` to the area where the SMA Inverter is located.
2. Through the `File Editor` add-on, create the file `/config/packages/balboa_spa.yaml`
3. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and after `  packages:` (mind the spaces):
```
    balboa_spa: !include packages/balboa_spa.yaml
```
4. Through the `File Editor` add-on, edit the file `/config/packages/balboa_spa.yaml` and add:
```
# This file includes all the items for the Balboa Spa Client add-on.

sensor:
  # We want to keep track of the time that the heater has been running (depends of course on polling).
  - platform: history_stats
    name: spa_heater_running_time
    entity_id: climate.nbp6013h_climate
    state: "heat"
    type: time
    start: "{{ now().replace(hour=0, minute=0, second=0) }}"
    end: "{{ now() }}"
```

