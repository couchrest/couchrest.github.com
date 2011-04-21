---
layout: default
title: Validations
id: model_validations
---

# Validations

CouchRest Model automatically includes the new ActiveModel validations, so they should work just as the traditional Rails validations. For more details, please see the ActiveModel::Validations documentation.

CouchRest Model adds the possibility to check the uniqueness of attributes using the `validates_uniqueness_of` class method, for example:

{% highlight ruby %}
class Person < CouchRest::Model::Base
  property :title, String
 
  validates_uniqueness_of :title
end
{% endhighlight %}

The uniqueness validation creates a new view for the attribute or uses one that already exists. You can
specify a different view using the `:view` option, useful for when the `unique_id` is specified and
you'd like to avoid the typical RestClient Conflict error:

{% highlight ruby %}
unique_id :code
validates_uniqueness_of :code, :view => 'all'
{% endhighlight %}

Given that the uniqueness check performs a request to the database, it is also possible to include a `:proxy` parameter. This allows you to call a method on the document and provide an alternate proxy object.

Examples:

{% highlight ruby %}
# Same as not including proxy:
validates_uniqueness_of :title, :proxy => 'class'

# Person#company.people provides a proxy object for people
validates_uniqueness_of :title, :proxy => 'company.people'
{% endhighlight %}


A really interesting use of `:proxy` and `:view` together could be where you'd like to ensure the ID is unique between several types of document. For example:

{% highlight ruby %}
class Product < CouchRest::Model::Base
  property :code

  validates_uniqueness_of :code, :view => 'by_product_code'

  view_by :product_code, :map => "
    function(doc) {
      if (doc['couchrest-type'] == 'Product' || doc['couchrest-type'] == 'Project') {
        emit(doc['code']);
      }
    }
  "
end

class Project < CouchRest::Model::Base
  property :code

  validates_uniqueness_of :code, :view => 'by_product_code', :proxy => 'Product'
end
{% endhighlight %}

Pretty cool!


