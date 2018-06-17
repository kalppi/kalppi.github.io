---
layout: post
title:  "Using colors in ssh sessions"
date:   2017-02-05 10:32:12 +0300
tags: [ssh, bash]
---

This is one easy way to use colors when using ssh to connect to a remote server.

<!--more-->

Put the following code in `~/.bash_aliases`:

{% highlight bash %}
REMOTE_COLORS='${debian_chroot:+($debian_chroot)}\[\033]0;\u@\h: \w\007\]\[\033[01;35m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;37m\]$ '

function ssh_color() {
    ssh $@ -t "bash --rcfile <(cat ~/.bashrc; echo ''; echo 'PS1=\"$REMOTE_COLORS\";')";
}

function ssh_color_motd() {
    ssh $@ -t "bash --rcfile <(cat ~/.bashrc; echo ''; echo 'PS1=\"$REMOTE_COLORS\"; for file in /etc/update-motd.d/*; do \$file; done')";
}

alias ssh=ssh_color_motd
{% endhighlight %}


Function `ssh_color` includes remote `.bashrc` normally, and then overrides PS1 variable. If you want to see motd like you would see when logging in normally, use `ssh_color_motd`.