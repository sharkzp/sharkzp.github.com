---
layout: post
title: "All about Duck Types"
date: 2013-06-16 23:26
comments: true
categories: [Rails, Ruby, OOP, SOLID, Duck types, best practices]
---
###Introduction
I heard a lot about Duck Types in Rails but I have never saw how can I used in a real-time application.
Today I will show you how to use duck types in examples.

{% blockquote %}
Duck Typing is a style of dynamic typing in which an object's methods and properties determine the valid semantics, rather than its inheritance from a particular class or implementation of a specific interface.
{% endblockquote %}
<!-- more -->

This sounds a bit weird for me, lets make an example.
For instance you have a class that responsible for generating some View-presenter data according to objects that you pass

{% codeblock lang:ruby %}
  class Presenter
    def description(subject)
      case subject
      when Post
        subject.as_json.merge!(commets_counter: true)
      when WallPost
        subject.to_hash.merge!(commets_counter: true)
      when Video
        subject.attributes.merge!(title: 'some title')
      when Audio
        subject.to_json
      when NilClass
        nil
      else
        raise ArgumentError
      end
    end
  end
{% endcodeblock %}

##What's wrong?
In some day you could came up to such solution. And on what you should take a look
This violates SOLID principles.

1. Why presenter class should know about view representation about each class?
2. We did not make any modifications with data that came up from class, than why we cannot move this method to class itself?

Let's refactoring this:
{% codeblock lang:ruby %}
class Presenter
  def description(subject)
    case subject
    when Post
      subject.description
    when WallPost
      subject.description
    when Account
      subject.description
    when Video
      subject.description
    when Audio
      subject.description
    when NilClass
      nil
    else
      raise ArgumentError
    end
  end
end

#For other classes this will be similar to
class Post
  def self.description
    as_json.merge!(commets_counter: true)
  end
end
{% endcodeblock %}
If subject will not implement description than rails will raise an exception.
Next iteration of refactoring
{% codeblock lang:ruby %}
class Presenter
  def description(subject)
    subject.description
  end
end
{% endcodeblock %}
Congratulation you successfully implement Duck type :)
But now our presenter looks like [lazy class](http://sourcemaking.com/refactoring/lazy-class).
So this class could be totally removed.

##Resume
If something looks like a duck and quack like a duck than it is a duck.
Also it will be good to test such Ducks using rspec [shared_examples](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)
