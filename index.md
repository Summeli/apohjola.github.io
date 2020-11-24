# Antti Pohjola's blog

Let's start with training for github pages

{% for post in site.posts %}
  <!-- Your post's summary goes here -->
  {{ post.excerpt }}
  [read more]({{post.url}})
{% endfor %}