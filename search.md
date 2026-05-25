---
layout: page
title: "Поиск"
permalink: /search/
---

<input id="search-input" type="text" placeholder="Введите текст для поиска по постам" style="width:100%;max-width:640px;padding:10px;font-size:16px;">

<ul id="search-results" style="margin-top:16px;">
{% for post in site.posts %}
  <li
    data-search="{{ post.title | downcase | escape }} {{ post.content | strip_html | downcase | escape }} {{ post.tags | join: ' ' | downcase | escape }}"
    style="margin-bottom:12px;"
  >
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <div style="font-size:0.9em;color:#666;">{{ post.date | date: "%Y-%m-%d" }}</div>
  </li>
{% endfor %}
</ul>

<script>
(function () {
  var input = document.getElementById('search-input');
  var items = Array.prototype.slice.call(document.querySelectorAll('#search-results li'));

  function normalize(s) {
    return (s || '').toLowerCase().trim();
  }

  function filter() {
    var q = normalize(input.value);
    items.forEach(function (item) {
      var hay = item.getAttribute('data-search') || '';
      item.style.display = !q || hay.indexOf(q) !== -1 ? '' : 'none';
    });
  }

  input.addEventListener('input', filter);
})();
</script>
