---
layout: default
title: Designs
no_comments: true_
id: model_desgins
---

# Defining Designs

To create views there is a special method *design*, that allow to create view
to find models according to attributes.
Inside a Model definition, add these lines

    design do
      view :by_name
    end

to create a view that get every model ordered by name

