---
layout: default
---

<div class="card-container">
  {% for post in site.posts %}
    <div class="card">
      <a href="{{ post.url }}">
        <img src="{{ post.image }}" alt="{{ post.title }}" style="width:100%; height:auto;">
      </a>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <div class="tags">
        {% for tag in post.tags %}
          <a href="/tags/{{ tag | slugify }}.html" class="tag">{{ tag }}</a> <!-- 使用 slugify 处理标签 -->
        {% endfor %}
      </div>
    </div>
  {% endfor %}
</div>

<style>
.card-container {
  display: flex;
  flex-wrap: wrap;
}
.card {
  border: 1px solid #ccc;
  margin: 10px;
  padding: 10px;
  width: calc(33% - 20px); /* 3列布局 */
}
.tag {
  display: inline-block;
  margin-right: 5px;
  padding: 5px;
  background-color: #007bff;
  color: white;
  text-decoration: none;
}
</style>
