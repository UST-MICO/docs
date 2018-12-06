# Documentation Repository

Documentation: [readthedocs](http://mico-docs.readthedocs.io)

![build badge](https://readthedocs.org/projects/mico-docs/badge/?version=latest)

## Useful links

* [Sphinx](http://www.sphinx-doc.org/en/master/)  
   Sphinx is a tool to compile ReStructuredText documentation into a variety of formats.
* [ReStructuredText](http://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)
* [Getting started (readthedocs)](https://docs.readthedocs.io/en/latest/intro/getting-started-with-sphinx.html#using-markdown-with-sphinx)

## Build the documentation locally

**Install Graphviz:**

Using apt (Ubuntu / Debian):

```bash
sudo apt-get install graphviz
```

Using brew (Mac OS X):

```bash
sudo brew install graphviz
```

**Install requirements:**

```bash
pip install -r requirements.txt
```

Make sure you have the `dot` command from `graphviz` and a basic `LaTeX` environment in your path!

**Build html:**

```bash
make html
```

open `_build/html/index.html` in your browser

## Enabled Extensions

* [sphinx.ext.intersphinx](http://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html)

   List all link targets in the documentation:
   ```bash
   python -m sphinx.ext.intersphinx ./_build/html/objects.inv
   ```

* sphinx.ext.autosectionlabel
* sphinx.ext.todo
* sphinx.ext.imgmath
* sphinx.ext.graphviz
* [sphinxcontrib.httpdomain](https://sphinxcontrib-httpdomain.readthedocs.io/en/stable/)
* [sphinxcontrib.openapi](https://sphinxcontrib-openapi.readthedocs.io)
