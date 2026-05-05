# linux.local

The following configuration uses [Docker Compose](https://docs.docker.com/compose/)
to run containers on a [mini PC](https://www.amazon.de/dp/B0GJ5HS2GC) running
Ubuntu 25.10.

## Services

- ### [Tailscale](https://tailscale.com)

  Requires a `TS_AUTHKEY` environment variable to be set before starting:

  ```sh
  export TS_AUTHKEY=tskey-auth-xxxxxxxxxxxxxxxx
  docker compose up -d tailscale
  ```

- ### [Homebridge](https://github.com/homebridge/docker-homebridge)

  Homebridge is used to enable several appliances in my apartment to be compatible with HomeKit:

  - Two [ceiling lights](https://amzn.to/3iQLGHk) controlled using
    [ESPHome](https://esphome.io)-powered [Sonoff Basic
    Switches](https://amzn.to/3mHHUSV) using
    <https://github.com/arachnetech/homebridge-mqttthing>.

- ### [Mosquitto](https://mosquitto.org)

  Mosquitto helps connect with my Sonoff lamps using MQTT. Here's the JSON setup
  I used in Homebridge-UI for the lamps:

  ```json
  {
    "type": "lightbulb-OnOff",
    "name": "Fridge",
    "url": "http://10.0.0.254:1883",
    "topics": {
        "getOnline": "fridge/status",
        "getOn": "fridge/switch/sonoff_lamp/state",
        "setOn": "fridge/switch/sonoff_lamp/command"
    },
    "onlineValue": "online",
    "offlineValue": "offline",
    "retryLimit": 3,
    "confirmationIndicateOffline": true,
    "onValue": "ON",
    "offValue": "OFF",
    "accessory": "mqttthing"
  }
  ```

  ## Nuki MQTT Setup

  To connect the Nuki Smart Lock with HomeKit (via HomeBridge and MQTT), configure the MQTT integration in the Nuki app:

  1. Open the Nuki app and go to lock settings
  2. Enable MQTT integration with the following settings:
     - **Host:** `10.0.0.254`
     - **Username:** `mqtt`
     - **Password:** `mosquito`
     - **Automatic discovery:** Off (disable the first setting for automatic device discovery)

  Here is the configuration to add latch functionality to the Nuki Smart Lock Pro (5th generation):

  ```json
  {
    "type": "switch",
    "name": "Türöffner",
    "topics": {
      "getOnline": "nuki/4529E16F/connected",
      "getOn": "nuki/4529E16F/state",
      "setOn": "nuki/4529E16F/lockAction"
    },
    "onlineValue": "true",
    "offlineValue": "false",
    "integerValue": true,
    "onValue": "3",
    "offValue": "1",
    "resetStateAfterms": 3,
    "manufacturer": "Nuki",
    "model": "Smart Lock Pro",
    "serialNumber": "4529E16F",
    "firmwareRevision": "1.2.0",
    "accessory": "mqttthing"
  }
  ```

- ### [Pi-hole](https://pi-hole.net)

  The password for the Pi-hole admin interface can be set/unset via:

  ```sh
  docker exec -it pihole pihole setpassword
  ```

- ### NGINX

  Nginx is used to render a simple website at <http://linux.local>.

  - <https://stackoverflow.com/a/38783433/326984>
  - <https://github.com/ream88/nginx-test>

  Some useful commands during development:

  ```sh
  bun run build
  rsync -vr dist linux.local:/home/mario/docker/nginx/
  rsync -v nginx.conf linux.local:/home/mario/docker/nginx/
  ssh linux.local 'cd ~/docker && docker compose restart nginx'
  ```

- ### [WatchYourLan](https://github.com/aceberg/WatchYourLAN)

## Setup

1. Install Docker:

   ```sh
   curl -fsSL https://get.docker.com | sudo sh
   sudo usermod -aG docker $USER
   ```

2. Start all services:

   ```sh
   cd ~/docker
   docker compose up -d
   ```

## Links

- <https://docs.docker.com/compose/>

## License

[MIT](LICENSE.md)
