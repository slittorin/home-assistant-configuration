# Home Assistant - Configuration

## Table of content

- [Governing principles](https://github.com/slittorin/home-assistant-setup#conceptual-design)
- [Governing principles](https://github.com/slittorin/home-assistant-setup#governing-principles)
- [Setup for Server 1](https://github.com/slittorin/home-assistant-setup#setup-for-server-1)

## Governing principles

- Install HA with [Home Assistance install](https://github.com/slittorin/home-assistant-install/).
- If not otherwise stated, the user `pi` performs all actions.

## Initial setup

1. Go to the HA page (servername:8123).
2. Create the first administrator account:
   - Choose 'admin' as the first account-name.
   - Choose a secure password.
   - Press 'Create account'.
3. Name and localization:
   - Enter the name of the HA installation.
   - Enter GPS-coordinates. Important to get the right weather data later on.
   - Enter the Timezone.
   - Enter the elevation.
   - Enter unit and currency.
   - Press 'Next'.
4. Analytics:
   - Decide if and what type of analytics to share.
   - Press 'Next'.
5. Devices and services.
   - Here we add devices and services later, so press 'Finish'.
6. Setup to utilize the MariaDB database.
   - Edit the file `/srv/ha/config/configuration.yaml` and add the following:
```

```
7. 
