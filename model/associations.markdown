---
layout: default
title: Associations
id: model_associations
---

# Associations

Two types at the moment:

{% highlight ruby %}
belongs_to :person
collection_of :tags
{% endhighlight %}


This is a somewhat controvesial feature of CouchRest Model that some document database purists may cringe at. CouchDB does not yet povide many features to support relationships between documents but the fact of that matter is that its a very useful paradigm for modelling data systems.

In the near future we hope to add support for a `has_many` relationship that takes of the _Linked Documents_ feature that arrived in CouchDB 0.11.

## Belongs To

Creates a property in the document with `_id` added to the end of the name of the foreign model with getter and setter methods to access the model. 

Example:


{% highlight ruby %}
class Cat < CouchRest::Model::Base
  belongs_to :mother
  property :name
end

kitty = Cat.new(:name => "Felix")
kitty.mother = Mother.find_by_name('Sophie')
{% endhighlight %}

Providing a object to the setter, `mother` in the example will automagically update the `mother_id` attribute. Retrieving the data later is just as expected:

{% highlight ruby %}
kitty = Cat.find_by_name "Felix"
kitty.mother.name == 'Sophie'
{% endhighlight %}

Belongs_to accepts a few options to add a bit more felxibility:

* `:class_name` - the camel case string name of the class used to load the model.
* `:foreign_key` - the name of the property to use instead of the attribute name with `_id` on the end.
* `:proxy` - a string that when evaluated provides a proxy model that responds to `#get`.

The last option, `:proxy` is a feature currently in testing that allows objects to be loaded from a proxy class, such as `ClassProxy`. For example:

{% highlight ruby %}
class Invoice < CouchRest::Model::Base
  attr_accessor :company
  belongs_to :project, :proxy => 'self.company.projects'
end
{% endhighlight %}

A project instance in this scenario would need to be loaded by calling `#get(project_id)` on `self.company.projects` in the scope of an instance of the Invoice. We hope to document and work on this powerful feature in the near future.

## Collection Of

A collection_of relationship is much like belongs_to except that rather than just one foreign key, an array of foreign keys can be stored. This is one of the great features of a document database. This relationship uses a proxy object to automatically update two arrays; one containing the objects being used, and a second with the foreign keys used to the find them.

The best example of this in use is with Labels:

{% highlight ruby %}
class Invoice < CouchRest::Model::Base
  collection_of :labels
end

invoice = Invoice.new
invoice.labels << Label.get('xyz')
invoice.labels << Label.get('abc')

invoice.labels.map{|l| l.name} # produces ['xyz', 'abc']
{% endhighlight %}

See the belongs_to relationship for the options that can be used. Note that this isn't especially efficient, a `get` is performed for each model in the array. As with a has_many relationship, we hope to be able to take advantage of the Linked Documents feature to avoid multiple requests.


