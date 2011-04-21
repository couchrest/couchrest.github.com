---
layout: default
title: Modelling
id: model_index
---

# CouchRest Model

CouchRest Models adds additional functionality to the standard CouchRest Document class such as
setting properties, callbacks, typecasting, and validations.

Originally called ExtendedDocument, the new Model structure uses ActiveModel, part of Rails 3, 
for validations and callbacks.

If your project is still running Rails 2.3, you'll have to continue using ExtendedDocument as 
it is not possible to load ActiveModel into programs that do not use ActiveSupport 3.0.

CouchRest Model is only properly tested on CouchDB version 1.0 or newer.

*WARNING:* As of April 2011 and the release of version 1.1.0, the default model type key is 'model' instead of 'couchrest-type'. Simply updating your project will not work unless you migrate your data or set the configuration option in your initializers:

{% highlight ruby %}
CouchRest::Model::Base.configure do |config|
  config.model_type_key = 'couchrest-type'
end
{% endhighlight %}

This is because CouchRest Model's are not couchrest specific and may be used in any other system such as a Javascript library, the model type should reflect this.


