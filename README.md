# Playing Azan on Google/Nest Home/Mini using Home Assistant

## Introduction
These configurations can be used to configure playing Azan on your Google Home / Mini Smart Speakers

## Pre-requisits
The following needs to be already setup before proceeding with this setup:

- Home Assistant
- Google Home / Mini or Nest Home / Mini already setup


## Steps

### Step - 1: In the `configuration.yaml`, please add the following: (*Please add Asr and Isha accordingly, accordingly*)

```
### Input slider to control gateway volume ###
input_number:
  gateway_volume:
    name: Volume
    initial: 10
    min: 0
    max: 100
    step: 1
    icon: mdi:volume-high

### Input slider to control loop delay ###
  loop_delay:
    name: "Loop Delay"
    initial: 1
    min: 0
    max: 15
    step: 1
    icon: mdi:loop


  azan_volume_maghrib:
    name: "Azan Volume Maghrib"
    initial: 0.8
    min: 0
    max: 1
    step: 0.01
    icon: mdi:volume-high

  azan_volume_fajr:
    name: "Azan Volume Fajr"
    initial: 0.01
    min: 0
    max: 1
    step: 0.01
    icon: mdi:volume-high

  azan_volume_zuhr:
    name: "Azan Volume Zuhar"
    initial: 0.8
    min: 0
    max: 1
    step: 0.01
    icon: mdi:volume-high

# It is recommended to put your automation in an external file `automations.yaml`.
automation: !include automations.yaml

# It is recommended to put your sensors in an exteranl file `sensors.yaml`. 
sensor: !include sensors.yaml


### Input select to select Azan file to play ###
input_select: !include input_select.yaml
```

### Step-2: `sensors.yaml`

```
- platform: rest
  name: Prayer Times
  resource: 'http://api.aladhan.com/v1/timings/:date_or_timestamp?latitude=45.668399&longitude=-73.944645&method=0&midnightMode=1'
  json_attributes:
    - data
  value_template: '{{ value_json.status }}'
  scan_interval: 3000
- platform: template
  sensors:
    sunrise:
      friendly_name: Sunrise Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Sunrise"] }}'
    fajr:
      friendly_name: Fajr Prayer Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Fajr"] }}'
    zhuhr:
      friendly_name: Zhuhr Prayer Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Dhuhr"] }}'
    asr:
      friendly_name: Asr Prayer Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Asr"] }}'
    sunset:
      friendly_name: Sunset Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Sunset"] }}'
    maghrib:
      friendly_name: Maghrib Prayer Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Maghrib"] }}'
    isha:
      friendly_name: Isha Prayer Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Isha"] }}'
    midnight:
      friendly_name: Midnight Time
      value_template: '{{ states.sensor.prayer_times.attributes["data"]["timings"]["Midnight"] }}'
```

### Step-2: `input_select.yaml`

```
azan_select_fajr:
  name: Azan Fajr
  options:
      - "azan1.mp3 - Abathar"
      - "azan2.mp3 - Iran"
      - "azan3.mp3 - Iraq Old"
  icon: mdi:bell-rin

azan_select_zuhr:
  name: Azan Zuhr
  options:
      - "azan1.mp3 - Abathar"
      - "azan2.mp3 - Iran"
      - "azan3.mp3 - Iraq Old"
  icon: mdi:bell-rin

azan_select_maghrib:
  name: Azan Maghrib
  options:
      - "azan1.mp3 - Abathar"
      - "azan2.mp3 - Iran"
      - "azan3.mp3 - Iraq Old"
  icon: mdi:bell-rin

```

### Step-3: In the `automations.yaml` add the following: (*Please add Asr and Isha accordingly, accordingly*)

Note: 
- Replace the IP address 192.168.2.30 with your RPI IP address where Home Assistant is running
- Replace the entity_ids with yours accordingly

The Automation accomplishes the following:
1. Set the Volume of the Smart Speaker, based on the slider value
2. Play Azan
3. Shut-down TV or any media (You can remove that part if not needed)

```
- alias: Play Fajr Azan
  trigger:
  - platform: time_pattern
    seconds: '30'
  condition:
  - condition: template
    value_template: '{{ now().time().strftime("%H:%M") == states.sensor.fajr.state }}'
  action:
    - service: media_player.volume_set
      data_template:
        entity_id: media_player.azan_speakers
        volume_level: '{{states.input_number.azan_volume_fajr.state}}'
    - service: media_player.play_media
      data_template:
        entity_id: media_player.azan_speakers
        media_content_id: http://192.168.2.30:8123/local/{{states.input_select.azan_select_fajr.state.split(' ')[0]}}
        media_content_type: audio/mp3
# Turn off TV
    - service: switch.turn_off
      data:
        entity_id: switch.18480742cc50e32c353a
# Turn off Mibox
    - service: media_player.turn_off
      data:
        entity_id: media_player.mibox3
        
- alias: Play Maghrib Azan
  trigger:
  - platform: time_pattern
    seconds: '30'
  condition:
  - condition: template
    value_template: '{{ now().time().strftime("%H:%M") == states.sensor.maghrib.state }}'
  action:
    - service: media_player.volume_set
      data_template:
        entity_id: media_player.azan_speakers
        volume_level: '{{states.input_number.azan_volume_maghrib.state}}'
    - service: media_player.play_media
      data_template:
        entity_id: media_player.azan_speakers
        media_content_id: http://192.168.2.30:8123/local/{{states.input_select.azan_select_maghrib.state.split(' ')[0]}}
        media_content_type: audio/mp3
# Turn off TV
    - service: switch.turn_off
      data:
        entity_id: switch.18480742cc50e32c353a
# Turn off Mibox
    - service: media_player.turn_off
      data:
        entity_id: media_player.mibox3
    - service: notify.alexa_media
      data:
        target:
          - media_player.home
        data:
          type: announce
        message: "Prayer Time..."


- alias: Play Zuhr Azan
  trigger:
  - platform: time_pattern
    seconds: '30'
  condition:
  - condition: template
    value_template: '{{ now().time().strftime("%H:%M") == states.sensor.zhuhr.state }}'
  action:
    - service: media_player.volume_set
      data_template:
        entity_id: media_player.azan_speakers
        volume_level: '{{states.input_number.azan_volume_zuhr.state}}'
    - service: media_player.play_media
      data_template:
        entity_id: media_player.azan_speakers
        media_content_id: http://192.168.2.30:8123/local/{{states.input_select.azan_select_zuhr.state.split(' ')[0]}}
        media_content_type: audio/mp3
# Turn off TV
    - service: switch.turn_off
      data:
        entity_id: switch.18480742cc50e32c353a
# Turn off Mibox
    - service: media_player.turn_off
      data:
        entity_id: media_player.mibox3
```
