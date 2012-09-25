---
layout: post
title: Metaprogramming ActiveRecord Connection Pools
author: David Graham
---
Using ActiveRecord database connections in a multi-threaded app, outside of Rails, is a bit of a mystery.  Since Rails typically takes care of connection cleanup, there's not much information on how to handle connections when using ActiveRecord in a stand-alone app.

But after piecing together a few helpful
[blog](http://blog.codefront.net/2009/06/15/activerecord-rails-metal-too-many-connections/)
[posts](http://bibwild.wordpress.com/2011/11/14/multi-threading-in-rails-activerecord-3-0-3-1/)
and finding the
[right class](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html)
to use inside ActiveRecord, it's actually pretty simple.

{% highlight ruby %}
def save_user(user)
  ActiveRecord::Base.connection_pool.with_connection do
    found = User.find_by_jid(user.jid)
    found.name = user.name
    found.password = user.password
    found.save
  end
end
{% endhighlight %}

ActiveRecord's with_connection method yields a database connection from its pool to the block.  When the block finishes, the connection is automatically checked back into the pool, avoiding connection leaks.

We don't use or even assign the yielded connection to a block variable. Any ActiveRecord call inside the block will use the connection automatically.

## Don't Repeat Ourselves

There are seven methods, today, in the Vines
[SQL storage class](https://github.com/negativecode/vines/blob/master/lib/vines/storage/sql.rb)
that need to use with_connection.  And while that's not a terrible burden, it would be nice if we didn't need to repeat this block of code in each place.

The storage layer in Vines is meant to be extended for custom database layouts, so it should give users an easy way to handle database connections out of the box.

## Metaprogramming to the Rescue

Using
[instance_method](http://www.ruby-doc.org/core-1.9.3/Module.html#method-i-instance_method)
together with
[define_method](http://www.ruby-doc.org/core-1.9.3/Module.html#method-i-define_method) is a simple technique for decorating a method with extra behavior. This takes an existing method and redefines it with some extra behavior wrapped around a call to the original implementation.

{% highlight ruby %}
def self.with_connection(method)
  old = instance_method(method)
  define_method method do |*args|
    ActiveRecord::Base.connection_pool.with_connection do
      old.bind(self).call(*args)
    end
  end
  defer(method)
end
{% endhighlight %}

In this case, the new method calls the original inside an ActiveRecord with_connection block.  This makes sure that connections are released back to the pool every time we use an ActiveRecord model in our methods.

Finally, the decorator defers the method call onto the EventMachine thread pool.  ActiveRecord uses blocking IO, which would block the EventMachine reactor thread.  Pushing these calls onto a thread pool allows the reactor thread to continue processing IO while we're waiting for the database query to return.

We won't get into the defer implementation here, but if you're interested,
[check out the code](https://github.com/negativecode/vines/blob/master/lib/vines/storage.rb).

## Putting It to Use

Now that we have the with_connection decorator written, we can use it to wrap all other methods that talk to ActiveRecord. Here's the save_user implementation using the decorator rather than dealing with ActiveRecord pools directly.

{% highlight ruby %}
def save_user(user)
  found = User.find_by_jid(user.jid)
  found.name = user.name
  found.password = user.password
  found.save
end
with_connection :save_user
{% endhighlight %}

And that's just one of our query methods.  We can wrap the others just as easily and consolidate connection pool handling in one place.

Metaprogramming is a pretty big hammer to use to solve a simple connection pool problem.  But define_method is a nice tool here, because it provides a simple way to write custom storage code without accidentally losing connections or blocking the EventMachine reactor thread.

\- {{ page.author }}
<br/>{{ page.date | date_to_string }}
