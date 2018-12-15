---
layout: post
comments: true
title:  "[Lovelace] Accessing Home Assistant states page with just one click"
twitter_text: "[Lovelace] Accessing @home_assistant states page with just one click"
date:   2018-11-15 10:30:00
tags: HomeAssistant Lovelace
permalink: /lovelace-accessing-home-assistant-states-page-with-just-one-click/
---
<!-- markdownlint-disable html -->
**WARNING: As of Home Assistant version 0.84 this method no longer works.**

Sometimes you need to access the Home Assistant `states` page \(old UI\) and when you have Lovelace as the default UI you have to type it in the address bar. Here is an easier way to access it.

Add the following code to your `configuration.yaml` file and restart Home Assistant.

**IMPORTANT:** Make sure you have the correct value in `url:`.

<br />

```yaml
{% raw %}
panel_iframe:
  states:
    title: States
    icon: mdi:home-assistant
    url: http://<your_home_assistant_url>/states
{% endraw %}
```

<br />

Now you can access the `states` page with just one click.