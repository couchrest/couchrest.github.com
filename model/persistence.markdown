---
layout: default
title: Database Persistence
no_comments: true
id: model_persistence
---

# Persistence

As you'd expect, CouchRest Model supports all the standard [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations you're used to in other object mappers. The key methods are:

 * [`Model.create`](#create)
 * [`Model.create!`](#create)
 * [`Model#save`](#save)
 * [`Model#save!`](#save)
 * [`Model#update_attributes`](#update_attributes)
 * [`Model#destroy`](#destroy)
 * [`Model#destroyed?`](#destroy)
 * [`Model#reload`](#reload)

<a id="create"> </a>
## `Model.create` and `Model.create!`

Insert a new document into the database with the provided attributes. Validations will check to see if the 
model is valid and an id will be assigned if saving has been successful. The `Model#pesisted?` call can be used
to check if the creation was successful. Using the bang method variation and error will be raised if the 
validation is not successful.

{% highlight ruby %}
# Create a new person object
person = Person.create(:first_name => "Homer", :last_name => "Simpson")

# Create a new person setting attributes using a block
Person.create(:first_name => "Homer") do |doc|
  doc.last_name = "Simpson"
end
{% endhighlight %}

<a id="save"> </a>
## `Model#save` and `Model#save!`

Save the current model to the database. Validations will be performed and errors will be provided in
ActiveModel's standard errors Hash (`Model#errors`). If the bang method is used, validation failures will
raise an error.

{% highlight ruby %}
# Insert a new person
person = Person.new(:first_name => "Homer", :last_name => "Simpson")
person.save

# Save without running validation
person.save(:validate => false)
{% endhighlight %}

<a id="update_attributes"> </a>
## `Model#update_attributes`

When called with a hash, the model's attributes will be updated with the matching data and saved. Returns true if the model has been saved correctly, false otherwise.

{% highlight ruby %}
# Find the person we want to update
person = Person.get(params[:id])

# Update their attributes
if person.update_attributes(params[:person])
  puts "Person updated successfully!"
end
{% endhighlight %}

<a id="destroy"> </a>
## `Model#destroy`

Remove the model from the database and freeze the attributes hash so no further modifications can be made. After removal, the old document id and revision will still be available should they be required in any callbacks.

If unsuccessful, an error is likely to have occurred in the request and an exception will be raised.

{% highlight ruby %}
# Find the person we want to destroy and remove them
person = Person.get(params[:id])
person.destroy
{% endhighlight %}


<a id="reload"> </a>
## `Model#reload`

Reload the model from the database replacing any changes.

{% highlight ruby %}
# Find the person, update an attribute and reload
person = Person.create(:first_name => "Homer")
person.first_name = "Bart"
person.reload
person.first_name  # => "Homer"
{% endhighlight %}


