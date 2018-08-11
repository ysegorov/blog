---
url: /2017/reloadable/
title: Reloadable decorator
modified: 2017-04-12T20:12:00+04:00
highlight: true
---
# Reloadable decorator

[wt][wt] has a feature I'd like to share here. It was one of the first features
of [wt][wt] I think.

## Problem

Suppose I have an idea for some post, just like this one.

I've opened terminal, started **wt** in *development* mode using
**`wt develop`** command and started **vim** for authoring.

I'm typing for some time...

Ok, now I'd like to check it in browser and I'm adding some metadata for
this post to configuration - **`wt.yaml`** file.

After this step I'm expecting refreshed page in the browser to contain updated
data from configuration.

## Solution

Internally [wt][wt] has a so called [engine][engine] to handle
incoming requests. It depends on **`wt.yaml`** configuration and reads it on
initialization.

It is quite possible for the [engine][engine] to contain logic to reload
configuration when needed but I thought this task shouldn't be implemented
there.

It would be better to have the possibility to easily turn this feature off when
needed (not while *authoring* but while *developing* the library) or even just
throw away.

So, here comes **reloadable** decorator:

```python
# -*- coding: utf-8 -*-

import logging
import os


class reloadable(object):

    def __init__(self, message):
        self._message = message
        self._values = {}
        self._lastmods = {}
        self.logger = logging.getLogger('wt.reloadable')

    def __call__(self, fn):

        def inner(filename):
            v = self._values.get(filename)
            m = self._lastmods.get(filename)
            try:
                modified = os.stat(filename).st_mtime
            except FileNotFoundError:
                self.logger.warn(
                    'File not found "%s", skipping stat', filename)
                pass
            else:
                if m is None or modified > m:
                    self.logger.debug(
                        'File "%s" modified, %s', filename, self._message)
                    self._lastmods[filename] = modified
                    v = None

            if v is None:
                self._values[filename] = v = fn(filename)
            return v
        return inner
```

It keeps last modified times for files and caches results of function calls. In
case file was modified cached value will be automatically updated. Pretty
simple yet very useful.

Real life usage example looks like this:

```python
# -*- coding: utf-8 -*-

from .decorators import reloadable
from .engine import WT


@reloadable('(re)loading configuration...')
def engine(fn):
    return WT(fn)
```

Currently [wt][wt] uses [aiohttp][aiohttp] as http backend.

It's only configured handler calls **`engine`** function to get rendering
[engine][engine] instance (you can check [server module][wt-server] sources
for details).

In case configuration file was updated this call will create new fresh instance
of [engine][engine] ready to render pages and posts.
This instance will be cached within **reloadable** decorator cache so any
subsequent calls to **`engine`** function will just have the very same
instance in return (till configuration was updated again).


-----

That's it for **reloadable** decorator. Stay tuned.


[wt]: https://ysegorov.github.io/wt-docs/
[engine]: https://github.com/ysegorov/wt/blob/master/wt/engine.py#L22
[aiohttp]: http://aiohttp.readthedocs.io/en/stable/
[wt-server]: https://github.com/ysegorov/wt/blob/master/wt/server.py
