---
layout: post
comments: true
title:  "Managing groups visibility in Home Assistant"
twitter_text: "Managing groups visibility in @home_assistant"
date:   2018-03-18 19:54:00
tags: HomeAssistant
permalink: /managing-groups-visibility-in-home-assistant/
---
<!-- markdownlint-disable html -->
After I wrote [Dynamically hide / unhide an entire view in Home Assistant](https://bonani.tech/dynamically-hide-unhide-an-entire-view/), many people asked me if it is possible to do the same with the cards \(groups\). It is, and it's easy too.

<br />

Here is an example code:

<br />

```yaml
{% raw %}
group:
  default_view:
    view: yes
    entities:
      - group.test_group
      - group.test_subgroup1
      - group.test_subgroup2
      - group.test_subgroup3

  test_group:
    name: "Test Group"
    control: hidden
    entities:
      - input_boolean.test_input_boolean1
      - input_boolean.test_input_boolean2
      - input_boolean.test_input_boolean3

  test_subgroup1:
    name: "Test Subgroup 1"
    entities:
      - sensor.sensor_1

  test_subgroup2:
    name: "Test Subgroup 2"
    entities:
      - sensor.sensor_2

  test_subgroup3:
    name: "Test Subgroup 3"
    entities:
      - sensor.sensor_3

input_boolean:
  test_input_boolean1:
    name: "Change Subgroup 1 visibility"
    initial: on
  test_input_boolean2:
    name: "Change Subgroup 2 visibility"
  test_input_boolean3:
    name: "Change Subgroup 3 visibility"

sensor:
  - platform: random
    name: Sensor 1
  - platform: random
    name: Sensor 2
  - platform: random
    name: Sensor 3

customize:
  group.test_subgroup2:
    visible: false
  group.test_subgroup3:
    visible: false

automation:
  - alias: "group_view_default"
    trigger:
      - platform: homeassistant
        event: start
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup1
        data:
          visible: true
      - service: group.set_visibility
        entity_id: group.test_subgroup2
        data:
          visible: false
      - service: group.set_visibility
        entity_id: group.test_subgroup3
        data:
          visible: false

  - alias: "subgroup1_visible"
    trigger:
      - platform: state
        entity_id: input_boolean.test_input_boolean1
        to: "on"
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup1
        data:
          visible: true

  - alias: "subgroup1_hidden"
    trigger:
      - platform: state
        entity_id: input_boolean.test_input_boolean1
        to: "off"
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup1
        data:
          visible: false

  - alias: "subgroup2_visible"
    trigger:
      - platform: state
        entity_id: input_boolean.test_input_boolean2
        to: "on"
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup2
        data:
          visible: true

  - alias: "subgroup2_hidden"
    trigger:
      - platform: state
        entity_id: input_boolean.test_input_boolean2
        to: "off"
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup2
        data:
          visible: false

  - alias: "subgroup3_visible"
    trigger:
      - platform: state
        entity_id: input_boolean.test_input_boolean3
        to: "on"
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup3
        data:
          visible: true

  - alias: "subgroup3_hidden"
    trigger:
      - platform: state
        entity_id: input_boolean.test_input_boolean3
        to: "off"
    action:
      - service: group.set_visibility
        entity_id: group.test_subgroup3
        data:
          visible: false
{% endraw %}
```

<br />

And here is the result:

<br />

![]({{ "/assets/img/groups.gif" | absolute_url }})
