---
layout: post
title: Why I Love Programming in Ruby
author: David Graham
---
A friend asked me recently why I enjoy programming in Ruby so much and I thought I'd share my thoughts here, with you, as well.  I'll focus on one piece of sample code, letting Ruby do most of the talking, that highlights some of my favorite features.

This example comes from some real code I was working on.  We'll refactor it step by step until we get something we're proud of.  Each time through, we'll improve the code by using more and more of Ruby's features.

## Split an Array Into Keys and Values

Our goal is to write a split method that accepts an array and partitions it into two different arrays of keys and values.  The unit test for split would look something like this.

{% highlight ruby %}
def test_split
  params = ['foo', 12, 'bar', 42]
  keys, values = split(params)
  assert_equal ['foo', 'bar'], keys
  assert_equal [12, 42], values
end
{% endhighlight %}

## Array#each

The first attempt creates a couple arrays, loops through params, and adds the item to keys if its loop index is even or to values if it's odd.

{% highlight ruby %}
def split(params)
  keys = []
  values = []
  index = 0
  params.each do |param|
    if index % 2 == 0
      keys << param
    else
      values << param
    end
    index += 1
  end
  [keys, values]
end
{% endhighlight %}

The neat thing about this is the Array#each iteration method that accepts a block of code (between the do/end keywords) and runs it once for each item in the array.  Internal iterators are one of my favorite Ruby features.

## Array#each_with_index

We can tighten this up a bit by using multiple assignment to define the keys and values arrays on one line.  And rather than define and increment the index variable ourselves, we'll let the Array#each_with_index iterator do it for us.  Now the outer scope isn't cluttered with index variables that are only needed for the iterator block.

{% highlight ruby %}
def split(params)
  keys, values = [], []
  params.each_with_index do |param, index|
    if index.even?
      keys << param
    else
      values << param
    end
  end
  [keys, values]
end
{% endhighlight %}

As a bonus, we used the Fixnum#even? method on the index, which reads a little easier than the modulo operator.

## Array#partition

Array's each and each_with_index methods are nice, but not as powerful as [Array#partition](http://www.ruby-doc.org/core-1.9.3/Enumerable.html#method-i-partition). This iterator splits one array into two, based on the block you've given it.  In our case, if the block returns true, the item goes into the keys array, false and it goes into values.

{% highlight ruby %}
def split(params)
  index = 0
  keys, values = params.partition do |param|
    even = index.even?
    index += 1
    even
  end
  [keys, values]
end
{% endhighlight %}

So Array#partition is convenient, but now we're back to maintaining the index variable again. This still feels like too much bookkeeping, just to split an array.

## Enumerators

It turns out that if you don't pass a block of code to Array#partition, it returns an [Enumerator](http://www.ruby-doc.org/core-1.9.3/Enumerator.html) object instead.  Enumerators have a with_index method that will maintain the current loop index for us.

{% highlight ruby %}
def split(params)
  params.partition.with_index {|param, index| index.even? }
end
{% endhighlight %}

By combining two Enumerators together, we can partition an array based on item indexes in a single, readable, beautiful line of code.

## What Do You Think?

Is there an easier way to split this array that I missed?  What's your favorite language feature, in Ruby or otherwise?  Let me know in the comments!

\- {{ page.author }}
<br/>{{ page.date | date_to_string }}
