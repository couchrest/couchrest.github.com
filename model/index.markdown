---
layout: default
title: Modelling
id: model_index
---

# CouchRest Model

CouchRest Model adds additional functionality to the standard CouchRest Document. ActiveModel is used as a base and provides features such as setting properties, callbacks, typecasting, dirty tracking, and validations.

## Instalation

Simply include the library in your Rails 3 (or 3.1) project's Gemfile:

{% highlight ruby %}
gem :couchrest_model, '~> 1.2.0.beta'
{% endhighlight %}

After a quick `bundle install` you'll now be able to begin creating your CouchRest Models. For more details on configuring the database connection, see the [configuration section](/model/configuring.html).

## Generating Models

Generate your first model:

{% highlight ruby %}
rails generate model person
{% endhighlight %}

This will create a `app/models/person.rb` containing the most basic model definition:

{% highlight ruby %}
class Foo < CouchRest::Model::Base
end
{% endhighlight %}

From here you can now start to add [properties](/model/properties.html) to the model.


