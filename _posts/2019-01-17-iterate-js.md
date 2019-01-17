---
layout: post
title:  "Haskell's `iterate` in Javascript"
date:   2019-01-17 12:24:00 -0500
excerpt: In which I implement Haskell's iterate function in JS.
---

Haskell's [`iterate`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Prelude.html#v:iterate) accepts a function `f` and an arugment `x`, and produces an infinite list of repeated application of `f` to `x`.

```
iterate f x == [x, f x, f (f x), ...]
```

It's handy for recursion.

With ES6's Generators, this is can be easily implemented in Javascript.

```
function* iterate(f, x) {
  yield x;
  while (true) {
    x = yield f(x);
  }
}
```

If a [`takeWhile`](https://hackage.haskell.org/package/base-4.12.0.0/docs/Prelude.html#v:takeWhile) is implemented alongside, then we have something that takes a predicate for the base case of the recursion.
