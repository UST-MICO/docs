# Documentation Repository for MICO (working title)

Documentation: [readthedocs](mico-docs.readthedocs.io)

![build badge](https://readthedocs.org/projects/mico-docs/badge/?version=latest)


## Useful links:

 *  [Sphinx](http://www.sphinx-doc.org/en/master/)

    Sphinx is a tool to compile ReStructuredText documentation into a variety of formats.
 *  [ReStructuredText](http://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)
 *  [Getting started (readthedocs)](https://docs.readthedocs.io/en/latest/intro/getting-started-with-sphinx.html#using-markdown-with-sphinx)

## Build the documentation locally:

install requirements:

```bash
pip install -r requirements.txt
```

build html:

```bash
make html
```

open `_build/html/index.html` in your browser
