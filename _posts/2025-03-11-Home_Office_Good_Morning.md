---
title: My Home Office Welcomes Me with a Personalized Morning Update
date: 2025-03-11 14:06:00 -500
categories: ["Home Assistant"]
tags: home-assistant automation bluetooth presense
image:
  path: /assets/img/headers/2025-03-11-Header.webp
  lqip: data:image/webp;base64,UklGRsYAAABXRUJQVlA4ILoAAABQBQCdASoUAAsAPpE4l0eloyIhMAgAsBIJbACdMoRwN4BTAUVULXmbf9l9BMu2qkX6U+AA/tz0F4IKFKnEBYFa29Y1GK8P3RGGu6L9HfC9Nf9Y9kqvBvFrrphw8vus9emBmB0iJUviG9rv2dAFhYVe3/8SSjGBKdu3oCxTPv00rt14xs/dpUPOsdo4bca+l19MHGDgXVsWZC9sYzVd6zyVj/qbpm1+t0O7ZhrshzAD4+Jno0YlOcYAAAA=

---

Mornings in my house are busy—with two young kids and three dogs, there’s always something going on. To stay on track, I built a desktop dashboard for my home office, but I wanted something hands-free to remind me of my schedule without checking a screen. That’s why I created a Good Morning Message that plays on my HomePod mini when I enter my office, giving me the weather, calendar updates, and important events. In this post, I’ll show you how I set it up using Home Assistant, Bermude BLE, and Chime TTS to make mornings smoother.

## What I Used for This Project

