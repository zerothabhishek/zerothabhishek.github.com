---
layout: post
permalink: /big-memcache-1.html
date: 2017-03-02
---

# Big memcache or how I learned to use database as a cache

I raised this query on our Ruby community Slack recently -

![slack-screenshot](/content/images/2017/01/Screen-Shot-2017-01-04-at-6.34.26-AM.png)

There were many useful answers, and much of the group was inclined towards using Redis. And nearly everyone ignored the 7MB part.

The data I want to cache is a somewhat big chunk of JSON - in the order of a few MBs. I'm getting this data from an external source through an API. Since the API serves paginated data, I have to hit the end-point several times to get it all. The whole process takes up to fifteen minutes sometimes - hence the need to store it in some kind of cache. After that, some reports are generated from the fetched data.

Fortunately, the size of the data is predictable. What looks 7MB today may increase up to 15MB or come down to 1.5MB in foreseeable future. I don't need to plan for unpredictably growing data.

### Giant in-memory data structures

And the idea of storing big data structures in memory to perform arbitrary processing is interesting. The last time I was in this situation, I had to work with a large (but fixed) product catalogue in a Mysql database. I thought I would just cache the entire data-set in Redis, and perform my *complex* operations on this giant list of hashes *in memory* instead of writing *complicated* SQL queries [1].

That didn't work. I ended up learning some SQL.

The point I didn't see was - cache stores work best when the cached data is small in size. Both Memcache and Redis. Caching is a win when the _process_ of arriving at the value is slow. They are not meant to store large units of data.

### The limit on Heroku 

On Heroku, there is a hard limit of 1MB per key-value size. We can't have a value greater than 1MB. This limit is actually enforced by Memcachier (the Memcache plugin on Heroku) and not by the Memcache software itself. The software *too* does have a limit, 1MB, but it can be raised to up to 128MB <sup>2</sup>.

Here's a bit from the [Heroku documentation](https://devcenter.heroku.com/articles/memcachier#key-value-size-limit-1mb) :
![memcachier-screenshot](/content/images/2017/01/Screen-Shot-2017-01-04-at-6.58.27-AM-1.png)

Redis doesn't have any such limits <sup>3</sup>.

### Blinded by abstraction

The caching API in Rails is so smooth, it almost feels like all our data is available in the local memory. There is nothing to connect, and there is no networking business. Here's an example - 

```
    data = Rails.cache.read('all-my-data')
    results = my_superfast_algorithm(data)
```

It will probably not work as fast as I thought. The `data` comes after traveling through a network stack, probably over the wire from a whole other computer. Its more like downloading a 7MB file than reading a local variable.

### Splitting it up

If like me, you too are hung up on the earlier idea - keeping it in Memcache, the next obvious thing would be to split the data in to smaller pieces. If Memcache doesn't allow us to have a single key value sized 7MB, we could create a ten of them sized 700KB, or a hundred of them sized 70 KB. 

In my case, the data was already structured as a collection. All you had to do was determine appropriate keys for each of the 70 KB piece. You could also store a list of these keys as a separate key-value pair - like an key-index. For the cache reads, you figure out all the necessary keys from the key-index, and then pull all the needed 70kb pieces.

That is easier said than done. Plus my application needed the entire 7MB chunk to produce reports. If we fetch by pieces, we'd have to fetch all the pieces. And the total time taken for all the pieces should be at par with the total time to fetch the original 7MB. There's little chance of that.

And then the whole wiring needed to accomplish this is too much. Its almost stretching the cache software beyond its limits. Cache stores are designed to work efficiently with key-value pairs, not rows and indexes.

There is another data-store built to handle data as rows and access them efficiently - a relational database.

### Store in a simple file.

Our data doesn't have a schema - at a unit level it is still unstructured. Its actually a list of dictionary like objects with variable size.  A relational database is may not be a strict fit here.

We could also store our 7MB JSON in a file. The read times are generally acceptable, and implementation is simple. There's only one problem though - it can't work on Heroku. The Heroku file-system is ephemeral. This means there is no guarantee that the file we create once will be in the same place the next time.

I could have tried storing the file in S3, but that looked too much work. And the performance would still be questionable given the file has to be fetched externally.

#### And Postgresql?

In my case, this time, I had Postgresql at hand. And I had seen years of hipster advocacy on how awesome it is for storing JSON data. So, well, it was time to give it a try. That too ended up being an interesting story - continued in the [next post](http://www.zerothabhishek.com/big-memcache-or-how-i-learned-to-use-database-as-a-cache-2/).

#### One more thing about Memcache/Redis read times

While writing this post, I tried to compare read-times for Memcache/Redis. The results were somewhat surprising.

For Memcache, the read performance is quite decent with 7MB or 16MB data - its almost instant. When we go upwards of a 100MB, there is a slight lag.

For Redis however, I struggled to get similar performance. I started seeing lags and timeouts at values sized 256kb <sup>4</sup>. This was surprising considering [several people](http://stackoverflow.com/questions/10558465/memcached-vs-redis) have claimed Redis performs better than Memcache.

[Continue to the next part](http://www.zerothabhishek.com/big-memcache-or-how-i-learned-to-use-database-as-a-cache-2/)

----

1. The operations weren't that complex, Redis is not same as in-memory, and the SQL was only somewhat complicated.

2. http://stackoverflow.com/a/11730559

3. It is still limited by system constraints such as RAM size, architecture etc. 

4. Both the measurements were taken with servers and clients residing on separate servers. I haven't yet researched enough on the issue, so don't take my word on it.
