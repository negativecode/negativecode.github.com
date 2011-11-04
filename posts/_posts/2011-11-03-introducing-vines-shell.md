---
layout: post
title: Introducing Vines Shell
author: David Graham
---
We're proud to announce the third preview release of the [Vines XMPP server](http://www.getvines.com/)!  Along with many bug fixes and improvements, we're introducing Vines shell, a new way to chat with servers.

## We &#x2764; the Command Line
Vines shell connects us with a bash session on a remote server through the same instant-messenger we're using to chat with friends. We can send the unix commands we already know and love to the server and it replies with the commands' output.

<figure class="screenshot">
<a href="/images/vines-shell.png"><img src="/images/vines-shell.png" alt="Vines Web Chat"/></a>
<figcaption>A multi-server chat session. <a href="/images/vines-shell.png">Click for larger image</a>.</figcaption>
</figure>

Chatting with a server is novel, but it's not much better than simply using ssh.  That's where services and permissions come in.

## Services
One limitation of the command line is that we can only control one machine at a time. So, Vines lets us group machines together into “services,” based on attributes they have in common. We might have a Rails Cluster service for our web application or even separate groups for our Fedora and Ubuntu servers.

We can chat with a service just like a single machine, except every server in the group runs the command simultaneously. For example, rather than patching a few dozen servers independently, we can send <em>yum update</em> to the service and patch all of them at once.

## Trust No One (Or Just Some People)
Granting access to servers is a pain. We need to create a user account, add it to the right groups, and update the sudoers file. Then repeat those steps on each server to which we need access. On top of that, we need to manage passwords.

A Vines service defines which unix accounts we're allowed to use when chatting with those servers. For example, we can limit our Development Web Servers users to the apache and postgresql accounts. Each developer doesn't need an account on these servers because they can get access to the tools they need through Vines.

We can give root access, to services, to the system administrators we already trust with that responsibility.

## Try It Out!
With services and centralized permissions, Vines shell adds some unique features to our trusty command line. Interacting with remote shell sessions is just the beginning of what we can do with servers connected to a real-time message system. There's much more to come.

Head on over to the <a href="http://www.getvines.com/">Vines website</a> to get started!

\- {{ page.author }}
<br/>{{ page.date | date_to_string }}
