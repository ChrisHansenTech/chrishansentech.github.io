---
title: "Automating Mealtime: My Aqara Pet Feeder Setup for Three Dogs"
date: 2025-02-26 10:30:00 -500
categories: ["Home Assistant"]
tags: home-assistant aqara pets
image:
  path: /assets/img/headers/2025-02-26-Header.webp
  lqip: data:image/webp;base64,UklGRrgAAABXRUJQVlA4IKwAAABQBACdASoUAAsAPpE4l0eloyIhMAgAsBIJYwCdMoAlrxW6i1dFeMjeqaz4AP6X7NFl4cnZgyOT4ckh9nguj9x+MixhDeISLaPbCrCm7cRRWGskvc4iUMx5ywkY2l2lNBn0t1cRnWnhrslkFwMpi7HK1VFuummWteGRfzn2MDHSWZRPvQL/+C3GrGV9dgMp2c5LvD/4ayGRYVwXWa2drqRd5E00dg5TAc8N5AAA
--- 

Being a tech lead means juggling meetings, deadlines, and constant problem-solving—sometimes all at once. Add three hungry dogs with synchronized feeding times into the mix, and even the best daily schedule can fall apart. That’s when I turned to automation. With Aqara smart feeders and Home Assistant, I built a system that ensures Jax, Lincoln, and Reagan get fed on time, every time—no matter how hectic my day gets.  

## What I Use  

>**Affiliate Disclosure**  
Chris Hansen Tech is a participant in the Amazon Services LLC Associates Program, an affiliate advertising program designed to provide a means for sites to earn advertising fees by advertising and linking to Amazon.com. As an Amazon Associate, I earn from qualifying purchases.
{: .prompt-info }  

- [Aqara Smart Pet Feeder C1](https://amzn.to/41xvL8D)  
- [SONOFF Zigbee 3.0 USB Dongle Plus](https://amzn.to/4i9eR6M)  

Since I have three dogs and three pet feeders, I also installed a [USB wall outlet](https://amzn.to/3XdiT5Y) to make it easier to plug them all in the same area. I covered the outlet with an [outlet cover](https://amzn.to/4ic5kMd) to keep the dogs and kids from messing with the plugs.  

## Connecting to Home Assistant  

The Aqara Pet Feeder works with both ZHA and Zigbee2MQTT. I use Zigbee2MQTT for my Zigbee network in Home Assistant.  

To connect the feeder, enable device pairing in Zigbee2MQTT, then hold the reset button on the Aqara Pet Feeder for five seconds. You'll see it join in Zigbee2MQTT.  

Once the pet feeder is connected, you can set up a feeding schedule in the Zigbee2MQTT interface. This saves the schedule to the feeder itself, allowing it to function even when your smart home is offline. In Home Assistant, the schedule is only displayed as an MQTT device entity.  

In Home Assistant, I configure the following settings for the feeder:  
- **Child lock**: Prevents my toddler from playing with the buttons.  
- **Mode**: Set to manual since I use an automation for the feeding schedule.  
- **Portion weight (grams)**: Used in statistics to track daily feeding amounts.  
- **Number of portions per feeding**: Configured based on food measurements.

![Device configuration](/assets/img/posts/2025-02-26-DeviceSettings.webp){: .shadow }
_Device Configuration_

>To determine the portion weight and number of portions, I measured the usual amount I feed my dogs (typically a quarter cup) and then weighed a single portion dispensed by the feeder. In my case, one portion was 8 grams, so I set five portions to match the weight of a quarter cup.  
{: .prompt-tip }

## Automation  

I wanted to allow manual feeding in case the dogs wanted to eat earlier than scheduled or if we had to leave during their normal mealtime. However, I didn’t want the automatic feeding to trigger if a manual feeding had occurred within the last hour. To achieve this, I created a `datetime` helper.  

![Datetime helper](/assets/img/posts/2025-02-26-DatetimeHelper.webp){: width="690" height="494" }{: .shadow }
_Datetime Helper_

I then created a script that executes feeding for each feeder and updates the `datetime` helper with the current timestamp.  

```yaml
{% raw %}
alias: Feed dogs
icon: mdi:dog
sequence:
  - target:
      entity_id:
        - select.jax_pet_feeder_feed
        - select.lincoln_pet_feeder_feed
        - select.reagan_pet_feeder_feed
    action: select.select_first
    data: {}
  - target:
      entity_id: input_datetime.dogs_last_feed
    data:
      timestamp: "{{ now().timestamp() }}"
    action: input_datetime.set_datetime
mode: single
{% endraw %}
```  

Finally, I created an automation for the feeding schedule that only triggers the script if the last feeding timestamp is more than an hour ago.  

```yaml
{% raw %}
alias: Feed Dogs
description: Feed the dogs based on the schedule.
triggers:
  - trigger: time
    at:
      - "05:30:00"
      - "10:30:00"
      - "13:30:00"
      - "16:30:00"
      - "18:30:00"
conditions:
  - condition: template
    value_template: >-
      {{ now() -
      as_datetime(state_attr('input_datetime.dogs_last_feed','timestamp')) >=
      timedelta(hours=1) }}
actions:
  - action: script.feed_dogs
    data: {}
mode: single
{% endraw %}
```  

>If you use the visual editor, you’ll need to create a separate time trigger for each feeding. Multiple times in one trigger can only be done in YAML.  
{: .prompt-info }

## Manual Feeding  

To trigger a manual feeding, I attached the script to a button on my mobile dashboard.  

Additionally, I exposed the `feed_dogs` script to Alexa. In the Alexa app, scripts show up as scenes. I then created a routine in Alexa so I can say, "Alexa, feed dogs," "Alexa, feed the dogs," or "Alexa, feed the puppies," and the feed dogs scene will activate, executing the script in Home Assistant.  

## Results & Observations  

Automating my dogs' mealtime has been one of my favorite automations. I no longer have to worry about forgetting to feed them when I’m busy. The dogs love it too—they run to the feeders as soon as they hear the food dropping into their bowls.  

I definitely recommend the Aqara Pet Feeder. I got my first one when we only had Jax, and it worked flawlessly for over a year before Lincoln and Reagan joined the family. When they did, I bought two more, and they’ve all worked great ever since.  

