---
title:  "Managing groups visibility in Home Assistant"
twitter_text: "Managing groups visibility in @home_assistant"
date:   2018-03-18 19:54:00
tags: [Home Assistant]
permalink: /managing-groups-visibility-in-home-assistant/
excerpt: "How to hide and unhide groups (cards) in the Home Assistant frontend."
---
<!-- markdownlint-disable html -->
**--- EDIT \(Feb 05, 2020\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

{% include important.html content="The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, this won't work anymore after that." %}

<br />

**--- ORIGINAL POST ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

After I wrote [Dynamically hide / unhide an entire view in Home Assistant](/dynamically-hide-unhide-an-entire-view/), many people asked me if it is possible to do the same with the cards \(groups\). It is, and it's easy too.

Here is an example code:

<br />

```yaml
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
```

<br />

And here is the result:

<br />

![]({{ "/assets/img/groups.gif" | absolute_url }})
