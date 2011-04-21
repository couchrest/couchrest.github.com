---
layout: default
title: Configuring
id: model_configuring
---

# Configuring

CouchRest Model supports a few configuration options. These can be set either for the whole Model code
base or for a specific model of your chosing. To configure globally, provide something similar to the 
following in your projects initializers or environments:

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

 * `mass_assign_any_attribute` - false by default, when true any attribute may be updated via the update_attributes or attributes= methods.
 * `model_type_key` - 'model' by default, is the name of property that holds the class name of each CouchRest Model.
 * `auto_update_design_doc` - true by default, every time a view is requested and this option is true, a quick check will be performed to ensure the model's design document is up to date. When disabled, you're design documents will never be updated automatically and you'll need to perform updates manually. Results are cached on a per-database and per-design basis to help lower the number of requests. See the View section for more details.

