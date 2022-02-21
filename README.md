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

