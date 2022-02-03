# Home Assistant - Configuration

## Table of content

- [Goal](https://github.com/slittorin/home-assistant-configuration#goal)
- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Packages and Integrations](https://github.com/slittorin/home-assistant-configuration#packages-and-integrations)
  - [Github Push](https://github.com/slittorin/home-assistant-configuration#github-push)
  - [Resource - Lovelace Card Mod](https://github.com/slittorin/home-assistant-configuration#resource---lovelace-card-mod)
  - [Resource - Apex Charts Card](https://github.com/slittorin/home-assistant-configuration#resource---apex-charts-card)
  - [Package - Home Assistant](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system)
  - [Package - Weather](https://github.com/slittorin/home-assistant-configuration#package---weather)
  - [Package - Nordpool](https://github.com/slittorin/home-assistant-configuration#package---nordpool)
  - [Package - SMA](https://github.com/slittorin/home-assistant-configuration#package---sma)
  - [Package - Balboa Spa](https://github.com/slittorin/home-assistant-configuration#package---balboa-spa)

## Goal

The goal of my Home Assistant installation is to primarily measure and get control of the major cost for my house, electricity.
Secondarily I would like to be able to control and perform automation activities.

## Generic information

#### For changes to Home Assistant configuration files:
1. Goto `Configuration` -> `Settings` -> `Server Controls` and press `Check Configuration`.
   - The output should state 'Configuration valid'. If not, isolate the error.
2. Try not to restart the full Home Manager through `Server Configuration`.
   - All states are lost, and sensors/data are lost that relates to states.
3. Restart only necessary part of Home Assistant at `YAML configuration reloading`.
4. Check logs to verify if there are warnings or errors.
   - Any warnings or errors can be found in the file `/config/home-assistant.log`.
   - It can take up to 1-2 minutes before updates are made.

#### Styles, naming convention, device and state class, and unit of measurement:
- Configuration-files/yaml:
  - Do not create more sensors than needed. Rely on the standard integration entities/attributes.
    - Since states (and therefore also InfluxDB states) do not track attribute changes, make sure to create sensors for the attributes you want to track.
      - And of course, where required, create new sensors if attributes cannot be utilized by templates, triggers or other.
  - Use [Modern configuration format](https://www.home-assistant.io/integrations/template/#configuration-variables).
  - Follow the [Style guide](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/).
  - Naming convention for `name:` and `unique_id:`of Sensor/Entity ID (and where required name) shall follow the naming standard:
    - `Device/Type` `area/attribute` `(if required location/zone)` `state/measure` `device` `unit of measurement`.
    - This will render unique id
      - balboa_spa_heater_on
      - balboa_spa_heater_running_time
      - balboa_spa_heater_consumption
      - balboa_spa_heater_consumption_daily
    - When testing, add `test_` before the entity id.
      - However remember that these sensors are not written to history, as we have excluded these sensors in the configuration file, so for instance history_stats will not work with `test_` entites.
  - Utilize [Packages](https://www.home-assistant.io/docs/configuration/packages/) to bundle together services/entities in packages.
  - Make sure that the new format to manage functions is utilized from [2021.10.x](https://community.home-assistant.io/t/updating-templates-with-the-new-default-values-in-2021-10-x/346198).
  - To allow Home Assistant to correctly identify and utilize sensors/data, always utilize:
    - When utilizing customized entities, utilize `device_class`. Valid types can be found under each platform [here](https://www.home-assistant.io/docs/configuration/customizing-devices/#device-class).
    - Utilize `state_class` where valid when defining sensor/template sensors. Valid types can be found [here](https://developers.home-assistant.io/docs/core/entity/sensor/#available-state-classes).
      - Note that legacy configuration format (see modern above), do not support `state_class`.
    - Utilize `unit of measurement` for all entities/sensors. Standard units is found [here](https://github.com/home-assistant/core/blob/dev/homeassistant/const.py).
    - For power/consumption related sensors one may also look to add integration [Powercalc](https://github.com/bramstroker/homeassistant-powercalc), specifically for binary sensors.

# Governing principles

- Setup HA with [Home Assistance setup](https://github.com/slittorin/home-assistant-setup).

# Resources, Packages and Integrations

## Github Push

We want to utilize Github Push instead of Pull as the original files are to reside on my HA-device.\
Therefore we do not utilize the standard Github Pull integration.\
Inspiration from the [community](https://community.home-assistant.io/t/sharing-your-configuration-on-github/195144).

Note: HASS.io already has git installed, so no need to install git.

Pre-requisities: That repository `home-assistant-config` is created in Github.

Perform the following:

1. If not already done, create a Github personal access token:
   - According [instructions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)-
   - Set preferably an Expiration date (and keep note of the date and update the token accordingly).
   - Set the following scopes:
     - `repo`.
     - `gists`.
   - Copy the token.
2. Through the `File Editor` add-on, add the following to `.env` in `/config/` (where TOKEN is the copied token, ensure that usename and repository is correct):
   `GITHUB_CONNECT_STRING=https://TOKEN@github.com/slittorin/home-assistant-config`
3. Through the `File Editor` add-on, create `.gitignore` in `/config/` with the following content:
```git config
# .gitignore for Home Assistant.

# An * ensures that everything will be ignored.
*

# Whitelisted files/folders, these will not be ignored.
!*.yaml
!.gitignore
!*.md
!*.sh

# Ignore folders.
.storage
.cloud
.google.token
logs

# Ignore .env that may store sensitive variables.
.env

# Ensure these YAML files are ignored, otherwise your secret data/credentials will leak.
ip_bans.yaml
secrets.yaml
```
4. Through the 'SSH & Web terminal' run the following in the `/config` directory (change your email and name to match your Github account, and ensure that all directories you want to add to Github are present):
   You will be asked to provide a comment at commit.
```bash
git init
git config user.email "you@example.com"
git config user.name "Your Name"
```
5. Through the 'SSH & Web terminal' run the following in the `/config` directory (where TOKEN is the copied token, ensure that usename and repository is correct):
   Note that git push can give error, that is nothing to trouble shoot.
```bash
git remote add origin https://TOKEN@github.com/slittorin/home-assistant-config
git push -u origin master
```
6. Through the `File Editor` add-on, add the file `/config/scripts/github_push.sh` and add:
```bash
#!/bin/bash
#
# Purpose:
# Push configuration to github.
#
# Usage:
# ./github_push.sh COMMENT
#
# COMMENT is the comment to add to the push-commit.
# If empty, the default comment will be: "Minor change."

# Load environment variables (mainly secrets).
if [ -f "/config/.env" ]; then
    export $(cat "/config/.env" | grep -v '#' | sed 's/\r$//' | awk '/=/ {print $1}' )
fi

# Variables:
base_dir="/config/scripts"
log_dir="/config/logs"
config_dir="/config"
logfile="${log_dir}/github_push.log"

_initialize() {
    touch "${logfile}"

    echo ""
    echo "$(date +%Y%m%d_%H%M%S): Starting Github push."
    
    # Check input.
    if [ -z "$1" ]
    then
        comment="Minor change."
        echo "$(date +%Y%m%d_%H%M%S): No input given, setting comment to default."
    else
        comment="$1"
    fi
}

_github_push() {
    cd ${config_dir}
    
    # Add all in config dor.
    git add .

    # Loop through all directories and add to git.
    for dir in */ ; do
        add_dir=1
        
        if [ ${dir} = "logs/" ] # Do not add logs-directory.
        then
           add_dir=0
        fi
        
        if [ ${add_dir} -eq 1 ]
        then
            echo "$(date +%Y%m%d_%H%M%S): Added directory: ${dir}"
            git add -f "${dir}"
        else
            echo "$(date +%Y%m%d_%H%M%S): Did not add directory: ${dir}"
        fi
    done

    exit_code=0
    status_error=""

    git status
    if [ $? -ne 0 ] 
    then
        exit_code=1
        status_error+=" status"
    fi

    git commit -m "${comment}"
    if [ $? -ne 0 ] 
    then
        exit_code=1
        status_error+=" commit"
    fi

    git push origin master
    if [ $? -ne 0 ] 
    then
        exit_code=1
        status_error+=" push"
    fi

    # Check if error occured with git commands.
    if [ ${exit_code} -eq 0 ] 
    then
        status=" No error."
    else
        status=" Error in: git${status_error}."
    fi
}

_finalize() {
    echo "$(date +%Y%m%d_%H%M%S): Git push performed:${status}"
    exit ${exit_code}
}

# Main
_initialize >> "${logfile}" 2>&1
_github_push >> "${logfile}" 2>&1
_finalize >> "${logfile}" 2>&1
```
7. Through the 'SSH & Web terminal' run the following in the `/config/script` directory:
   - `chmd ug+x github_push.sh`.
   - `./github_push.sh`-
8. Check the log-file `/config/logs/github_push.log`.
   - Isolate if there are errors, and if needed isolate the problem.
9. Go to the github repositor and check that all files has been properly pushed.
   - Isolate if there are errors, and if needed isolate the problem.

## Resource - Lovelace Card Mod

We want to tweak lovelace with CSS styles to various elements of the Home Assistant frontend.

Perform the following:

1. Download the latest `card-mod.js` file from [Github Lovelace Card Mod](https://github.com/thomasloven/lovelace-card-mod)
2. Through the `File Editor` add-on, save `card-mod.js` in `/config/www`.
3. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and add:
```yaml
frontend:
  extra_module_url:
    - /local/card-mod.js
```
4. Go to `Configiuration` -> `Dashboards` -> `Resources`:
   - Press `Add resource`:
     - For `URL` type `/local/card-mod.js`.
     - For `Resource type` choose `Javascript Module`.
6. Check configuration and restart Home Assistant.
7. Check logs for errors.
   - If errors, find the problem.

## Resource - Apex Charts Card

We want to be able to add more customized graph cards.

Perform the following:

1. Download the latest `apexcharts-card.js` file from [Github Apex Charts](https://github.com/RomRider/apexcharts-card)
2. Through the `File Editor` add-on, save `apexcharts-card.js` in `/config/www`.
4. Go to `Configiuration` -> `Dashboards` -> `Resources`:
   - Press `Add resource`:
     - For `URL` type `/local/apexcharts-card.js`.
     - For `Resource type` choose `Javascript Module`.
6. Check configuration and restart Home Assistant.
7. Check logs for errors.
   - If errors, find the problem.

## Package - Home Assistant system

We want to keep track of the following for the HA system:
- Database sizes.
- Number of objects managed by Home Assistant.

Perform the following:

1. Through the `File Editor` add-on, create the file `/config/packages/ha_system.yaml`
2. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and after `  packages:` (mind the spaces):
```yaml
    ha_system: !include packages/ha_system.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/ha_system.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/ha_system.yaml) and add:
   - Sensor for MariaDB size.
   - Triggers for keeping track of domain-entities.

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
```yaml
    ha_system: !include packages/weather.yaml
```
7. Through the `File Editor` add-on, edit the file `/config/packages/weather.yaml` and add:
```yaml
# This file includes all the items for the Weather integration.

# Logic:
# - For historical purpuse we track:
#   - The sun over the horizon, time in readable format and in seconds.
#   - Time over the horizon when it reaches my solar panels (approximatelly of course):
#     - 4 degrees above the horizon at sunrise.
#     - 6 degrees above the horizon at sunset.

# Modern format.
# If required, utilize name, and set unique_id.
template:
  # Set a snapshot in seconds and readable format since 00:00 when sun is above 0 degrees.
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        above: 0
      - platform: state
        entity_id: sun.sun
        attribute: rising
        to: true
    sensor:
      - name: test_weather_sun_snapshot_elevation_above_0_degrees_seconds
        unit_of_measurement: 'time'
        state_class: measurement
        state: "{{ (now().hour * 3600) + (now().minute * 60) + now().second }}"
      - name: test_weather_sun_snapshot_elevation_above_0_degrees_hhmmss
        unit_of_measurement: 'time'
        state_class: measurement
        state: >
          {{ "%02d"|format(now().hour) }}:{{ "%02d"|format(now().minute) }}:{{ "%02d"|format(now().second) }}

  # Set a snapshot in seconds and readable format since 00:00 when sun is above 4 degrees, for when the sun reaches the solar panels.
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        above: 4
      - platform: state
        entity_id: sun.sun
        attribute: rising
        to: true
    sensor:
      - name: test_weather_sun_snapshot_elevation_reaches_solarpanels_seconds
        unit_of_measurement: 'time'
        state_class: measurement
        state: "{{ (now().hour * 3600) + (now().minute * 60) + now().second }}"
      - name: test_weather_sun_snapshot_elevation_reaches_solarpanels_hhmmss
        unit_of_measurement: 'time'
        state_class: measurement
        state: >
          {{ "%02d"|format(now().hour) }}:{{ "%02d"|format(now().minute) }}:{{ "%02d"|format(now().second) }}

  # Set a snapshot in seconds and readable format since 00:00 when sun is below 0 degrees.
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        below: 0
      - platform: state
        entity_id: sun.sun
        attribute: rising
        to: false
    sensor:
      - name: test_weather_sun_snapshot_elevation_below_0_degrees_seconds
        unit_of_measurement: 'time'
        state_class: measurement
        state: "{{ (now().hour * 3600) + (now().minute * 60) + now().second }}"
      - name: test_weather_sun_snapshot_elevation_below_0_degrees_hhmmss
        state: >
          {{ "%02d"|format(now().hour) }}:{{ "%02d"|format(now().minute) }}:{{ "%02d"|format(now().second) }}
          
  # Set time in seconds and readable format when sun is over horizon.
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        below: 0
      - platform: state
        entity_id: sun.sun
        attribute: rising
        to: false
    sensor:
      - name: test_weather_sun_time_above_horizon_seconds
        unit_of_measurement: 'time'
        state_class: measurement
        state: >
          {% if (states('sensor.test_weather_sun_snapshot_elevation_above_0_degrees_seconds') == "unknown") %}
          {%   set start = int(0) %}
          {% else %}
          {%   set start = int(states('sensor.test_weather_sun_snapshot_elevation_above_0_degrees_seconds')) %}
          {% endif %}
          {% set stop = (now().hour * 3600) + (now().minute * 60) + now().second %}
          {% set time = (stop - start) %}
          {{ int(time) }}
      - name: test_weather_sun_time_above_horizon_hhmmss
        state_class: measurement
        state: >
          {% if (states('sensor.test_weather_sun_snapshot_elevation_above_0_degrees_seconds') == "unknown") %}
          {%   set start = int(0) %}
          {% else %}
          {%   set start = int(states('sensor.test_weather_sun_snapshot_elevation_above_0_degrees_seconds')) %}
          {% endif %}
          {% set stop = int((now().hour * 3600) + (now().minute * 60) + now().second) %}
          {% set time  = (stop - start) %}
          {% set hours = int(time/3600) %}
          {% set time = (time - (hours*3600)) %}
          {% set minutes = int(time/60) %}
          {% set time = (time - (minutes*60)) %}
          {% set seconds = time %}
          {{ "%02d"|format(hours) }}:{{ "%02d"|format(minutes) }}:{{ "%02d"|format(seconds) }}

  # Set a snapshot in seconds and readable format since 00:00 when sun is below 6 degrees.
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        below: 6
      - platform: state
        entity_id: sun.sun
        attribute: rising
        to: false
    sensor:
      - name: test_weather_sun_snapshot_elevation_leaves_solarpanels_seconds
        unit_of_measurement: 'time'
        state_class: measurement
        state: "{{ (now().hour * 3600) + (now().minute * 60) + now().second }}"
      - name: test_weather_sun_snapshot_elevation_leaves_solarpanels_hhmmss
        state: >
          {{ "%02d"|format(now().hour) }}:{{ "%02d"|format(now().minute) }}:{{ "%02d"|format(now().second) }}

  # Set time in seconds and readable format when sun can reach the solar panels
  - trigger:
      - platform: numeric_state
        entity_id: sun.sun
        attribute: elevation
        below: 0
      - platform: state
        entity_id: sun.sun
        attribute: rising
        to: false
    sensor:
      - name: test_weather_sun_time_reaches_solarpanels_seconds
        unit_of_measurement: 'time'
        state_class: measurement
        state: >
          {% if (states('sensor.test_weather_sun_snapshot_elevation_reaches_solarpanels_seconds') == "unknown") %}
          {%   set start = int(0) %}
          {% else %}
          {%   set start = int(states('sensor.test_weather_sun_snapshot_elevation_reaches_solarpanels_seconds')) %}
          {% endif %}
          {% set stop = (now().hour * 3600) + (now().minute * 60) + now().second %}
          {% set time = (stop - start) %}
          {{ int(time) }}
      - name: test_weather_sun_time_reaches_solarpanels_hhmmss
        state_class: measurement
        state: >
          {% if (states('sensor.test_weather_sun_snapshot_elevation_reaches_solarpanels_seconds') == "unknown") %}
          {%   set start = int(0) %}
          {% else %}
          {%   set start = int(states('sensor.test_weather_sun_snapshot_elevation_reaches_solarpanels_seconds')) %}
          {% endif %}
          {% set stop = int((now().hour * 3600) + (now().minute * 60) + now().second) %}
          {% set time  = (stop - start) %}
          {% set hours = int(time/3600) %}
          {% set time = (time - (hours*3600)) %}
          {% set minutes = int(time/60) %}
          {% set time = (time - (minutes*60)) %}
          {% set seconds = time %}
          {{ "%02d"|format(hours) }}:{{ "%02d"|format(minutes) }}:{{ "%02d"|format(seconds) }}

  - sensor:
      # We want to keep track of weather wind speed in m/s, as SMHI integration gives km/h.
      - name: Weather wind speed ms
        unique_id: weather_wind_speed_ms
        unit_of_measurement: 'm/s' # SMHI integration gives km/h, so we convert.
        state_class: measurement
        state: "{{ (state_attr('weather.smhi_home', 'wind_speed') / 3.6) | round(1) }}"
        
      # We want to keep track of weather wind gust speed in m/s, as SMHI integration gives km/h.
      - name: Weather wind gust speed ms
        unique_id: weather_wind_gust_speed_ms
        unit_of_measurement: 'm/s' # SMHI integration gives km/h, so we convert.
        state_class: measurement
        state: "{{ (state_attr('weather.smhi_home', 'wind_gust_speed') / 3.6) | round(1) }}"
        
      # We want to measure the chill-effect of wind speed on temperature
      # https://www.smhi.se/kunskapsbanken/meteorologi/vind/vindens-kyleffekt-1.259
      - name: Weather wind feels like
        unique_id: weather_wind_feels_like
        unit_of_measurement: '°C'
        state_class: measurement
        state: >
          {% set t = float(state_attr('weather.smhi_home', 'temperature')) %}
          {% set v = float(state_attr('weather.smhi_home', 'wind_speed') / 3.6) %}
          {% set t_calc = v %}
          {% if (((v > 2) and (v < 40)) and ((t > -40) and (t < 10))) %}
          {%   set t1 = 0.6215 * t %}
          {%   set t2 = -13.956 * (v**0.16) %}
          {%   set t3 = 0.48669 * t * (v**0.16) %}
          {%   set t_calc = 13.12 + t1 + t2 + t3 %}
          {% endif %}
          {{ t_calc | round(1) }}
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
```yaml
    tariff_electrical: !include packages/tariff_electrical.yaml
```
5. Through the `File Editor` add-on, edit the file `/config/packages/tariff_electrical.yaml` and add:
```yaml
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

See also errors that can occur with this Integration: [Incorrect SMA Energy data](https://github.com/slittorin/home-assistant-maintenance#incorrect-sma-energy-data)

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
```yaml
# This file includes all the items for the Balboa Spa Client add-on.

sensor:
# This file includes all the items for the Balboa Spa Client add-on.

# The following are the consumptions for Denform Montana:
# - Heater: 3 kW.
# - Circulation pump: 0.2 kW + 0.1 kW for the ozone-device (that takes 0.2 kW, but we assume that the ozone-device is only running 50% of the time).
#
# Logic:
# - We utilize history_stat to increase time running over each hour, when binary sensor is true/on.
# - Based on increase time, we create energy sensor that with state_class 'total_increasing' to allow HA to capture energy correct.

# History stats is kept in the legacy format.
# We keep the names to 'entity_id' format.
# ---------------------------------------------------------
sensor:
  # We want to keep track of the time that the circulation pump has been running each hour (depends of course on polling).
  # We can utilize the binary sensor from the Balboa Spa Client integration.
  - platform: history_stats
    name: balboa_spa_circulationpump_running_time_daily
    entity_id: binary_sensor.nbp6013h_circ_pump
    state: 'on'
    type: time
    start: "{{ now().replace(minute=0, second=0) }}"
    end: "{{ now() }}"

   # We want to keep track of the time that the heater has been running each hour (depends of course on polling).
   # Since we cannot track attribute-state in history_stats, we need to utilize the binary template-sensor created below.
  - platform: history_stats
    name: balboa_spa_heater_running_time_daily
    entity_id: binary_sensor.balboa_spa_heater_on
    state: 'on'
    type: time
    start: "{{ now().replace(minute=0, second=0) }}"
    end: "{{ now() }}"

# Modern format.
# Utilize friendly name, and set unique_id.
# ---------------------------------------------------------
template:
  - sensor:
      # We want to keep track of accumulated (daily) consumption for the circulation pump.
      - name: Balboa Spa circulationpump consumption
        unique_id: balboa_spa_circulationpump_consumption
        unit_of_measurement: 'kWh'
        device_class: energy
        state_class: total_increasing
        state: "{{ float(states('sensor.balboa_spa_circulationpump_running_time_daily'), 0) * 0.3 }}"

  - binary_sensor:
      # Since we cannot track attribute-state in history_stats, we need to create a binary template-sensor.
      # Returns true if
      - name:  Balboa Spa heater on
        unique_id: balboa_spa_heater_on
        device_class: heat
        state: "{{ is_state_attr('climate.nbp6013h_climate', 'hvac_action', 'heating') }}"

  - sensor:
      # We want to keep track of accumulated (daily) consumption for the heater.
      - name: Balboa Spa heater consumption
        unique_id: balboa_spa_heater_consumption
        unit_of_measurement: 'kWh'
        device_class: energy
        state_class: total_increasing
        state: "{{ float(states('sensor.balboa_spa_heater_running_time_daily'), 0) * 3 }}"
```
