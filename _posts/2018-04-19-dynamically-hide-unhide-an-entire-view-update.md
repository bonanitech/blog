---
layout: post
comments: true
title:  "Dynamically hide/unhide an entire view in Home Assistant (UPDATE)"
twitter_text: "Dynamically hide/unhide an entire view in @home_assistant (UPDATE)"
date:   2018-04-19 11:25:00
tags: HomeAssistant
permalink: /dynamically-hide-unhide-an-entire-view-update/
---
<!-- markdownlint-disable html -->
Reddit user [/u/lordjustice17](https://www.reddit.com/user/lordjustice17) sent me a message \(see below\) on [Reddit](https://www.reddit.com/r/homeassistant/comments/84rogz/dynamically_hideunhide_an_entire_view_in_home/dxlv4ql/) with a different use of [Dynamically hide/unhide an entire view in Home Assistant](https://bonani.tech/dynamically-hide-unhide-an-entire-view/)

<br />

![]({{ "/assets/img/Screenshot 2018-04-19 12.23.34.png" | absolute_url }})

<br />

His code:

<br />

```yaml
{% raw %}
- platform: template
 switches:
   console:
     friendly_name: Control Panel
     value_template: "{{ is_state_attr('group.console', 'view', true) }}"
     turn_on:
       service: group.set
       data:
         object_id: console
         view: true
     turn_off:
       service: group.set
       data:
         object_id: console
         view: false
{% endraw %}
```

<br />

This creates a `switch` that shows and hides a `view` called console. Clever!
