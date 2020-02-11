---
title:  "Making Home Assistant's Presence Detection not so Binary (Node-RED version)"
twitter_text: "Making @home_assistant Presence Detection not so Binary (@NodeRED version)"
date:   2018-11-14 10:30:00
tags: [Home Assistant, Node-RED]
permalink: /making-home-assistants-presence-detection-not-so-binary-node-red-version/
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
        "id": "b7a85521.c083f8",
        "type": "switch",
        "z": "b528e5db.f2efd8",
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
        "x": 340,
        "y": 60,
        "wires": [
            [
                "30d95f27.40a2e"
            ],
            [
                "4d838391.cfd4bc"
            ]
        ]
    },
    {
        "id": "16e435df.fb29ea",
        "type": "change",
        "z": "b528e5db.f2efd8",
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
        "x": 235,
        "y": 60,
        "wires": [
            [
                "b7a85521.c083f8"
            ]
        ],
        "l": false
    },
    {
        "id": "30d95f27.40a2e",
        "type": "template",
        "z": "b528e5db.f2efd8",
        "name": "",
        "field": "payload.entity_id",
        "fieldType": "msg",
        "format": "handlebars",
        "syntax": "mustache",
        "template": "input_select.{{topic}}",
        "output": "str",
        "x": 435,
        "y": 40,
        "wires": [
            [
                "78eb1550.e6398c"
            ]
        ],
        "l": false
    },
    {
        "id": "78eb1550.e6398c",
        "type": "api-current-state",
        "z": "b528e5db.f2efd8",
        "name": "Status?",
        "server": "5066f6f1.f02178",
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
        "x": 540,
        "y": 40,
        "wires": [
            [
                "3a6d3ce5.8437e4",
                "d198ac47.2475e"
            ],
            [
                "e2b28013.ee912",
                "3a6d3ce5.8437e4"
            ]
        ]
    },
    {
        "id": "3a6d3ce5.8437e4",
        "type": "change",
        "z": "b528e5db.f2efd8",
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
        "x": 635,
        "y": 60,
        "wires": [
            [
                "53ab96b9.5e98f8",
                "3c158779.6076f8"
            ]
        ],
        "l": false
    },
    {
        "id": "bf6051fc.0d51b",
        "type": "change",
        "z": "b528e5db.f2efd8",
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
        "x": 795,
        "y": 60,
        "wires": [
            [
                "113e36fd.eee399",
                "3c158779.6076f8"
            ]
        ],
        "l": false
    },
    {
        "id": "4d838391.cfd4bc",
        "type": "api-call-service",
        "z": "b528e5db.f2efd8",
        "name": "Just Left",
        "server": "5066f6f1.f02178",
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
        "x": 680,
        "y": 100,
        "wires": [
            [
                "53ab96b9.5e98f8",
                "bf6051fc.0d51b"
            ]
        ]
    },
    {
        "id": "d198ac47.2475e",
        "type": "api-call-service",
        "z": "b528e5db.f2efd8",
        "name": "Just Arrived",
        "server": "5066f6f1.f02178",
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
        "x": 690,
        "y": 20,
        "wires": [
            [
                "113e36fd.eee399"
            ]
        ]
    },
    {
        "id": "dc02a480.578828",
        "type": "api-call-service",
        "z": "b528e5db.f2efd8",
        "name": "Away",
        "server": "5066f6f1.f02178",
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
        "x": 950,
        "y": 100,
        "wires": [
            [
                "3c158779.6076f8"
            ]
        ]
    },
    {
        "id": "d3f017bd.0042e8",
        "type": "api-call-service",
        "z": "b528e5db.f2efd8",
        "name": "Extended Away",
        "server": "5066f6f1.f02178",
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
        "x": 1220,
        "y": 100,
        "wires": [
            []
        ]
    },
    {
        "id": "53ab96b9.5e98f8",
        "type": "trigger",
        "z": "b528e5db.f2efd8",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10 min",
        "x": 830,
        "y": 100,
        "wires": [
            [
                "dc02a480.578828"
            ]
        ]
    },
    {
        "id": "3c158779.6076f8",
        "type": "trigger",
        "z": "b528e5db.f2efd8",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "24",
        "extend": false,
        "units": "hr",
        "reset": "",
        "bytopic": "topic",
        "name": "24 hrs",
        "x": 1070,
        "y": 100,
        "wires": [
            [
                "d3f017bd.0042e8"
            ]
        ]
    },
    {
        "id": "113e36fd.eee399",
        "type": "trigger",
        "z": "b528e5db.f2efd8",
        "op1": "",
        "op2": "{\"payload\":{\"data\":{\"option\":\"\"}}}",
        "op1type": "nul",
        "op2type": "json",
        "duration": "10",
        "extend": false,
        "units": "min",
        "reset": "",
        "bytopic": "topic",
        "name": "10 min",
        "x": 830,
        "y": 20,
        "wires": [
            [
                "e2b28013.ee912"
            ]
        ]
    },
    {
        "id": "e2b28013.ee912",
        "type": "api-call-service",
        "z": "b528e5db.f2efd8",
        "name": "Home",
        "server": "5066f6f1.f02178",
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
        "x": 950,
        "y": 20,
        "wires": [
            []
        ]
    },
    {
        "id": "78986bc7.815274",
        "type": "trigger-state",
        "z": "b528e5db.f2efd8",
        "name": "Family",
        "server": "5066f6f1.f02178",
        "entityid": "^person\\..*$",
        "entityidfiltertype": "regex",
        "debugenabled": false,
        "constraints": [],
        "constraintsmustmatch": "all",
        "outputs": 2,
        "customoutputs": [],
        "outputinitially": false,
        "state_type": "str",
        "x": 90,
        "y": 60,
        "wires": [
            [
                "ef562e3b.fdb24"
            ],
            []
        ]
    },
    {
        "id": "ef562e3b.fdb24",
        "type": "rbe",
        "z": "b528e5db.f2efd8",
        "name": "",
        "func": "rbe",
        "gap": "",
        "start": "",
        "inout": "out",
        "property": "payload",
        "x": 175,
        "y": 60,
        "wires": [
            [
                "16e435df.fb29ea"
            ]
        ],
        "l": false
    }
]
```

{% endraw %}

{% include important.html content="Don't forget to review the Server configuration in the Home Assistant nodes if you are going to import this sequence." %}
