---
layout: post
title: "Monads the heart of the matter"
date: 2013-04-27 12:33
comments: true
categories: [Rails, Ruby, OOP, Monads, functional programming, best practices]
---
###Introduction
Monads came form functional programming like Haskell. And this is not a secret from no-one that Ruby takes some parts from this languages.
So, what is Monad about?

{% blockquote %}
Monad is a structure that represents computations. This allows the programmer to build pipelines that process data in steps, in which each action is decorated with additional processing rules provided by the monad. Today we will create monad and see how it will help us.
{% endblockquote %}
<!-- more -->
Monad is made up of three things:

* Container for a value.
* Wrap the parameter into instance of monad.
* Binding functions to the container.

Let's create it step by step
{% codeblock lang:ruby%}
class Monad
  def initialize(value)
    @v = value
  end

  def unit(value)
    self.class.new(value)
  end

  def bind(f)
    self.unit(f.call(@v))
  end
end
{% endcodeblock %}

It will produce
{% codeblock lang:ruby%}
  h = { a: { c: { d: { e: 4 } } } }
  Monad.new(h).bind(lambda{|c| c.fetch(:a)}).bind(lambda{|c| c.fetch(:c)})
  #=> {:d=>{:e=>4}}

  v = 10
  Monad.new(v).bind(lambda{|c| c**2 }).bind(lambda{|c| c / 2 })
  #=> 50
{% endcodeblock %}

This is the basic realization of IO Monads.

###There are few other types.
1. IO Monad - take result from first function and pass it to second.
2. Maybe - if first function return value than pass it to second function, otherwise return nil. This one implemented as a gem [This link](https://github.com/pzol/monadic).
3. Either monad - if function will fail it will return Failure object, otherwise it will return success. This used for handling errors and fighting exceptions.
And others.

###Simple implementation of Maybe monad
{% codeblock lang:ruby%}
class Monad
  def initialize(value)
    @v = value
  end

  def unit(value)
    self.class.new(value)
  end

  def bind(f)
    begin
      self.unit(f.call(@v))
    rescue
      self.unit(nil)
    end
  end
end
{% endcodeblock %}

It will produce
{% codeblock lang:ruby%}
h1 = { a: { c: { d: { e: 4 } } } }
h2 = {}
f1 = lambda {|c| c.fetch(:a) }
f2 = lambda {|c| c.fetch(:c) }
f3 = lambda {|c| c.fetch(:d) }
f4 = lambda {|c| c.fetch(:e) }

Monad.new(h1).bind(f1).bind(f2).bind(f3).bind(f4)
#=> 4
Monad.new(h2).bind(f1).bind(f2).bind(f3).bind(f4)
#=> nil
{% endcodeblock %}
This way you could avoid exception handling in you applications that depend on external request
For example
{% codeblock lang:ruby%}
class TwitterCallbackController < ActionController::Base
  def create
    if partner_uuid = Monad.new(request).fetch(:body).fetch(:partner_uuid)
      User.authenticate_by_uuid(partner_uuid)
    end
  end
end
{% endcodeblock %}

###Other fields of interest:
* Fetching Data from Database
* Processing XML/SOAP results
* Complex calculations
* etc

##Resume
Monad its a powerful technique that could be used in your application in various places. But it could encapsulate some intermediate results that you may want to see. For example error handling could be more useful when you will see real answer from 3rd party applications.
