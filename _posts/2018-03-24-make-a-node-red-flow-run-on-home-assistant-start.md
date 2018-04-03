---
layout: post
comments: true
title:  "How to run a Node-RED flow on Home Assistant start"
date:   2018-03-24 14:34:00 -0400
tags: home-assistant node-red
---

Here is an easy and simple way to run a Node-RED flow when Home Assistant starts.

We start in Node-RED with an HTTP node.

<br />

![]({{ "/assets/Screenshot-2018-03-24-14.05.48.png" | absolute_url }})

<br />

Then we edit it to listen to `/hastart` and choose a name.

<br />

![]({{ "/assets/Screenshot-2018-03-24-14.05.57.png" | absolute_url }})

<br />

This will result in this node which you can insert in your flow and/or create a subflow to use it in multiple flows. Don't forget to **DEPLOY**.

<br />

![]({{ "/assets/Screenshot-2018-03-24-14.06.03.png" | absolute_url }})

<br />

Now, in Home Assistant, we create a `shell_command` and an `automation` that will run it.

<br />

{% highlight yaml %}
shell_command:
  ha_start: 'curl http://localhost:1880/hastart'

automation:
  - alias: homeassistant_start
    trigger:
      - platform: homeassistant
        event: start
    action:
      - service: shell_command.ha_start
{% endhighlight %}

<br />

And that's it. From now on, every time Home Assistant is started the node is triggered.

<br />

{% include disqus.html %}