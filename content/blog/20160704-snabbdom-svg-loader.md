+++
path = "/2016/svg-snabbdom-loader/"
title = "Webpack loader to use svg icons as snabbdom VNode"
date = 2016-07-04
[extra]
modified = 2016-07-04T08:45:00+00:00
+++
# Webpack loader to use svg icons as snabbdom VNode

<div class="note">

**Update.** This library has been moved to archive, no plans to develop it further.

</div>

I'm exploring FRP for frontend and it looks very interesting to me, especially
for dataflow using streams.

Unidirectional data flow, introduced as **Flux** by **Facebook** engineers, is
an excellent paradigm and I was thinking about finding a way to use it without
**React** (due to the fact I don't like it - just the same way I don't like
**Angular** as both of them look overengineered to me).

It took some time to understand the subject of FRP and there are a lot of
useful resources available (just a couple of links here - [Mostly Adequate
Guide to Functional Programming][1], [React-less Virtual DOM with Snabbdom][2],
[The introduction to Reactive Programming][3]).

My first attempt was to understand [snabbdom][snabbdom] / [flyd][flyd] /
[union-type][union-type] / [ramda][ramda] approach and the most challenging
part was **union types**.

Studying [The ELM Architecture][4] helped a lot in this.

To dive deeper on the subject I've created a very simple implementation of
union types which in it's core just tags values using specified tag name:

```javascript
// type.js

var curryN = require('ramda/src/curryN');

function Type(spec) {
    var o = {},
        keys = Object.keys(spec);

    keys.forEach(function (key) {
        var fn = spec[key];
        if (spec[key].length > 2) {
            o[key] = curryN(fn.length - 1, function () {
                return {
                    value: [].slice.apply(arguments),
                    tag: key,
                    handler: fn
                };
            });
        } else {
            o[key] = function (smth) {
                return {
                    value: smth,
                    tag: key,
                    handler: fn
                };
            };
        }
    });
    function update(tagged, model) {
        var tag = tagged.tag,
            value = tagged.value,
            fn = tagged.handler;
        value = Array.isArray(value) ? value
                                        : (value != null ? [value] : []);
        value.push(model);
        return fn.apply(null, value);
    }
    o.update = curryN(2, update);
    return o;
}
```

Example of union type definition:

```javascript
// component.js

var Type = require('./type');
var R = require('ramda');

var Msg = Type({
    Click: function (evt, model) { return R.assoc('clicks', model.clicks + 1, model); },
    Remove: function (evt, model) { return R.assoc('_removed', true, model); },
    Tick: function (model) {
        return R.assoc('count', model.count > 359 ? 0 : model.count + 1, model);
    }
});
```

It is a bit simplier than [union-type][union-type], doesn't have type checking and
probably missing something in this implementation.

Anyway, it helped me to understand the data flow.

My experiments are available in [this project][vdom-experiment]
(work in progress).

I like to use svg icons for ui components and I used to create separate
html file with all icons embedded in it using single **svg** and multiple
**symbol** definitions and referencing them using **xlink:href** attribute.

I was planning to use icons to enrich the ui of the experiments and thought it
would be great to have **svg** icons available as [snabbdom][snabbdom] **VNodes**.

So, here is [the webpack loader][loader] for this (this is my first published
**npm** module).

Usage is pretty simple just like any webpack loader:

```javascript
// icons.js

var home = require('./svg/home.svg'),
    alert = require('./svg/alert.svg'),
    ...

module.exports = {
    home: home,
    alert: alert,
    ...
};
```

```javascript
// alerts.js

var icons = require('./icons'),
    h = require('snabbdom/h');

function view(model) {
    return h('div.info', {}, [icons.alert, 'Some alert here']);
}
```

The loader can be chained with **svgo-loader**, please check the docs for this.

That's it for now.



[1]: https://www.gitbook.com/book/drboolean/mostly-adequate-guide/details
[2]: https://medium.com/@yelouafi/react-less-virtual-dom-with-snabbdom-functions-everywhere-53b672cb2fe3#.fnuaga25e
[3]: https://gist.github.com/staltz/868e7e9bc2a7b8c1f754
[snabbdom]: https://github.com/paldepind/snabbdom
[flyd]: https://github.com/paldepind/flyd
[union-type]: https://github.com/paldepind/union-type
[ramda]: https://github.com/ramda/ramda
[4]: http://guide.elm-lang.org/architecture/index.html
[loader]: https://github.com/ysegorov/svg-snabbdom-loader
[vdom-experiment]: https://github.com/ysegorov/vdom-experiment
