---
layout: post
comments: true
title:  "[Lovelace] Accessing Home Assistant states page with just one click"
twitter_text: "[Lovelace] Accessing Home Assistant states page with just one click"
date:   2018-10-31 10:30:00
tags: HomeAssistant Lovelace
permalink: /lovelace-accessing-home-assistant-states-page-with-just-one-click/
---

Sometimes you have to access the Home Assistant `states` page \(old UI\) and when you have Lovelace as the default UI you need to type the URL in the address bar. Here is an easier way to access it.

Add the following code to your `configuration.yaml` file and restart Home Assistant.

**\(IMPORTANT\):** Make sure you have the correct value in `url:`.

<br />

{% highlight yaml %}
{% raw %}
panel_iframe:
  states:
    title: States
    icon: mdi:home-assistant
    url: http://<your_home_assistant_url>/states
{% endraw %}
{% endhighlight %}

<br />

Now you can access the `states` page with just one click.