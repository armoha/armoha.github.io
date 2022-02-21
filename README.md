zolaschool
==

Step 00
--

To start with zola the init command comes in handy:

```
zola init zolaschool
cd zolaschool
```

it creates the directory structure needed and a config file. I just pressed enter to use the default settings. After that i can run a server to deliver the pages with:

```
zola serve
```

if i visit [http://127.0.0.1:1111/](http://127.0.0.1:1111/) i read:

> To modify this page, create a index.html file in the templates directory

Step 01
--

So i did that:

```
cat > templates/index.html <<EOF
<h1>Something</h1>

EOF
```

and in the browser the page reloads without any intervention. Nice!

Step 02
--

Thats more or less static html serving - where does my content come in? The "index"-page is called `_index` in the content space:

```
cat > content/_index.md <<EOF
+++
title = "Anything"
+++
EOF
```

if i now adapt the template accordingly the title switches:

```
cat > templates/index.html <<EOF
<h1>{{ section.title }}</h1>
EOF
```

Step 03
--

More content files in the same directory are pages, the content is formulated in markdown with a meta data header surrounded by "+++". The syntax of the meta data is toml:

```
cat > content/nothing.md <<EOF
+++
title = "Nothing"
+++

Nothing
==

EOF
```

if i navigate to [http://127.0.0.1:1111/nothing](http://127.0.0.1:1111/nothing) zola tells me `page.html` is missing:

```
cat > templates/page.html <<EOF
<h1> {{ page.title }} </h1>
{{ page.content | safe }}

EOF
```

Which shows Nothing twice, once from the meta data `page.title`, another from the markdown part.

Step 04
--

Navigating back to the main page shows that i have no hint, that "Nothing" exists. I can add links to all pages in this directory. A directory is called "section" in zola. A section can contain multiple pages and we can iterate over those:

```
cat >> templates/index.html <<EOF
<h2>Pages:</h2>
<ul>
	{% for page in section.pages %}
		<li><a href="{{ page.permalink }}">{{ page.title }}</a></li>
	{% endfor %}
</ul>
```

Watching the browser shows that the "Nothing"-page shows up as an item in an unordered list.

Step 05
--

I want a subsection

```
mkdir content/things
cat > content/things/_index.md << EOF
+++
title = "Things"
+++

More things
--

EOF
```

The subsection doesnt show up as a page in "/" and manually navigating to [http://127.0.0.1:1111/things/](http://127.0.0.1:1111/things/) shows that section.html is missing:

```
cp templates/index.html templates/section.html
```

Step 06
--

The initial page doesnt show the subsections, we can add that ability:

```
cat >> templates/index.html <<EOF
<h2>Sections:</h2>
<ul>
	{% for subsection in section.subsections %}
		{% set sub = get_section(path=subsection, metadata_only=true) %}
		<li><a href="{{ sub.permalink | safe }}">{{ sub.title }}</a></li>
	{% endfor %}
</ul>
EOF
```

And i did the same for sections:

```
cat >> templates/section.html <<EOF
<h2>Sections:</h2>
<ul>
	{% for subsection in section.subsections %}
		{% set sub = get_section(path=subsection, metadata_only=true) %}
		<li><a href="{{ sub.permalink | safe }}">{{ sub.title }}</a></li>
	{% endfor %}
</ul>
EOF
```

Step 07
--

I can now add some more content:

```
cat > content/things/everything.md <<EOF
+++
title = "Everything"
+++

This is every thing
==

EOF
```

It will show up in [http://127.0.0.1:1111/things/](http://127.0.0.1:1111/things/).

Step 08
--

I want templates! This is the base for all my pages:

```
cat > templates/base.html <<EOF
<!DOCTYPE html>
<html>
	<head>
		<title>Thingieworld</title>
	</head>
	<body>
		<section>
			<div>
				{% block content %}
				{% endblock %}
			</div>
		</section>
	</body>
</html>
EOF
```

Now i need to surround my html files with the extends & block commands:

```
for n in templates/index.html templates/page.html templates/section.html; do (
	echo '{% extends "base.html" %}'
	echo '{% block content %}'
	cat $n
	echo '{% endblock content %}' ) > x
	mv x $n
done
```

Step 09
--

I want some styling:

```
cat > sass/site.scss <<EOF
body {
	background-color: #444;
	color: #ccc;
	font-family: Lucida Console, Liberation Mono, DejaVu Sans Mono, monospace;
	padding: 2em;
	margin: 2em;
}

a {
	color: #e90;
}
EOF
```

Which is included by a headerline in templates/base.html:

```
<link rel="stylesheet" href="{{ get_url(path="site.css", trailing_slash=false) | safe }}">
```

Step 10
--

Special section / page handling is done when using _index or index files in a directory. An `_index` file denotes a subsection while an `index` file is just another page. A file like `a.md` is the same page as `a/index.md`. I created both cases:

```
mkdir content/a content/b
cat > content/a/index.md <<EOF
+++
title = "a"
+++
EOF
cat > content/b/_index.md <<EOF
+++
title = "b"
+++
EOF
```

The browser shows the links in different lists (i added header lines inbetween the lists for better visibility, see above).

Step 11
--

I enabled a taxonomy of "tags" in config by adding

```
taxonomies = [
	{name = "tags"},
]
```

into the main section of `config.toml`. Two templates to render taxonomies are needed:

```
cat > templates/taxonomy_single.html <<EOF
{% extends "base.html" %}
{% block content %}
	<h1>{{ term.name }}</h1>
	{% for page in term.pages %}
		<a href="{{ page.permalink | safe }}">{{ page.title }}</a>
	{% endfor %}
{% endblock content %}
EOF
cat > templates/taxonomy_list.html <<EOF
{% extends "index.html" %}
{% block content %}
	{% if terms %}
		{% for term in terms %}
			<p><a href="{{ term.permalink | safe }}">{{ term.name }}</a> ({{ term.pages | length }})</p>
		{% endfor %}
	{% endif %}
{% endblock content %}
EOF
```

and in `templates/page.html` i added a footer with the tags of that page:

```
<h2>Tags:</h2>
{% if page.taxonomies.tags %}
	{% for tag in page.taxonomies.tags %}
		<a href="{{ get_taxonomy_url(kind="tags", name=tag) | safe }}">{{ tag }}</a>
	{% endfor %}
{% endif %}
```

finally i added a taxonomy "tags" to `content/nothing.md` and `content/things/everything.md`:

```
[taxonomies]
tags = ["thing", "no"]
```

i added

```
	<a href="/tags/">all tags</a>
```

to `templates/index.html` and `templates/section.html` to be able to navigate to "taxonomy_list". I didnt find out a way to programmatically retrieve the url and thus hardcoded it to be "/tags/" (hints welcome).

Step 12
--

For search to work i enabled it in `config.toml`:

```
build_search_index = true
```

added `static/search.js` and `sass/_search.scss` from (https://github.com/getzola/zola/blob/master/docs/), added `@import "search";` to `sass/site.scss`. Finally i added the box to `templates/base.html`.


Disclaimer
--

zolaschool is licensed under ![https://creativecommons.org/licenses/by-sa/4.0/](https://mirrors.creativecommons.org/presskit/buttons/88x31/png/by-sa.png)

