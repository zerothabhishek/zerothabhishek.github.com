---
layout: post
permalink: /beware-rails-pseudo-ids.html
date: 2016-03-11
---

# Beware Rails pseudo ids

ActiveRecord provides auto-increment `id` for models that act as the default primary key. It is so useful and convenient that it almost makes us forget about the idea of primary keys. 

Until we write this somewhere in our project:

```
sample_post = Post.find(1)
```

We have created a certain post we want to refer to as a sample. A post can have a title, body and few other fields, but nothing that can identify it uniquely. The `Post` model doesn't have a real primary key.

The above code will not work across different environments. The id assigned to our sample may be different across development and production. And it will surely be different in testing.

_If you're thinking about engineering solutions to fix it across the different environments - stop. Rails `ids` are not intended to be used that way. Its time to start thinking about real primary keys._

For a use-case like this, not only do we need a unique identifier to our post, but also one that is not likely to change very often. We could have made the post's `title` as the primary key. But then we'd have to change the code every-time the sample post's title changes.

One possible solution is to use something like a label. We can add a label field, add a uniqueness validation on it, and then -

```
sample_post = Post.find_by_label('sample')
```

The point here is, ids provided by ActiveRecord are not real identifiers. They are **pseudo ids**. A sure sign of incorrect usage of the pseudo ids is when it is referred directly as a literal.

__Real unique identifiers come from the data.__ And in case there is no real unique identifier, we should add one separately instead of relying on the id.

