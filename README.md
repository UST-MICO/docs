# Documentation Repository for MICO (working title)

Documentation: [readthedocs](http://mico-docs.readthedocs.io)

![build badge](https://readthedocs.org/projects/mico-docs/badge/?version=latest)


## Useful links:

 *  [Sphinx](http://www.sphinx-doc.org/en/master/)

    Sphinx is a tool to compile ReStructuredText documentation into a variety of formats.
 *  [ReStructuredText](http://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)
 *  [Getting started (readthedocs)](https://docs.readthedocs.io/en/latest/intro/getting-started-with-sphinx.html#using-markdown-with-sphinx)

## Build the documentation locally:

**Install Graphviz**
Usign apt (Ubuntu / Debian):
```bash
sudo apt-get install graphviz 
```

Using brew (Mac OS X):
```bash
sudo brew install graphviz
```

**Upgrade pip:**
```bash
sudo -H pip2 install --upgrade pip
sudo -H pip3 install --upgrade pip
```

**Install requirements:**
```bash
pip install -r requirements.txt
```

**Build html:**
```bash
make html
```

open `_build/html/index.html` in your browser
