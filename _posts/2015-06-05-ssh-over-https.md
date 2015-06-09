---
layout: post
title:  "Connecting to SSH over HTTPS"
date:   2015-05-06
---
Some network administrators block the outbound port 22 preventing users connecting via SSH to the outside world, or, in my case, using the git to connect to GitHub.

There are hosts that allow SSH connection via the HTTPS port 443. Simply edit the `~/.ssh/config` file with the following information:

{% highlight ruby %}
Host github.com
    Hostname ssh.github.com
    Port 443
{% endhighlight %}

[Source][1].

[1]: https://help.github.com/articles/using-ssh-over-the-https-port/
