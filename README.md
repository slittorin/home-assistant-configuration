# Home Assistant - Configuration

## Table of content

- [Goal](https://github.com/slittorin/home-assistant-configuration#goal)
- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Resources, Packages and Integrations](https://github.com/slittorin/home-assistant-configuration#resources-packages-and-integrations)
  - [Backup](https://github.com/slittorin/home-assistant-configuration#backup)
  - [Copy backup files to server1](https://github.com/slittorin/home-assistant-configuration#copy-backup-files-to-server1)
  - [Git for Grafana](https://github.com/slittorin/home-assistant-configuration#git-for-grafana).
  - [Github Push](https://github.com/slittorin/home-assistant-configuration#github-push)
  - [Resource - Lovelace Card Mod](https://github.com/slittorin/home-assistant-configuration#resource---lovelace-card-mod)
  - [Resource - Layout Card](https://github.com/slittorin/home-assistant-configuration#resource---layout-card)
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

#### Styles, naming convention, device and state class, and unit of measurement
- Configuration-files/yaml:
  - Do not create more sensors than needed. Rely on the standard integration entities/attributes.
    - Since states (and therefore also InfluxDB states) do not track attribute changes, make sure to create sensors for the attributes you want to track.
      - And of course, where required, create new sensors if attributes cannot be utilized by templates, triggers or other.
  - When retrieving states of sensors, in Jinja2, make sure to check if the value is `unknown` or `unavailable`.
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
    - Utilize `unit_of_measurement` for all entities/sensors. Standard units is found [here](https://github.com/home-assistant/core/blob/dev/homeassistant/const.py).
    - For power/consumption related sensors one may also look to add integration [Powercalc](https://github.com/bramstroker/homeassistant-powercalc), specifically for binary sensors.
- Scheduled tasks (triggers, automation, backups):
  - Spread out evenly to not put load on specific times (includes towards databases).
  - Use the following as base:
    - 00:01 - 00:59 - Backup timeslot.
    - 01:00 - 01:58 - Triggers, automations and similar.

#### Persistence

Since Home Assistant is a event/state machine there will be some sensors that will not survive a restart.\
This is due to the architectural design and governing principles on how the platform works.

Therefore all integrations must be able to support persistence over restart, dependent on the nature of the integration what the sensors measure.\
It also means that manually added sensors/entities will need to be adapted to support persistence over reboots.

Note must be taken on the following:
- Adding persistence will not only add complexity, but may also affect measurement of statistics and so forth, therefore try not to add persistence.
  - Sensors that for instance are used to track sum of values over time, cannot simply be made persistent, since if we add persistence, it will add value to the sum that was not intended.
- Snapshots are good candidates to made persistent.

Persistence can be made according to the following:
- For automations:
  - Add the following to automation triggers to allow the trigger to be fired after restart of HA:
    ```yaml
    - platform: homeassistant
      event: start
    ```
- For sensors:
  - Create `input_number` of `input_text` sensor to store sensor values (input sensors are persistent over restarts):
    ```yaml
    input_number:
      electrical_consumption_intake_hour_snapshot_persistent:
        name: 'Persistent for electrical_consumption_intake_hour_snapshot'
        icon: mdi:database
        mode: box
        min: 0
        max: 10000000
        unit_of_measurement: "kWh"
    ```
  - Add automation to update input sensor based on (original/tracked) state changes:
    ```yaml
    automation:
    # For persistence, On every state change for electrical_consumption_intake_hour_snapshot, set input_number sensor.
    - id: automation_electrical_consumption_intake_hour_snapshot
      alias: 'Automation for persistance of electrical_consumption_intake_hour_snapshot'
      trigger:
        - platform: state
          entity_id: sensor.electrical_consumption_intake_hour_snapshot
      action:
        service: input_number.set_value
        data_template:
          entity_id: input_number.electrical_consumption_intake_hour_snapshot_persistent
          value: "{{ states('sensor.electrical_consumption_intake_hour_snapshot') }}"
    ```
  - Use template sensors and Jinja2 to check if sensor value is `unknown` or `unavailable` and retrieve the value from the input sensor:
    ```yaml
    - name:  electrical_solar_production_hour
      device_class: 'energy'
      unit_of_measurement: 'kWh'
      state_class: measurement
      state: >
        {% set persistent_snapshot = states('input_number.electrical_consumption_intake_hour_snapshot_persistent') %}
        {% set snapshot = states('sensor.electrical_consumption_intake_hour_snapshot') %}
        {% if (snapshot == 'unknown' or snapshot == 'unavailable') %}
        {%   set snapshot = persistent_snapshot %}
        {% endif %}
    ```

# Resources, Packages and Integrations

## Backup

We want to have the following backup-scheme:
- Daily backups Monday through Saturday, kept for 7 days.
- Weekly backups on Sunday, kept for 8 weeks (56+1 days).

We could have utilized a standard automation scheme, and run a shell-script daily to remove older files, however since the filenaming-convention on file-level för backup files are not readable, we instead utilize a backup-integration [auto-backup](https://github.com/jcwillox/hass-auto-backup) that will take care of the work for us.

1. Through `HACS` in the menu:
   - Add Integration: `Auto Backup` by `@jcwillox`.
2. Restart the Home Assistant server under `Server management`.
   - This may take a while as the custom component is installed.
3. Go to `Configuration` -> `Devices and services` and add Integration `auto backup`.
4. Through the `File Editor` add-on, edit the file [/config/packages/backup.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/backup.yaml) and add the following:
   - Daily backup scheme according above.
   - Weekly backup schema according above.
5. For the Auto backup integration:
   - Change backup timeout to 60 minutes.

## Copy backup files to server1

In case we get a server corruption, we also want to copy the files to server1.

Pre-requisite is [Preparation in setup](https://github.com/slittorin/home-assistant-setup#preparation).

See also [Github Copy to server1 in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#copy-to-server1).

Here we needed to take into consideration the following:
- Ideally we should have have used `rsync`, but this is not installed by default in HA/add-on for Terminal/SSH.
  - Therefore we need to create a script to do a copy.
- Command shell scripts are terminated by HA after 60 seconds.
  - We therefore add `&` to get the script to not be terminated by HA.
  - With the below solution, we will therefore get errors in the log if the copy takes more than 1 minute.
    ```
    2022-03-19 03:01:00 ERROR (MainThread) [homeassistant.components.shell_command] Timed out running command: `/config/scripts/copy_backup.sh pi@192.168.2.30 /srv/ha/backup &`, after: 60s
    ```
- Directory `/backup` is not directly reachable by command shell triggered by HA.
  - But exists the add-on for Terminal/SSH.

Perform the following:
1. Through the `File Editor` add-on, add the file `/config/scripts/copy_backup.sh` and add:
```bash
#!/bin/bash
#
# Purpose:
# Copy backup to other server.
#
# Usage:
# ./copy_backup.sh SERVER REMOTEDIR
#
# Example: ./copy_backup.sh pi@192.168.2.30 /srv/ha/backup
#
# SERVER    - Mandatory. Is the remote server in format: user@NAME/IP.
#             Remember that the server must be able to run commands without password (such as key based login).
# REMOTEDIR - Mandatory. Remote directory.
#
# Since shell commands by default do not have /backup mounted, we do this manually: https://community.home-assistant.io/t/tips-for-shell-command-and-auto-deleting-old-snapshots/207688
# We could have utilized this approach, but chose manual mount: https://community.home-assistant.io/t/shell-command-backup-not-found/273055/8

# Load environment variables (mainly secrets).
if [ -f "/config/.env" ]; then
    export $(cat "/config/.env" | grep -v '#' | sed 's/\r$//' | awk '/=/ {print $1}' )
fi

# Variables:
config_dir="/config"
base_dir="/config/scripts"
log_dir="/config/logs"
logfile="${log_dir}/copy_backup.log"
logfile_tmp="${log_dir}/copy_backup.tmp"
rsa_file="/config/.ssh/id_rsa"
known_hosts_file="/config/.ssh/known_hosts"

touch ${logfile}

# Check server.
if [ -z "$1" ]; then
    echo "ERROR. Server must be given."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Server must be given." >> ${logfile}
    exit 1
else
    SERVER="$1"
fi

# Remote dir.
if [ -z "$1" ]; then
    echo "ERROR. Remote directory must be given."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Remote directory must be given." >> ${logfile}
    exit 1
else
    REMOTEDIR="$2"
fi

echo "$(date +%Y%m%d_%H%M%S): Starting copy of backup." >> ${logfile}

# We can run this script in differect context, either in shell add-on, or as shell command.
# Therefore we need to check this before we start to copy.
backup_dir="/backup"
if [ ! -d "${backup_dir}" ]; then
    backup_dir="/mnt/data/supervisor/backup"
    echo "$(date +%Y%m%d_%H%M%S): Script triggered as shell_command, mounting ${backup_dir}" >> ${logfile}
    umount /mnt/data 2> /dev/null
    mkdir -p /mnt/data 2>> ${logfile}
    mount /dev/sda8 /mnt/data 2>> ${logfile}
    RESULT_CODE=$?
    if [ ${RESULT_CODE} -ne 0 ]; then
        echo "ERROR. Mount error. Exit code: ${RESULT_CODE}"
        echo "$(date +%Y%m%d_%H%M%S): ERROR. Mount error. Exit code: ${RESULT_CODE}" >> ${logfile}
        exit 1
    fi 
else
    echo "$(date +%Y%m%d_%H%M%S): Script triggered through shell add-on." >> ${logfile}
fi

echo "$(date +%Y%m%d_%H%M%S): Local directory: ${backup_dir}" >> ${logfile}
echo "$(date +%Y%m%d_%H%M%S): Remote directory: ${SERVER}:${REMOTEDIR}" >> ${logfile}

# Since rsync does not exists in HA addon for shell, we need to make exact copy manually.
# Check if all files in remote directory exists locally, if not, they are removed remote.
# We check here if the file size is equal, but do not check hash of file.
FILES_REMOVED=0
FILES=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "ls -1 ${REMOTEDIR} && ls -1 *.tar 2> /dev/null" 2>> ${logfile}`
if [ ! -z "${FILES}" ]; then
    for FILE in ${FILES}; do
        FILE=`basename ${FILE}`
        if [ ! -z ${FILE} ]; then
            if [ ! -f "${backup_dir}/${FILE}" ]; then
                RESULT=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "rm ${REMOTEDIR}//${FILE}" 2>> ${logfile}`
                RESULT_CODE=$?
                if [ ${RESULT_CODE} -ne 0 ]; then
                    echo "ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${RESULT}"
                    echo "$(date +%Y%m%d_%H%M%S): ERROR. SSH command error when removing remote file. Exit code: ${RESULT_CODE}: ${RESULT}" >> ${logfile}
                    exit 1
                else
                    FILES_REMOVED=$((${FILES_REMOVED}+1))
                    echo "$(date +%Y%m%d_%H%M%S): File ${FILE} does not exists locally, and is removed in remote directory." >> ${logfile}
                fi
            else
                SIZE_REMOTE=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "stat -c %s ${REMOTEDIR}/${FILE}" 2>> ${logfile}`
                RESULT_CODE=$?
                if [ ${RESULT_CODE} -ne 0 ]; then
                    echo "ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${SIZE_REMOTE}"
                    echo "$(date +%Y%m%d_%H%M%S): ERROR. SSH command error when getting hash for file. Exit code: ${RESULT_CODE}: ${RESULT}" >> ${logfile}
                    exit 1
                fi
                
                SIZE=`stat -c %s ${backup_dir}/${FILE} 2>> ${logfile}`
                if [ "${SIZE_REMOTE}" == "${SIZE}" ]; then
                    echo "$(date +%Y%m%d_%H%M%S): Remote file ${FILE} exists locally and are equal in size, not required to be transfered." >> ${logfile}
                else
                    RESULT=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "rm ${REMOTEDIR}//${FILE}" 2>> ${logfile}`
                    RESULT_CODE=$?
                    if [ ${RESULT_CODE} -ne 0 ]; then
                        echo "ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${RESULT}"
                        echo "$(date +%Y%m%d_%H%M%S): ERROR. SSH command error when removing remote file. Exit code: ${RESULT_CODE}: ${RESULT}" >> ${logfile}
                        exit 1
                    else
                        FILES_REMOVED=$((${FILES_REMOVED}+1))
                        echo "$(date +%Y%m%d_%H%M%S): Remote file ${FILE} exists locally but files are not equal in size, therefore removed in remote directory." >> ${logfile}
                    fi
                fi
            fi
        fi
    done
else
    echo "$(date +%Y%m%d_%H%M%S): No files in remote directory." >> ${logfile}
fi

# Since rsync does not exists in HA addon for shell, we need to make exact copy manually.
# Check if file exists remotely, if not, transfer.
cd ${backup_dir}
FILES_COPIED=0
FILES=`ls -1 *.tar 2> /dev/null`
if [ ! -z "${FILES}" ]; then
    for FILE in ${FILES}; do
        if [ ! -z ${FILE} ]; then
            CHECK=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "ls -1 ${REMOTEDIR}/${FILE} 2> /dev/null" 2>> ${logfile}`
            if [ -z ${CHECK} ]; then
                echo "$(date +%Y%m%d_%H%M%S): File ${FILE} does not exists remotely, and is being transfered." >> ${logfile}
                RESULT=`scp -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} -p ${backup_dir}/${FILE} ${SERVER}:${REMOTEDIR} 2>> ${logfile}`
                RESULT_CODE=$?
                if [ ${RESULT_CODE} -ne 0 ] 
                then
                    echo "ERROR. SSH command error when transferring file. Exit code: ${RESULT_CODE}: ${RESULT}"
                    echo "$(date +%Y%m%d_%H%M%S): ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${RESULT}" >> ${logfile}
                    exit 1
                else
                    FILES_COPIED=$((${FILES_COPIED}+1))
                fi
            fi
        fi
    done
else
    echo "$(date +%Y%m%d_%H%M%S): No files in local directory." >> ${logfile}
fi

# Log, and limit logfile to 1000 rows.
RESULT="$(date +%Y%m%d_%H%M%S): No error. Removed files: ${FILES_REMOVED}. Copied files: ${FILES_COPIED}."
echo "${RESULT}" >> ${logfile}
tail -n1000 ${logfile} > ${logfile_tmp}
rm ${logfile}
mv ${logfile_tmp} ${logfile}

# Show output
echo "${RESULT}"

exit 0
```
2. Through the 'SSH & Web terminal' run the following in the `/config/script` directory (change to fit your installation):
   - `chmod ug+x copy_backup.sh`.
   - `copy_backup.sh pi@192.168.2.30 /srv/ha/backup`.
3. Check the log-file `/config/logs/copy_backup.log`.
   - Isolate if there are errors, and if needed isolate the problem.
4. Go to server1 and check that the files exists under `/srv/ha/backup`.
   - Isolate if there are errors, and if needed isolate the problem.
5.  Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and add after `shell_command:` (if not already present, add `shell_command:`, mind the spaces. Adapt after your installation):
```yaml
  # We run copy_backup as background process, since otherwise HA will terminate the process after 60 seconds.
  copy_backup: /config/scripts/copy_backup.sh pi@192.168.2.30 /srv/ha/backup &
```
6. Through the `File Editor` add-on, create the file `/config/packages/copy-backup.yaml`
7. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    copy_backup: !include packages/backup.yaml
```
8. Through the `File Editor` add-on, edit the file [/config/packages/backup.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/backup.yaml) and add the following:
   - Input button.
   - Automation to trigger shell_command on press of button.
   - Sensor for retrieving the last row of the log-file for `/config/logs/copy_backup.log`.

## Github Push

We want to utilize Github Push instead of Pull as the original files are to reside on my HA-device.\
Therefore we do not utilize the standard Github Pull integration.\
Inspiration from the [community](https://community.home-assistant.io/t/sharing-your-configuration-on-github/195144).

Note: HASS.io already has git installed, so no need to install git.

Pre-requisities:
- [Preparation in setup](https://github.com/slittorin/home-assistant-setup#preparation).
- That repository `home-assistant-config` is created in Github.

See also [Github Push in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization/blob/main/README.md#dashboard---home-assistant).

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
config_dir="/config"
base_dir="/config/scripts"
log_dir="/config/logs"
logfile="${log_dir}/github_push.log"
logfile_tmp="${log_dir}/github_push.tmp"

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

# Limit the number of rows in the logfile
tail -n1000 ${logfile} > ${logfile_tmp}
rm ${logfile}
mv ${logfile_tmp} ${logfile}

exit ${exit_code}
```
7. Through the 'SSH & Web terminal' run the following in the `/config/script` directory:
   - `chmod ug+x github_push.sh`.
   - `./github_push.sh`.
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
   - Sensor for retrieving the last row of the log-file for `/config/logs/`github_push..log`.
   - Triggers for keeping track of domain-entities.

## Git for Grafana

Besides backup of Grafana, we also want to push the dashboards to Github.
This to have them also backed up there, and allow them to be shared.

Pre-requisities:
- [Preparation in setup](https://github.com/slittorin/home-assistant-setup#preparation).
- That the setup for [Git for Grafana](https://github.com/slittorin/home-assistant-setup#git-for-grafana) is done.

See also [Git for Grafana in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#git-for-grafana).

Perform the following:
1. Through the `File Editor` add-on, add the file `/config/scripts/grafana_github_push.sh` and add:
```bash
#!/bin/bash
#
# Purpose:
# Push Grafana configuration on other server to github.
#
# Usage:
# ./grafana_github_push.sh SERVER HOST COMMENT
#
# SERVER    - Mandatory. Is the remote server in format: user@NAME/IP.
#             Remember that the server must be able to run commands without password (such as key based login).
# HOST      - The Grafana host/IP to connect to, including port.
# COMMENT   - The comment to add to the push-commit.
#             If empty, the default comment will be: "Minor change."
#
# Pre-req. is that the script for pushing Grafana JSON-files to Github exists on the other server as: /srv/grafana-git.sh

# Load environment variables (mainly secrets).
if [ -f "/config/.env" ]; then
    export $(cat "/config/.env" | grep -v '#' | sed 's/\r$//' | awk '/=/ {print $1}' )
fi

# Variables:
config_dir="/config"
base_dir="/config/scripts"
log_dir="/config/logs"
logfile="${log_dir}/grafana_github_push.log"
logfile_tmp="${log_dir}/grafana_github_push.tmp"
rsa_file="/config/.ssh/id_rsa"
known_hosts_file="/config/.ssh/known_hosts"

touch ${logfile}

# Check server.
if [ -z "$1" ]; then
    echo "ERROR. Server must be given."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Server must be given." >> ${logfile}
    exit 1
else
    SERVER="$1"
fi

# Check host.
if [ -z "$2" ]; then
    echo "ERROR. Host must be given."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Grafana host must be given." >> ${logfile}
    exit 1
else
    HOST="$2"
fi

# Check comment.
if [ -z "$3" ]; then
    NO_COMMENT=1
    COMMENT="Minor change."
else
    NO_COMMENT=0
    COMMENT="$3"
fi

echo "$(date +%Y%m%d_%H%M%S): Starting Grafana Github push." >> ${logfile}
echo "$(date +%Y%m%d_%H%M%S): Server: ${SERVER}" >> ${logfile}
echo "$(date +%Y%m%d_%H%M%S): Host: ${HOST}" >> ${logfile}

if [ ${NO_COMMENT} -eq 1 ]; then
    echo "$(date +%Y%m%d_%H%M%S): No comment given, setting comment to default." >> ${logfile}
fi

# Initialize the remote script.
RESULT=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "sudo /srv/grafana-git.sh ${HOST} '${COMMENT}'" 2>> ${logfile}`
RESULT_CODE=$?
if [ ${RESULT_CODE} -ne 0 ]; then
    echo "ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${RESULT}"
    echo "$(date +%Y%m%d_%H%M%S): ERROR. SSH command error when removing triggering remote script. Exit code: ${RESULT_CODE}: ${RESULT}" >> ${logfile}
    exit 1
else
    echo "$(date +%Y%m%d_%H%M%S): File ${FILE} remote script triggered." >> ${logfile}
fi

# Get the last row in the log.
RESULT=`ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "tail -1 /srv/grafana-git.log" 2>> ${logfile}`

# Log, and limit logfile to 1000 rows.
echo "${RESULT}" >> ${logfile}
tail -n1000 ${logfile} > ${logfile_tmp}
rm ${logfile}
mv ${logfile_tmp} ${logfile}

# Show output
echo "${RESULT}"

exit 0
```
2. Through the 'SSH & Web terminal' run the following in the `/config/script` directory (change to fit your installation):
   - `chmod ug+x grafana_github_push.sh`.
   - `grafana_github_push.sh pi@192.168.2.30 192.168.2.30:3000 Test`.
3. Check the log-file `/config/logs/copy_backup.log`.
   - Isolate if there are errors, and if needed isolate the problem.
4. Go to server1 and check logfile `/srv/grafana-git.log`.
   - Isolate if there are errors, and if needed isolate the problem.
5.  Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and add after `shell_command:` (if not already present, add `shell_command:`, mind the spaces. Adapt after your installation):
```yaml
  grafana_github_push: /config/scripts/grafana_github_push.sh pi@192.168.2.30 192.168.2.30:3000 "{{ value }}"
```
6. Through the `File Editor` add-on, create the file `/config/packages/grafana_github_push.yaml`
7. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    grafana_github_push: !include packages/grafana_github_push.yaml
```
8. Through the `File Editor` add-on, edit the file [/config/packages/grafana_github_push.yaml]https://github.com/slittorin/home-assistant-config/blob/master/packages/grafana_github_push.yaml) and add the following:
   - Input text for commit message.
   - Input button.
   - Automation to trigger shell_command on press of button.
   - Sensor for retrieving the last row of the log-file `/config/logs/grafana_github.log`.

## Resource - Lovelace Card Mod

We want to tweak lovelace with CSS styles to various elements of the Home Assistant frontend.\
Therefore we add [Card Mod](https://github.com/thomasloven/lovelace-card-mod).

Perform the following:

1. Through `HACS` in the menu:
   - Add Frontend: `Card Mod` by `@thomasloven`.

## Resource - Layout Card

We want to be able to customize cards.\
Therefore we add [Layout Card](https://github.com/thomasloven/lovelace-layout-card).

Perform the following:

1. Through `HACS` in the menu:
   - Add Frontend: `layout-card` by `@thomasloven`.

## Resource - Apex Charts Card

We want to be able to add more customized graph cards.\
Therefore we add [Apex Charts](https://github.com/RomRider/apexcharts-card).

Perform the following:

1. Through `HACS` in the menu:
   - Add Frontend: `apex-chart-card` by `@RomRider`.

## Package - server1 system - Database and tables data

We want to keep track of statistics for server1:
- CPU Temp
- CPU usage (15 min average)
- Disk usage
- Memory usage
- Swap usage
- Uptime

Pre-requisities:
- [Preparation in setup](https://github.com/slittorin/home-assistant-setup#preparation).
- [Setup of OS/HW statistics](https://github.com/slittorin/home-assistant-setup#oshw-statistics).

See also [XXX in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#git-for-grafana).

Perform the following:

1. Through the `File Editor` add-on, create the file `/config/packages/server1.yaml`
2. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    server1: !include packages/server1.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/server1.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/server1.yaml) and add command line sensors to retrieve the stats-file from server1 according above.

## Package - Home Assistant system - Database and tables data

We want to keep track of the following for the HA system:
- Recorder database size (in MariaDB), each 6 hour (was first 30 minutes, then 1 hour, thereafter updated to 6 hour 2022-03-17, since these takes quite a load on the server, and also is not required that often).
  - Size of the main tables:
    - states
    - events
    - statistics
    - statistics_short_term
- To keep track of the earlies data in the tables, each day:
  - First created data (date) for the main tables:
    - states
    - events
    - statistics
    - statistics_short_term

See also [Size of recorderdatabase and tables in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#size-of-recorder-database-and-tables), and  [min(created) date in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#mincreated-date-for-tables).

Perform the following:

1. Through the `File Editor` add-on, create the file `/config/packages/database_table_sizes.yaml`
2. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    database_table_sizes: !include packages/database_table_sizes.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/database_table_sizes.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/database_table_sizes.yaml) and add the sensors according to above.

## Package - Home Assistant system - InfluxDB Size

We want to keep track of the database size for InfluxDB.
As there is no native way to do this, we take the size of the XXXX.

Pre-requisities:
- [Preparation in setup](https://github.com/slittorin/home-assistant-setup#preparation).

See also [InfluxDB size in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization/blob/main/README.md#influxdb-size).

Perform the following:

1. Through the `File Editor` add-on, add the file `/config/scripts/remote_docker_volume_size.sh` and add:
```bash
#!/bin/bash
#
# Purpose:
# Retrieve docker volume size information from remote server.
#
# Usage:
# ./remote_docker_volume_size.sh SERVER VOLUME TYPE
#
# Example: ./remote_docker_volume_size.sh pi@192.168.2.30 /var/lib/influxdb2 value
#
# SERVER    - Mandatory. Is the remote server in format: user@NAME/IP.
#             Remember that the server must be able to run commands without password (such as key based login).
# VOLUME    - Mandatory. The docker volume name. Does not need to match the entire volume-name, but give only one result.
# TYPE      - Default will be used if not given or not recognized. The measure, one of the following:
#             value             - The size of the volume in MB [default].
#             datetime          - The datetime, in format YYYYMMDD HHMMSS.

# Load environment variables (mainly secrets).
if [ -f "/config/.env" ]; then
    export $(cat "/config/.env" | grep -v '#' | sed 's/\r$//' | awk '/=/ {print $1}' )
fi

# Variables:
config_dir="/config"
base_dir="/config/scripts"
log_dir="/config/logs"
logfile="${log_dir}/remote_docker_volume_size.log"
logfile_tmp="${log_dir}/remote_docker_volume_size.tmp"
rsa_file="/config/.ssh/id_rsa"
known_hosts_file="/config/.ssh/known_hosts"

touch ${logfile}

# Check server.
if [ -z "$1" ]; then
    echo "ERROR. Server must be given."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Server must be given." >> ${logfile}
    exit 1
else
    SERVER="$1"
fi

# Check volume.
if [ -z "$2" ]; then
    echo "ERROR. Volume must be given."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Volume must be given." >> ${logfile}
    exit 1
else
    VOLUME="$2"
fi

# Check type to retrieve.
if [ -z "$3" ]; then
    TYPE="value"
else
    TYPE="$3"
fi

# Set the file to look into.
FILE="/srv/stats/docker_volume_sizes.txt"

# Run command.
RESULT=$(ssh -o BatchMode=yes -o UserKnownHostsFile=${known_hosts_file} -i ${rsa_file} ${SERVER} "cat ${FILE} | grep ${VOLUME}" 2>> ${logfile})
RESULT_CODE=$?
if [ ${RESULT_CODE} -ne 0 ] 
then
    echo "ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${RESULT}" 
    echo "$(date +%Y%m%d_%H%M%S): ERROR. SSH command error. Exit code: ${RESULT_CODE}: ${RESULT}" >> ${logfile}
    exit 1
fi

# We should only get one line.
LINES=$(echo "${RESULT}" | wc -l)
if [ ${LINES} -ne 1 ]; then
    echo "ERROR. Volume name too ambigous, several volumes found."
    echo "$(date +%Y%m%d_%H%M%S): ERROR. Volume name too ambigous, several volumes found." >> ${logfile}
    exit 1
fi

# Get the values.
DATETIME=`echo ${RESULT} | awk -F, '{ print $1 }'i | sed 's/_/ /'`
VALUE=`echo ${RESULT} | awk -F, '{ print $4 }'`

# Show the result
if [ "${TYPE}" == "datetime" ] ;then
    RESULT="${DATETIME}"
else
    RESULT="${VALUE}"
fi

# Log, and limit logfile to 1000 rows.
echo "$(date +%Y%m%d_%H%M%S): INFO. Server: ${SERVER}. File: ${FILE}. Measure: ${MEASURE}. Result: ${RESULT}." >> ${logfile}
tail -n1000 ${logfile} > ${logfile_tmp}
rm ${logfile}
mv ${logfile_tmp} ${logfile}

# Show output
echo "${RESULT}"

exit 0
```
2. Through the `File Editor` add-on, create the file `/config/packages/database_table_sizes.yaml`
2. Through the `File Editor` add-on, edit the file [/config/configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml) and after `  packages:` (mind the spaces):
```yaml
    database_table_sizes: !include packages/database_table_sizes.yaml
```
3. Through the `File Editor` add-on, edit the file [/config/packages/database_table_sizes.yaml](https://github.com/slittorin/home-assistant-config/blob/master/packages/database_table_sizes.yaml) and add the sensors according to above.

XXX
XXX
XXX
XXX





## Package - Home Assistant system - Domains and entities

Note that we need to add senors manually for domains that are not present, see [Regular maintenance - Add domain sensors](https://github.com/slittorin/home-assistant-maintenance#add-domain-sensors).

See also [Number of domains and entities in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#number-of-domains-and-entities).

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

See also [XXX in Visualizations - Dashboard Home Assistant](https://github.com/slittorin/home-assistant-visualization#number-of-domains-and-entities).

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

1. Through `HACS` in the menu:
   - Add Integration: `Nordpool`.
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
