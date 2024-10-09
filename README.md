# Home Assistant Automated Hot Water 

## Background

In Israel, it is a [requirement](https://en.wikipedia.org/wiki/Solar_power_in_Israel#:~:text=After%20the%20energy%20crisis%20in,towers%20with%20insufficient%20roof%20area.) to have a solar water heater. These water heaters also have an option for electrically heating the water.
In the summer, the electrical heating is not usually required. In the winter, they are usually required. In the spring and fall the electricity  may be required based on weather conditions.

## The Goal

Given that the weather changes and the family doesn't want to have a cold shower, I decided to implement (at the recommendation of a friend Tal) a Home Assistant Automation to determine if the Dude (Hebrew for Hot Water Heater) should be turned on or not and for how long.

![ha-dude](https://github.com/user-attachments/assets/f274bf26-50af-438f-98d1-930cf03f9a0c)

## How does it work

First, I needed to install a [Switcher](https://switcher.co.il/) Hot Water smart switch. This was available locally and I could connect it to the Home Assistant as an integration.
At 3am and 3pm (times I decided on for family patterns), an automation will kick off. 
- If the temperature is below 25C (77F) OR if there is more than 15% cloud cover, turn on the dude. (It also sends me a notification)
  - If neither of those conditions are met, then it does nothing.
- After the dude has been on for an hour (no matter how it was turned on, even manually or via a smart home device like Alexa)
  - If the temperate is above 19C (66F), then do nothing
- After the dude has been on for an hour and a half (again, no matter how it was turned on)
  - turn it off

With this, the dude is only coming on when needed (to electrically heat the water when there is not enough sun or it's too cold out) and when it's severly cold out, the dude is staying on longer

## Automations

### Dude Enable
```
alias: "Dude-enable "
description: ""
trigger:
  - platform: time
    at: "03:00:00"
    enabled: true
  - platform: time
    at: "15:00:00"
condition:
  - condition: or
    conditions:
      - condition: numeric_state
        entity_id: weather.forecast_home
        attribute: temperature
        below: 25
      - condition: numeric_state
        entity_id: weather.forecast_home
        attribute: cloud_coverage
        above: 15
action:
  - type: turn_on
    device_id: <ID>
    entity_id: <ID>
    domain: switch
  - action: notify.<method>
    metadata: {}
    data:
      message: Dude Turned On
      title: Home Assistant
mode: single
```

### Dude-Auto-Off-60-above-65

```
alias: Dude-Auto-Off-60-above-65
description: ""
trigger:
  - platform: device
    type: turned_on
    device_id: <ID>
    entity_id: <ID>
    domain: switch
    for:
      hours: 1
      minutes: 0
      seconds: 0
condition:
  - condition: numeric_state
    entity_id: weather.forecast_home
    attribute: temperature
    above: 19
action:
  - type: turn_off
    device_id: <ID>
    entity_id: <ID>
    domain: switch
  - action: notify.<method>
    metadata: {}
    data:
      title: Home Assistant
      message: Dude Automation, turned off after 60 min (temp above 65F)
mode: single

```

### Dude-Auto-Off-90

```
alias: Dude-Auto-Off-90
description: ""
trigger:
  - platform: device
    type: turned_on
    device_id: <ID>
    entity_id: <ID>
    domain: switch
    for:
      hours: 1
      minutes: 30
      seconds: 0
condition: []
action:
  - type: turn_off
    device_id: <ID>
    entity_id: <ID>
    domain: switch
  - action: notify.<method>
    metadata: {}
    data:
      title: Home Assistant
      message: Dude Automation, turned off after 90 min
mode: single

```
