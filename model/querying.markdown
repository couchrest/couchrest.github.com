---
layout: default
title: Simple Queries
no_comments: true
id: model_queries
---

# Simple Queries

CouchRest Model supports a few basic queries for retrieving your data from the database. For more sophisticated ways of accessing your data, see the [views section](/model/view_objects.html).

 * [`Model.all`](#all)
 * [`Model.count`](#count)
 * [`Model.first`](#first)
 * [`Model.last`](#first)
 * [`Model.get`](#get)
 * [`Model.get!`](#get)
 * [`Model.find`](#get)

<a id="all"> </a>
## `Model.all`

Load all the documents of the matching type using the model's "all" view.

{% highlight ruby %}
# Find a list of all the people
Person.all.each do |person|
  puts "Person: #{person.first_name}"
end

# Limit the number of people returned
Person.all(:limit => 10).each do |person|
  puts "Person: #{person.first_name}"
end
{% endhighlight %}

<a id="count"> </a>
## `Model.count`

Perform a request for the 'all' view but set limit to 0 and only use
the `total_rows` field returned by CouchDB.

{% highlight ruby %}
# Find the total number of people
People.count  # => 23
{% endhighlight %}

<a id="first"> </a>
## `Model.first` and `Model.last`

Using the 'all' query to find the first or last document. CouchDB orders by the `id`
field which is generated randomly upon saving. You'll probably want to use a view
with an index on a specific value to get something more meaningful from these
requests.

{% highlight ruby %}
# Fetch the first person available
Person.first  # <#Person ....>
{% endhighlight %}


<a id="get"> </a>
## `Model.get`, `Model.get!`, and `Model.find`

Fetch a single document from the database using document id provided. This request does not use a view to fetch the results, so if the id you provide is for an object of a different type, an instance of that will be returned instead!

{% highlight ruby %}
# Load a single Person from the database
person = Person.get("12345")

# If the id belongs to an object of a different type:
person = Person.get('felix')
person.class # => Cat !!
{% endhighlight %}

