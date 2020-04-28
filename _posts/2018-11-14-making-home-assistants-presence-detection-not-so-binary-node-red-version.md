---
title:  "Making Home Assistant's Presence Detection not so Binary (Node-RED version)"
twitter_text: "Making @home_assistant Presence Detection not so Binary (@NodeRED version)"
date:   2018-11-14 10:30:00
tags: [Home Assistant, Node-RED]
permalink: /making-home-assistants-presence-detection-not-so-binary-node-red-version/
excerpt: "\"Translating\" Phil Hawthorne's *Making Home Assistant’s Presence Detection not so Binary* to Node-RED"
---
<!-- markdownlint-disable html -->
**--- EDIT \(Nov 04, 2019\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Here's the most recent version of the flow. Now I'm using the `person` integration to track people. If you're using `device_tracker`, remember to change the first and third nodes according to your needs.

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v3.png" | absolute_url }})

The JSON code of this sequence is [here](#json-code-v3).

<br />

**--- EDIT \(Dec 25, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

If you use `node-red-contrib-home-assistant-websocket` (default in Node-RED Community Hass.io Add-on), the sequence can be even smaller.

Since its [version 0.3.0](https://github.com/zachowj/node-red-contrib-home-assistant-websocket/releases/tag/v0.3.0):

> Call Service and Fire Event nodes are now able to render mustache templates in the data property.

This means that we can remove almost all of the template nodes used in the original sequence and move their content to the call service nodes right after them.

<br />

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence_v2.png" | absolute_url }})

The JSON code of this sequence is [here](#json-code-v2).

<br />

**--- ORIGINAL POST ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

My special thanks to Phil Hawthorne for letting me use the title of his post here.

In early 2018 I found Phil's excellent blog post "[Making Home Assistant’s Presence Detection not so Binary](https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/)" and began using the setup suggested by him almost immediately.

A few months later I started migrating my automations to Node-RED and posted the initial results in the [comments section](https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/#comment-3945751887) of Phil's post.

This works great, but the caveat is that you have to have one sequence for each device tracked.

I started looking for a way to have only one sequence for all the tracked devices, after some time (with some hits and many, many misses) this is what I came out with.

<br />

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence.png" | absolute_url }})

The JSON code of this sequence is [here](#json-code).

<br />

Now all I needed was to set up an `input_select` with the same name of a `device_tracker` in Home Assistant.

In `known_devices.yaml`:

<br />

```yaml
homer:
  hide_if_away: false
  icon:
  mac: 00:00:00:00:00:00
  name: Homer
  picture: /local/homer.png
  track: true
```

<br />

In `configuration.yaml`:

<br />

```yaml
input_select:
  homer:
    options:
      - Home
      - Just Arrived
      - Just Left
      - Away
      - Extended Away
```

<br />

And in Node-RED an `events: state` node that should be connected to the change node in the beginning of the sequence.

<br />

![Homer]({{ "/assets/img/2018-11-14-homer_node.png" | absolute_url }})

<br />

This must be repeated for each person being tracked, just replacing the names where needed.

For example:

<br />

```yaml
marge:
  hide_if_away: false
  icon:
  mac: 11:11:11:11:11:11
  name: Marge
  picture: /local/marge.png
  track: true
```

<br />

```yaml
input_select:
  marge:
    options:
      - Home
      - Just Arrived
      - Just Left
      - Away
      - Extended Away
```

<br />

![Marge]({{ "/assets/img/2018-11-14-marge_node.png" | absolute_url }})

<br />

The final result should be something like this:

<br />

![Node-RED]({{ "/assets/img/2018-11-14-node-red.png" | absolute_url }})

<br />

![Home Assistant]({{ "/assets/img/2018-11-14-home-assistant.png" | absolute_url }})

<br />

## JSON code

Node-RED JSON code for the sequence in the beginning of this post.

<br />

{% raw %}

```json
[
    {
        "id": "63bc1c41.4818b4",
        "type": "switch",
        "z": "92d8d5f4.738fe8",
        "name": "Home?",
        "property": "status",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "home",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "not_home",
                "vt": "str"
            }
        ],
        "checkall": "false",
        "repair": false,
        "outputs": 2,
        "x": 240,
        "y": 100,
        "wires": [
            [
                "48116c4c.4e2024"
            ],
            [
                "e665e2e.424d02"
            ]
        ]
    },
    {
        "id": "e665e2e.424d02",
        "type": "template",
        "z": "92d8d5f4.738fe8",
        "name": "Just Left",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Just Left\"\n   }\n}",
        "output": "str",
        "x": 380,
        "y": 140,
        "wires": [
            [
                "fd43f67e.91b0d8"
            ]
        ]
    },
    {
        "id": "48116c4c.4e2024",
        "type": "template",
        "z": "92d8d5f4.738fe8",
        "name": "",
        "field": "payload.entity_id",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "input_select.{{topic}}",
        "output": "str",
        "x": 380,
        "y": 60,
        "wires": [
            [
                "70796e6c.ac84d"
            ]
        ]
    },
    {
        "id": "70796e6c.ac84d",
        "type": "api-current-state",
        "z": "92d8d5f4.738fe8",
        "name": "Status?",
        "server": "f7db64c8.551da8",
        "version": "1",
        "outputs": 1,
        "halt_if": "",
        "halt_if_type": "str",
        "halt_if_compare": "is",
        "override_topic": false,
        "entity_id": "",
        "state_type": "str",
        "state_location": "payload",
        "override_payload": "msg",
        "entity_location": "data",
        "override_data": "msg",
        "blockInputOverrides": false,
        "x": 520,
        "y": 60,
        "wires": [
            [
                "dfda57cf.84b928",
                "27264311.e3345c"
            ]
        ]
    },
    {
        "id": "dfda57cf.84b928",
        "type": "switch",
        "z": "92d8d5f4.738fe8",
        "name": "Just Left?",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "neq",
                "v": "Just Left",
                "vt": "str"
            },
            {
                "t": "else"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 660,
        "y": 40,
        "wires": [
            [
                "cd88b28d.36f0b"
            ],
            [
                "2bbf8591.d344ba"
            ]
        ]
    },
    {
        "id": "27264311.e3345c",
        "type": "change",
        "z": "92d8d5f4.738fe8",
        "name": "RESET",
        "rules": [
            {
                "t": "set",
                "p": "reset",
                "pt": "msg",
                "to": "true",
                "tot": "bool"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 650,
        "y": 80,
        "wires": [
            [
                "fe6163e2.6ef36",
                "a5420f5c.c41da"
            ]
        ]
    },
    {
        "id": "e522ae3b.d86a7",
        "type": "change",
        "z": "92d8d5f4.738fe8",
        "name": "RESET",
        "rules": [
            {
                "t": "set",
                "p": "reset",
                "pt": "msg",
                "to": "true",
                "tot": "bool"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 1010,
        "y": 140,
        "wires": [
            [
                "2e48c4eb.a8c30c",
                "a5420f5c.c41da"
            ]
        ]
    },
    {
        "id": "fd43f67e.91b0d8",
        "type": "api-call-service",
        "z": "92d8d5f4.738fe8",
        "name": "HA",
        "server": "f7db64c8.551da8",
        "version": "1",
        "service_domain": "",
        "service": "",
        "entityId": "",
        "data": "",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 510,
        "y": 140,
        "wires": [
            [
                "fe6163e2.6ef36",
                "e522ae3b.d86a7"
            ]
        ]
    },
    {
        "id": "7d55c4ca.ebcb3c",
        "type": "api-call-service",
        "z": "92d8d5f4.738fe8",
        "name": "HA",
        "server": "f7db64c8.551da8",
        "version": 1,
        "service_domain": "",
        "service": "",
        "entityId": "",
        "data": "",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 950,
        "y": 20,
        "wires": [
            [
                "2e48c4eb.a8c30c"
            ]
        ]
    },
    {
        "id": "fa4008f4.b2b498",
        "type": "api-call-service",
        "z": "92d8d5f4.738fe8",
        "name": "HA",
        "server": "f7db64c8.551da8",
        "version": "1",
        "service_domain": "",
        "service": "",
        "entityId": "",
        "data": "",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 890,
        "y": 120,
        "wires": [
            [
                "a5420f5c.c41da"
            ]
        ]
    },
    {
        "id": "f517c6e4.0f7b08",
        "type": "api-call-service",
        "z": "92d8d5f4.738fe8",
        "name": "HA",
        "server": "f7db64c8.551da8",
        "version": "1",
        "service_domain": "",
        "service": "",
        "entityId": "",
        "data": "",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 1370,
        "y": 80,
        "wires": [
            []
        ]
    },
    {
        "id": "fe6163e2.6ef36",
        "type": "trigger",
        "z": "92d8d5f4.738fe8",
        "op1": "",
        "op2": "",
        "op1type": "nul",
        "op2type": "pay",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10 Min",
        "x": 650,
        "y": 120,
        "wires": [
            [
                "ae21edc5.d0ec1"
            ]
        ]
    },
    {
        "id": "a5420f5c.c41da",
        "type": "trigger",
        "z": "92d8d5f4.738fe8",
        "op1": "",
        "op2": "",
        "op1type": "nul",
        "op2type": "pay",
        "duration": "24",
        "extend": false,
        "units": "hr",
        "reset": "",
        "bytopic": "topic",
        "name": "24 Hrs",
        "x": 1070,
        "y": 80,
        "wires": [
            [
                "6721c902.c5f848"
            ]
        ]
    },
    {
        "id": "2e48c4eb.a8c30c",
        "type": "trigger",
        "z": "92d8d5f4.738fe8",
        "op1": "",
        "op2": "",
        "op1type": "nul",
        "op2type": "pay",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10 Min",
        "x": 1070,
        "y": 20,
        "wires": [
            [
                "2bbf8591.d344ba"
            ]
        ]
    },
    {
        "id": "cd88b28d.36f0b",
        "type": "template",
        "z": "92d8d5f4.738fe8",
        "name": "Just Arrived",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Just Arrived\"\n   }\n}",
        "output": "str",
        "x": 810,
        "y": 20,
        "wires": [
            [
                "7d55c4ca.ebcb3c"
            ]
        ]
    },
    {
        "id": "2bbf8591.d344ba",
        "type": "template",
        "z": "92d8d5f4.738fe8",
        "name": "Home",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Home\"\n   }\n}",
        "output": "str",
        "x": 1190,
        "y": 40,
        "wires": [
            []
        ]
    },
    {
        "id": "6721c902.c5f848",
        "type": "template",
        "z": "92d8d5f4.738fe8",
        "name": "Extended Away",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Extended Away\"\n   }\n}",
        "output": "str",
        "x": 1220,
        "y": 80,
        "wires": [
            [
                "f517c6e4.0f7b08"
            ]
        ]
    },
    {
        "id": "ae21edc5.d0ec1",
        "type": "template",
        "z": "92d8d5f4.738fe8",
        "name": "Away",
        "field": "payload",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Away\"\n   }\n}",
        "output": "str",
        "x": 770,
        "y": 120,
        "wires": [
            [
                "fa4008f4.b2b498"
            ]
        ]
    },
    {
        "id": "bb846ce5.4cd4a",
        "type": "change",
        "z": "92d8d5f4.738fe8",
        "name": "Change",
        "rules": [
            {
                "t": "move",
                "p": "payload",
                "pt": "msg",
                "to": "status",
                "tot": "msg"
            },
            {
                "t": "change",
                "p": "topic",
                "pt": "msg",
                "from": "device_tracker.",
                "fromt": "str",
                "to": "",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 100,
        "y": 100,
        "wires": [
            [
                "63bc1c41.4818b4"
            ]
        ]
    }
]
```

{% endraw %}

{% include important.html content="Don't forget to review the Server configuration in the Home Assistant nodes if you are going to import this sequence." %}

<br />

**--- EDIT \(Dec 25, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

## JSON code v2

Node-RED JSON code for the sequence in the beginning of this post.

<br />

{% raw %}

```json
[
    {
        "id": "e817c354.2e18",
        "type": "switch",
        "z": "fd7e6f11.57548",
        "name": "Home?",
        "property": "status",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "home",
                "vt": "str"
            },
            {
                "t": "eq",
                "v": "not_home",
                "vt": "str"
            }
        ],
        "checkall": "false",
        "repair": false,
        "outputs": 2,
        "x": 240,
        "y": 100,
        "wires": [
            [
                "6faf2677.041528"
            ],
            [
                "ddc97c15.f5558"
            ]
        ]
    },
    {
        "id": "6faf2677.041528",
        "type": "template",
        "z": "fd7e6f11.57548",
        "name": "",
        "field": "payload.entity_id",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "input_select.{{topic}}",
        "output": "str",
        "x": 380,
        "y": 60,
        "wires": [
            [
                "cc718e97.c5df5"
            ]
        ]
    },
    {
        "id": "cc718e97.c5df5",
        "type": "api-current-state",
        "z": "fd7e6f11.57548",
        "name": "Status?",
        "server": "f7db64c8.551da8",
        "version": "1",
        "outputs": 1,
        "halt_if": "",
        "halt_if_type": "str",
        "halt_if_compare": "is",
        "override_topic": false,
        "entity_id": "",
        "state_type": "str",
        "state_location": "payload",
        "override_payload": "msg",
        "entity_location": "data",
        "override_data": "msg",
        "blockInputOverrides": false,
        "x": 520,
        "y": 60,
        "wires": [
            [
                "4b8e4dd1.55db84",
                "255bf0c.522981"
            ]
        ]
    },
    {
        "id": "4b8e4dd1.55db84",
        "type": "switch",
        "z": "fd7e6f11.57548",
        "name": "Just Left?",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "neq",
                "v": "Just Left",
                "vt": "str"
            },
            {
                "t": "else"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 660,
        "y": 40,
        "wires": [
            [
                "97fc9f3e.e9845"
            ],
            [
                "5d9d99eb.12bf68"
            ]
        ]
    },
    {
        "id": "255bf0c.522981",
        "type": "change",
        "z": "fd7e6f11.57548",
        "name": "RESET",
        "rules": [
            {
                "t": "set",
                "p": "reset",
                "pt": "msg",
                "to": "true",
                "tot": "bool"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 650,
        "y": 80,
        "wires": [
            [
                "aa153516.f666b8",
                "ddbb8120.446fc"
            ]
        ]
    },
    {
        "id": "d5752739.886ce8",
        "type": "change",
        "z": "fd7e6f11.57548",
        "name": "RESET",
        "rules": [
            {
                "t": "set",
                "p": "reset",
                "pt": "msg",
                "to": "true",
                "tot": "bool"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 790,
        "y": 140,
        "wires": [
            [
                "751e6205.ecbd3c",
                "ddbb8120.446fc"
            ]
        ]
    },
    {
        "id": "ddc97c15.f5558",
        "type": "api-call-service",
        "z": "fd7e6f11.57548",
        "name": "Just Left",
        "server": "f7db64c8.551da8",
        "version": 1,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Just Left\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 380,
        "y": 140,
        "wires": [
            [
                "aa153516.f666b8",
                "d5752739.886ce8"
            ]
        ]
    },
    {
        "id": "97fc9f3e.e9845",
        "type": "api-call-service",
        "z": "fd7e6f11.57548",
        "name": "Just Arrived",
        "server": "f7db64c8.551da8",
        "version": 1,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Just Arrived\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 810,
        "y": 20,
        "wires": [
            [
                "751e6205.ecbd3c"
            ]
        ]
    },
    {
        "id": "c6fa9f19.d7b77",
        "type": "api-call-service",
        "z": "fd7e6f11.57548",
        "name": "Away",
        "server": "f7db64c8.551da8",
        "version": 1,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Away\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 790,
        "y": 100,
        "wires": [
            [
                "ddbb8120.446fc"
            ]
        ]
    },
    {
        "id": "e7bf876d.8ba068",
        "type": "api-call-service",
        "z": "fd7e6f11.57548",
        "name": "Extended Away",
        "server": "f7db64c8.551da8",
        "version": 1,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Extended Away\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 1100,
        "y": 80,
        "wires": [
            []
        ]
    },
    {
        "id": "aa153516.f666b8",
        "type": "trigger",
        "z": "fd7e6f11.57548",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10 Min",
        "x": 650,
        "y": 120,
        "wires": [
            [
                "c6fa9f19.d7b77"
            ]
        ]
    },
    {
        "id": "ddbb8120.446fc",
        "type": "trigger",
        "z": "fd7e6f11.57548",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "24",
        "extend": false,
        "units": "hr",
        "reset": "",
        "bytopic": "topic",
        "name": "24 Hrs",
        "x": 950,
        "y": 80,
        "wires": [
            [
                "e7bf876d.8ba068"
            ]
        ]
    },
    {
        "id": "751e6205.ecbd3c",
        "type": "trigger",
        "z": "fd7e6f11.57548",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10 Min",
        "x": 950,
        "y": 20,
        "wires": [
            [
                "5d9d99eb.12bf68"
            ]
        ]
    },
    {
        "id": "5d9d99eb.12bf68",
        "type": "api-call-service",
        "z": "fd7e6f11.57548",
        "name": "Home",
        "server": "f7db64c8.551da8",
        "version": 1,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Home\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 1070,
        "y": 40,
        "wires": [
            []
        ]
    },
    {
        "id": "f37171b7.f1caa",
        "type": "change",
        "z": "fd7e6f11.57548",
        "name": "Change",
        "rules": [
            {
                "t": "move",
                "p": "payload",
                "pt": "msg",
                "to": "status",
                "tot": "msg"
            },
            {
                "t": "change",
                "p": "topic",
                "pt": "msg",
                "from": "device_tracker.",
                "fromt": "str",
                "to": "",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 100,
        "y": 100,
        "wires": [
            [
                "e817c354.2e18"
            ]
        ]
    }
]
```

{% endraw %}

{% include important.html content="Don't forget to review the Server configuration in the Home Assistant nodes if you are going to import this sequence." %}

<br />

**--- EDIT \(Dec 25, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

## JSON code v3

Node-RED JSON code for the sequence in the beginning of this post.

<br />

{% raw %}

```json
[
    {
        "id": "ce09e65.8f84418",
        "type": "switch",
        "z": "9e2d545a.b4a658",
        "name": "Home?",
        "property": "status",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "home",
                "vt": "str"
            },
            {
                "t": "else"
            }
        ],
        "checkall": "false",
        "repair": false,
        "outputs": 2,
        "x": 360,
        "y": 60,
        "wires": [
            [
                "80d7953f.4817f8"
            ],
            [
                "e76d582a.9a8f88"
            ]
        ]
    },
    {
        "id": "dccf0e91.545e9",
        "type": "change",
        "z": "9e2d545a.b4a658",
        "name": "Change",
        "rules": [
            {
                "t": "move",
                "p": "payload",
                "pt": "msg",
                "to": "status",
                "tot": "msg"
            },
            {
                "t": "change",
                "p": "topic",
                "pt": "msg",
                "from": "person.",
                "fromt": "str",
                "to": "",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 255,
        "y": 60,
        "wires": [
            [
                "ce09e65.8f84418"
            ]
        ],
        "l": false
    },
    {
        "id": "80d7953f.4817f8",
        "type": "template",
        "z": "9e2d545a.b4a658",
        "name": "",
        "field": "payload.entity_id",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "input_select.{{topic}}",
        "output": "str",
        "x": 455,
        "y": 40,
        "wires": [
            [
                "4d8f933f.aa124c"
            ]
        ],
        "l": false
    },
    {
        "id": "4d8f933f.aa124c",
        "type": "api-current-state",
        "z": "9e2d545a.b4a658",
        "name": "Status?",
        "server": "fc6553e6.c5975",
        "version": 1,
        "outputs": 2,
        "halt_if": "Just Left",
        "halt_if_type": "str",
        "halt_if_compare": "is_not",
        "override_topic": false,
        "entity_id": "",
        "state_type": "str",
        "state_location": "payload",
        "override_payload": "msg",
        "entity_location": "data",
        "override_data": "msg",
        "blockInputOverrides": false,
        "x": 560,
        "y": 40,
        "wires": [
            [
                "d3f35a3d.8281a8",
                "241457b2.2cc128"
            ],
            [
                "c1544077.8f84a",
                "d3f35a3d.8281a8"
            ]
        ]
    },
    {
        "id": "d3f35a3d.8281a8",
        "type": "change",
        "z": "9e2d545a.b4a658",
        "name": "RESET",
        "rules": [
            {
                "t": "set",
                "p": "reset",
                "pt": "msg",
                "to": "true",
                "tot": "bool"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 655,
        "y": 60,
        "wires": [
            [
                "6a5f8f00.23807",
                "8bd1d643.a115e8"
            ]
        ],
        "icon": "node-red/timer.svg",
        "l": false
    },
    {
        "id": "21de8580.22285a",
        "type": "change",
        "z": "9e2d545a.b4a658",
        "name": "RESET",
        "rules": [
            {
                "t": "set",
                "p": "reset",
                "pt": "msg",
                "to": "true",
                "tot": "bool"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 815,
        "y": 60,
        "wires": [
            [
                "712d8721.d63168",
                "8bd1d643.a115e8"
            ]
        ],
        "icon": "node-red/timer.svg",
        "l": false
    },
    {
        "id": "e76d582a.9a8f88",
        "type": "api-call-service",
        "z": "9e2d545a.b4a658",
        "name": "Just Left",
        "server": "fc6553e6.c5975",
        "version": 1,
        "debugenabled": false,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Just Left\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 700,
        "y": 100,
        "wires": [
            [
                "6a5f8f00.23807",
                "21de8580.22285a"
            ]
        ]
    },
    {
        "id": "241457b2.2cc128",
        "type": "api-call-service",
        "z": "9e2d545a.b4a658",
        "name": "Just Arrived",
        "server": "fc6553e6.c5975",
        "version": 1,
        "debugenabled": false,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Just Arrived\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 710,
        "y": 20,
        "wires": [
            [
                "712d8721.d63168"
            ]
        ]
    },
    {
        "id": "d5810a67.0f3498",
        "type": "api-call-service",
        "z": "9e2d545a.b4a658",
        "name": "Away",
        "server": "fc6553e6.c5975",
        "version": 1,
        "debugenabled": false,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Away\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 970,
        "y": 100,
        "wires": [
            [
                "8bd1d643.a115e8"
            ]
        ]
    },
    {
        "id": "c0187e63.5a9e1",
        "type": "api-call-service",
        "z": "9e2d545a.b4a658",
        "name": "Extended Away",
        "server": "fc6553e6.c5975",
        "version": 1,
        "debugenabled": false,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Extended Away\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 1240,
        "y": 100,
        "wires": [
            []
        ]
    },
    {
        "id": "6a5f8f00.23807",
        "type": "trigger",
        "z": "9e2d545a.b4a658",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10min",
        "x": 850,
        "y": 100,
        "wires": [
            [
                "d5810a67.0f3498"
            ]
        ]
    },
    {
        "id": "8bd1d643.a115e8",
        "type": "trigger",
        "z": "9e2d545a.b4a658",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "24",
        "extend": false,
        "units": "hr",
        "reset": "",
        "bytopic": "topic",
        "name": "24hr",
        "x": 1090,
        "y": 100,
        "wires": [
            [
                "c0187e63.5a9e1"
            ]
        ]
    },
    {
        "id": "712d8721.d63168",
        "type": "trigger",
        "z": "9e2d545a.b4a658",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10min",
        "x": 850,
        "y": 20,
        "wires": [
            [
                "c1544077.8f84a"
            ]
        ]
    },
    {
        "id": "c1544077.8f84a",
        "type": "api-call-service",
        "z": "9e2d545a.b4a658",
        "name": "Home",
        "server": "fc6553e6.c5975",
        "version": 1,
        "debugenabled": false,
        "service_domain": "input_select",
        "service": "select_option",
        "entityId": "",
        "data": "{\"entity_id\":\"input_select.{{topic}}\",\"option\":\"Home\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "payload",
        "output_location_type": "msg",
        "mustacheAltTags": false,
        "x": 970,
        "y": 20,
        "wires": [
            []
        ]
    },
    {
        "id": "4f5a58f6.75a848",
        "type": "trigger-state",
        "z": "9e2d545a.b4a658",
        "name": "Person",
        "server": "fc6553e6.c5975",
        "exposeToHomeAssistant": false,
        "haConfig": [
            {
                "property": "name",
                "value": ""
            },
            {
                "property": "icon",
                "value": ""
            }
        ],
        "entityid": "^person\\..*$",
        "entityidfiltertype": "regex",
        "debugenabled": false,
        "constraints": [
            {
                "id": "eaeskffh1a",
                "targetType": "this_entity",
                "targetValue": "",
                "propertyType": "previous_state",
                "propertyValue": "old_state.state",
                "comparatorType": "includes",
                "comparatorValueDatatype": "list",
                "comparatorValue": "home,not_home"
            }
        ],
        "constraintsmustmatch": "all",
        "outputs": 2,
        "customoutputs": [],
        "outputinitially": false,
        "state_type": "str",
        "x": 100,
        "y": 60,
        "wires": [
            [
                "414184b6.858bdc"
            ],
            []
        ]
    },
    {
        "id": "414184b6.858bdc",
        "type": "rbe",
        "z": "9e2d545a.b4a658",
        "name": "",
        "func": "rbe",
        "gap": "",
        "start": "",
        "inout": "out",
        "property": "payload",
        "x": 195,
        "y": 60,
        "wires": [
            [
                "dccf0e91.545e9"
            ]
        ],
        "l": false
    }
]
```

{% endraw %}

{% include important.html content="Don't forget to review the Server configuration in the Home Assistant nodes if you are going to import this sequence." %}
