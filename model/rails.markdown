---
layout: default
title: Start with Rails
id: model_start_with_rails
---

# Getting start with Rails and Couchrest_model

It’s not difficult to use Couchdb with Rails 3. Most of it comes down to making sure that you’re not loading ActiveRecord and understanding how to manager the new Rails dependency.

## Installing Couchdb

1. For couchdb, We need install  g++, erlang  and few adapter, so that couchdb will work smoothly.

Run below command form terminal.

{% highlight ruby %}
$ sudo apt-get install g++

$ sudo apt-get install erlang-base erlang-dev erlang-eunit erlang-nox

$ sudo apt-get install libmozjs185-dev libicu-dev libcurl4-gnutls-dev libtool
{% endhighlight %}

2. One we done with above installation, go to couchdb site and download coach db source file.

In a terminal, go to the folder where you have downloaded the file, extract and go to bin folder and run below commands.

{% highlight ruby %}
$ ./configure

$ makee

$ sudo make install
{% endhighlight %}

That's it, you done with coachdb installation.

## Creating a new Rails Application With couchrest_model

The important thing here is to avoid loading ActiveRecord. So we will use --skip-active-record to skip ActiveRecord initialization.

{% highlight ruby %}
$rails new cast --skip-active-record
{% endhighlight %}

The first thing we need to do is to add the Couchrest_model gem to the Gemfile.

{% highlight ruby %}
source 'https://rubygems.org' application

gem 'rails', '3.2.13'
gem 'couchrest_model'
group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'therubyracer', :platforms => :ruby
  gem 'uglifier', '>= 1.0.3'
end
gem 'jquery-rails
{% endhighlight %}

## Bundling

Once you’ve configured our Gemfile, We can then install the gems running the bundle installer:

{% highlight ruby %}
$ bundle install

Installing couchrest (1.1.3)
Installing couchrest_model (1.1.2)
Using jquery-rails (3.0.4)
Using rails (3.2.13)
Using sass (3.2.9)
Using sass-rails (3.2.6)
Using uglifier (2.1.2)
Your bundle is updated!
{% endhighlight %}

## Configuring

Once the gems have installed we’ll need to run the Couchdbrest_model configuration generator so that it can create the configuration YAML file.

{% highlight ruby %}
$rails generate couchrest_model:config
{% endhighlight %}

The default file is shown below. We can leave it as it is while we’re developing our application.

{% highlight ruby %}
config/couchdb.yml

development: &development
  protocol: 'http'
  host: localhost
  port: 5984
  prefix: cast
  suffix: development
  username:
  password:

test:
  <<: *development
  suffix: test

production:
  protocol: 'https'
  host: localhost
  port: 5984
  prefix: cast
  suffix: production
  username: root
  password: 123
{% endhighlight %}

Everything is in place now for us to begin building our application. We’ll start by creating an User model with email, first_name and last_name fields and use Rails’ scaffolding to create the associated controller and view code.

{% highlight ruby %}
$rails g scaffold user email:string fname:string lname:string
{% endhighlight %}

If we open up the model file we’ll see that it’s a simple class that inherit  CouchRest::Model::Base

{% highlight ruby %}
class User < CouchRest::Model::Base
end
{% endhighlight %}

you can now start to add properties to the model.

To define a property, simply call the property class method in your model definition:

{% highlight ruby %}
class User < CouchRest::Model::Base
  property :email, String
  property :fname, String
  property :lname, String   
end
{% endhighlight %}

Our application is now ready to run. We don’t need to run any database migrations as CouchDB is schema-less. Now it's time to run server. Make sure your Couchdb(sudo couchdb) is running in other terminal OR background.