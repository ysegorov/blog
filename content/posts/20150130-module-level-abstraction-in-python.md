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
