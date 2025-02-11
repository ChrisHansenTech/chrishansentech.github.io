---
title: Integrate SharkIQ with Home Assistant using Alexa
date: 2025-02-11 11:10:00 -500
categories: ["Home Assistant"]
tags: homeassistant alexa
---

# Integrate SharkIQ with Home Assistant using Alexa

I have two SharkIQ robot vacuums and initially set up the SharkIQ integration in Home Assistant. Of course, as soon as I finished configuring everything the way I liked it, with dashboard buttons to control the vacuums, Shark decided to block third-party access to their API.

For a while, I resorted to using the SharkIQ app on my iPhone, but I quickly became frustrated with its slow performance and frequent sign-in prompts.

The other day, I came across a blog post by Garrett Morgan at [GarrettDevelops.com](https://garrettdevelops.com/essential-scripts-for-shark-clean-ha-integration-workaround/) that demonstrated how to use Google Home to integrate SharkIQ robots with Home Assistant. Since I don't use Google Home for anything else but have Alexa set up via Home Assistant Cloud, I decided to create my own version using Alexa.

## Requirements

To implement this integration, you'll need the following:

- **Home Assistant Cloud**
- **Alexa integration** for Home Assistant Cloud enabled
- **Home Assistant Community Store (HACS)**
- **Alexa Media Player** from HACS

## Creating a Template Vacuum

To enable vacuum controls in Home Assistant, we will create a [Template Vacuum](https://www.home-assistant.io/integrations/vacuum.template/).

1. In your `configuration.yaml` file, add the following line:
   ```yaml
   vacuum: !include vacuums.yaml
   ```
2. Create a `vacuums.yaml` file and add the following configuration:

```yaml
- platform: template
  vacuums:
    living_room:  # The ID for your robot vacuum.
      friendly_name: Clean Elizabeth
      unique_id: # Insert a GUID for the unique ID.
      start:
        action: script.vacuum_start
        data:
          robotname: Elizabeth
      pause:
        action: script.vacuum_pause
        data:
          robotname: Elizabeth
      return_to_base:
        action: script.vacuum_return_to_base
        data:
          robotname: Elizabeth
      locate:
        action: script.vacuum_locate_vacuum
        data:
          robotname: Elizabeth
      set_fan_speed:
        action: script.vacuum_set_fan_speed
        data:
          robotname: Elizabeth
          speed: "{{ fan_speed }}"
      fan_speeds:
        - ECO
        - NORMAL
        - MAX
```

Repeat this configuration for each vacuum if you have multiple units.

Restart Home Assistant to apply the new configuration, and a new vacuum entity will appear.

![Vacuum Entity](/assets/img/posts/2025-02-11-vacuum_entity.png)

Since the SharkIQ Alexa Skill will not provide status updates to Home Assistant, I did not include status templates. When you send a command to Alexa, the response is spoken through the Echo device, rather than being reflected in Home Assistant.

## Creating the Scripts

I created scripts for the following actions: **Start, Pause, Return to Base, and Set Fan Speed**. For the scripts to work correctly with the vacuum entity type, they must follow the proper naming convention. If you use different script names, Home Assistant will return an error.

To use these scripts, you will need the entity ID of the Alexa Media Player that will send the voice commands. Any response from Alexa will be played through this device.

We use the `media_player.play_media` action with `media_content_id`, which contains the voice command to send to Alexa, and `media_content_type` set to `custom`.

### Vacuum Start

This script must be named `script.vacuum_start`:

```yaml
sequence:
  - service: media_player.play_media
    target:
      entity_id: media_player.ecobee_smart_thermostat
    data:
      media_content_id: Alexa ask Shark to start {{ robotname }}
      media_content_type: custom
fields:
  robotname:
    selector:
      text: {}
    name: RobotName
    description: The name of the robot vacuum
    required: true
alias: Vacuum Start
```  

### Vacuum Pause

This script must be named `script.vacuum_pause`:

```yaml
sequence:
  - service: media_player.play_media
    target:
      entity_id: media_player.ecobee_smart_thermostat
    data:
      media_content_id: Alexa ask Shark to pause {{ robotname }}
      media_content_type: custom
fields:
  robotname:
    selector:
      text: {}
    name: RobotName
    description: The name of the robot vacuum
    required: true
alias: Vacuum Pause
```

### Vacuum Return to Base

This script must be named `script.vacuum_return_to_base`:

```yaml
sequence:
  - service: media_player.play_media
    target:
      entity_id: media_player.ecobee_smart_thermostat
    data:
      media_content_id: Alexa ask Shark to dock {{ robotname }}
      media_content_type: custom
fields:
  robotname:
    selector:
      text: {}
    name: RobotName
    description: The name of the robot vacuum
    required: true
alias: Vacuum Return to Base
```

### Vacuum Set Fan Speed

This script must be named `script.vacuum_set_fan_speed`. It includes a field for the speed setting.

```yaml
sequence:
  - service: media_player.play_media
    target:
      entity_id: media_player.christopher_s_echo_dot
    data:
      media_content_id: Alexa set mode to {{ speed }} on {{ robotname }}
      media_content_type: custom
fields:
  robotname:
    selector:
      text: {}
    name: RobotName
    description: The name of the robot vacuum
    required: true
  speed:
    selector:
      select:
        options:
          - ECO
          - NORMAL
          - MAX
    name: Speed
    description: The fan speed setting
alias: Vacuum Set Fan Speed
```

## Room Cleaning Script

I also created a script for cleaning specific rooms, which allows me to add buttons to the dashboard.

```yaml
sequence:
  - service: media_player.play_media
    target:
      entity_id: media_player.ecobee_smart_thermostat
    data:
      media_content_id: Alexa ask Shark to {{ cleantype }} the {{ room }} using {{ robotname }}
      media_content_type: custom
fields:
  cleantype:
    selector:
      select:
        options:
          - clean
          - matrix clean
    name: Clean Type
    description: Type of clean to perform
  room:
    selector:
      text: {}
    name: Room
    description: The room to clean
  robotname:
    selector:
      text: {}
    name: RobotName
    description: The name of the robot vacuum
alias: Robot Clean Command
```

## Summary

With this setup, your SharkIQ vacuum is back in Home Assistant! Now you can run commands quickly without dealing with the sluggish SharkIQ app. Feel free to customize and expand upon this integration, and most importantly—stay curious!

