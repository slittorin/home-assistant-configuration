# Home Assistant - Configuration

## Table of content

- [Goal](https://github.com/slittorin/home-assistant-configuration#goal)
- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Resources, Packages and Integrations](https://github.com/slittorin/home-assistant-configuration#resources-packages-and-integrations)
  - [Github Push](https://github.com/slittorin/home-assistant-configuration#github-push)
  - [Resource - Lovelace Card Mod](https://github.com/slittorin/home-assistant-configuration#resource---lovelace-card-mod)
  - [Resource - Apex Charts Card](https://github.com/slittorin/home-assistant-configuration#resource---apex-charts-card)
  - [Package - Home Assistant - Database and tables data](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system---database-and-tables-data)
  - [Package - Home Assistant - Domains and entities](https://github.com/slittorin/home-assistant-configuration/blob/main/README.md#package---home-assistant-system---domains-and-entities)
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

# Governing principles

- Setup HA with [Home Assistance setup](https://github.com/slittorin/home-assistant-setup).

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
- Scheduled tasks (triggers, automation, backups):
  - Spread out evenly to not put load on specific times (includes towards databases).
  - Use the following as base:
    - 00:01 - 00:59 - Backup timeslot.
    - 01:00 - 01:58 - Triggers, automations and similar.

# Resources, Packages and Integrations

## Github Push

We want to utilize Github Push instead of Pull as the original files are to reside on my HA-device.\
Therefore we do not utilize the standard Github Pull integration.\
Inspiration from the [community](https://community.home-assistant.io/t/sharing-your-configuration-on-github/195144).

Note: HASS.io already has git installed, so no need to install git.

Pre-requisities: That repository `home-assistant-config` is created in Github.

Se also [Github Push in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization/blob/main/README.md#dashboard---home-assistant).

Perform the following:

1. If not already done, create a Github personal access token:
   - According [instructions](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).
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

# First rule: Ignore everything, all files in root and all directories.
*
*/*

# Second rule: Whitelisted folders, these will not be ignored.
!/blueprints/
!/blueprints/**
!/deps/
!/deps/**
!/packages/
!/packages/**
!/scripts/
!/scripts/**
!/tts/
!/tts/**
!/www/
!/www/**

# Third rule: Whitelisted files, these will not be ignored.
!*.yaml
!.gitignore
!*.md
!*.sh

# Fourth rule is more specific, as we want to whitelist certain files in .storage
!/.storage/
!/.storage/lovelace
!/.storage/lovelace.*
!/.storage/lovelace_*

# Last rule: If we make a mistake above, ensure these files are ignored, otherwise your secret data/credentials will leak.
.env
ip_bans.yaml
secrets.yaml
```
4. Through the 'SSH & Web terminal' run the following in the `/config` directory (change your email and name to match your Github account, and ensure that all directories you want to add to Github are present):\
   You will be asked to provide a comment at commit.
```bash
git init
git config user.email "you@example.com"
git config user.name "Your Name"
```
5. Through the 'SSH & Web terminal' run the following in the `/config` directory (where TOKEN is the copied token, ensure that usename and repository is correct):\
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

# Check input.
if [ -z "$1" ]
then
    no_comment=1
    comment="Minor change."
else
    no_comment=0
    comment="$1"
fi

# Variables:
base_dir="/config/scripts"
log_dir="/config/logs"
config_dir="/config"
logfile="${log_dir}/github_push.log"

_initialize() {
    touch "${logfile}"

    echo ""
    echo "----------------------------------------------------------------------------------------------------------------"
    echo "$(date +%Y%m%d_%H%M%S): Starting Github push."

    if [ ${no_comment} -eq 1 ] 
    then
        echo "$(date +%Y%m%d_%H%M%S): No input given, setting comment to default."
    fi
}

_github_push() {
    cd ${config_dir}
    
    exit_code=0
    status_error=""
    
    # Add all in /config dir (according to .gitignore).
    echo "$(date +%Y%m%d_%H%M%S): Added all in base directory"
    git add .
    
    # We also add .storage, but allow .gitignore to only allow whitelisted files.
    # This to be able to add Lovelace files managed by UI.
    echo "$(date +%Y%m%d_%H%M%S): Added directory: .storage/"
    git add .storage/

    # Loop through all directories and add to git (according to .gitignore).
    for dir in */ ; do
        echo "$(date +%Y%m%d_%H%M%S): Added directory: ${dir}"
        git add "${dir}"
    done

    git status
    git_exit_code=$?
    if [ ${git_exit_code} -ne 0 ] 
    then
        exit_code=1
        status_error+=" status (${git_exit_code})"
    fi

    git commit -m "${comment}"
    git_exit_code=$?
    if [ ${git_exit_code} -ne 0 ] 
    then
        exit_code=1
        status_error+=" commit (${git_exit_code})"
    fi

    git push origin master
    git_exit_code=$?
    if [ ${git_exit_code} -ne 0 ] 
    then
        exit_code=1
        status_error+=" push (${git_exit_code})"
    fi

    # Check if error occured with git commands.
    if [ ${exit_code} -eq 0 ] 
    then
        status="No error."
    else
        status="Error in: git${status_error}."
    fi
}

_finalize() {
    echo "$(date +%Y%m%d_%H%M%S): ${status}"
}

# Main
_initialize >> "${logfile}" 2>&1
_github_push >> "${logfile}" 2>&1
_finalize >> "${logfile}" 2>&1
exit ${exit_code}
```
7. Through the 'SSH & Web terminal' run the following in the `/config/script` directory:
   - `chmod ug+x github_push.sh`.
   - `./github_push.sh`-
8. Check the log-file `/config/logs/github_push.log`.
   - Isolate if there are errors, and if needed isolate the problem.
9. Go to the github repositor and check that all files has been properly pushed.
   - Isolate if there are errors, and if needed isolate the problem.
10.  Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and add after `shell_command:` (if not already present, add `shell_command:`, mind the spaces):
```yaml
  github_push: /config/scripts/github_push.sh "{{ value }}"
```
11. Through the `File Editor` add-on, create the file `/config/packages/github_push.yaml`
2. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    github_push: !include packages/github_push.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/github_push.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/github_push.yaml) and add the following:
   - Input text for committ message.
   - Input button.
   - Automation to trigger shell_command on press of button.
   - Sensor for retrieving the last row of the log-file for `github_push.sh`.
   - Triggers for keeping track of domain-entities.

## Resource - Lovelace Card Mod

We want to tweak lovelace with CSS styles to various elements of the Home Assistant frontend.

Perform the following:

1. Download the latest `card-mod.js` file from [Github Lovelace Card Mod](https://github.com/thomasloven/lovelace-card-mod)
2. Through the `File Editor` add-on, save `card-mod.js` in `/config/www`.
3. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and add:
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

## Package - Home Assistant system - Database and tables data

We want to keep track of the following for the HA system:
- Recorder database size (in MariaDB), each 30 minutes.
- Size of the main tables, each 30 minutes:
  - states
  - events
  - statistics
  - statistics_short_term
- To keep track of the earlies data in the tables, each day:
  - First created data for the main tables:
    - states
    - events
    - statistics
    - statistics_short_term
- To keep track of entities with most rows in the table states, each day:
  - 30 entities with most rows.

Perform the following:

1. Through the `File Editor` add-on, create the file `/config/packages/database_table_sizes.yaml`
2. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    database_table_sizes: !include packages/database_table_sizes.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/database_table_sizes.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/database_table_sizes.yaml) and add the sensors according to above.

## Package - Home Assistant system - Domains and entities

Note that we need to add senors manually for domains that are not present, see [Regular maintenance - Add domain sensors](https://github.com/slittorin/home-assistant-maintenance#add-domain-sensors).

We want to keep track of the following for the HA system:
- Number of all domains.
- List of all domains.
- Number of entities.
- Number of entities per domain.

1. Through the `File Editor` add-on, create the file `/config/packages/domains_entities.yaml`
2. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    domains_entities: !include packages/domains_entities.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/domains_entities.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/domains_entities.yaml) and add the sensors according to above.

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
6. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    ha_system: !include packages/weather.yaml
```
7. Through the `File Editor` add-on, edit the file [/config/packages/weather.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/weather.yaml) and add the following:
   - Binary sensors to keep track of when sun is over and under horizon.
   - Binary sensors to keep track of when sun reaches and leaves my solar panels.
   - Snapshot in seconds and HH:MM:SS when sun is over and under horizon.
   - Snapshot in seconds and HH:MM:SS when sun reaches and leaves my solar panels.
   - Time in seconds and HH:MM:SS when sun is over the horizon.
   - Time in seconds and HH:MM:SS when sun reaches my solar panels (so I can track the time I should get sun on the panels).
   - Sensors for all SMHI weather integration attributes.
   - Conversion of wind speeds from km/h to m/s.
   - 'Feel' effect of temperature and wind speed (From SMHI in Sweden).
   - Readable Swedish format for wind-bearing.
   - Readable Swedish format (from SMHI Sweden) for wind speeds.
   - Sensors for Sun integration attributes for elevation and azimuth.

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
4. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    tariff_electrical: !include packages/tariff_electrical.yaml
```
5. Through the `File Editor` add-on, edit the file [/config/packages/tariff_electrical.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/tariff_electrical.yaml) and add the following:
   - Nordpool data to get correctly for my region.

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
3. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```
    balboa_spa: !include packages/balboa_spa.yaml
```
4. Through the `File Editor` add-on, edit the file [/config/packages/balboa_spa.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/balboa_spa.yaml) and add the following:
   - History stats to keep track of how long time the circulation pump and header has been running.
   - We cannot track attribute-state in history_stats, we need to create a binary template-sensor.
   - Keep track of accumulated (daily) consumption for the circulation pump and heater.
     - Circulation pump: 0.2 kW + 0.1 kW for the ozone-device (that takes 0.2 kW, but we assume that the ozone-device is only running 50% of the time).
     - Heater 3 kW.
