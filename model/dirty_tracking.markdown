---
layout: default
title: Dirty Tracking
id: model_dirty_tracking
---

# Dirty Tracking

Since version 1.1.0, CouchRest Model has support for dirty tracking based on the support modules provided by ActiveModel. This works as you'd expect by allow special method calls to check if a specific attribute or anything in the object has changed.

Unlike a relational database that only has to deal with two dimensional tables, CouchRest Model supports dirty tracking on nested models. Changes on a deeply nested object will be passed through the parent properties and documents until reaching the base document.

## Viewing and checking for changes

{% highlight ruby %}
class Article < CouchRest::Model::Base
  property :title, String
end

@article = Article.new(:title => "Test Article")

# Has the article changed?
@article.changed?  #=> true

# Which properties have changed?
@article.changed   #=> ['title']

# Get a hash of all the changes (maybe long if there are lots of nested models)
@article.changes   #=> { 'title' => [nil, "Test  Article"] }

# Perform checks on specific properties:
@article.title_changed?   #=> true
@article.title_change     #=> [nil, "Test Article"]
@article.title_was        #=> nil
{% endhighlight %}

## Undoing changes

{% highlight ruby %}
@article = Article.first
@article.title = "Changed Title"
# Undo the change
@article.reset_title!
@article.title    #=> "Test Article"
{% endhighlight %}

