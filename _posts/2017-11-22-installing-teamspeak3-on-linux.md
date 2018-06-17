---
layout: post
title:  "Installing Teamspeak 3 on Linux"
date:   2017-11-22 13:32:12 +0300
tags: [linux, teamspeak3, systemd]
---

How to install Teamspeak 3 on Linux.

<!--more-->

Create a new user for the teamspeak process

{% highlight bash %}
> sudo adduser --disabled-login ts3
{% endhighlight %}

Fetch the latest release, and extract it

{% highlight bash %}
> wget http://dl.4players.de/ts/releases/3.0.13/teamspeak3-server_linux_amd64-3.0.13.tar.bz2
> tar xjf teamspeak3-server_linux_amd64-3.0.13.tar.bz2
> sudo mv teamspeak3-server_linux-amd64 /usr/local/ts3
{% endhighlight %}

Change the owner and group of the folder to the new ts3 user

{% highlight bash %}
> sudo chown -R ts3:ts3 /usr/local/ts3
{% endhighlight %}

Open firewall ports

{% highlight bash %}
> sudo ufw allow 9987/udp
> sudo ufw allow 30033/tcp
> sudo ufw allow 10011/tcp
{% endhighlight %}

Create start and stop scripts (optional)

{% highlight bash %}
> sudo sh -c 'echo "#!/bin/bash
/usr/local/ts3/ts3server_startscript.sh start" > /usr/local/bin/ts3start.sh'

> sudo sh -c 'echo "#!/bin/bash
/usr/local/ts3/ts3server_startscript.sh stop" > /usr/local/bin/ts3stop.sh'

> sudo chmod +x /usr/local/bin/ts3start.sh
> sudo chmod +x /usr/local/bin/ts3stop.sh
{% endhighlight %}

Create systemd file

{% highlight bash %}
> sudo sh -c 'echo "[Unit]
Description=Teamspeak 3 Server
Wants=network-online.target
After=network-online.target

[Service]
Type=forking
ExecStart=/usr/local/ts3/ts3server_startscript.sh start
ExecStop=/usr/local/ts3/ts3server_startscript.sh stop
WorkingDirectory=/usr/local/ts3
User=ts3
Group=ts3
Restart=always
StandardOutput=journal+console
StandardError=journal+console

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/ts3.service'
{% endhighlight %}

Enable service

{% highlight bash %}
> sudo systemctl enable ts3
> sudo systemctl start ts3
{% endhighlight %}

Show admin token

{% highlight bash %}
> sudo bash -c 'cat /usr/local/ts3/logs/*1.log | grep -Po "token=\K(.+)"'
{% endhighlight %}

