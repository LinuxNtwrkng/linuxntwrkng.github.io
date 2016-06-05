---
layout: post
title: Adding Search
date: '2016-06-04 17:39'
---

I used [this](http://jekyll.tips/jekyll-casts/jekyll-search-using-lunr-js/) as a guide to get search set up.  

1.  Download the minified [lunr.js](http://lunrjs.com/), which is what will do the heavy lifting of the search:

```` bash
mkdir js

cd js

curl https://raw.githubusercontent.com/olivernn/lunr.js/master/lunr.min.js > lunr.min.js
````

2.  Create `/js/search.js` as follows:

{% raw %}

~~~~ javascript
(function() {
  function displaySearchResults(results, store) {
    var searchResults = document.getElementById('search-results');

    if (results.length) { // Are there any results?
      var appendString = '';

      for (var i = 0; i < results.length; i++) {  // Iterate over the results
        var item = store[results[i].ref];
        appendString += '<li><a href="' + item.url + '"><h3>' + item.title + '</h3></a>';
        appendString += '<p>' + item.content.substring(0, 100) + '...</p></li>';
      }

      searchResults.innerHTML = appendString;
    } else {
      searchResults.innerHTML = '<li>No results found</li>';
    }
  }

  function getQueryVariable(variable) {
    var query = window.location.search.substring(1);
    var vars = query.split('&');

    for (var i = 0; i < vars.length; i++) {
      var pair = vars[i].split('=');

      if (pair[0] === variable) {
        return decodeURIComponent(pair[1].replace(/\+/g, '%20'));
      }
    }
  }

  var searchTerm = getQueryVariable('query');

  if (searchTerm) {
    document.getElementById('search-box').setAttribute("value", searchTerm);

    // Initalize lunr with the fields it will be searching on. I've given title a boost of 10 and tags a boost of 5 to indicate matches on these fields are more important.
    var idx = lunr(function () {
      this.field('id');
      this.field('title', { boost: 10 });
      this.field('tags', { boost: 5 });
      this.field('author');
      this.field('category');
      this.field('content');
    });

    for (var key in window.store) { // Add the data to lunr
      idx.add({
        'id': key,
        'title': window.store[key].title,
        'tags': window.store[key].tags,
        'author': window.store[key].author,
        'category': window.store[key].category,
        'content': window.store[key].content
      });

      var results = idx.search(searchTerm); // Get lunr to perform a search
      displaySearchResults(results, window.store); // We'll write this in the next section
    }
  }
})();
~~~~

{% endraw %}

3.  Create `/search.html` as follows:

{% raw %}

~~~~ javascript
---
layout: "search"
title: "Search Results"
---

<ul id="search-results"></ul>

<form action="{{ site.baseurl }}/search" method="get">
  <label for="search-box"></label>
  <input type="hidden" id="search-box" name="query">
  <input type="hidden" value="search">
</form>

<script>
  window.store = {
    {% for post in site.posts %}
      "{{ post.url | slugify }}": {
        "title": "{{ post.title | xml_escape }}",
        "tags": "{{ post.tags }}",
        "author": "{{ post.author | xml_escape }}",
        "category": "{{ post.category | xml_escape }}",
        "content": {{ post.content | strip_html | strip_newlines | jsonify }},
        "url": "{{ post.url | xml_escape }}"
      }
      {% unless forloop.last %},{% endunless %}
    {% endfor %}
  };
</script>

<script src="/js/lunr.min.js"></script>
<script src="/js/search.js"></script>
~~~~

{% endraw %}

4.  Create `/_layouts/search.html` as follows:

{% raw %}

~~~~ html
---
layout: default
---

<article class="search">

  <h1>{{ page.title }}</h1>

  <div class="entry">
    {{ content }}
  </div>
</article>
~~~~

{% endraw %}
