+++
path = "/2015/module-level-abstraction-in-python/"
title = "Module level abstraction in python"
date = 2015-01-31
[extra]
modified = 2015-01-30T17:40:00+00:00
+++
# Module-level abstraction in Python

Sample code first.

```python
# -*- coding: utf-8 -*-
#
# file: jukoro/pickle.py

import cPickle


load = cPickle.load
loads = cPickle.loads


def dump(obj, f):
    return cPickle.dump(obj, f, protocol=cPickle.HIGHEST_PROTOCOL)


def dumps(obj):
    return cPickle.dumps(obj, protocol=cPickle.HIGHEST_PROTOCOL)

```


No magic inside - in our own project we've created **pickle.py** module,
the one to be used in place of **cPickle** from standart library.

As a result we've got an [abstraction][1], which helps us to keep code DRY and
improves code manageability.

I like this approach.


[1]: http://en.wikipedia.org/wiki/Abstraction_principle_(computer_programming)
