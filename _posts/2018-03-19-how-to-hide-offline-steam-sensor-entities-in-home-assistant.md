---
layout: post
comments: true
title:  "How to hide offline Steam sensor entities in Home Assistant"
date:   2018-03-19 08:27:00 -0400
tags: home-assistant
---

I received the message below on [Reddit🔗](https://www.reddit.com/r/homeassistant/comments/85fbob/managing_groups_visibility_in_home_assistant/dvx0473/) and decided to find a way to do what was proposed.

<br />

>Hi, this is great thanks so much for your contributions to the community. Blogs like yours are the reason I got in to HA in the first place.  
>I’d be interested to see if you find a way to hide individual sensors from appearing groups. For example I have a group of steam sensors in a group called steam and would love to hide them individually when someone is not online without having to hide the whole group.

<br />

A few hours later...

<br />

{% highlight yaml %}
{% raw %}
sensor:
  - platform: steam_online
    api_key: !secret steam_api_key
    accounts:
      - 12345678901234567
      - 98765432109847553
      - 98409840789049048
      - 90848949084989804

automation: 
  - alias: 'Steam Group Entities 1'
    trigger:
      - platform: homeassistant
        event: start
    action:
      - service: group.set
        data_template:
          object_id: steam
          entities: "{% if not(is_state('sensor.steam_12345678901234567', 'offline')) %}sensor.steam_12345678901234567,{% endif %}
          {% if not(is_state('sensor.steam_98765432109847553', 'offline')) %}sensor.steam_98765432109847553,{% endif %}
          {% if not(is_state('sensor.steam_98409840789049048', 'offline')) %}sensor.steam_98409840789049048,{% endif %}
          {% if not(is_state('sensor.steam_90848949084989804', 'offline')) %}sensor.steam_90848949084989804{% endif %}"

  - alias: 'Steam Group Entities 2'
    trigger:
      - platform: time
        minutes: '/1'
        seconds: 0
    action:
      - service: group.set
        data_template:
          object_id: steam
          entities: "{% if not(is_state('sensor.steam_12345678901234567', 'offline')) %}sensor.steam_12345678901234567,{% endif %}
          {% if not(is_state('sensor.steam_98765432109847553', 'offline')) %}sensor.steam_98765432109847553,{% endif %}
          {% if not(is_state('sensor.steam_98409840789049048', 'offline')) %}sensor.steam_98409840789049048,{% endif %}
          {% if not(is_state('sensor.steam_90848949084989804', 'offline')) %}sensor.steam_90848949084989804{% endif %}"
{% endraw %}
{% endhighlight %}

<br />

**It works!**

*Note:* These SteamIDs do not exist

Some important points:

*  Pay attention to the commas after each sensor in the templates, the last doesn't need one.

*  If you use views, don't forget to add the group (in this case `group.steam`) to the desired view.

*  You don't need to create the group, it will be created by the automations.

<br />

{% include disqus.html %}