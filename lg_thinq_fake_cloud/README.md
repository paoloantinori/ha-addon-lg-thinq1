# LG ThinQ1 fake-cloud

A Home Assistant OS app that runs a local server impersonating the LG ThinQ1 cloud, so legacy
LG ThinQ1 appliances (washer, dryer, fridge) report state to Home Assistant over MQTT without
the LG cloud. Always-on; survives reboots.

This app builds the [local_smarthinq1](https://github.com/paoloantinori/local_smarthinq1)
server at a pinned commit (see the Dockerfile's `MAIN_COMMIT`) and runs it on the HAOS host
network.

## Setup

1. Add this repository to Home Assistant (Settings -> Add-ons -> Add-on store -> Repositories),
   then install the **LG ThinQ1 fake-cloud** app.
2. Create a dedicated MQTT user in the Mosquitto add-on (e.g. `lgthinq`).
3. Configure the app form:
   - **mqtt_host**: `127.0.0.1` (the Mosquitto add-on, same host)
   - **mqtt_user** / **mqtt_password**: the dedicated user
   - **mode**: `standalone` (default; impersonate the cloud locally)
   - **allow_control**: leave `false` unless doing a supervised control test
4. Start the app. The log should show `LG fake-cloud (standalone) on :46030`.
5. Route the appliance's traffic to this HAOS host via OpenWrt DNAT on ports `:46030` and
   `:47878` (see the main repo's `deploy/haos-addon/routing-setup.sh`).

## Ports

- `:46030` (TLS) for ThinQ1 telemetry (`report/diagmon`).
- `:47878` (raw TCP) for the ThinQ1 control channel (only active if `allow_control` is on).

## Safety

`allow_control` is off by default. Enabling it allows HA to actuate the appliance (physical
control). Washer/dryer buttons are not published until their wire format is captured and
approved; only the fridge's temperature selects are exposed.

## Updating

To pick up a newer server version, bump `MAIN_COMMIT` in the Dockerfile to a newer pushed
commit of [local_smarthinq1](https://github.com/paoloantinori/local_smarthinq1), then rebuild
the app in HA (Add-on store -> Refresh, then Rebuild, or `ha store reload && ha apps rebuild lg_thinq_fake_cloud`).
