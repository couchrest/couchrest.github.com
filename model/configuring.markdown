---
layout: default
title: Configuring
id: model_configuring
---

# Configuring

CouchRest Model supports several configuration options coverting connection settings, environment, and special object modelling options. They can be set either for the entire project or for a specific model of your choosing.

To configure globally, provide something similar to the following in your projects initializers or environments:

{% highlight ruby %}
CouchRest::Model::Base.configure do |config|
  config.mass_assign_any_attribute = true
  config.model_type_key = 'couchrest-type'
end
{% endhighlight %}

To set for a specific model:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  mass_assign_any_attribute true
end
{% endhighlight %}

Options currently avilable are:

 * [`:model_type_key`](#model_type_key)
 * [`:mass_assign_any_attribute`](#mass_assign)
 * [`:auto_update_design_doc`](#design_doc)
 * [`:environment`](#environment)
 * [`:connection`](#connection)
 * [`:connection_config_file`](#connection)

<a id="model_type_key"> </a>
## `model_type_key`

By default, this option is just `type`.

CouchDB does not distinguish between documents in a database, so in order for us to know what type of model we're dealing with, we need to define and set an attribute in each document that defines the class that should be used to instantiate a new object.

The `model_type_key` should not be changed in a database that has already been used for storing data. Unless a manual data migration is performed, CouchRest Model view requests would not be able to find your data as the design docs would still be using the old key.

In versions of CouchRest Model before 1.1.0, the key was changed from `couchrest-type` to `type`. If you are upgrading from an earlier version, you might want to use this configuration option to avoid migrating data.

<a id="mass_assign"> </a>
## `mass_assign_any_attribute`

False by default. When true any attribute may be updated via the `new`, `update_attributes` or `attributes=` methods.

This is disabled by default to follow the convention set by normal object mappers and to avoid any possible security problems. For example, if you enable this option and fail to set any checks in any HTML forms received from the browser, the client would be able to store anything in your database:

Take the following action in a controller:

{% highlight ruby %}
# Find and update person
def update
  @person = Person.get(params[:id])
  @person.update_attributes(params[:person])
end
{% endhighlight %}

Imagine the following simplified HTML form that has been modified by a malicious user:

{% highlight html %}
<form>
  <input type="text" name="person[name]" />
  <input type="hidden" name="person[spam]" value="Some random text you don't want">
</form>
{% endhighlight %}

If `mass_assign_any_attribute` is true, the `spam` attribute in the html form will be stored is if it was any other attribute.

<a id="design_doc"> </a>
## `auto_update_design_doc`

True by default. Every time a view is requested and this option is true, a quick check will be performed to ensure the model's design document is up to date. When disabled, design documents will never be updated automatically and you'll need to perform updates manually.

Results are cached on a per-database and per-design basis to help lower the number of requests. See the [View section](/model/view_objects.html) for more details.


<a id="environment"> </a>
## `environment`

Default is determined from the Rails `Rails.env` value or resorts to `:development` if Rails is not available. Used essentially for determining the path of the connection configuration file (next).


<a id="connection"> </a>
## `connection` and `connection_config_file`

The `connection` option is a Hash of options used to connect to the database. The default is as follows:

{% highlight ruby %}
config.connection = {
  :protocol => 'http',
  :host     => 'localhost',
  :port     => '5984',
  :prefix   => 'couchrest',
  :suffix   => nil,
  :join     => '_',
  :username => nil,
  :password => nil
}
{% endhighlight %}

If your couchrest model is used in a Rails project, the prefix option is automatically overriden using the project's name determined from Rails' `config/application.rb` file. If a name cannot be determined or you're not developing a Rails project, 'couchrest' is the default database prefix.

The prefix and suffix options are used to form a database name by combining them using the join option. The most basic option just uses the prefix as the database name, however if you define a suffix and models that define they're own database, more complex names can be used.

{% highlight ruby %}
# Define a simple person model with database and alter suffix
class Person < CouchRest::Model::Base
  use_database "sample"
  connection.update(:suffix => 'test')
end

# Get the database
Person.database # => http://localhost:5984/couchrest_sample_test
{% endhighlight %}


The `connection_config_file` is determined from the current working directory plus 'config/couchdb.yml'. This should be fine for most Rails applications. Any configuration options found in this file for the current environment will override those in the default config. Heres what a typical `couchdb.yml` might look like to access an external provider using the https protocol:

{% highlight yaml %}
development:
  protocol: 'https'
  host: sample.cloudant.com
  port: 443
  prefix: project
  suffix: text
  username: test
  password: user
{% endhighlight %}


