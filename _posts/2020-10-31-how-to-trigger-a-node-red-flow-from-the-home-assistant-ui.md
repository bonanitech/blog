---
title: "How to trigger a Node-RED flow from the Home Assistant UI (new version)"
twitter_text: "How to trigger a @NodeRED flow from the @home_assistant UI (new version)"
date: 2020-10-31
tags: [Home Assistant, Node-RED]
permalink: /how-to-trigger-a-node-red-flow-from-the-home-assistant-ui-new-version/
excerpt: "Now we have a better and more elegant way to do it."
---
<!-- markdownlint-disable html -->
Until a few months ago, I was using [this method](https://bonani.tech/how-to-trigger-a-node-red-flow-from-the-home-assistant-ui/) to trigger flow from the Home Assistant UI. Although it works fine, it’s not very elegant. Empty scripts cause linting warnings, and we don’t know if it will be forever supported by Home Assistant.

As [pointed out](https://bonani.tech/how-to-trigger-a-node-red-flow-from-the-home-assistant-ui/#comment-4794636679) by iantrich (a long time ago! 😳 ) in the comments section of my old post, since version [0.20.0](https://github.com/zachowj/node-red-contrib-home-assistant-websocket/releases/tag/v0.20.0) of `node-red-contrib-home-assistant-websocket`, we can "Trigger an exposed event node from a service call".

> With the release of the Node-RED [custom component](https://github.com/zachowj/hass-node-red) version 0.3.0, it adds the ability to trigger an event node from a service call.

That makes triggering a Node-RED flow from the Home Assistant UI even easier.

First, in Node-RED, we need to add an `entity` node to the flow.

<br />

![entity-node]({{ "/assets/img/2020-10-31-entity-node.png" | absolute_url }})

<br />

Change its type to switch, and add a name in the Home Assistant Config section.

<br />

![edit-entity-node]({{ "/assets/img/2020-10-31-edit-entity-node.png" | absolute_url }})

<br />

Then we can connect our flow to the first output of the node.

<br />

![flow]({{ "/assets/img/2020-10-31-flow.png" | absolute_url }})

<br />

After that, in Home Assistant, we add the following code to a Lovelace view.

```yaml
- type: button
  entity: script.good_morning
  name: "Good Morning"
  icon: mdi:coffee
  tap_action:
    action: call-service
    service: nodered.trigger
    service_data:
      entity_id: switch.scene_good_morning
```

<br />

This will create the this button.

<br />

![button]({{ "/assets/img/2019-07-14-button.png" | absolute_url }})

<br />

And when we press the button, it will trigger the flow we created above.

<br />

## Exposing to voice assistants

If we want to use our favorite voice assistant to trigger that flow, we can create a script.

```yaml
good_morning:
  alias: Good Morning
  sequence:
  - service: nodered.trigger
    data: {}
    entity_id: switch.scene_good_morning
  mode: single
```

And then we expose it using [Home Assistant Cloud](https://www.home-assistant.io/cloud/) or other method.
