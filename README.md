# Home Assistant - Configuration

## Table of content

- [Governing principles](https://github.com/slittorin/home-assistant-configuration#governing-principles)
- [Generic integrations]()
- [Integration - Weather]()
- [Integration - SQL]()

## Governing principles

- Setup HA with [Home Assistance setup](https://github.com/slittorin/home-assistant-setup).

# Generic integrations

## Integration - Weather

We want to have a more accurate weather integration for Sweden than the built in, so we utilize SMHI.

1. Add and enable integration `SMHI`, utilize the GPS coordinates set in setup/onboarding stage.
2. Disable the default integration (in my setup, 'met.no').
3. Delete thereafter the default integration.
   - If we this at initial setup of the HA, we do not loose any valid data.
   - If you want to keep historical data, do not delete this integration.

## Integration - SQL

To be able to gather information on the size and state of MariaDB database we enable the SQL integration.

1. 
2.
3.
