# Higher Order Recursion Schemes

## Contents

 * [Anamorphism](#anamorphism)
 * [Apomorphism](#apomorphism)
 * [Catamorphism](#catamorphism)
 * [Paramorphism](#paramorphism)
 * [References](#references)

## Anamorphism

Signature: `a → F(a)`

CoAlgebra (takes value "up" a type).

Co-recursion

Functional equivalent of a while-loop.

Dual of [catamorphism](#catamorphism).

```js
const unfold = (f, seed) => {
    const go = (f, seed, acc) => {
        const res = f(seed);
        return res ? go(f, res[1], acc.concat([res[0]])) : acc;
    }
    return go(f, seed, []);
}
```

```js
const ana = (g, a) => g(a).map(x => ana(g, x));
```

## Apomorphism

## Catamorphism

Signature: `F(a) → a`

Algebra (takes value "down" a type).

Dual of [anamorphism](#anamorphism).

Can be thought of as the type-level version of the Y combinator.

Relies on a different kind of `map` that is interested in the tail/children.

Map over everything and recall itself until it hits the fixed point of the functor.

Operates bottom-up.

Reduce is an implementation of a catamorphism:

```js
const reduce = (f, acc, xs) => {
    if (xs.length === 0) { return acc; }
    return reduce(f, f(acc, first(xs)), rest(xs));
}
```

```js
// Cata takes an algebra function and folds the given structure down.
const cata = (f, xs) => f(xs.map(ys => cata(f, ys)));
```

## Paramorphism

Accumulator function gets reference to the rest of the list

```js
const para = (f, acc, xs) => {
    if (xs.length === 0) { return acc; }
    return para(f, f(acc, first(xs), xs), rest(xs));
}
```

## References

- A Million Ways to Fold in JS
  - [video](http://forwardjs.com/university/a-million-ways-to-fold-in-js)
  - [slides](http://www.slideshare.net/drboolean/millionways)
  - [GitHub](https://github.com/DrBoolean/RecursionTalk)
