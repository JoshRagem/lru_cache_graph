lru_cache_graph
=====

An escript to demonstrate a way to build an lru cache in Erlang without cyclic data structures.

### no cyclic data structures

Since nothing in Erlang is mutable, it is not possible to create truly cyclic structures such as:
```
# can't do this in erlang
a = {other: None}
b = {other: a}
a.other = b
```

To accomplish the goal of an lru cache, I decided to model the doubly-linked list as a directed graph like:
```
a <-> b <-> c <-> d <-> e
```
with an `edge` record standing in for an edge headed left and another edge headed right. The problem now becomes one of properly ordering the set of `edge` records.

#### get a value in the graph

For a key that already exists in the graph, it must be reordered; to do this I break the graph down into a list of vertices, filter out the key for the get, and rebuild the edges:
```
# get 'c' from
a <-> b <-> c <-> d <-> e
# becomes
[ c a b d e ]
# becomes
c <-> a <-> b <-> d <-> e
```

That looks pretty simple, but it greater detail it looks like this:
```
# get 'c' from a <-> b <-> c <-> d <-> e
[ {a, b} {b, c} {c, d} {d, e} ]
# becomes
[ a b b c c d d e ]
[ a b b d d e] # remove 'c'
[ c a b b d d e]
# build edges, but ignore repeated vertices
[ {c, a} {a, b} {b, d} {d, e} ]
```

#### get a value not in the graph
```
# get 'j' from a <-> b <-> c <-> d <-> e
# first read the value for 'j' from storage
[ {a, b} {b, c} {c, d} {d, e} ]
# becomes
[ a b b c c d d e ]
[ j a b b c c d d e ]
# build edges, but ignore repeated vertices
[ {j, a} {a, b} {b, c} {c, d} {d, e} ]
# this graph is too big, we must trim it to the correct length
[ {j, a} {a, b} {b, c} {c, d} ] # to trim => [ {d, e} ]
# we only have to remove the right hand values from any edges in the 'to trim' list: 'e'
# remove 'e' from cache
# insert 'j' into cache
```

Build
-----

    $ rebar3 escriptize

Run
---

    $ _build/default/bin/lru_cache_graph
