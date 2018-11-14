---
layout: post
comments: true
title:  "Making Home Assistant's Presence Detection not so Binary (Node-RED version)"
twitter_text: "Making Home Assistant's Presence Detection not so Binary (Node-RED version)"
date:   2018-11-14 17:30:00
tags: HomeAssistant NodeRED
permalink: /making-home-assistants-presence-detection-not-so-binary-node-red-version/
---

My special thanks to Phil Hawthorne for letting me use the title of his post here.

In early 2018 I found Phil's excellent blog post "[Making Home Assistant’s Presence Detection not so Binary](https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/)", I began using the setup suggested by him almost immediately.

A few months later I started migrating my automations to Node-RED and posted the initial results in the [comments section](https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/#comment-3945751887) of Phil's post.

This works great, but the caveat is that you have to have one sequence for each device tracked.

I started looking for a way to have only one sequence for all the tracked devices, after some time (with some hits and many, many misses) this is what I came out with.

<br />

![Template Sequence]({{ "/assets/img/2018-11-14-template_sequence.png" | absolute_url }})

<br />

**NOTE:** The JSON code of this sequence is at the end of this page.

<br />

Now all I needed was to set up an `input_select` with the same name of a `device_tracker` in Home Assistant.

<br />

In `known_devices.yaml`:

<br />

{% highlight yaml %}
{% raw %}
homer:
  hide_if_away: false
  icon:
  mac: 00:00:00:00:00:00
  name: Homer
  picture: /local/homer.png
  track: true
{% endraw %}
{% endhighlight %}

<br />

In `configuration.yaml`:

<br />

{% highlight yaml %}
{% raw %}
input_select:
  homer:
    options:
      - Home
      - Just Arrived
      - Just Left
      - Away
      - Extended Away
{% endraw %}
{% endhighlight %}

<br />

And in Node-RED an `events: state` node that should be connected to the change node in the beginning of the sequence.

<br />

![Homer]({{ "/assets/img/2018-11-14-homer_node.png" | absolute_url }})

<br />

This must be repeated for each person being tracked, just replacing the names where needed.

For example:

<br />

{% highlight yaml %}
{% raw %}
marge:
  hide_if_away: false
  icon:
  mac: 11:11:11:11:11:11
  name: Marge
  picture: /local/marge.png
  track: true
{% endraw %}
{% endhighlight %}

<br />

{% highlight yaml %}
{% raw %}
input_select:
  marge:
    options:
      - Home
      - Just Arrived
      - Just Left
      - Away
      - Extended Away
{% endraw %}
{% endhighlight %}

<br />

![Marge]({{ "/assets/img/2018-11-14-marge_node.png" | absolute_url }})

<br />

The final result should be something like this:

<br />

![Node-RED]({{ "/assets/img/2018-11-14-node-red.png" | absolute_url }})

<br />

![Home Assistant]({{ "/assets/img/2018-11-14-home-assistant.png" | absolute_url }})

<br />

Node-RED JSON code for the sequence in the beginning of this post:

<br />

{% highlight json %}
{% raw %}
[{"id":"d40df6c2.d47028","type":"switch","z":"8563a7bd.384e78","name":"Home?","property":"status","propertyType":"msg","rules":[{"t":"eq","v":"home","vt":"str"},{"t":"eq","v":"not_home","vt":"str"}],"checkall":"false","repair":false,"outputs":2,"x":400,"y":100,"wires":[["c06ebef7.051a7"],["1c5fa89c.e32cc7"]]},{"id":"1c5fa89c.e32cc7","type":"template","z":"8563a7bd.384e78","name":"Just Left","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Just Left\"\n   }\n}","output":"str","x":540,"y":140,"wires":[["c6209771.e44568"]]},{"id":"c06ebef7.051a7","type":"template","z":"8563a7bd.384e78","name":"","field":"payload.entity_id","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"input_select.{{topic}}","output":"str","x":540,"y":60,"wires":[["ee350274.26dfb"]]},{"id":"ee350274.26dfb","type":"api-current-state","z":"8563a7bd.384e78","name":"Status?","halt_if":"","override_topic":false,"override_payload":true,"entity_id":"","x":680,"y":60,"wires":[["b9e404f5.ac9ee8","72adcd60.18ef94"]]},{"id":"b9e404f5.ac9ee8","type":"switch","z":"8563a7bd.384e78","name":"Just Left?","property":"payload","propertyType":"msg","rules":[{"t":"neq","v":"Just Left","vt":"str"},{"t":"else"}],"checkall":"true","repair":false,"outputs":2,"x":820,"y":40,"wires":[["bff37b9c.a09628"],["49cf7023.7e66"]]},{"id":"72adcd60.18ef94","type":"change","z":"8563a7bd.384e78","name":"RESET","rules":[{"t":"set","p":"reset","pt":"msg","to":"true","tot":"bool"}],"action":"","property":"","from":"","to":"","reg":false,"x":810,"y":80,"wires":[["7becd2f9.0dd21c","f4835fc3.0baf2"]]},{"id":"85a514ff.5e7dd8","type":"change","z":"8563a7bd.384e78","name":"RESET","rules":[{"t":"set","p":"reset","pt":"msg","to":"true","tot":"bool"}],"action":"","property":"","from":"","to":"","reg":false,"x":1170,"y":140,"wires":[["6586b018.1ce49","f4835fc3.0baf2"]]},{"id":"c6209771.e44568","type":"api-call-service","z":"8563a7bd.384e78","name":"HA","service_domain":"","service":"","data":"","mergecontext":"","x":670,"y":140,"wires":[["7becd2f9.0dd21c","85a514ff.5e7dd8"]]},{"id":"1213e3a7.af404c","type":"api-call-service","z":"8563a7bd.384e78","name":"HA","service_domain":"","service":"","data":"","mergecontext":"","x":1110,"y":20,"wires":[["6586b018.1ce49"]]},{"id":"9f3a6cdc.4bc01","type":"api-call-service","z":"8563a7bd.384e78","name":"HA","service_domain":"","service":"","data":"","mergecontext":"","x":1050,"y":120,"wires":[["f4835fc3.0baf2"]]},{"id":"a6ac7a43.a32818","type":"api-call-service","z":"8563a7bd.384e78","name":"HA","service_domain":"","service":"","data":"","mergecontext":"","x":1530,"y":80,"wires":[[]]},{"id":"7becd2f9.0dd21c","type":"trigger","z":"8563a7bd.384e78","op1":"","op2":"","op1type":"nul","op2type":"pay","duration":"10","extend":false,"units":"min","reset":"","bytopic":"topic","name":"10 Min","x":810,"y":120,"wires":[["f78cad65.d2879"]]},{"id":"f4835fc3.0baf2","type":"trigger","z":"8563a7bd.384e78","op1":"","op2":"","op1type":"nul","op2type":"pay","duration":"24","extend":false,"units":"hr","reset":"","bytopic":"topic","name":"24 Hrs","x":1230,"y":80,"wires":[["a73294ba.2c45b8"]]},{"id":"6586b018.1ce49","type":"trigger","z":"8563a7bd.384e78","op1":"","op2":"","op1type":"nul","op2type":"pay","duration":"10","extend":false,"units":"min","reset":"","bytopic":"topic","name":"10 Min","x":1230,"y":20,"wires":[["49cf7023.7e66"]]},{"id":"bff37b9c.a09628","type":"template","z":"8563a7bd.384e78","name":"Just Arrived","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Just Arrived\"\n   }\n}","output":"str","x":970,"y":20,"wires":[["1213e3a7.af404c"]]},{"id":"49cf7023.7e66","type":"template","z":"8563a7bd.384e78","name":"Home","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Home\"\n   }\n}","output":"str","x":1350,"y":40,"wires":[["2e01cc90.59e594"]]},{"id":"a73294ba.2c45b8","type":"template","z":"8563a7bd.384e78","name":"Extended Away","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Extended Away\"\n   }\n}","output":"str","x":1380,"y":80,"wires":[["a6ac7a43.a32818"]]},{"id":"f78cad65.d2879","type":"template","z":"8563a7bd.384e78","name":"Away","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{ \"domain\": \"input_select\",\n  \"service\": \"select_option\",\n  \"data\": {\n    \"entity_id\": \"input_select.{{topic}}\",\n    \"option\": \"Away\"\n   }\n}","output":"str","x":930,"y":120,"wires":[["9f3a6cdc.4bc01"]]},{"id":"25670fd1.5262b","type":"change","z":"8563a7bd.384e78","name":"Change","rules":[{"t":"move","p":"payload","pt":"msg","to":"status","tot":"msg"},{"t":"change","p":"topic","pt":"msg","from":"device_tracker.","fromt":"str","to":"","tot":"str"}],"action":"","property":"","from":"","to":"","reg":false,"x":260,"y":100,"wires":[["d40df6c2.d47028"]]},{"id":"2e01cc90.59e594","type":"api-call-service","z":"8563a7bd.384e78","name":"HA","service_domain":"","service":"","data":"","mergecontext":"","x":1470,"y":40,"wires":[[]]}]
{% endraw %}
{% endhighlight %}

<br />

**\(IMPORTANT\):** Don't forget to review the Server configuration in the Home Assistant nodes if you are going to use this sequence.