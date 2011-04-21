---
layout: default
title: Properties
id: model_properties
---

# CouchRest Model Properties

A property is the definition of an attribute, it describes what the attribute is called, how it should
be type casted and other options such as it's default value. These replace your typical 
`add_column` methods found in relational database migrations.

To define a property, simply call the `property` class method in your model definition:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  property :name,     String
  property :birthday, Date
  property :likes_catnip, TrueClass, :default => true
end
{% endhighlight %}

Properties are defined with a name, class, and any of the following options:

 * `:default` - a value to assign when instantiating
 * `:read_only` - when true, only creates a getter
 * `:alias` - create an alias to the getter and setter methods
 * `:protected` - do not allow mass-assignment
 * `:accessible` - makes this attribute available for mass-assignment, but removes all others
 * `:init_method` - default is 'new', the method called on the class to instantiate a new object

When defined, getters and setters similar to the following are added to your model:

{% highlight ruby %}
def name
  read_attribute('name')
end

def name=(value)
  write_attribute('name', value)
end
{% endhighlight %}

These can be overwriten in your code should you want to perform any special treatment of an attribute. The `#read_attribute` and `#write_attribute` methods perform any typecasting required.

Here are a few examples of properties in use:

{% highlight ruby %}
# Define a new Cat model
class Cat < CouchRest::Model::Base
  property :name,        String
  property :last_fed_at, Time
  property :awake,       TrueClass, :default => false
end

# Assign values to the properties on instantiation
@cat = Cat.new(:name => 'Felix', :last_fed_at => 10.minutes.ago)

# Access the values
@cat.name   #=> 'Felix'
@cat.last_fed_at   #=> Time.parse('2011-04-21T10:32:00Z')

@cat.save

# Reload the object
@cat = Cat.find(@cat.id)

# The properties are typecast on loading
@cat.last_fed_at < 20.minutes.ago   #=> true

# TrueClass properties will create a getter with question mark at the end:
@cat.awake?   #=> false
{% endhighlight %}

A special property macro is available called `timestamps!` that will create the `created_at` and `updated_at` accessors. As you'd expect, these are updated automatically when creating and saving a document and are set as Time objects.

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  property :name,        String
  timestamps!
end
{% endhighlight %}

In CouchRest, any class can be used as a property type as long as it has a to_json method. Problems however may arrise when trying to load the stored data back into your model, most classes cannot be restored from a Hash with JSON values. To get around this problem, the [`CastedModel` include](/model/casted_models.html) allows you to create complex embdedded documents that will be loaded correctly.

Properties do not need to be defined with a type or class, but in this case no type casting will be performed when loading the model, so you'll always get back whatever the `to_json` method converted your property into:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  property :name
  property :birthday
end

# Assign some values and save
@cat = Cat.new(:name => 'Felix', :birthday => 2.years.ago)
@cat.name        #=> 'Felix'
@cat.birthday.is_a?(Time)  #=> true
@cat.save

# Reload the model
@cat = Cat.find(@cat.id)
@cat.name        # 'Felix'

# The birthday will not be converted back into a Time object
@cat.birthday.is_a?(Time)  #=> false
{% endhighlight %}

Properties defined with the `:read_only` option will only have a getter method, and its value is set when the document
is read from the database. You can however update a read-only attribute using the `write_attribute` method:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  property :name, String
  property :lives, Integer, :default => 9, :read_only => true  

  def fall_off_balcony!
    write_attribute(:lives, lives - 1)
    save
  end
end

@cat = Cat.new(:name => "Felix")
@cat.fall_off_balcony!
@cat.lives    # Now 8!
{% endhighlight %}

Mass assigning attributes, as you'l already have realised, is also possible in a similar fashion to ActiveRecord:

{% highlight ruby %}
# Update matching attributes
@cat.attributes = { :name => "Felix", :birthday => Date.new(2011, 1, 1) }
@cat.save

# is the same as:
@cat.update_attributes(:name => "Felix", :birthday => Date.new(2011, 1, 1))
{% endhighlight %}

Attributes sent to `#attribtues=` or `#update_attributes` that do not have a property definition will not be updated. This provents useless data being passed to database, such as from an HTML form, and also because most projects will need to do some kind of typecasting to allow the ruby code to understand the data. However, if you would like truely
dynamic attributes (CouchDB is schema-less after all!), the `mass_assign_any_attribute` configuration option when set to true will store everything you put into the mass assignment methods.

## Property Arrays

One of the most attractive features of CouchDB is the ability to store arrays of data. CouchRest Model makes it very easy to define properties that will accept and array of items of a specific class, simply add `[` and `]` around the class:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  # Single text string
  property :name, String
  # Multiple text strings
  property :nicknames, [String]
end
{% endhighlight %}

By default, the array will be ready to use from the moment the object as been instantiated:

{% highlight ruby %}
@cat = Cat.new(:name => 'Fluffy')
@cat.nicknames << 'Buffy'
@cat.nicknames == ['Buffy']
{% endhighlight %}

When anything other than a String is set as the class of a property, the array will be converted
into special wrapper called a `CastedArray`. If the child objects respond to the `casted_by` and `casted_by_property` methods (such as those created with [`CastedModel`](/model/casted_models.html)) it will contain a reference to the parent object and property, not the casted array itself. More details are availble in the following section on [casted models](/model/casted_models.html)


