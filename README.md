# Load Shedding

A Home Assistant integration to track your load schedding schedule.


> ℹ️ **_NOTE:_**  This integration makes use of [this Python library](https://gitlab.com/wernerhp/load-shedding) which only supports schedules for Eskom Direct customers.  If you can find your schedule on https://loadshedding.eskom.co.za/ then you'll have schedule info available.  
> If you are not an Eskom Direct customer, then a work-around is to find an Eskom Direct schedule which matches yours and use that instead.  There are no immediate plans to add other municipalities, but Merge Requests on [the library](https://gitlab.com/wernerhp/load-shedding) to expand support are welcome.

# HACS
[![hacs_badge](https://img.shields.io/badge/HACS-Default-41BDF5.svg)](https://github.com/hacs/integration)
1. Go to HACS Integrations on your Home Assitant instance
2. Select "+ Explore & Download Repositories" and search for "Load Shedding"
3. Select "Load Shedding" and "Download this repository with HACS"
![image](https://user-images.githubusercontent.com/2578772/167293308-d3ef2131-bc71-431e-a1ff-6e02f02af000.png)
4. Once downloaded, click the "My Integrations" button to configure the integration.  
[![Open your Home Assistant instance and start setting up a new integration.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=load_shedding)
5. Setup cards and automations

<a href="https://www.buymeacoffee.com/wernerhp" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png" alt="Buy Me A Coffee" style="height: auto !important;width: auto !important;" ></a>

# Manual Install
<details>
<summary>Expand</summary>

1. Download and unzip to your Home Assistant `config/custom_components` folder.
  <details>
  <summary>Screenshot</summary>
  
![image](https://user-images.githubusercontent.com/2578772/164681660-57d56fc4-4713-4be5-9ef1-bf2f7cf96b64.png)
  </details>
  
2. Restart Home Assistant.
3. Go to Settings > Devices & Services > + Add Integration (or click [![Open your Home Assistant instance and start setting up a new integration.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=load_shedding))
4. Search for 'Load Shedding' and follow the config flow.
5. If you're coming from a previous version of this integration, you may need to delete the `.json` files in `/config/.cache`.
<details>
  <summary>Screenshot</summary>
  
![image](https://user-images.githubusercontent.com/2578772/164681929-e3afc6ea-5821-4ac5-8fa8-eee04c819eb6.png)
  </details>
</details>

# Sensor
The load shedding sensor State will always reflect the current load shedding stage.  
i.e When load shedding is suspended, it will show **No Load Shedding**.  When Stage 2 is active, it will show **Stage 2**.  
> Since the schedules differ depending on the Stage, the `next_start`, `next_end` and `schedule` times only show once there is an active Stage as it needs to know which stage to query.  

When load shedding ends the last state's schedule will remain in the sensor's Attributes until your restart Home Assistant.
<details>
  <summary>Screenshot</summary>

![image](https://user-images.githubusercontent.com/2578772/168296185-af97139a-6170-4273-8414-18a2f9d140c2.png)
  
![image](https://user-images.githubusercontent.com/2578772/168240243-27c7fd3b-d7e2-4918-a74d-97b13155aa90.png)
  </details>

# Cards
I created this card with the help of [template-entity-row](https://github.com/thomasloven/lovelace-template-entity-row)  
<details>
  <summary>Screenshot</summary>
 
![image](https://user-images.githubusercontent.com/2578772/168237722-9ce09b94-310c-4f08-bcc1-40a7ffe257b0.png)

  </details>
<details>
  <summary>Code</summary>
  
```yaml
type: entities
entities:
  - type: custom:template-entity-row
    icon: mdi:lightning-bolt-outline
    name: Milnerton
    state: '{{states(''sensor.load_shedding_milnerton'')}}'
    active: '{{ not is_state(''sensor.load_shedding_milnerton'', ''No Load Shedding'') }}'
  - type: custom:template-entity-row
    icon: mdi:timer-outline
    name: Next
    state: >-
      {{ state_attr('sensor.load_shedding_milnerton', 'next_start') |
      as_timestamp | timestamp_custom("%H:%M") }} - {{
      state_attr('sensor.load_shedding_milnerton', 'next_end' ) | as_timestamp |
      timestamp_custom("%H:%M") }}
    condition: '{{ not is_state(''sensor.load_shedding_milnerton'', ''No Load Shedding'') }}'
  - type: custom:template-entity-row
    icon: mdi:timer-sand
    name: Time Until
    state: >-
      {{ (state_attr('sensor.load_shedding_milnerton', 'next_start') |
      as_timestamp - now().strftime('%Y-%m-%d %H:%M%z') | as_timestamp)|
      timestamp_custom("%Hh%M", False) }}
    condition: '{{ not is_state(''sensor.load_shedding_milnerton'', ''No Load Shedding'') and state_attr(''sensor.load_shedding_milnerton'', ''next_start'') != None }}'
show_header_toggle: false
```
  </details>

# Automation Ideas
These are just some automations I've got set up.  They are not perfect and will require some tweaking on your end.  Feel free to contribute your automations ideas and custom panels by posting them on [this Issue thread](https://github.com/wernerhp/ha_integration_load_shedding/issues/5)

In order to clean up some of the automation code, you can define a [template sensors](https://www.home-assistant.io/integrations/template) for "Next Start" and "Next End" and reference them in automations instead.
<details>
  <summary>Code</summary>
  
  ```
- platform: template
  sensors:
    load_shedding_next_start:
      friendly_name: "Next Start"
      value_template: >-
        {% if state_attr('sensor.load_shedding_milnerton', 'next_start') != None %}
          {{ state_attr("sensor.load_shedding_milnerton", "next_start") | as_datetime - now().strftime("%Y-%m-%d %H:%M%z") | as_datetime }}
        {% else %}
          Unknown
        {% endif %}
    load_shedding_next_end:
      friendly_name: "Next End"
      value_template: >-
        {% if state_attr('sensor.load_shedding_milnerton', 'next_start') != None %}
          {{ state_attr("sensor.load_shedding_milnerton", "next_start") | as_datetime - now().strftime("%Y-%m-%d %H:%M%z") | as_datetime }}
        {% else %}
          Unknown
        {% endif %}

  ```
  </details>

### Announce Load Shedding stage changes on speakers and push notifications.
<details>
  <summary>Code</summary>
  
```yaml
alias: Load Shedding (Stage)
description: ''
trigger:
  - platform: template
    value_template: '{{ states.sensor.load_shedding_milnerton.state }}'
condition: []
action:
  - choose:
      - conditions:
          - condition: time
            after: input_datetime.sleep
            before: '23:59:59'
          - condition: time
            after: '00:00:00'
            before: input_datetime.wake
        sequence:
          - wait_for_trigger:
              - platform: time
                at: input_datetime.wake
            continue_on_timeout: false
    default: []
  - service: notify.mobile_app_nokia_8_sirocco
    data:
      title: Load Shedding
      message: '{{ states.sensor.load_shedding_milnerton.state }}'
  - service: tts.home_assistant_say
    data:
      entity_id: media_player.assistant_speakers
      cache: true
      message: >-
        {% if is_state("sensor.load_shedding_milnerton", "No Load Shedding") %}
        Load Shedding suspended {% else %} Load Shedding {{
        states.sensor.load_shedding_milnerton.state }} {% endif %}
mode: single
```
  </details>
  
### 15 minutes warning on speaker and telegram before load shedding starts.
<details>
  <summary>Code</summary>
  
```yaml
alias: Load Shedding (Warning)
description: ''
trigger:
  - platform: template
    value_template: >-
      {{ state_attr('sensor.load_shedding_milnerton', 'next_start') |
      as_datetime - now().strftime('%Y-%m-%d %H:%M%z') | as_datetime ==
      timedelta(minutes=15) }}
condition:
  - condition: and
    conditions:
      - condition: time
        after: input_datetime.alarm
        before: input_datetime.sleep
      - condition: not
        conditions:
          - condition: state
            entity_id: sensor.load_shedding_milnerton
            state: Unknown
          - condition: state
            entity_id: sensor.load_shedding_milnerton
            state: No Load Shedding
action:
  - service: telegram_bot.send_message
    data:
      message: Load Shedding starts in 15 minutes.
      title: Load Shedding
  - service: media_player.volume_set
    data:
      volume_level: 0.7
    target:
      entity_id: media_player.assistant_speakers
  - service: tts.home_assistant_say
    data:
      entity_id: media_player.assistant_speakers
      message: Load Shedding starts in 15 minutes.
      cache: true
mode: single
```
</details>

    
### Dim lights or turn off devices before load shedding and turn them back on afterwards.

### Update your Slack status

Setup a REST Command and two automations to set your Slack status when Load Shedding starts and ends.

<details>
  <summary>Code</summary>
  
`secrets.yaml`
```yaml
slack_token: Bearer xoxp-XXXXXXXXXX-XXXXXXXXXXXX-XXXXXXXXXXXXX-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```  
  
  [rest_command](https://www.home-assistant.io/integrations/rest_command/)
  
```yaml
slack_status:
  url: https://slack.com/api/users.profile.set
  method: POST
  headers:
    authorization: !secret slack_token
    accept: "application/json, text/html"
  payload: '{"profile":{"status_text": "{{ status }}","status_emoji": "{{ emoji }}"}}'
  content_type: "application/json; charset=utf-8"
  verify_ssl: true
```
</details>

<details>
  <summary>Code</summary>
  
```yaml
alias: Load Shedding (Start)
description: ''
trigger:
  - platform: template
    value_template: >-
      {{ state_attr('sensor.load_shedding_milnerton', 'next_start') |
      as_datetime - now().strftime('%Y-%m-%d %H:%M%z') | as_datetime ==
      timedelta(minutes=0) }}
condition:
  - condition: not
    conditions:
      - condition: state
        entity_id: sensor.load_shedding_milnerton
        state: Unknown
      - condition: state
        entity_id: sensor.load_shedding_milnerton
        state: No Load Shedding
action:
  - service: rest_command.slack_status
    data:
      emoji: ':loadsheddingtransparent:'
      status: >-
        Load Shedding until {{
        (state_attr('sensor.load_shedding_milnerton','next_end') | as_datetime |
        as_local).strftime('%H:%M (%Z)') }}
mode: single
```
</details>

<details>
  <summary>Code</summary>
  
```yaml
alias: Load Shedding (End)
description: ''
trigger:
  - platform: template
    value_template: >-
      {{ state_attr('sensor.load_shedding_milnerton', 'next_end') | as_datetime
      - now().strftime('%Y-%m-%d %H:%M%z') | as_datetime == timedelta(minutes=0)
      }}
condition:
  - condition: not
    conditions:
      - condition: state
        entity_id: sensor.load_shedding_milnerton
        state: Unknown
      - condition: state
        entity_id: sensor.load_shedding_milnerton
        state: No Load Shedding
action:
  - service: rest_command.slack_status
    data:
      emoji: ':speech_balloon:'
      status: is typing...
mode: single

```
</details>
