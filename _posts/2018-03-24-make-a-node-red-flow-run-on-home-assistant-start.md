---
title:  "How to run a Node-RED flow on Home Assistant start"
twitter_text: "How to run a @NodeRED flow on @home_assistant start"
date:   2018-03-24 12:00:00
tags: HomeAssistant NodeRED
permalink: /make-a-node-red-flow-run-on-home-assistant-start/
---
<!-- markdownlint-disable html -->
**--- EDIT \(Jul 09, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

I've just found a simpler and better way to do that. All we need is a status node and a switch node.

<br />

![]({{ "/assets/img/Screenshot 2018-07-09 12.06.44.png" | absolute_url }})

<br />

In this case the status node checks for status changes in a selected node. I selected one of my home assistant "events: state" nodes. The switch node checks if the status node is sending the string "*connected*" in `msg.status.text`.

Following is the JSON code of the sequence. Do not forget to change the status node configuration according to your environment.

<br />

```json
[
    {
        "id": "45fc4a1a.70c274",
        "type": "status",
        "z": "152abd73.69dcbb",
        "name": "",
        "scope": [
            "95360a40.8cb83"
        ],
        "x": 100,
        "y": 660,
        "wires": [
            [
                "65572a91.71965c"
            ]
        ]
    },
    {
        "id": "65572a91.71965c",
        "type": "switch",
        "z": "152abd73.69dcbb",
        "name": "",
        "property": "status.text",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "connected",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 230,
        "y": 660,
        "wires": [
            []
        ]
    }
]
```

<br />

**--- ORIGINAL POST ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Here is an easy and simple way to run a Node-RED flow when Home Assistant starts.

We start in Node-RED with an HTTP node.

<br />

![]({{ "/assets/img/Screenshot-2018-03-24-14.05.48.png" | absolute_url }})

<br />

Then we edit it to listen to `/hastart` and choose a name.

<br />

![]({{ "/assets/img/Screenshot-2018-03-24-14.05.57.png" | absolute_url }})

<br />

This will result in this node which you can insert in your flow and/or create a subflow to use it in multiple flows. Don't forget to **DEPLOY**.

<br />

![]({{ "/assets/img/Screenshot-2018-03-24-14.06.03.png" | absolute_url }})

<br />

Now, in Home Assistant, we create a `shell_command` and an `automation` that will run it.

<br />

```yaml
shell_command:
  ha_start: 'curl http://localhost:1880/hastart'
  # http://localhost:1880/endpoint/hastart if you're using the Node-RED Community Hass.io Add-on.

automation:
  - alias: homeassistant_start
    trigger:
      - platform: homeassistant
        event: start
    action:
      - service: shell_command.ha_start
```

<br />

And that's it. From now on, every time Home Assistant is started the node is triggered.
