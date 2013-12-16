---
layout: post
title: "Callbacks Extraction"
date: 2013-12-16 17:22
comments: true
categories: [Rails, Ruby, OOP, best practices, Callback, hell, Extraction]
---
{% blockquote Edsger W. Dijkstra http://www.u.arizona.edu/~rubinson/copyright_violations/Go_To_Considered_Harmful.html %}

The unbridled use of the go to statement has an immediate consequence that it becomes terribly hard to find a meaningful set of coordinates in which to describe the process progress.

{% endblockquote %}

Rails provide powerful callback technique and powerful tool it could injure in newbie hands.

<!-- more -->

###Understanding the problem
For example we have simple blog engine. Where we have <code>Post, User, Comment</code>
And system should create a default post with random comments when <code>User</code> with type <code>admin</code> creates. This could through rails callbacks

{% codeblock lang:ruby%}
class User < ActiveRecord::Base
  after_create :create_post, if: :admin?

  private
  def create_post
    posts << Post.create(default_post_attributes)
  end
end

class Post < ActiveRecord::Base
  after_create :create_comments

  private
  def create_comments
    comments << Comment.create(defaul_comment_attributes)
  end
end

class Comment < ActiveRecord::Base
  before_create :ensure_user_existance

  private
  def ensure_user_existance
    unless user
      user = User.create(default_user_attrubutes)
    end
  end
end
{% endcodeblock %}

After some time passed you decide that creating user for comments was a bad idea and you remove it.
But when you create another one admin you need to search through three different objects to find out the root of problem. This is the simplest example but you could find even more weird problems and real hell in practice.

###Solution
For extracting this behavior out of objects we need to keep in mind some of best practices. Sandi Matz recommends to keep your objects unitary and easy to follow check out [this](http://robots.thoughtbot.com/sandi-metz-rules-for-developers) blog post for more details.

So lets introduce new object that for now will encapsulate logic around admin user

{% codeblock lang:ruby%}
class AdminUser
  def save
    user.save
    create_post
  end

  private
  def create_post
    posts << Post.create(comments: [Comment.create(user: user)])
  end
end

class User < ActiveRecord::Base
end

class Post < ActiveRecord::Base
end

class Comment < ActiveRecord::Base
end
{% endcodeblock %}

This way we could extract any kind of callbacks out of User class that belongs to specific type of user of certain conditions.

{% codeblock lang:ruby%}
class AdminUser
  attr_reader :attributes

  def initialize(options = {})
    before_initialize
    attributes = options[:user]
    after_initialize
  end

  def save
    before_save
    user.save
    after_save
  end

  private
  def user
    User.new(attributes)
  end

  #...
end
{% endcodeblock %}

I recommend to always keep your objects as clean and small as possible, so any changes that you need to introduce touch single class that easy to understand and follow.

###Resume
Extracting callbacks out of Domain Class could be useful refactoring technique and any of us should use to avoid callback hells and code smells. Unitary class is much easier to test and change.
