---
layout: post
title: Vines XMPP Server, Now With Clustering!
author: David Graham
---
We're proud to announce the fourth preview release of the [Vines XMPP server](http://www.getvines.org)!  In addition to bug fixes and performance improvements, there are three important new features in this release.

## Two Servers Are Better Than One
Thanks to EventMachine, a single Vines chat server can handle plenty of traffic. But eventually, we'll need a second server (or more) to handle higher loads and provide high-availability should one server fail.  In this release, several Vines servers can cluster together, using Redis to route messages among themselves.

I'll write more in-depth about how the clustering feature works in the future.  For now, it's easy to get started by adding this snippet to your config file.

{% highlight ruby %}
cluster do
  host 'redis.wonderland.lit'
  port 6379
end
{% endhighlight %}

Point two or more Vines servers to the same Redis database and you're done!

## Publish/Subscribe
The XMPP standard includes an extension to support the [Publish/Subscribe](http://xmpp.org/extensions/xep-0060.html) (a.k.a pubsub) design pattern.  This pattern allows connected users to publish messages to a channel without necessarily knowing who's subscribed to that channel.

This feature is big because Vines can now support more than just instant-messaging applications.  Publish/Subscribe is the basis of a new service we're about to launch into beta. Stay tuned for more on that in the near future!

## MongoDB
And finally, user accounts can now be stored in [MongoDB](http://www.mongodb.org/) databases. Vines can connect to a single MongoDB instance or to a replica set for high-availability.

So Vines can store information in pretty much any database out of the box: simple YAML files, SQL databases, Redis, CouchDB, and MongoDB.  And it still has an easy to customize storage API in case one of these options doesn't quite fit your environment.

## Try It Out!
If this is the first time you're installing Vines, make sure to follow our [Ruby install instructions](http://www.getvines.org/ruby) and then run *gem install vines*.  If you already have Vines installed, just update the Ruby gem with *gem update vines* to install the new version.

\- {{ page.author }}
<br/>{{ page.date | date_to_string }}
