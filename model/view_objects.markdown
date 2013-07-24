---
layout: default
title: Views and View Objects
id: model_view_objects
---

# Views

CouchDB views can be quite difficult to get grips with at first as they are quite different from what you'd expect with SQL queries in a normal Relational Database. Checkout some of the [CouchDB documentation on views](http://wiki.apache.org/couchdb/Introduction_to_CouchDB_views) to get to grips with the basics. The key is to remember that CouchDB will only generate indexes from which you can extract consecutive rows of data, filtering other than between two points in a data set is not possible.

CouchRest Model has great support for views, and since version 1.1.0 we added support for a View objects that make accessing your data even easier.

### Definitions and Design Block

Since version 1.1.0 it has been possible to create views that return chain-able objects, similar to those you'd find in the [sequel](http://sequel.rubyforge.org/) or Rails 3's Arel.

View definitions are provided in a model's `design` block. This is a special region of the class where view object accessors are defined and will be stored in the model's design document. In the future, other types of design methods may also be supported.

A view is defined by providing a view name to the `view` method. If no map function is provided, an attempt will be made to determine one for you based on the name. The following example shows this:

{% highlight ruby %}
# Define a new Post class with a design block
class Post < CouchRest::Model::Base
  property :name
  design do
    view :by_name
  end
end

# The model should have a by_name method
Post.respond_to?(:by_name) # => true

# Request a set of posts ordered by name
Post.by_name.count # => 23
Post.by_name.all   # => posts ordered by name...
{% endhighlight %}

Following the example, the name of the view is provided as `by_name`. Couchrest model automatically determines that we want to create a view whose index contains the document's name field. If we wanted to create views based on more than one key, they can be separated with an `and`:

{% highlight ruby %}
# Define a new Post class with a design block and joint view
class Post < CouchRest::Model::Base
  property :name, String
  property :date, Date
  design do
    view :by_date_and_name
  end
end
{% endhighlight %}

The following code section shows an example model covering several of the options supported by the view definitions.

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
          if (doc['type'] == 'Post' && doc.tags) {
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

Acessing data is the fun part:

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

The view objects have built in support for pagination based on the [kaminari](https://github.com/amatsuda/kaminari) gem. Methods are provided to match those required by the library to perform pagination.

For your view to support paginating the results, it must use a reduce function that provides a total count of the documents in the result set. By default, auto-generated views include a reduce function that supports this.

Use pagination as follows:

{% highlight ruby %}
# Prepare a query:
@posts = Post.by_title.page(params[:page]).per(10)

# In your view, with the kaminari gem loaded:
paginate @posts
{% endhighlight %}


### Multiple Design Documents

Since version 1.2.0, it is now possible to define multiple designs for a single model. Views can often take a long time to generate, especially if you have lots of documents, your application will practically lock-up until all the indexes have updated. It makes sense therefore to try and avoid updating designs where a delay in execution may be significant and effect responses to users.

Creating multiple designs helps differentiate between essential views, like finding all documents by date or joins which are critical to the functionality of the app, and non-essential views such as for statistics or testing purposes. Changes to the critical design can be avoided, and the stats can be updated whenever required. Here's how:


{% highlight ruby %}
class Post < CouchRest::Model::Base
  property :title
  property :body
  property :posted_at, DateTime
  property :tags, [String]

  design do
    view :by_title
    view :by_posted_at_and_title
  end

  design :tags do
    view :tag_list,
      :map =>
        "function(doc) {
          if (doc['#{model_type_key}'] == 'Post' && doc.tags) {
            doc.tags.forEach(function(tag){
              emit(doc.tag, 1);
            });
          }
        }",
      :reduce => "function(k, v, r) { return sum(v); }"
    end
  end
end
{% endhighlight %}

The second design block includes the `:tags` argument so that it will be created in its own document in CouchDB called `_design/Post_tags`. Just as with a normal design block, each view will have its own method created in the model without a prefix, ready to be called.



### Inheriting Designs

Another new feature in 1.2.0. When a Couchrest Model is inherited, it's design blocks will be automatically copied to the new child model. Each model stores an array of all the design doc definitions that have come before it so they are ready to be provided to a new model and re-interpreted. The only thing to be careful about in this scenario is ensuring that your own view map definitions do not fix the document type (generally a good idea anyway).

Heres an example of defining a simplified example of when this might be useful:


{% highlight ruby %}
class User < CouchRest::Model::Base
  property :name,     String
  property :email,    String
  property :password, String

  timestamps!

  design do
    view :by_email
    view :by_name,
      :map => "
        function(d) {
          if (d['type'] == '#{model}' && d['name']) { emit(d.name, d['email']); }
        }
      "
  end
end

class Admin < Mammal
  property :role
  design do
    view :by_role
  end
end

# Some basic queries
Admin.by_role.key('manager')  # find all admins with manager role
Admin.by_email.key('foo@bar.com').first # Find admin by email address
{% endhighlight %}

Of course, CouchDB views are not like SQL queries, so we cannot get a list of all users and admins without creating our own map method:

{% highlight ruby %}
design do
  view :by_email, :map => "
    function(d) {
      t = d['type'];
      if ((t == 'User' || t == 'Admin') && d['email']) {
        emit(d.email, 1);
      }
    }"
{% endhighlight %}

Given that this is such a new feature, expect changes and refinements to be made to cope with different environments.


### View Migrations

Deciding when to update the model's design doc is a difficult issue. In production you don't want to be constantly checking for updates and in development maximum flexibility is important. CouchRest Model solves this issue by providing the `auto_update_design_doc` configuration option, set true by default.

When a view is requested a quick GET for the design document will be sent to ensure it is up to date with the latest version defined in the model. Results are cached in the current thread for the complete design document's URL, including the database, to try and limit requests. This should be fine for most projects, but dealing with multiple sub-databases may require a different strategy.

Setting the option to false will require a manual update of each model's design doc whenever you know a change has happened. This will be useful in cases when you do not want CouchRest Model to interfere with the views already store in the CouchRest database, or you'd like to deploy your own update strategy. Here's an example of a module that will update all submodules:

{% highlight ruby %}
module CouchRestMigration
  def self.update_all_design_docs
    CouchRest::Model::Base.subclasses.each{|klass| klass.design_doc.sync! if klass.respond_to?(:design_doc)}
  end
end

# Running this from your applications initializers would be a good idea,
# for example in Rail's application.rb or environments/production.rb:
config.after_initialize do
  CouchRestMigration.update_all_design_docs
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
    CouchRest::Model::Base.subclasses.each{|klass| klass.design_doc.sync!(db) if klass.respond_to?(:save_design_doc!}
  end
end

# Command to run after a capistrano migration:
$ rails runner "CouchRestMigratin.update_all_design_docs"
{% endhighlight %}


### What happened to the old view methods?

They been deleted. :)

In older bits of code, you might find references to `view_by` definitions and calls to `has_view?` and just `view`. These methods have now all been removed since version 1.2 along with all the missing method helpers that went with them. This is due to new focus on design documents and to simplify usage.


