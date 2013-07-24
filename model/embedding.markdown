---
layout: default
title: Embedding Models
id: model_embedding
---

# Embedded or Nesting Models

The CouchRest Model `Embeddable` module allows you to take full advantage of CouchDB's ability to store complex
documents and retrieve them. Simply include the module in a Class and set any properties you'd like to use.
For example:

{% highlight ruby %}
class CatToy
  include CouchRest::Model::Embeddable

  property :name, String
  property :purchased, Date
end

class Cat < CouchRest::Model::Base
  property :name, String
  property :toys, CatToy, :array => true
end

@cat = Cat.new(:name => 'Felix', :toys => [{:name => 'mouse', :purchased => 1.month.ago}])
@cat.toys.first.class == CatToy
@cat.toys.first.name == 'mouse'
{% endhighlight %}

Any hashes sent to the property will be automatically converted:

{% highlight ruby %}
@cat.toys << {:name => 'catnip ball'}
@cat.toys.last.is_a?(CatToy) # True!
{% endhighlight %}

Obviously, to avoid Ruby raising a missing constant error, classes *must* be defined before the parent uses them.
To avoid this, or if you have a simple set of data to model, CouchRest Model supports creating anonymous classes:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  property :name, String

  # define a property with a nested anonymous array of embeddable models
  property :toys, :array => true do
    property :name, String
    property :rating, Integer
  end
end

# Inialize a new Cat with nested toys
@cat = Cat.new(
  :name => 'Felix', 
  :toys => [
    {:name => 'mouse', :rating => 3},
    {:name => 'catnip ball', :rating => 5}
  ])

# Access the values
@cat.toys.last.rating   #=> 5
@cat.toys.last.name     #=> 'catnip ball'

# Update them in place
@cat.toys.first.rating = 2
{% endhighlight %}

The example above defines an array of the annonymous class on the `toys` attribute, but it is often useful 
to group data into a simple hash:

{% highlight ruby %}
class Journey < CouchRest::Model::Base
  property :title, String
  property :start do
    property :name, String
    property :loc,  Point
    property :time, Time
  end
end

@journey = new Journey(
  :title => "Sample Journey",
  :start => {
    :name => "Calle San Bernardo 13",
    :loc  => [40.4221, -3.7081],
    :time => "2013-07-24T16:44:23Z"
  }
)

@journey.start.name   # => "Calle San Bernardo 13"
{% endhighlight %}

If you prefer a more traditional usage of blocks and avoid the magical `instance_eval`, a block can be provided with a parameter. This might be useful if you need to access a variable outside of the block as the scope will not have been altered.

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  # use a traditional block to set add properties in nested array
  property :toys, :array => true do |toy|
    toy.property :name, String
    toy.property :rating, Integer
  end
end
{% endhighlight %}
