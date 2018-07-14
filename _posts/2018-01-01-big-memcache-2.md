---
layout: post
permalink: /big-memcache-2.html
---

# Big memcache or how I learned to use database as a cache (part-2)

This a continuation the [previous post](http://www.zerothabhishek.com/big-memcache-or-how-i-learned-to-use-database-as-a-cache) where I explored how suitable Memcache and Redis are when the cache values are too big. Here I narrate my efforts on using the Postgresql database for the purpose.

Since my data was already a collection, each element of it looked like a row in a table. Every such row has a key-value dictionary (and somewhat big), so we need some kind of serialized storage for it. I start with `text` data-type because that's like a default (for Rails programmers). Here's the schema -
 
    data_cache:
      label: varchar
      raw: text

`raw` is where we store the whole row-data. ActiveRecord has a `serialize` feature that does the serialisation for us, so the model looks like this -

    class DataCache
      serialize :raw
    end

The `label` field is for storing some kind of identifier - similar to a cache key. This may be used as a filter in queries, for example, to fetch the rows imported today -

    DataCache.where(label: "Contacts-2017-01-15")

After inserting my sample data, which is 15MB <sup> 1 </sup> in this case, the read times don't look good:

    l = "Contacts-2017-01-15"  ## 15 MB
    DataCache.where(label:l).select(:raw).map(&:raw)  ## ~ 40 seconds
 

#### Serialize with care

It is not right to comment about Postgresql performance at this point. We are running a rather inefficient query, and then passing the results through a Ruby ORM. The problem could be anywhere, and Ruby software are not know to be very fast.

Actually when I try the same query directly from on Postgres, it is much faster.

    explain analyse select raw from datacache where label='Contacts-2017-01-15'
    Planning time: 0.063 ms
    Execution time: 1.188 ms

After some tinkering I noticed it was `serialize` that took most of the time. The time taken is far less (around 0.3 seconds) if we remove the `serialize` macro. The result is just plain text and not a Hash, so this is not a fair comparison. But it does indicate that `serialize` could be the problem.

At this point I decided<sup>2</sup> to try the Postgresql `jsonb` datatype.

#### The jsonb type in Postgres

As the name sounds, the `jsonb` datatype stores JSON directly in a column. *b* is for binary - the storage format is binary not text, which gets us some additional performance. Postgresql also allows us to have indexes on keys _within_ the JSON, which is another powerful feature.

It was fairly easy to use too - just change the column data-type to `jsonb` -

    data_cache:
      label: varchar
      raw: jsonb

and add some code to deserialize -

    class DataCache
      def rawj
        JSON.parse(self.raw)
      end
    end

And after seeding the data again:

    l = "Contacts-2017-01-15"  ## 15 MB
    DataCache.where(label:l)
             .select(:raw)
             .map(&:rawj)  ##  1.14 seconds

That's much better. Not great, but much better.

#### The Oj gem also helps

Oj claims to be the fastest JSON handling library in Ruby. If we try it instead of the default `Activesupport:JSON`, and the results improve futher.

    class DataCache
      def rawj
         Oj.load(self.raw)
      end
    end

    l = "Contacts-2017-01-15"  ## 15 MB
    DataCache.where(label:l).select(:raw).map(&:rawj)  ##  1.02 seconds

Not a massive improvement. But not bad.

#### Adding some redundancy

Another thing I tried was adding a few redundant columns in the same data-cache table. We had some queries needed only a smaller subset of the `raw` data - these new columns would store exactly that. Since the size of this subset is smaller, the queries are faster. We populate these columns during the import process <sup>3</sup> .

This helped a lot. The only down-side in this approach would be keeping the new columns consistent. That was a non issue in my case, since the data undergoes no updates.

#### And changing the serializer

By default ActiveRecord uses the YAML serializer. It converts our Ruby `Hash` (that represents our JSON data) to YAML before storing it in the text column. If we specify the format as JSON instead - the hash will be stored as JSON -

    serialize :raw, JSON

And now when we check the time -

    DataCache.where(label:l).select(:raw).map(&:raw)  ## 2.18 seconds

This is slightly poorer than the `jsonb` time, but still an improvement on the original.

In hindsight, I should have brought this option  up for discussion. Although it is a little slower than using `jsonb`, it could have saved me a lot of deployment trouble.

### The Postgres upgrade problem.

The `jsonb` datatype was introduced in Postgres 9.4. Our Heroku Postgres instance was using 9.5 - that made me somewhat care-free about deployment planning.

Unfortunately, only the dev server I was using was on 9.5, and the production was using 9.3. I noticed that much later. Who would in their right minds create a dev server with different version number?

Well, Heroku, it gives you the latest Postgres on any new application (which a sensible default behaviour). It doesn't prompt you for a version - you can supply it, but if you miss it, you get the latest. And if you application was first deployed three years back - there's a big chance you are running a different PG version in a new dev/staging. 

And ==Postgres is hard to upgrade==. Here's how we do a Postgres upgrade -

    1. Provision a new db
    2. Copy everything from the old to the new
    3. Switch to the new

Postgres comes with a decent tool for the copying - `pg:copy`. But if something goes wrong in copying - you are on your own. Or worse, if copying happens wrongly, you are on your own.

Copying also takes quite some time. It could be 2-5 mins for few MBs of data, and hours if it is GBs.

And in that duration, you'll have to keep your database offline. After copying starts, we can't let any new data enter the application. Think about hours of planned downtime whenever a new version of Postgres comes out<sup> 4 </sup>. 

Fortunately for me, my product owners were keen on doing the upgrades. They allocated time for the effort, and it went smoothly. 

#### A note on pagination

My scenario was a bit of a stretch since pagination was not preferred. We needed all the data in a single go, and were okay with some delay. The CSV export of the same data was a more primary use-case. A somewhat poor page load experience was a trade-off.

#### Conclusion

With this experience, my conclusion was that a database table can serve as a good data-cache. It is a great fit if the data too big, and is structured as a collection.  With the right choice of serializer, some data-redundancy and the right indexes, a relational database can be a fairly good choice.

Postgres _Jsonb_ is also a decent solution - thought I didn't get to use its full power (indexes). I would recommend against rushing into it, because the impact on infrastructure change can outweigh its performance benefits. If that is the only option somehow, keep my experience in mind.

Also, its best not to blindly treat Redis as a replacement of Memcache. In my case, both didn't work, but I can imagine situations where Memcache can work better than Redis. It may be a good idea to experiment with Memcache a bit before rejecting it.

---

[1] This is a variation of the 7MB data-set I mentioned in the previous article. This is also a real use case, and since the numbers are worser here, it illustrates the point better.

[2] Logic dictates that I analyse `serialize` further at this point. But because I was so enthusiastic about `jsonb`, I rushed into experimenting with that first.

[3] Since the import happens in the background, this additional step is not a problem.
 
[4] The Uber engineering post about Postgres to Mysql migration mentions this. 

