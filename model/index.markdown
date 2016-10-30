---
layout: default
title: Modelling
id: model_index
---

# CouchRest Model

CouchRest Model adds additional functionality to the standard CouchRest Document. ActiveModel is used as a base and provides features such as setting properties, callbacks, typecasting, dirty tracking, and validations.

## Installation

### Rails 5

#### Gem

    $ sudo gem install couchrest_model

#### Bundler

Simply put the following line into your Gemfile

    gem 'couchrest_model'

### Rails 3
Simply include the library in your Rails 5 (or 3.1) project's Gemfile:

{% highlight ruby %}
gem :couchrest_model, '~> 1.2.0.beta'
{% endhighlight %}

After a quick `bundle install` you'll now be able to begin creating your CouchRest Models. For more details on configuring the database connection, see the [configuration section](/model/configuring.html).

## Configuration

The library is looking for a file in *config/couchdb.yml*. You can generate it
by using the following command:

    $ rails generate couchrest_model:config

It should looks like that:

    development: &development
      protocol: 'http'
      host: localhost
      port: 5984
      prefix: your_prefix
      suffix: development
      username: your_username
      password: your_password
    test:
      <<: *development
      suffix: test
    production:
      protocol: 'https'
      host: localhost
      port: 5984
      prefix: your_prefix
      suffix: production
      username: root
      password: 123


## Generating Models

Generate your first model:

{% highlight ruby %}
rails generate model person
{% endhighlight %}

This will create a `app/models/person.rb` containing the most basic model definition:

{% highlight ruby %}
class Person < CouchRest::Model::Base
end
{% endhighlight %}

From here you can now start to add [properties](/model/properties.html) to the model.


