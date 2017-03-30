# Logger service using epoll - Part 2

<div class="note" markdown="1">
This is a follow-up for [part 1][part1] post about logger service.
</div>

So, the task is to have logger service capable of handling, let's say, 1k
logging records per second. I am expecting about 100 client connections to this
service and It's clear that it must be stable enough to handle incoming
requests.

*Please do remember that all figures in this post I've got running the code on
my laptop - YMMV*.

To test the service I have this script:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import time
import random
import logging
import logging.config
import multiprocessing as mp


def setup_logging():
    conf = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'default': {
                'format': (
                    '%(levelname)1.1s %(asctime)s %(name)s:%(lineno)3s '
                    '%(message)s'),
                'datefmt': '%Y.%m.%d %H:%M:%S',
            }
        },
        'handlers': {
            'service_logger': {
                'level': 'DEBUG',
                # must be importable
                'class': 'svctools.handlers.SocketHandler',
                'host': '127.0.0.1',
                'port': 5000,
            },
        },
        'loggers': {
            '': {
                'level': 'DEBUG',
                'handlers': ['service_logger'],
                'propagate': False,
            },
        },
    }
    logging.config.dictConfig(conf)


def run(idx):
    msg = '[%06d] log message %08d'
    pid = os.getpid()
    logger = logging.getLogger(__name__)
    cnt = count = random.randint(300, 800)
    while True:
        logger.debug(msg, pid, cnt, extra={'trace_id': idx})
        cnt -= 1
        time.sleep(random.choice((0.01, 0.02, 0.03, 0.04, 0.03, 0.02, 0.01)))
        if cnt == 0:
            break

    logger.debug('[%06d] --------', pid, extra={'trace_id': idx})

    return count


def main():
    pool = mp.Pool(200, setup_logging)
    r = pool.map_async(run, range(1000))

    data = r.get()
    print('sent %s records' % sum(data))


if __name__ == "__main__":
    main()

```

We have pool of 200 processes performing the same action - just calling
`logger.debug`.
Single action to be performed by a process is the following:

- randomly select number of logging messages to be send to service (somewhere
  between 300 and 800 messages)
- send messages one by one with a tiny delay in between (to simulate some kind
  of real load)

We will be running 1000 of such actions which means we will send about 500k
messages to our service (this is just an approximation).


### Step 1 - logging to a file (only)

It was not interesting at all as I've got about **8700 records per second** -
very impressive. But we need all records to be stored in database - to do
a query later. Let's see.


### Step 2 - logging to a file and to PostgreSQL using logging handler

The code for PostgreSQL logging handler is available [here][pghandler]. This
time results were disappointing - about **200 records per second** (at the same
time there was no data loss which is good).


### Step 3 - logging to a file and to PostgreSQL using self-made handler

It was clear that our service everytime runs the same query inserting just one
record to database. So, the trick is to employ buffering and to insert records
in bulk using, for example, [`psycopg2.extras.execute_values`][execute_values].
This time I've got about **3600 records per second** using 1k-blocks - much
better. For a batches of 100 records I've got about **3000 records per second**
- a bit slower but still good enough.

This solution has one trick though. As we are prebuffering records there will
be times buffer is not full and there is no incoming records for some time.
This can lead to data loss which is bad. To solve this problem we can use
[`signal.setitimer`][setitimer] to setup simple timer to send `SIGALRM` on a
regular basis and to flush receiving buffer on it.


Here are the features of the logger service:

- no threads or child processes
- edge-triggered epoll for i/o
- PostgreSQL backend
- the code looks ugly

Last feature will be fixed soon. Stay tuned.


[part1]: /2017/logger/
[pghandler]: https://gist.github.com/ysegorov/8947d99a016aa00ace51d9ab4d89c428#file-pghandler-py
[execute_values]: http://initd.org/psycopg/docs/extras.html#psycopg2.extras.execute_values
[setitimer]: https://docs.python.org/2/library/signal.html#signal.setitimer
