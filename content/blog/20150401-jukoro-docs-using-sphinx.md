+++
path = "/2015/jukoro-docs-using-sphinx/"
title = "Jukoro documentation using Sphinx"
date = 2015-04-01
[extra]
modified = 2015-04-01T10:40:00+00:00
+++
# Jukoro documentation using Sphinx

<div class="note">

**Update.** `jukoro` library has been moved to archive, no plans to develop it further.

</div>

Summary about creating python library documentation using [Sphinx][sphinx] and
publishing it to [GitHub Pages][ghp] (based on [this article][1]).

```bash
$ git clone git@github.com:ysegorov/jukoro.git
$ cd jukoro

$ mkdir docs
$ pip install sphinx
$ sphinx-apidoc -A "Yuri Egorov" -F -o docs jukoro

$ git add docs
$ git commit -m "sphinx-apidoc generated docs folder contents"

$ mkdir gh-pages
$ vim docs/Makefile  ## change BUILDDIR to ../gh-pages

$ /usr/share/git/workdir/git-new-workdir . gh-pages/html
$ cd gh-pages/html

$ git checkout --orphan gh-pages
$ git rm -rf .

$ cd ../../docs
$ make html
$ cd -

$ git add .
$ git commit -m "sphinx generated docs"
$ git push -u origin gh-pages

$ touch .nojekyll  ## to skip Jekyll
$ git add .nojekyll
$ git commit -m ".nojekyll"
$ git push
```


~~And now there is jukoro's documentation available online~~ (still requires
some attention).

For some reason I've started [to use markdown][3] for ``readme`` and ``changelog``
files but now it seems it would be better to use [reStructuredText][4].

This way it will be possible to include ``readme`` and ``changelog`` to
documentation (which is not finished yet).


[sphinx]: http://sphinx-doc.org/
[ghp]: https://pages.github.com/
[1]: http://raxcloud.blogspot.co.uk/2013/02/documenting-python-code-using-sphinx.html
[3]: /2015/markdown-live/
[4]: http://docutils.sourceforge.net/rst.html
