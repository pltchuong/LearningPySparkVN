Tác giả: **{{site.data.glossary.tomasz_drabas}}** và **{{site.data.glossary.denny_lee}}**

Người dịch: **{{site.data.glossary.phan_chuong}}**

<ul>
  {% for post in site.posts reversed %}
    <li>
      <a href="{{ post.url | relative_url }}">
      {% if post.number %}
        {{ post.number }} -
      {% endif %}
      {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
