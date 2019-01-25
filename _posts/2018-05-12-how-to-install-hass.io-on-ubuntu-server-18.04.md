---
layout: post
comments: true
title:  "How to install Hass.io on Ubuntu Server 18.04"
twitter_text: "How to install Hass.io on Ubuntu Server 18.04"
date:   2018-11-17 10:30:00
tags: HomeAssistant
permalink: /how-to-install-hass.io-on-ubuntu-server-18.04/
---
<!-- markdownlint-disable html -->
**--- EDIT \(Nov 17, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

The best method to install Hass.io on a Generic Ubuntu/Debian machine is described at [https://gist.github.com/frenck/32b4f74919ca6b95b30c66f85976ec58](https://gist.github.com/frenck/32b4f74919ca6b95b30c66f85976ec58)

My thanks to Frenck for the awesome script.

<br />

**\* DO NOT \*** follow the instructions below. I'm keeping them here just as a historical reference.

<br />

**--- ORIGINAL POST ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Until last month, I was running Hass.io on an Ubuntu Server 16.04. Now I decided to upgrade the host O.S. to the newest version. This is a step by step guide on how I did it.
{: style="text-decoration: line-through;"}

<br />

**STEP 1:** Perform a BACKUP
{: style="text-decoration: line-through;"}

I like to download the entire `/usr/share/hassio` directory. This is where the configuration files for Home Assistant \(in `/usr/share/hassio/homeassistant`\) and its add-ons \(in `/usr/share/hassio/addons`\) are located in Hass.io.
{: style="text-decoration: line-through;"}

Write down the address of each add-on repository that you have added to your installation, the name of all installed add-on and their configuration, they will be needed later.
{: style="text-decoration: line-through;"}

<br />

**--- EDIT \(Oct 24, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Like John Mesberg [well remembered in the comments section](https://bonani.tech/how-to-install-hass.io-on-ubuntu-server-18.04/#comment-4159094116), you can use the Hass.io snapshot feature for the backup and restore process.
{: style="text-decoration: line-through;"}

<br />

**STEP 2:** Install host O.S.
{: style="text-decoration: line-through;"}

You can follow [this guide](https://www.howtoforge.com/tutorial/ubuntu-lts-minimal-server/) explaining how to install Ubuntu Server 18.04.
{: style="text-decoration: line-through;"}

<br />

**STEP 3:** Make sure the host system is up to date.
{: style="text-decoration: line-through;"}

<br />

```bash
sudo apt-get update

sudo apt-get -y upgrade
```
{: style="text-decoration: line-through;"}

<br />

Some would say this is an optional step, but I think it's a good practice.
{: style="text-decoration: line-through;"}

<br />

**STEP 4:** Install needed packages.
{: style="text-decoration: line-through;"}

<br />

**--- EDIT \(Sep 12, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

On the latest Ubuntu server, `jq` and `docker.io` are in the universe repository. Enable it with.
{: style="text-decoration: line-through;"}

<br />

```bash
sudo add-apt-repository universe && sudo apt-get update
```
{: style="text-decoration: line-through;"}

<br />

Install the packages.
{: style="text-decoration: line-through;"}

<br />

```bash
sudo apt-get install docker.io avahi-daemon jq
```
{: style="text-decoration: line-through;"}

<br />

These are the only packages required to install Hass.io on an Ubuntu Server.
{: style="text-decoration: line-through;"}

<br />

**\(OPTIONAL\):** If you want to make sure docker was correctly installed and it's running you can run one of the commands below.
{: style="text-decoration: line-through;"}

```bash
sudo systemctl status docker
```
{: style="text-decoration: line-through;"}

![sudo systemctl status docker]({{ "/assets/img/Screenshot 2018-05-11 12.00.41.png" | absolute_url }})
{: style="opacity: 0.5;"}

<br />

```bash
sudo docker ps
```
{: style="text-decoration: line-through;"}

![sudo docker ps]({{ "/assets/img/Screenshot 2018-05-11 12.01.07.png" | absolute_url }})
{: style="opacity: 0.5;"}

<br />

**STEP 5:** Install Hass.io
{: style="text-decoration: line-through;"}

To install Hass.io in this environment you need to run the following command as described in [this page](https://www.home-assistant.io/hassio/installation/#alternative-install-on-generic-linux-server).
{: style="text-decoration: line-through;"}

<br />

```bash
curl -sL https://raw.githubusercontent.com/home-assistant/hassio-build/master/install/hassio_install | sudo bash -s
```
{: style="text-decoration: line-through;"}

<br />

Then you can access `http://server_ip_address:8123` in your web browser.
{: style="text-decoration: line-through;"}

You should see this message. Just wait.
{: style="text-decoration: line-through;"}

<br />

![installing]({{ "/assets/img/Screenshot 2018-05-11 12.12.18.png" | absolute_url }})
{: style="opacity: 0.5;"}

<br />

After a few minutes the page should be refreshed automatically and you should see the page below.
{: style="text-decoration: line-through;"}

<br />

![running]({{ "/assets/img/Screenshot 2018-05-11 12.17.24.png" | absolute_url }})
{: style="opacity: 0.5;"}

<br />

In some cases, after some time, you may receive an error message saying the page is inaccessible, a manual page refresh should resolve that.
{: style="text-decoration: line-through;"}

<br />

**STEP 6:** Reinstall, reconfigure and start add-ons.
{: style="text-decoration: line-through;"}

Click *Hass.io* in the left pane, and then click *ADD-ON STORE*.
{: style="text-decoration: line-through;"}

Add the repositories you wrote down in **STEP 1**, install the add-ons you had in your previous install, reconfigure and start them.
{: style="text-decoration: line-through;"}

<br />

**STEP 7:** Restore Home Assistant configuration files from backup.
{: style="text-decoration: line-through;"}

If you followed the guide in **STEP 2** you should have access to the server via SSH.
{: style="text-decoration: line-through;"}

Connect to it using your preferred SCP client, remove all the files in `/usr/share/hassio/homeassistant` and upload the Home Assistant configuration files from your backup to that directory.
{: style="text-decoration: line-through;"}

<br />

**STEP 8:** Restart Home Assistant or reboot the host system.
{: style="text-decoration: line-through;"}

Then you can restart Home Assistant or reboot your system \(only if you really want to\).
{: style="text-decoration: line-through;"}

That's it! Now you have Hass.io running on Ubuntu Server 18.04.
{: style="text-decoration: line-through;"}

<br />

**--- EDIT \(May 18, 2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

If you prefer to run Home Assistant in Python Virtual Environment [BurnsHA](https://www.youtube.com/channel/UCSKQutOXuNLvFetrKuwudpg) has a great video explaining how to do it

{% include youtubePlayer.html id="OThxdTKVjHU" %}
