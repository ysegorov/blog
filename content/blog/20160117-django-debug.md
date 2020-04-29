+++
path = "/2016/django-debug/"
title = "Django project debugging"
date = 2016-01-17
[extra]
modified = 2016-01-17T12:32:00+00:00
summary = "Simple tools to debug complex code"
+++
# Django project debugging


Quite often there is a need to debug some code, to see what's going on and why
there is so much time spent generating pretty simple web page.

There are tools making this task easier, [django-debug-toolbar][1] looks like
**must have** debug swiss knife but I'd like to use a self-made solution.

I used to use logging as a way to get the information needed so [here are][2]
two classes to debug code execution time and SQL queries and timings.
They can be used as decorators or as context managers (see below).

{{ gist(url="https://gist.github.com/ysegorov/7191601fd58bb37a9efd") }}


To initialize these classes you will have to pass ``prefix`` as the only
argument (needed to grep logging output).

Here are some abstract usage examples:

```python

# -*- coding: utf-8 -*-

from django.shortcuts import render

from project.articles import get_articles, get_stats
from project.debug import timer, debug_sql


@debug_sql('index')
@timer('index')
def index(request):
    return render(request, 'index.html')


def articles(request):
    with timer('articles'):
        articles = get_articles()
        with debug_sql('articles'):
            stats = get_stats(articles)
    return render(request, 'articles.html', context={'articles': articles,
                                                     'stats': stats})

```

It's pretty easy to use the helpers, there is no need for a project-wide
configuration and you can focus on the code you are debugging to avoid logging
noise.

[1]: https://github.com/django-debug-toolbar/django-debug-toolbar
[2]: https://gist.github.com/ysegorov/7191601fd58bb37a9efd
