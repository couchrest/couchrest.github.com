---
layout: default
title: Definition
no_comments: true
---

# Defining a CouchRest Model

## Basics

To create a new model, we need to define a class that inherits from the `CouchRest::Model::Base` class in much the same way as an ActiveRecord model.

{% highlight ruby %}
class Person < CouchRest::Model::Base
  property :first_name, String
  property :last_name, String
  timestamps!
end
{% endhighlight %}

Unlike a regular database mapper, CouchRest Model requires you to define the properties or attributes your model needs to use directly in the class. There are no migrations or automatic column detection in a Document Database! The [properties section](/model/properties.html) goes into more detail about how to define your own.

## Setting the database

CouchDB is accessed via HTTP requests, as such, there is no requirement to create a binary connection to a specific database. This allows databases to be defined _at run time_ as opposed to on initialisation. For example:

{% highlight ruby %}
# Create a new people class using a specific database
class Person < CouchRest::Model::Base
  use_database 'people'
  property :first_name, String
  property :last_name, String
  timestamps!
end
{% endhighlight %}

The `use_database` method tells the model which database it should use. This can be provided as a `CouchRest::Database` object or as simple string that will be concatenated to the connection prefix. See the [configuration section](/model/configuring.html) for more details.

## Document ids

When a CouchDB document is saved, or "persisted" to the database, it is automatically assigned `_id` and `_rev` variables. These can be accessed using the `Model#id` and `Model#rev` methods.



