# Home Assistant - Configuration

## Table of content

- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Generic integrations](https://github.com/slittorin/home-assistant-configuration#generic-integrations)
  - [Integration - Weather](https://github.com/slittorin/home-assistant-configuration#integration---weather)
  - [Integration - SQL](https://github.com/slittorin/home-assistant-configuration#integration---sql)

## Governing principles

- Setup HA with [Home Assistance setup](https://github.com/slittorin/home-assistant-setup).

# Generic integrations

## Integration - Weather

We want to have a more accurate weather integration for Sweden than the built in, so we utilize SMHI.

1. Add and enable integration `SMHI`.
   - Utilize the GPS coordinates set in setup/onboarding stage.
3. Disable the default integration (in my setup, 'met.no').
4. Delete thereafter the default integration.
   - If we this at initial setup of the HA, we do not loose any valid data.
   - If you want to keep historical data, do not delete this integration.

## Integration - SQL

We do not need any integration as this is built into HA.

### Home Assistant database size

To be able to gather information on the size and state of MariaDB database we utilize the built in SQL integration.

1. Through the `File Editor` add-on, edit the file `/config/configuration.yaml` and add after `sensor:` (add the row `sensor:` if not created, change also `password`):
     ```
  - platform: sql
    db_url: mysql://homeassistant:alskling2002!@localhost/hass
    queries:
      - name: HA DB size
        query: 'SELECT table_schema "database", Round(Sum(data_length + index_length) / 1024 / 1024, 1) "value" FROM information_schema.tables WHERE table_schema="hass" GROUP BY table_schema;'
        column: "value"
        unit_of_measurement: MB
     ```
2. Goto `Configuration` -> `Settings` -> `Server Controls` and press `Check Configuration`.
   - The output should state 'Configuration valid'. If not, change the recorder config above.
   - On the same page press `Restart` under `Server management`.
3. Add a sensor card to the Overview page.
   - Entity: `HA DB size`
   - Name: `Home Assistant Database size`
   - Icon: `mdi:database`
   - Unit: `MB`
