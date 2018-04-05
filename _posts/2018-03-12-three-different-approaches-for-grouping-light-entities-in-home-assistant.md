---
layout: post
comments: true
title:  "3 different ways to group light entities in Home Assistant"
date:   2018-03-12 21:00:00 -0400
tags: home-assistant
---

I have three LIFX bulbs in a room, two in a ceiling fixture and one in a floor lamp. At first (months ago) I was controlling each bulb individually in the [Home Assistant🔗](https://home-assistant.io) [frontend🔗](https://home-assistant.io/components/frontend/), it was kinda annoying because any change in one of the ceiling fixture light bulbs should be manually replicated to the other one. Then I realized it would be much better to control them using groups.

<br />

## Group

The original way to group entities was by using the [Group component🔗](https://home-assistant.io/components/group/). Code and resulting card below.

<br />

{% highlight yaml %}
{% raw %}
group:
  room_lights:
    name: "Group"
    entities:
      - group.room_ceiling_lights
      - group.room_lamp_lights
  room_ceiling_lights:
    name: "Ceiling Lights"
    entities:
      - light.ceiling1
      - light.ceiling2
  room_lamp_lights:
    name: "Lamp Lights"
    entities:
      - light.lamp1

customize:
  group.room_ceiling_lights:
    friendly_name: "Ceiling Lights"
    icon: mdi:ceiling-light
  group.room_lamp_lights:
    friendly_name: "Lamp Lights"
    icon: mdi:lamp
{% endraw %}
{% endhighlight %}

<br />

![Using the group component]({{ "/assets/Screenshot-2018-03-11-17.05.39-1.png" | absolute_url }})

<br />

When you click/tap on the name (or icon) of the group, Home Assistant displays this card where you can control some attributes.

<br />

![Group component - attributes and controls]({{ "/assets/Screenshot-2018-03-11-17.05.54-1.png" | absolute_url }})

<br />

This was a great improvement in day-to-day use. Now I could turn on or off both ceiling lights at the same time, I could still change their attributes like brightness, color and color temperature at the same time. I was amazed.

But \(there is always a but\) the color of the icons were not changed when the group was turned on or off \(lights\). This can be a bit frustrating if you use only the frontend to view and control those groups. That was the biggest drawback of this approach in my opinion.

<br />

## Switch

Then I found [this discussion🔗](https://community.home-assistant.io/t/three-smart-bulbs-in-a-group-can-i-get-the-default-dynamic-bulb-icon-to-work/10318) on the [Home Assistant Community Forum🔗](https://community.home-assistant.io) which led me to [this issue🔗](https://github.com/home-assistant/home-assistant-polymer/issues/186) on [GitHub🔗](https://github.com) where the use of the [Switch component🔗](https://home-assistant.io/components/switch/) was suggested. I decided to try it using a [Template Switch🔗](https://home-assistant.io/components/switch.template/).  Code and resulting card below.

<br />

{% highlight yaml %}
{% raw %}
switch:
  - platform: template
    switches:
      room_ceiling_lights:
        value_template: "{{ is_state('group.room_ceiling_lights', 'on') }}"
        turn_on:
          service: light.turn_on
          entity_id: group.room_ceiling_lights
        turn_off:
          service: light.turn_off
          entity_id: group.room_ceiling_lights
  - platform: template
    switches:
      room_lamp_lights:
        value_template: "{{ is_state('group.room_lamp_lights', 'on') }}"
        turn_on:
          service: light.turn_on
          entity_id: group.room_lamp_lights
        turn_off:
          service: light.turn_off
          entity_id: group.room_lamp_lights

customize:
  switch.room_ceiling_lights:
    friendly_name: Ceiling
    icon: mdi:ceiling-light
  switch.room_lamp_lights:
    friendly_name: Lamp
    icon: mdi:lamp

group:
  switch_group:
    name: "Switch"
    control: hidden
    entities:
      - switch.room_ceiling_lights
      - switch.room_lamp_lights
{% endraw %}
{% endhighlight %}

<br />

![Using the switch component]({{ "/assets/Screenshot-2018-03-11-17.06.16-1.png" | absolute_url }})

<br />

![Switch component - attributes and controls]({{ "/assets/Screenshot-2018-03-11-17.06.20-1.png" | absolute_url }})

<br />

Now the icons were changed every time the corresponding group was turned on or off. On the other hand we lose the possibility of changing attributes.

<br />

## Light Group

Beginning in version 0.65 we have a new platform called [Light Group🔗](https://home-assistant.io/components/light.group/) in the [Light component🔗](https://home-assistant.io/components/light/). So I decided to test it. Code and resulting card below.

<br />

{% highlight yaml %}
{% raw %}
light:
  - platform: lifx
  - platform: group
    name: Ceiling Lights
    entities:
      - light.ceiling1
      - light.ceiling2
  - platform: group
    name: Lamp Lights
    entities:
      - light.lamp1

customize:
  light.ceiling_lights:
    friendly_name: "Ceiling Lights"
    icon: mdi:ceiling-light
  light.lamp_lights:
    friendly_name: "Lamp Lights"
    icon: mdi:lamp

group:
  light_group:
    name: "Light Group"
    control: hidden
    entities:
      - light.ceiling_lights
      - light.lamp_lights
{% endraw %}
{% endhighlight %}

<br />

![Using the new light group platform]({{ "/assets/Screenshot-2018-03-11-17.06.44-1.png" | absolute_url }})

<br />

![Light group platform - attributes and controls]({{ "/assets/Screenshot-2018-03-11-17.07.15-1.png" | absolute_url }})

<br />

Awesome. Now, besides the icons being changed with each change of state (on or off) of the groups, we have the ability to change attributes again.

<br />

## Conclusion

Particularly I prefer using the new light group platform because it's easier to setup, specially when compared to the template switch approach and yet has the advantage of controlling the groups attributes.

Of course the choice depends on the scenario and use cases. How do you group your light entities? Let us know in the comments section below.

<br />

{% include disqus.html %}