zolaschool
==

Step 00
--

To start with zola the init command comes in handy:

```
zola init zolaschool
cd zolaschool
```

it creates the directory structure needed and a config file. I just pressed enter to use the default settings. After that you can run a server to deliver the pages with:

```
zola serve
```

if you visit [http://127.0.0.1:1111/](http://127.0.0.1:1111/) you read:

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

