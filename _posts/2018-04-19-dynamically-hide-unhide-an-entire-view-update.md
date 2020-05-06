---
title:  "Dynamically hide/unhide an entire view in Home Assistant"
twitter_text: "Dynamically hide/unhide an entire view in @home_assistant"
date:   2018-04-19 11:25:00
tags: [Home Assistant]
permalink: /dynamically-hide-unhide-an-entire-view-update/
excerpt: "How to hide and unhide view in Home Assistant (Update)."
---
<!-- markdownlint-disable html -->
**--- EDIT \(Feb 05, 2020\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

{% include important.html content="The States UI is now [deprecated](https://www.home-assistant.io/blog/2020/02/05/release-105/#the-old-states-ui-is-now-deprecated) and will be completely removed from Home Assistant in version 0.107.0. Therefore, this won't work anymore after that." %}

<br />

**--- ORIGINAL POST ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Reddit user [/u/lordjustice17](https://www.reddit.com/user/lordjustice17) sent me a message \(see below\) on [Reddit](https://www.reddit.com/r/homeassistant/comments/84rogz/dynamically_hideunhide_an_entire_view_in_home/dxlv4ql/) with a different use of [Dynamically hide/unhide an entire view in Home Assistant](/dynamically-hide-unhide-an-entire-view/).

<br />

![]({{ "/assets/img/Screenshot 2018-04-19 12.23.34.png" | absolute_url }})

<br />

His code:

<br />

{% raw %}

```yaml
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
```

{% endraw %}

<br />

This creates a `switch` that shows and hides a `view` called console. Clever!
