# Playing Azan on Google/Nest Home/Mini using Home Assistant

## Introduction
These configurations can be used to configure playing Azan on your Google Home / Mini Smart Speakers

## Pre-requisits
The following needs to be already setup before proceeding with this setup:

- Home Assistant
- Google Home / Mini or Nest Home / Mini already setup


## Steps

### Step - 1: In the `configuration.yaml`, please add the following: (*Please add Asr and Isha accordingly*)

```
### Input slider to control speaker volume ###
input_number:
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

### Step-2: Create `sensors.yaml` and add the following to it.

I am using `api.aladhan.com` to get the updated timings (updated every `3000 seconds` - This can be increased to 24hours as the times won't change frequently). 

Note a few things:

1. Change the `method` accoding to your school of fiqh. I have set it to `0 - Shia Ithna-Ansari`
2. Change the `logitute` and `latitude` based on your location
3. Change the `midnightMode` accordingy. I have set it to `1 for Jafari (Mid Sunset to Fajr)`

For complete API documentation, please refer here:
https://aladhan.com/prayer-times-api#GetTimings

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

### Step-3: Create `input_select.yaml` and add the following yaml to it. (*Please add Asr and Isha accordingly to it*)

These inputs will be available to use in your UI. 

Note:
1. place the Azan MP3 file under `www/azan1.mp3
2. If you would like to have multiple options, to select which Azan to be played at what time, you and add. I have added three azans as follows:
 - `www/azan1.mp3`
 - `www.azan2.mp3`
 - `www.azan3.mp3`
 
3. Make sure the options value below has this convention:
```
"<exact_azan_file_name> - <custom title>"
e.g.
"azan1.mp3 - Abathar"
```

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

### Step-4: In the `automations.yaml` add the following: (*Please add Asr and Isha accordingly, accordingly*)

Note: 
- Replace the IP address 192.168.2.30 with your RPI IP address where Home Assistant is running
- Replace the entity_ids with yours accordingly

The Automation accomplishes the following:
1. Set the Volume of the Smart Speaker, based on the slider value
2. Play Azan (The Azan is also selected based on the Input Select
3. Shut-down TV or any media (You can remove that part if not needed)
4. The Automation runs every 30 second, so there might be a max delay of 30 seconds when it's time for prayers.

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