- [ESPHome Integration](https://www.home-assistant.io/integrations/esphome/)
- [Private BLE Device Integration](https://www.home-assistant.io/integrations/private_ble_device)
- [Bermuda BLE Trilateration - HACS](https://github.com/agittins/bermuda)
- [Chime TTS - HACS](https://nimroddolev.github.io/chime_tts/) – To create a more natural announcement
- [Home Assistant Cloud](https://www.nabucasa.com/) – For Nabu Casa Cloud TTS
- [Apple HomePod mini](https://www.apple.com/shop/buy-homepod/homepod-mini)
- [ELEGOO 3PCS ESP-32 Development Board](https://amzn.to/3FeKUEl)
- [Aqara Presence Sensor FP2](https://amzn.to/4hgxC7j)

## Setting Up the ESP32 Boards

1. Install the ESPHome integration in Home Assistant.
2. Install ESPHome on the ESP32 boards as a **Bluetooth Proxy**. You can use either the [ESPHome Ready-Made Projects](https://esphome.io/projects/) website or the ESPHome Builder add-on for Home Assistant.
3. After adding the Bluetooth proxies to Home Assistant, assign them to **Areas**. These Areas will be used by the **Bermuda BLE Trilateration** integration to determine where devices are detected.

![ESPHome](/assets/img/posts/2025-03-11-ESPHome.webp){: w="1200" h="182" .shadow lqip="data:image/webp;base64,UklGRngAAABXRUJQVlA4WAoAAAAQAAAAEwAAAgAAQUxQSBcAAAABD6AQQADE37nRiIgYCAlSiM2I/kdGBgBWUDggOgAAADADAJ0BKhQAAwA+kTiXR6WjIiEwCACwEglpAABb7CSXawAA/vePJ7cbrKtGT4/ZY6IXsYGUqKtggAA=" }
_ESP Home Builder_

## Private BLE Integration

Since everyone in my home uses Apple devices, I am using the **Private BLE integration**. Follow the instructions in the [integration’s documentation](https://www.home-assistant.io/integrations/private_ble_device/#getting-your-identity-resolving-key-irk) to retrieve the **Identity Resolving Key (IRK)** for your devices.

Once you have the IRK, add an entry for your device key in the integration.

![Private BLE](/assets/img/posts/2025-03-11-PrivateBLE.webp){: w="669" h="226" .shadow lqip="data:image/webp;base64,UklGRoAAAABXRUJQVlA4WAoAAAAQAAAAEwAABwAAQUxQSBQAAAABD3Dt/4iIICQgaP6vNiGi/ykxJFZQOCBGAAAAsAIAnQEqFAAIAD6ROphHpaMioTAIALASCWcAAHsgAP71AMy0eZB5E/wwDlXHYOZTfWhvwZfjW0fUpjSMsC3jrTglFud0AA==" }
_Add private BLE entry_

## Bermuda BLE Trilateration

1. Install **Bermuda BLE Trilateration** from HACS.
2. Restart Home Assistant.
3. Open **Devices & Services**.
4. Add the **Bermuda BLE Trilateration** integration.

After installation, the integration will automatically discover your **Bluetooth Proxies**.

### Configuring Bermuda BLE Trilateration

1. In the Bermuda BLE Trilateration integration settings, use **Select Devices** to choose which devices you want to track.
2. This will create additional entities for each tracked Bluetooth device, showing:
    - The **area** where the device is detected
    - The **distance** from each BLE proxy

>The USB Bluetooth adapter on your Home Assistant server does not work well with this integration, as it does not include a timestamp in the advertisement packets. **Area distance tracking is not supported** when using the server’s Bluetooth.
{: .prompt-tip}

![Bermuda BLE Config](/assets/img/posts/2025-03-11-BermudaConfig.webp){: w="588" h="725" .shadow lqip="data:image/webp;base64,UklGRowAAABXRUJQVlA4WAoAAAAQAAAADwAAEwAAQUxQSCUAAAABHyCmIXObDnK6RUQ0QiEjSRLSIq72d3cFcAIR/8iLun7JYsAAFZQOCBAAAAA0AMAnQEqEAAUAD6ROpdHpaMiITAIALASCWkAAFvpPDjOAp8qaLQAAP72nkM9OPh84oqzYiIrfPjfGCPbyNIAAA==" }
_Bermuda BLE Config_

## Setting Up Chime TTS

1. Install **Chime TTS** from HACS.
2. Restart Home Assistant.
3. Open **Devices & Services**.
4. Add the **Chime TTS** integration.
5. Configure **Chime TTS** and set the **default TTS platform**.

## The Announcement Script

I use the **AccuWeather** integration for weather updates and the **CalDAV integration** to retrieve my calendar events.

My morning announcement includes:

- The **date**
- The **current and high temperatures**
- The **daily and nightly weather conditions**
- Events from my **family and work calendars**

### How the Script Works

1. **Retrieving Calendar Events:** The script fetches events from my **family** and **work** calendars for the next **12 hours**. I chose 12 hours to avoid announcements about events happening the next day if I enter my office later in the morning. The events are stored in the `agenda` variable.
2. **Generating the Audio Announcement:** The **Chime TTS** integration generates the spoken message, using a **chime sound** at the beginning to signal that an announcement is coming.
3. **Personalized Greeting:** The message dynamically selects “Good morning,” “Good afternoon,” or “Good evening” based on the current time.

```yaml
{% raw %}
sequence:
  - action: calendar.get_events
    target:
      entity_id:
        - calendar.family_jackie
        - calendar.christopher_hansen
    data:
      duration:
        hours: 12
        minutes: 0
        seconds: 0
    response_variable: agenda
  - action: chime_tts.say
    metadata: {}
    data:
      chime_path: bells
      announce: false
      message: >-
        {% set current_hour = now().strftime('%H') | int %}
        {% if current_hour < 12 %}
          {% set greeting = 'Good morning' %}
        {% elif current_hour < 18 %}
          {% set greeting = 'Good afternoon' %}
        {% else %}
          {% set greeting = 'Good evening' %}
        {% endif %}
        
        {{ greeting }} Chris, it is {{ now().date() }}. 

        Currently the Real Feel temperature is {{ state_attr('weather.home',
        'temperature') }} degrees, with a high of {{
        states('sensor.home_realfeel_temperature_max_day_0') }}

        Today it will {{ states('sensor.home_condition_day_0') }}

        Tonight {{ states('sensor.home_condition_night_0') }}

        Your family calendar

        {% for event in agenda["calendar.family_jackie"]["events"] %}   
          {{ (event.start |   as_datetime).strftime('%I:%M %p') }} {{event.summary }}. 
        {% endfor %}

        Your work calendar

        {% for event in agenda["calendar.christopher_hansen"]["events"] %}   
          {{ (event.start |   as_datetime).strftime('%I:%M %p') }} {{event.summary }}. 
        {% endfor %}

        Have a productive day!
    target:
      entity_id: media_player.office
alias: Good morning Chris
description: Good morning message for Chris
{% endraw %}
```

## Automations

To ensure the **Good Morning Message** plays only **once per day**, I created an **input boolean**.

### **Good Morning Chris Automation**

This automation triggers the script when:

- The **presence sensor** detects the office becoming occupied.
- My **Apple Watch** is detected in the office area.
- The **input boolean** is **off**.

```yaml
{% raw %}
alias: Good Morning Chris
triggers:
  - trigger: state
    entity_id: binary_sensor.office_presence_sensor
    to: "on"
conditions:
  - condition: state
    entity_id: sensor.chris_apple_watch_area
    attribute: area_name
    state: Office
  - condition: state
    entity_id: input_boolean.good_morning_message
    state: "off"
actions:
  - action: script.good_morning_chris
  - action: input_boolean.turn_on
    target:
      entity_id: input_boolean.good_morning_message
mode: single
{% endraw %}
```

### **Nightly Reset Automation**

At **4:00 AM**, an automation resets the `good_morning_message` boolean, allowing the announcement to trigger again the next day.

```yaml
{% raw %}
alias: Nightly helper reset
triggers:
  - trigger: time
    at: "04:00:00"
conditions: []
actions:
  - action: input_boolean.turn_off
    target:
      entity_id: input_boolean.good_morning_message
mode: single
{% endraw %}
```

## Conclusion

This project has been a great addition to my morning routine. It keeps me **on track** by giving me an **audible summary** of my day while I settle into work. Combined with my **desk dashboard**, I now have a complete overview of my agenda every morning.

>**Affiliate Disclosure**  
Chris Hansen Tech is a participant in the Amazon Services LLC Associates Program, an affiliate advertising program designed to provide a means for sites to earn advertising fees by advertising and linking to Amazon.com. As an Amazon Associate, I earn from qualifying purchases.
{: .prompt-info }

