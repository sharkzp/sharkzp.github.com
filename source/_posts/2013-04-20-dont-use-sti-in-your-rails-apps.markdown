---
layout: post
title: "Don't use STI in your Rails Apps"
date: 2013-04-20 11:33
comments: true
categories: [Rails, Ruby, OOP, STI, best practices]
---
{% blockquote Sincerely Your Wiki http://en.wikipedia.org/wiki/Single_Table_Inheritance %}

Single table inheritance is a way to emulate object-oriented inheritance in a
relational database. When mapping from a database table to an object in an
object-oriented language, a field in the database identifies what class in the hierarchy the object belongs to

{% endblockquote %}

Single Table Inheritance(STI) is a useful and sharp technique,
and like any other high level instrument you should use it in an appropriate place.
Let's take a closer look with examples
<!-- more -->
For example you have a Blog, inside this blog you will have for sure such objects:

* Image
* User Avatar
* Comments Attachments

First thing that you should create is a table from which you will inherit:

{% codeblock lang:ruby%}
# app/models/asset.rb
class Asset < ActiveRecord::Base
  # Fields:
    #   String attachment
    #   String type
end

# app/models/image.rb
class Image < Asset
  mount_uploader :attachment, ImageUploader
end

# app/models/avatar.rb
class Avatar < Asset
  mount_uploader :attachment, AvatarUploader
end

# app/models/comment_attachment.rb
class CommentAttachment < Asset
  mount_uploader :attachment, CommentAttachmentUploader
end
{% endcodeblock %}

###Let's see what will happens in terminal:
{% codeblock lang:ruby%}
  1.9.3-p194 :001 > asset = Asset.create
  => #<Asset id: 1, type: nil, attachment: nil>

  1.9.3-p194 :004 > comment_attachment = CommentAttachment.create
  => #<CommentAttachment id: 4, type: "CommentAttachment", attachment: nil>

  1.9.3-p194 :006 > Asset.select('id, type')
  => [#<Asset id: 1, type: nil>, #<Image id: 2, type: "Image">, #<Avatar id: 3, type: "Avatar">, #<CommentAttachment id: 4, type: "CommentAttachment">]
{% endcodeblock %}

###How does it work?

{% blockquote http://api.rubyonrails.org/classes/ActiveRecord/Base.html %}
Active Record allows inheritance by storing the name of the class in a column that by default is named “type” (can be changed by overwriting Base.inheritance_column)
{% endblockquote %}

So when you inherit from some table your child tables will insert class name in <code>type</code> column.
This looks good, you create one table, make an abstraction and right now everything work nice.
When you will specify each uploader you could create different behavior and image size(I will not cover it in this post feel free to visit [This link](https://github.com/jnicklas/carrierwave))
You could go to your boss and say they your are awesome :)

##Where is the evil?
What will happens when you will create a User?
{% codeblock lang:ruby%}
# app/models/user.rb
class User < ActiveRecord::Base
  has_one :avatar
end

# app/models/avatar.rb
class Avatar < Asset
  belongs_to :user
end
{% endcodeblock %}

You should add <code>user_id</code> to your asset table.
Oh, yes you also need a <code>Post</code>, that will include <code>Images</code>, and yes you have a <code>Comments</code>, that attach to you <code>CommentAttachment</code>.
And in the end you will face this kind of scheme:
{% codeblock lang:ruby%}
  create_table "assets", :force => true do |t|
    t.string   "type"
    t.string   "attachment"
    t.string   "user_id"
    t.string   "post_id"
    t.string   "comment_id"
    t.string   "commentator_id"
    t.datetime "created_at", :null => false
    t.datetime "updated_at", :null => false
  end
{% endcodeblock %}

Half of your table will be filled with <code>NULL</code>.
Maybe someday you will decide to use your <code>CommentAttachment</code> for attaching <code>Audio</code> and <code>video</code>?
{% codeblock lang:ruby%}
  create_table "assets", :force => true do |t|
    t.string   "type"
    t.string   "attachment"
    t.string   "user_id"
    t.string   "post_id"
    t.string   "comment_id"
    t.string   "commentator_id"
    t.string   "audio_bitrate"
    t.string   "thumbnail"
    t.boolean  "is_in_gallery" #for cases when you create a gallery of images
    t.string   "artist"
    t.string   "album"
    t.integer  "year"
    t.string   "genre"
    t.integer  "bitrate"

    t.datetime "created_at", :null => false
    t.datetime "updated_at", :null => false
  end
{% endcodeblock %}

Right now your table will looks like a crap. And your project will became soo huge that you could not to refactoring well.

##Conclusion
Use STI only when you totally sure that you will never change it.
This post based on a true story.
