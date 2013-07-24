---
layout: default
title: Proxying
id: model_proxying
---

# Proxying

CouchDB makes it really easy to create databases on the fly, so easy in fact that it is perfectly
feasible to have one database per user or per company or per whatever makes sense to split into
its own individual database. CouchRest Model now makes it really easy to support this scenario
using the proxy methods. Here's a quick example:

{% highlight ruby %}
# Define a master company class, its children should be in their own DB
class Company < CouchRest::Model::Base
  use_database COUCHDB_DATABASE
  property :name
  property :slug

  proxy_database_method :slug
  proxy_for :invoices
end

# Invoices belong to a company
class Invoice < CouchRest::Model::Base
  property :date
  property :total

  proxied_by :company

  design do
    view :by_date
  end
end
{% endhighlight %}

By setting up our models like this, the invoices should be accessed via a company object:

{% highlight ruby %}
company = Company.first
company.invoices.new            # build a new invoice
company.invoices.by_date.first  # find company's first invoice by date
{% endhighlight %}

Internally, all requests for invoices are passed through a model proxy. Aside from the 
basic methods and views, it also ensures that some of the more complex queries are supported
such as validating for uniqueness and associations.

The `proxy_database_method` defines the name of the method that should be called to return 
the desired name of the database for the instance. The project's database prefix and suffix
will be added if they are specified in the configuration. If you'd like to provide your own
method for setting the database, simply override the `proxy_database` method so that a 
CouchRest Database of your choosing is returned.

For obvious reasons, it is not a good idea to use a variable that can change for the database
name, so the object's id might be a reasonable choice.

Deleting an proxy model will not automatically delete the associated database, you'd have to
do this manually in a callback.


