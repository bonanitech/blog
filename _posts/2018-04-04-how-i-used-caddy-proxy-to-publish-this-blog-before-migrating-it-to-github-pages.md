---
layout: post
comments: true
title:  "How I used Caddy Proxy to publish this blog before migrating it to GitHub Pages"
date:   2018-04-04 10:40:00 -0400
tags: home-assistant
---

Before migrating this blog to [GitHub Pages🔗](https://pages.github.com) I used this [Caddy Proxy add-on🔗](https://github.com/bestlibre/hassio-addons/tree/master/caddy_proxy) on Hass.io to help me publish it.

When using Caddy Proxy, there is no need to worry about certificates in the `configuration.yaml` file, it takes care of that automatically. Just let Home Assistant run on the default port \(`8123`\).

As I don't have a static IP on my internet connection I had to use the [DuckDNS add-on🔗](https://www.home-assistant.io/addons/duckdns/) to translate my dynamic IP address to a hostname.

The first step was to create an account and a domain on the [DuckDNS website🔗](https://www.duckdns.org).

After that I copied the domain and token to the DuckDNS add-on Config, saved and started the add-on.

<br />

DuckDNS add-on Config example:  

{% highlight json %}
{
  "lets_encrypt": {
    "accept_terms": true,
    "certfile": "fullchain.pem",
    "keyfile": "privkey.pem"
  },
  "token": "sdfj-2131023-dslfjsd-12321",
  "domains": ["my-domain.duckdns.org"]
}
{% endhighlight %}

<br />

Time to edit some DNS entries.  

I use [Cloudflare🔗](https://www.cloudflare.com) to manage my domain. All I had to do was add a `CNAME` record pointing the root of my domain \(`@`\) to my DuckDNS address.

<br />

![Cloudflare]({{ "/assets/cloudflare.jpg" | absolute_url }})

<br />

I don't like NAT Reflection/NAT Loopback/NAT Hairpinning, so I created an internal DNS zone to be able to access the blog in the LAN.

<br />

I set up a forward on ports `80` and `443` to the IP address of the Caddy Proxy \(in this case the same IP of my Home Assistant install\) in my router.

<br />

I also set up the Caddy Proxy add-on like below:  

{% highlight json %}
{
  "homeassistant": "hassio_hostname",
  "vhosts": [
    {
      "vhost": "bonani.tech",
      "port": "8081",
      "remote": "10.10.10.10"
    }
  ],
  "raw_config": [],
  "email": "user@email.com"
}
{% endhighlight %}

<br />

Then I installed [WordPress🔗](https://hub.docker.com/_/wordpress/) on port `8081` \(because it was running on the same server\).

<br />

`docker run --network=bridge -p 8081:80 -v /opt/wordpress/html:/var/www/html --name wordpress -d wordpress`

<br />

And VOILÀ!

<br />

{% include disqus.html %}