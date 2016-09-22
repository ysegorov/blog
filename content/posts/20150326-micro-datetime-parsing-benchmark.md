# Micro datetime parsing benchmark

Just a note for myself about datetime parsing in python.

Let's assume we know ahead that we have a datetime string in [ISO 8601][iso8601]
format and timezone is [UTC][utc].

Here is a benchmark.

```python
In [1]: import arrow, re, datetime as dt

In [2]: iso = arrow.utcnow().isoformat()

In [3]: iso
Out[3]: '2015-03-26T07:59:44.210642+00:00'

In [4]: r = re.compile(r'[\-T:\.]')

In [5]: arrow.get(iso)
Out[5]: <Arrow [2015-03-26T07:59:44.210642+00:00]>

In [6]: arrow.get(dt.datetime.strptime(iso.rpartition('+')[0], '%Y-%m-%dT%H:%M:%S.%f'), 'UTC')
Out[6]: <Arrow [2015-03-26T07:59:44.210642+00:00]>

In [7]: arrow.get(dt.datetime(*map(int, r.split(iso.rpartition('+')[0]))), 'UTC')
Out[7]: <Arrow [2015-03-26T07:59:44.210642+00:00]>

In [8]: %timeit arrow.get(iso)
10000 loops, best of 3: 54.7 µs per loop

In [9]: %timeit arrow.get(dt.datetime.strptime(iso.rpartition('+')[0], '%Y-%m-%dT%H:%M:%S.%f'), 'UTC')
100000 loops, best of 3: 17.5 µs per loop

In [10]: %timeit arrow.get(dt.datetime(*map(int, r.split(iso.rpartition('+')[0]))), 'UTC')
100000 loops, best of 3: 8.83 µs per loop
```


Unfortunatelly, the third way of parsing is wrong because it doesn't take into
account microseconds zero-padded to the left.

[Arrow][arrow] library is just great and is certainly worth trying out.


[iso8601]: http://en.wikipedia.org/wiki/ISO_8601
[utc]: https://en.wikipedia.org/wiki/Coordinated_Universal_Time
[arrow]: https://github.com/crsmithdev/arrow
