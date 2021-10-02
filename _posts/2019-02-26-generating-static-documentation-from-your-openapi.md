---
title: Generating static documentation from your OpenAPI 3.0
layout: post
comments: true
published: false
description: 
keywords: openapi, swagger, yaml, documentation, rest, restful
---

## api2html

This is my favorite. With api2html you can transform Swagger / OpenAPI files to beautiful HTML:

* https://github.com/tobilg/api2html

Other tools...

## Widdershins

### Requirements

* [NPM](https://nodejs.org/en/download/)
* [widdershins](https://github.com/Mermade/widdershins)
* A OpenAPI v3 file, example [petstore.yaml](https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore.yaml)

### Installation

With widdershins you can produce a static documentation from your OpenAPI 3.0 / Swagger 2.0 / AsyncAPI 1.x / Semoasa 0.1.0 definition.

To install widdershins run:

```bash
mkdir widdershins
cd widdershins
npm install widdershins
```

### Conversion into a Markdown file

Create a config file for widdershins: `widdershins.json`

Content:

```json
{
  "search": false,
  "codeSamples": false,
  "lang": false,
  "httpsnippet": false,
  "language_tabs": [],
  "language_clients": [],
  "verbose": true,
  "tocSummary": false
}
```

Then run widdershins to generate the markdown file.

Linux: 

```bash
node widdershins --environment widdershins.json petstore.yaml -o resources.md
```

Windows:

```cmd
> call node_modules\.bin\widdershins.cmd --environment widdershins.json petstore.yaml -o resources.md
```

## Mkdocs

### Conversion into a HTML documentation

Mkdocs helps you create beautiful API documentation.

### Requirements

* [Python](https://www.python.org/) + PIP
* [Mkdocs](https://pandoc.org/installing.html#windows)
* Some markdown files: index.md, security.md, resources.md

Assuming you have Python already, install Mkdocs with Python PIP:

```bash
pip install mkdocs
```

Create a file: `mkdocs.yml`:

```yml
site_name: Petstore
nav:
    - Home: index.md
    - Security: security.md
    - Resources: resources.md
theme:
  name: readthedocs
  navigation_depth: 3
  titles_only: false
  custom_dir: "custom_theme/"
  highlightjs: false
  hljs_languages:
      - javascript

use_directory_urls: false

# disable all plugins for offline browsing (e.g. search)
plugins: []
```

Create a directory inside your project to hold your docs:

```
mkdir docs/
```

Copy the markdown files into the `docs/` directory.

Also create a file `index.md` and `security.md` within the `docs/` directory.

Now we have to fix the navigation items by overwriting the `toc.html` template.

Create a directory `custom_theme/` in the base path of the project.

Create a file `toc.html` in the `custom_theme/` directory.

Content of: `toc.html`:

{% raw %}
```twig
{% for toc_item in page.toc %}
    <li class="toctree-l{{ navlevel + 1 }}">
        <a href="{{ toc_item.url }}">{{ toc_item.title }}</a>
    </li>  
{% endfor %}
```
{% endraw %}

Ok, let's generate the html files with this command:

```bash
mkdocs build
```

Now you should find the html documentation in the `site/` directory.

Screenshot:

![image](https://user-images.githubusercontent.com/781074/53407600-d283d200-39bc-11e9-8d8d-2b094fc4772e.png)

## Pandoc

### Conversion into a single HTML file

### Requirements

* pandoc: [Download](https://pandoc.org/installing.html#windows)

If multiple input files are given, pandoc will concatenate them all (with blank lines between them) before parsing.

```bash
pandoc -s index.md security.md resources.md -o petstore.html
```

Add this [CSS](https://gist.github.com/killercup/5917178) to your Pandoc HTML documents using `--css pandoc.css` to make them look more awesome. 

```bash
pandoc --css pandoc.css --to=html5 --highlight-style=haddock --self-contained -s index.md security.md resources.md -o petstore.html
```

On windows the `--self-contained` option may crash. In this case just remove this option:

```bash
pandoc --css pandoc.css --to=html5 --highlight-style=haddock -s index.md security.md resources.md -o petstore.html
```

Screenshot:

![image](https://user-images.githubusercontent.com/781074/53409319-ed584580-39c0-11e9-8d34-6a3518ffd63b.png)


### Conversion into a DOCX file

```bash
pandoc -s index.md security.md resources.md -o petstore.docx
```

### Conversion into a PDF file

Just open the docx file with Microsoft Word and export the file to PDF.

## Other solutions

* <https://swaggerdown.io/> - Generate documentation from your OpenAPI specs.

