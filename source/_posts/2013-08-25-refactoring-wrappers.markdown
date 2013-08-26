---
layout: post
title: "Refactoring Wrappers"
date: 2013-08-25 20:11
comments: true
categories: [Rails, Ruby, OOP, Wrapper, Paterns, service, best practices]
---
###Introduction
Very often in my everyday practice I faced a task when you need to create a communication between your own services or some 3rd party service that don't have a implementation yet. So what is the best way to doing that?

{% blockquote %}
Wrapper - it is a piece of code that will be responsible for communication between your application and external service.
{% endblockquote %}
<!-- more -->

###Identicating problem
When you have a task that implies creation of wrapper you could came up with the easiest solution like:

{% codeblock lang:ruby%}
class ServiceWrapper
  def self.server_rise(server)
    request = Net::HTTP.post_form(URI(Settings.service_url + "server_rise"), { server_name: server.name })
    if request['success']
      true
    else
      false
    end
  end
end
{% endcodeblock %}

But as everyone knew requirements are changes often :) So this could be finished like this:

{% codeblock lang:ruby%}
class ServiceWrapper
  def self.server_rise(server)
    request = Net::HTTP.post_form(service_url("server_rise"), { server_name: server.name })
    if request['success']
      true
    else
      false
    end
  end

  def self.add_platform(platform)
    request = Net::HTTP.post_form(service_url("add_platform"), { platform: platform.name})
    if request['success']
      request['response']
    else
      false
    end
  end

  def self.platform_rise(platform)
    request = Net::HTTP.post_form(service_url("platform_rise"), { platform: platform.name})
    if request['success']
      true
    else
      false
    end
  end

  def exceed_storage(server)
    request = Net::HTTP.post_form(service_url("exceed_storage"), { server_name: server.name })
    if request['success']
      request['response']
    else
      false
    end
  end

  def three_fourths_traffic(server)
    request = Net::HTTP.post_form(service_url("three_fourths_traffic"), { server_name: server.name })
    if request['success']
      request['response']
    else
      false
    end
  end

  def exceed_traffic(server)
    request = Net::HTTP.post_form(service_url("exceed_traffic"), { server_name: server.name })
    if request['success']
      request['response']
    else
      false
    end
  end

  def server_down(server)
    request = Net::HTTP.post_form(service_url("server_down"), { server_name: server.name })
    if request['success']
      request['response']
    else
      false
    end
  end

  private
  def self.service_url(url, options)
    URI(Settings.service_url + command)
  end
end
{% endcodeblock %}

This doesn't even looks like a object oriented approach, this is complete mess that could be extended became even worse. So what we could do about that?

###Understanding problem

When you develop application especially application in ruby(You remember that this is Object Oriented language, right?) you should keep in mind what objects you create and how messages goes thought them. Let's indicate our objects regarding this task scope:

* Server
* Platform
* Notification

So basically we have:

* Initiate Server creation ( isn't this sounds like <code>Object#new</code>? )
* Server down ( isn't this changing the state of object ? )
* Same for platform
* Simple notifications that actually could be treated exactly how you think about your [mailers](http://guides.rubyonrails.org/action_mailer_basics.html)

###Solution

Now we see what objects we have, lets write simple implementations for <code>Server</code>.

{% codeblock lang:ruby%}
class Wrapper::Server
  def new(server)
    @server = server
  end

  def self.save!
    response = Wrapper::Request.post("server_rise", server_name: @server.name)
    response.success?
  end
end
{% endcodeblock %}

From your code it will looks like this <code>Wrapper::Server.new(@server).save!</code> name it as a <code>wrapper</code> is a bad idea, so you should probably name it with a service_name and add Wrapper and the end if necessary. Let's see how <code>request</code> will looks like:

{% codeblock lang:ruby%}
module Wrapper::Request
  class Error < StandardError; end

  def self.post(url, params = {})
    response = RestClient.post(Settings.service_url + url, params.to_json, content_type: :json, accept: :json)
    Wrapper::Response.new(response.body)

  rescue RestClient::Exception => e
    Wrapper::Response.new(e.response)
  rescue Errno::ECONNREFUSED => e
    raise Wrapper::Request::Error, "Service is down"
  end
end
{% endcodeblock %}

And <code>response</code>

{% codeblock lang:ruby%}
class Wrapper::Response
  def initialize(json_body)
    json_body = json_body
  end

  def success?
    !!json_body['response']
  end
end
{% endcodeblock %}

This is just a proof of concept that will give your a direction in which you should move on. Now you will works with objects instead of hacks.

###Conclusion
You should always treat your messages as a communication between objects. Even if coding in this way takes more time. But the more time your spend today the less time you will need to spend in future.

