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
**--- EDIT \(Nov 17,2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

The best method to install Hass.io on a Generic Ubuntu/Debian machine is described at [https://gist.github.com/frenck/32b4f74919ca6b95b30c66f85976ec58](https://gist.github.com/frenck/32b4f74919ca6b95b30c66f85976ec58)

My thanks to Frenck for the awesome script.

<br />

**DO NOT** follow the instructions below. I'm keeping them here just as a historical reference.

**--- ORIGINAL POST ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Until last month, I was running Hass.io on an Ubuntu Server 16.04. Now I decided to upgrade the host O.S. to the newest version. This is a step by step guide on how I did it.

<br />

**STEP 1:** Perform a BACKUP

I like to download the entire `/usr/share/hassio` directory. This is where the configuration files for Home Assistant \(in `/usr/share/hassio/homeassistant`\) and its add-ons \(in `/usr/share/hassio/addons`\) are located in Hass.io.

Write down the address of each add-on repository that you have added to your installation, the name of all installed add-on and their configuration, they will be needed later.

**--- EDIT \(Oct 24,2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

Like John Mesberg [well remembered in the comments section](https://bonani.tech/how-to-install-hass.io-on-ubuntu-server-18.04/#comment-4159094116), you can use the Hass.io snapshot feature for the backup and restore process.

<br />

**STEP 2:** Install host O.S.

You can follow [this guide](https://www.howtoforge.com/tutorial/ubuntu-lts-minimal-server/) explaining how to install Ubuntu Server 18.04.

<br />

**STEP 3:** Make sure the host system is up to date.

<br />

```bash
sudo apt-get update

sudo apt-get -y upgrade
```

<br />

Some would say this is an optional step, but I think it's a good practice.

<br />

**STEP 4:** Install needed packages.

**--- EDIT \(Sep 12,2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

On the latest Ubuntu server, `jq` and `docker.io` are in the universe repository. Enable it with.

<br />

```bash
sudo add-apt-repository universe && sudo apt-get update
```

<br />

Install the packages.

<br />

```bash
sudo apt-get install docker.io avahi-daemon jq
```

<br />

These are the only packages required to install Hass.io on an Ubuntu Server.

<br />

**\(OPTIONAL\):** If you want to make sure docker was correctly installed and it's running you can run one of the commands below.

```bash
sudo systemctl status docker
```

![sudo systemctl status docker]({{ "/assets/img/Screenshot 2018-05-11 12.00.41.png" | absolute_url }})

<br />

```bash
sudo docker ps
```

![sudo docker ps]({{ "/assets/img/Screenshot 2018-05-11 12.01.07.png" | absolute_url }})

<br />

**STEP 5:** Install Hass.io

To install Hass.io in this environment you need to run the following command as described in [this page](https://www.home-assistant.io/hassio/installation/#alternative-install-on-generic-linux-server).

<br />

```bash
curl -sL https://raw.githubusercontent.com/home-assistant/hassio-build/master/install/hassio_install | sudo bash -s
```

<br />

Then you can access `http://server_ip_address:8123` in your web browser.

You should see this message. Just wait.

<br />

![installing]({{ "/assets/img/Screenshot 2018-05-11 12.12.18.png" | absolute_url }})

<br />

After a few minutes the page should be refreshed automatically and you should see the page below.

<br />

![running]({{ "/assets/img/Screenshot 2018-05-11 12.17.24.png" | absolute_url }})

<br />

In some cases, after some time, you may receive an error message saying the page is inaccessible, a manual page refresh should resolve that.

<br />

**STEP 6:** Reinstall, reconfigure and start add-ons.

Click *Hass.io* in the left pane, and then click *ADD-ON STORE*.

Add the repositories you wrote down in **STEP 1**, install the add-ons you had in your previous install, reconfigure and start them.

<br />

**STEP 7:** Restore Home Assistant configuration files from backup.

If you followed the guide in **STEP 2** you should have access to the server via SSH.

Connect to it using your preferred SCP client, remove all the files in `/usr/share/hassio/homeassistant` and upload the Home Assistant configuration files from your backup to that directory.

<br />

**STEP 8:** Restart Home Assistant or reboot the host system.

Then you can restart Home Assistant or reboot your system \(only if you really want to\).

That's it! Now you have Hass.io running on Ubuntu Server 18.04.

<br />

**--- EDIT \(May 18,2018\) ---**
{: style="color:gray; font-size: 80%; text-align: center;"}

If you prefer to run Home Assistant in Python Virtual Environment [BurnsHA](https://www.youtube.com/channel/UCSKQutOXuNLvFetrKuwudpg) has a great video explaining how to do it

{% include youtubePlayer.html id="OThxdTKVjHU" %}
