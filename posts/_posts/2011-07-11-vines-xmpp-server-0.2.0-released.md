---
layout: post
title: Vines XMPP Server 0.2.0 Released
author: David Graham
---
We're proud to announce the second preview release of the [Vines XMPP server](http://www.getvines.com)!  Along with many bug fixes and improvements, there are three important new features in this release.

## File Transfers
The chat server now supports in-band and out-of-band file transfers.  You can drag and drop a file onto any XMPP chat client and transfer it to anyone on your buddy list.

## HTTP Server
We've enhanced our web support with a simple HTTP server capable of serving static files. You'll typically run this, in production, behind a caching proxy server like nginx, Apache mod_proxy, or HAproxy to gain TLS and clustering support.

## Web Chat
This release includes a new web chat client built entirely with [CoffeeScript](http://www.coffeescript.org). There's no application server to configure (e.g. Rails) as the app connects directly to the XMPP server using HTTP.  The web application is available immediately after starting the chat server at http://localhost:5280/chat/.

<figure class="screenshot">
<a href="/images/vines-web.png"><img src="/images/vines-web.png" alt="Vines Web Chat"/></a>
<figcaption>Vines Web. <a href="/images/vines-web.png">Click for larger image</a>.</figcaption>
</figure>

The user interface is designed for desktop browsers as well as tablets. We're still tweaking some details on iPad, but give it a try and let us know what you think!

## Try It Out!
If this is the first time you're installing Vines, make sure to follow our [Ruby install instructions](http://www.getvines.com/ruby) and then run *gem install vines*.  If you already have Vines installed, just update the Ruby gem with *gem update vines* to install the new version.

\- {{ page.author }}
<br/>{{ page.date | date_to_string }}
