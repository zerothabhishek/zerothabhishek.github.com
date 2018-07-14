---
# layout: 
---

### Abhishek Yadav

<!-- [Articles](/articles) -->

<!-- [Projects](/projects) -->

<!-- [Contact](/contact) -->


<div id="blog_index">
  <div class="posts">
    {% for post in site.posts %}
      <div class="post_item">
        <!-- <div class="date">{{ post.date | date_to_string }}</div> -->
        <div class="post_title"><a href="{{ post.url }}"> {{ post.title }}</a></div>
      </div>
    {% endfor %}
  </div>
</div>
