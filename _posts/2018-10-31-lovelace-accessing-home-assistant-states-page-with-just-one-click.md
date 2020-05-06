---
title:  "[Lovelace] Accessing Home Assistant states page with just one click"
twitter_text: "[Lovelace] Accessing @home_assistant states page with just one click"
date:   2018-10-31 10:30:00
tags: [Home Assistant, Lovelace]
permalink: /lovelace-accessing-home-assistant-states-page-with-just-one-click/
excerpt: "Sometimes you need to access the Home Assistant `states` page (old UI) and when you have Lovelace as the default UI you have to type it in the address bar. Here is an easier way to access it."
---
<!-- markdownlint-disable html -->
{% include warning content="The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, this won't work anymore after that." %}

Sometimes you need to access the Home Assistant `states` page \(old UI\) and when you have Lovelace as the default UI you have to type it in the address bar. Here is an easier way to access it.

Add the following code to your `configuration.yaml` file and restart Home Assistant.

```yaml
panel_iframe:
  states:
    title: "States"
    icon: mdi:home-assistant
    url: http://<your_home_assistant_url>/states
```

{% include important content="Make sure you have the correct value in `url:`." %}

<br />

Now you can access the `states` page with just one click.

<br />

## Update - Dec 24, 2018

An alternative way to achieve this is to place a `weblink` in a Lovelace entities card.

```yaml
- type: entities
  show_header_toggle: false
  entities:
    - type: weblink
      url: /states
      name: "States"
      icon: mdi:home-assistant
```

<br />

![States]({{ "/assets/img/2018-10-31-states.png" | absolute_url }})
