---
layout: default
title: Views and View Objects
id: model_view_objects
---

# Views and View Objects

CouchDB views can be quite difficult to get grips with at first as they are quite different from what you'd expect with SQL queries in a normal Relational Database. Checkout some of the [CouchDB documentation on views](http://wiki.apache.org/couchdb/Introduction_to_CouchDB_views) to get to grips with the basics. The key is to remember that CouchDB will only generate indexes from which you can extract consecutive rows of data, filtering other than between two points in a data set is not possible.

CouchRest Model has great support for views, and since version 1.1.0 we added support for a View objects that make accessing your data even easier. 

## The Old Way

Here's an example of adding a view to our Cat class:

{% highlight ruby %}
class Cat < CouchRest::Model::Base
  property :name, String
  property :toys, [CatToy]

  view_by :name
end
{% endhighlight %}

The `view_by` method will create a view in the Cat's design document called "by_name". This will allow searches to be made for the Cat's name attribute. Calling `Cat.by_name` will send a query of to the database and return an array of all the Cat objects available. Internally, a map function is generated automatically and stored in CouchDB's design document for the current model, it'll look something like the following:

{% highlight javascript %}
function(doc) {
  if (doc['couchrest-type'] == 'Cat' && doc['name']) {
    emit(doc.name, null);
  }
}
{% endhighlight %}

By default, a special view called `all` is created and added to all couchrest models that allows you access to all the documents in the database that match the model. By default, these will be ordered by each documents id field.

It is also possible to create views of multiple keys, for example:

{% highlight ruby %}
view_by :birthday, :name
{% endhighlight %}

This will create an view of all the cats' birthdays and their names called `by_birthday_and_name`.

Sometimes the automatically generate map function might not be sufficient for more complicated queries. To customize, add the :map and :reduce functions when creating the view:

{% highlight ruby %}
view_by :tags,
  :map =>
    "function(doc) {
      if (doc['model'] == 'Post' && doc.tags) {
        doc.tags.forEach(function(tag){
          emit(doc.tag, 1);
        });
      }
    }",
  :reduce =>
    "function(keys, values, rereduce) {
      return sum(values);
    }"
{% endhighlight %}

Calling a view will return document objects by default, to get access to the raw CouchDB result add the `:raw => true` option to get a hash instead. Custom views can also be queried with `:reduce => true` to return reduce results. The default is to query with `:reduce => false`.
 
Views are generated (on a per-model basis) lazily on first-access. This means that if you are deploying changes to a view, the views for
that model won't be available until generation is complete. This can take some time with large databases. Strategies are in the works.

### View Objects

Since CouchRest Model 1.1.0 it is now possible to create views that return objects chainable objects, similar to those you'd find in the Sequel Ruby library or Rails 3's Arel. Heres an example of creating a few views:

{% highlight ruby %}
class Post < CouchRest::Model::Base
  property :title
  property :body
  property :posted_at, DateTime
  property :tags, [String]

  design do
    view :by_title
    view :by_posted_at_and_title
    view :tag_list,
      :map =>
        "function(doc) {
          if (doc['model'] == 'Post' && doc.tags) {
            doc.tags.forEach(function(tag){
              emit(doc.tag, 1);
            });
          }
        }",
      :reduce =>
        "function(keys, values, rereduce) {
          return sum(values);
        }"
  end
end
{% endhighlight %}

You'll see that this new syntax requires all views to be defined inside a design block. Unlike the old version, the keys to be used in a query are determined from the name of the view, not the other way round. Acessing data is the fun part:

{% highlight ruby %}
# Prepare a query:
view = Post.by_posted_at_and_title.skip(5).limit(10)

# Fetch the results:
view.each do |post|
  puts "Title: #{post.title}"
end

# Grab the CouchDB result information with the same object:
view.total_rows   => 10
view.offset       => 5

# Re-use and add new filters
filter = view.startkey([1.month.ago]).endkey([Date.current, {}])

# Fetch row results without the documents:
filter.rows.each do |row|
  puts "Row value: #{row.value} Doc ID: #{row.id}"
end

# Lazily load documents (take last row from previous example):
row.doc      => Fetch last Post document from database

# Using reduced queries is also easy:
tag_usage = Post.tag_list.reduce.group_level(1)
tag_usage.rows.each do |row|
  puts "Tag: #{row.key}  Uses: #{row.value}"
end
{% endhighlight %}

#### Pagination

The view objects have built in support for pagination based on the [kaminari](https://github.com/amatsuda/kaminari) gem. Methods are provided to match those required by the library to peform pagination.

For your view to support paginating the results, it must use a reduce function that provides a total count of the documents in the result set. By default, auto-generated views include a reduce function that supports this.

Use pagination as follows:

{% highlight ruby %}
# Prepare a query:
@posts = Post.by_title.page(params[:page]).per(10)

# In your view, with the kaminari gem loaded:
paginate @posts
{% endhighlight %}

### Design Documents and Views

Views must be defined in a Design Document for CouchDB to be able to perform searches. Each model therefore must have its own Design Document. Deciding when to update the model's design doc is a difficult issue, as in production you don't want to be constantly checking for updates and in development maximum flexability is important. CouchRest Model solves this issue by providing the `auto_update_design_doc` configuration option and is true by default.

Each time a view or other design method is requested a quick GET for the design will be sent to ensure it is up to date with the latest changes. Results are cached in the current thread for the complete design document's URL, including the database, to try and limit requests. This should be fine for most projects, but dealing with multiple sub-databases may require a different strategy.

Setting the option to false will require a manual update of each model's design doc whenever you know a change has happened. This will be useful in cases when you do not want CouchRest Model to interfere with the views already store in the CouchRest database, or you'd like to deploy your own update strategy. Here's an example of a module that will update all submodules:

{% highlight ruby %}
module CouchRestMigration
  def self.update_design_docs
    CouchRest::Model::Base.subclasses.each{|klass| klass.save_design_doc! if klass.respond_to?(:save_design_doc!)}
  end
end

# Running this from your applications initializers would be a good idea,
# for example in Rail's application.rb or environments/production.rb:
config.after_initialize do
  CouchRestMigration.update_design_docs
end
{% endhighlight %}

If you're dealing with multiple databases, using proxied models, or databases that are created on-the-fly, a more sophisticated approach might be required:

{% highlight ruby %}
module CouchRestMigration
  def self.update_all_design_docs
    update_design_docs(COUCHREST_DATABASE)
    Company.all.each do |company|
      update_design_docs(company.proxy_database)
    end
  end
  def self.update_design_docs(db)
    CouchRest::Model::Base.subclasses.each{|klass| klass.save_design_doc!(db) if klass.respond_to?(:save_design_doc!}
  end
end

# Command to run after a capistrano migration:
$ rails runner "CouchRestMigratin.update_all_design_docs"
{% endhighlight %}



