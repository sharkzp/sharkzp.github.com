---
layout: post
title: "Contract Tests"
date: 2014-01-20 23:31
comments: true
categories: [Rails, Ruby, OOP, RSpec, Capybara, testing, test]
---
{% blockquote Joe Rainsberger http://www.infoq.com/presentations/integration-tests-scam %}

Integration Tests are a Scam

{% endblockquote %}

If you still not using it, read this article and we will go through the process.

<!-- more -->

###Intro
I gave a [talk](https://slid.es/sharkzp/contract-tests) few weeks ago about this topic on a conference and understood that still a lot of people have not known about such kind of tests. So I will help you out to understand the problem and what we could do with that.

###Understanding the problem
The larger application we have more time we need to run our test suite. I hope a lot of us wrote integration tests, where your test tool click around application and check business process and UI. But the larger application we have - more objects will be involved in this process,
so when we go up to 10k LOC each and every integration test involves big amount of logic and spends a lot of time to pass through. Also when test is failing who could say which exactly piece of code is broken? But don't get me wrong I still think that we need to write integration tests, but I think it will be better to run them only on CI for master branch in a background.

Lets look on a problem from different perspectives.

###Purpose of Unit tests
When we write unit tests we always keep in mind object under test, this means:

{% img center /images/mock.png %}

These means that we mocking or stubbing all dependencies that we have in a code.
If Unit tests intend to test only one object, and integration tests does not provide great feedback what could we do?

###Test pyramid
I'm not a good painter but I think its pretty easy to understand.

{% img center /images/test_pyramid.png %}

This pyramid was popularized by Martin Fowler. It shows how your test suite should looks like in terms of coverage. But here we are missing contract tests. So here is updated version.

{% img center /images/test_pyramid_with_contract.png %}

So contracts test is a layer between integration tests and unit.

###What the hack?
Basically contract is a communication between two objects in application.

{% img center /images/contract.png %}

And testing this contract implies checking dependency between this objects.

###Practice! Finally! :)
For example we have two objects:

{% codeblock lang:ruby%}
class Tracker
  def time_to_handle_route(car, length)
    "#{(length.to_f / car.speed).round(2)} hour"
  end
end

class Honda
  def speed
    120
  end
end
{% endcodeblock %}

And we have tests for this objects
{% codeblock lang:ruby%}
#.../spec/tracker_spec.rb
require 'spec_helper'

describe Tracker do
  describe '#drive' do
    let(:tracker) { Tracker.new }
    let(:car)     { double(speed: 60) }
    let(:route)   { 120 }

    it 'returns time that will be spend for driving 120 km' do
      expect(tracker.time_to_handle_route(car, route)).to eq('2.0 hour')
    end
  end
end

#.../spec/honda_spec.rb
require 'spec_helper'

describe Honda do
  describe '#speed' do
    it 'returns speed for Honda' do
      expect(Honda.new.speed).to eq(120)
    end
  end
end
{% endcodeblock %}

What will be if we do a refactoring for <code>Honda</code> class. Because we think that method <code>speed</code> not descriptive enough.

{% codeblock lang:ruby%}
class Honda
  def speed_per_hour
    120
  end
end
{% endcodeblock %}

Also we change the test suite for this class
{% codeblock lang:ruby%}
require 'spec_helper'

describe Honda do
  describe '#speed_per_hour' do
    it 'returns speed for Honda' do
      expect(Honda.new.speed_per_hour).to eq(120)
    end
  end
end
{% endcodeblock %}

But take a look on our test for tracker class. Since we expecting method <code>speed</code>. Class Honda will be not applicable for Tracker. But our test suite will be still passing.
{% codeblock lang:ruby%}
"#{(length.to_f / car.speed).round(2)} hour"

#.../spec/tracker_spec.rb
let(:car)     { double(speed: 60) }
{% endcodeblock %}

Here is where contract test will be useful.
{% codeblock lang:ruby%}
require 'spec_helper'

describe 'Contract between Honda and Tracker' do
  context 'Tracker' do
    it 'responds to #time_to_handle_route' do
      Tracker.new.respond_to?(:time_to_handle_route).should be_true
    end
  end

  context 'Honda' do
    it 'responds to speed' do
      Honda.new.respond_to?(:speed).should be_true
    end
  end
end
{% endcodeblock %}

In this way we will test communication between objects. We don't need to know how this methods will be implementing but we expecting that this method will have such methods in their public interface.

###Summary
You can always write such simple and easy tests by your own, but if you want to go beyond of that I recommend you to read about [bogus](https://github.com/psyho/bogus). And watch this talks: J. B. (Joe) Rainsberger - Integration Tests Are a Scam, Ruby Conf 12 - Boundaries by Gary Bernhardt.
