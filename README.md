# Home Assistant - Configuration

## Table of content

- [Generic information](https://github.com/slittorin/home-assistant-configuration#generic-information)
- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Integrations](https://github.com/slittorin/home-assistant-configuration#integrations)
  - [Generic - Home Assistant](https://github.com/slittorin/home-assistant-configuration/blob/main/README.md#generic---home-assistant)
  - [Integration - Weather](https://github.com/slittorin/home-assistant-configuration#integration---weather)
  - [Integration - Unifi](https://github.com/slittorin/home-assistant-configuration#integration---unifi)

## Generic information

For all changes to Home Assistant configuration files, you usually need to restart:
-  Goto `Configuration` -> `Settings` -> `Server Controls` and press `Check Configuration`.
   - The output should state 'Configuration valid'. If not, change the recorder config above.
   - On the same page press `Restart` under `Server management`.
- Any warnings or errors can be found in the file `/config/home-assistant.log`.
- Naming convention:
  - Entity ID: type, area, location, device, lower-case with `_` as delimiter.

## Governing principles

- Setup HA with [Home Assistance setup](https://github.com/slittorin/home-assistant-setup).

# Integrations

## Generic - Home Assistant

We want to keep track of database sizes and number of objects managed by Home Assistant.

1. Through the `File Editor` add-on, edit the file `/config/sensors.yaml` and add:
     ```
     # We keep track of the MariaDB database size.
     - platform: sql
       db_url: !secret recorder_db_url
       # Scan every 10:t minute.
       scan_interval: 600
       queries:
         - name: home_assistant_db_size
           query: 'SELECT table_schema "database", Round(Sum(data_length + index_length) / 1024 / 1024, 1) "value" FROM information_schema.tables WHERE table_schema="homeassistant" GROUP BY table_schema;'
           column: "value"
           unit_of_measurement: MB
     ```

## Integration - Weather

We want to have a more accurate weather integration for Sweden than the built in, so we utilize SMHI.

1. Add and enable integration `SMHI`.
   - Utilize the GPS coordinates set in setup/onboarding stage.
3. Disable the default integration (in my setup, 'met.no').
4. Delete thereafter the default integration.
   - If we this at initial setup of the HA, we do not loose any valid data.
   - If you want to keep historical data, do not delete this integration.
5. Through the `File Editor` add-on, edit the file `/config/templates.yaml` and add:
     ```
       - platform: sql
        db_url: !secret recorder_db_url
        # Scan every 10:t minute.
        scan_interval: 600
        queries:
          - name: home_assistant_db_size
            query: 'SELECT table_schema "database", Round(Sum(data_length + index_length) / 1024 / 1024, 1) "value" FROM information_schema.tables WHERE table_schema="homeassistant" GROUP BY table_schema;'
            column: "value"
            unit_of_measurement: MB
     ```

## Integration - Unifi

We want to have gather information from our Unifi network.

1. Add a read-only local user to the Unify controller.
2. Add the integration `Unifi`.
   - Host is the IP of the Cloud Key.
   - User and password for the read-only local user created above.
