---
layout: post
title: Fibers &amp; Resumable Recursive Descent JSON Parsing
author: David Graham
---
The first thing I tried while developing the [json-stream](http://www.lousymedia.com/json-stream/) gem was writing a [recursive descent parser](http://en.wikipedia.org/wiki/Recursive_descent_parser). They're great because the parsing code looks like the syntax description of the language being parsed. Unfortunately, they can't pause while waiting for more text to complete the parse.

We need to provide a recursive descent parser a complete JSON document, then wait for the entire parse to finish before continuing on. This is fine for a programming language compiler, but doesn't work when we're receiving JSON data in small pieces over the network.

Now that Ruby 1.9 is widely used, I thought it would be fun to revisit this problem with [Fibers](http://www.rubyinside.com/ruby-fibers-8-useful-reads-on-rubys-new-concurrency-feature-1769.html).

## Resumable Parsing

Here's a sample program that demonstrates a worst-case scenario: JSON text arriving to our parser one character at a time.

{% highlight ruby %}
parser = Parser.new
json = '[1, 2, 3, [4, 5, 6]]'
json.each_char do |ch|
  if result = parser << ch
    puts result.inspect
    # parse finished, process result object
  end
  # do other work
end
{% endhighlight %}

We pass a JSON chunk into the parser and process the result object that's returned when parsing is complete.  But if this piece of data didn't complete the parse, we need the parser to remember its current state, so we can feed it more data when it's available.

## Pausing Recursion With Fibers

Each Ruby Fiber gets its own call stack separate from our main program.  They're similar to threads in that way; however, fibers always run on the thread that calls them.

So if we wrap a Fiber around our recursive descent parser, we can feed it data, parse up to the end of the available chunk, and suspend the call stack until we have more data with which to complete the parse. The level of recursion inside the parser is maintained so it can pick up where it left off.

Let's look at the two classes responsible for parsing a stream of JSON data: Parser and Scanner. These classes use a Fiber to coordinate the parsing.

## Scanner

The Scanner is responsible for maintaining the buffer of JSON data that hasn't been parsed yet.  It's basically just the data source for the Parser.  In a traditional JSON parser the data source would typically be a String or StringIO object.

{% highlight ruby %}
class Scanner
  def <<(data)
    # more data has arrived
    @buf += data.split('')
  end

  def peek
    # pause when we're out of data
    Fiber.yield if @buf.empty?
    @buf.shift
  end
end
{% endhighlight %}

The trick to this data source is that it yields/pauses the Fiber it's running in when it's out of data.  It will feed the Parser characters, one at a time, until there's nothing left to parse and pause the Fiber's stack.  Now we need something that will resume the Fiber when there's more data in the buffer.

## Parser

The Parser is responsible for reading characters from the Scanner and building Ruby objects from the JSON tokens it recognizes.  If it encounters a character it didn't expect, it raises an error for the improperly formatted JSON it was given.

Most importantly, in this case, it must resume the Fiber when it's given more data to parse.

{% highlight ruby %}
class Parser
  def initialize
    @scanner = Scanner.new
    # run the parse method in a fiber
    @fiber = Fiber.new do
      Fiber.yield parse
    end
  end

  def <<(data)
    # resume parsing when more data arrives
    @scanner << data
    @fiber.resume
  end
end
{% endhighlight %}

The Scanner yields the Fiber and the Parser resumes it. This back and forth happens until a full JSON object is parsed from the stream. Fibers allowed us to write an easy-to-read recursive descent parser that also handles partial JSON data streams.

I simplified these code samples to highlight just the Fiber interaction.  A working, fibered JSON parser is available at [this gist](https://gist.github.com/2210713), though.

So, is this technique just for crazy people who need specialized JSON parsers or are there other places this is useful?

\- {{ page.author }}
<br/>{{ page.date | date_to_string }}
