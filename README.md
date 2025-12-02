# slow-wake-light
blueprint:
  name: Yavaş Sabah Lambası Açılma
  description: >
    Switch açar, ışığı önce %10 yakar ve hedef parlaklığa kadar yavaşça artırır.
  domain: automation

  input:
    wake_time:
      name: Çalışma Saati
      selector:
        time: {}

    target_light:
      name: Hedef Lamba
      selector:
        target:
          entity:
            domain: light

    target_switch:
      name: Açılacak Switch
      selector:
        target:
          entity:
            domain: switch

    final_brightness:
      name: Hedef Parlaklık (%)
      default: 65
      selector:
        number:
          min: 1
          max: 100
          step: 1
          unit_of_measurement: "%"

    step_increase:
      name: Parlaklık Artış Adımı (%)
      default: 5
      selector:
        number:
          min: 1
          max: 20
          step: 1

    step_delay:
      name: Artış Adımları Arası Bekleme (sn)
      default: 25
      selector:
        number:
          min: 1
          max: 120
          step: 1
          unit_of_measurement: seconds

trigger:
  - platform: time
    at: !input wake_time

action:

  - choose:
      - conditions:
          - condition: state
            entity_id: !input target_switch
            state: "off"
        sequence:
          - service: switch.turn_on
            target: !input target_switch
          - delay: "00:00:02"

  - service: light.turn_on
    target: !input target_light
    data:
      brightness_pct: 10

  - repeat:
      while:
        - condition: template
          value_template: >
            {% set light = (inputs.target_light.entity_id | list)[0] %}
            {% set current = state_attr(light, 'brightness') | int %}
            {% set target = (inputs.final_brightness | int) * 255 / 100 %}
            {{ current < target }}
      sequence:
        - delay:
            seconds: !input step_delay

        - service: light.turn_on
          target: !input target_light
          data:
            brightness: >
              {% set light = (inputs.target_light.entity_id | list)[0] %}
              {% set current = state_attr(light, 'brightness') | int %}
              {% set target = (inputs.final_brightness | int) * 255 / 100 %}
              {% set step = (inputs.step_increase | int) * 255 / 100 %}
              {{ [current + step, target] | min | int }}

mode: single
