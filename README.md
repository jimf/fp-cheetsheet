# Functional Programming Cheatsheet

Functional programming cheetsheet, oriented toward JavaScript.

This is an ongoing work in progress.

## Table of Contents
- [Method and Operator Glossary](#method-and-operator-glossary)
  - [`<>`](#)
  - [`>>=`](#)
  - [`chain`](#chain)
  - [`concat`](#concat)
  - [Endomorphism](#endomorphism)
  - [`flatMap`](#flatmap)
  - [`mappend`](#mappend)
  - [`mbind`](#mbind)
- [JavaScript FP References and Libraries](#javascript-fp-references-and-libraries)

## Method and Operator Glossary

#### `<>`

Haskell infix alias for `mappend`. See `concat`.

#### `>>=`

Haskell [`chain`](#chain) operator.

#### `chain`

Conceptually, `chain` is `map` followed by a "flatten" operation. More
concretely, if operating with arrays, `chain` would map the given
function over the data set, and rather than return an array of arrays,
would instead join all those arrays together into a single array. Chain is
useful when working with Monads to flatten out nesting. For example, to
return `Just(42)` rather than `Just(Just(42))`.

Aliases: [`>>=`](#), [`flatMap`](#flatMap), [`mbind`](#mbind)

#### `concat`

```js
const concat = (a, b) => a.concat(b);
```

Append one value to another.

Aliases: [`<>`](#), [`mappend`](#mappend)

#### Endomorphism

A function whose input and output are the same type.

#### `flatMap`

See [`chain`](#chain).

#### `mappend`

Haskell append operation. See [`concat`](#concat).

#### `mbind`

See [`chain`](#chain).

## JavaScript FP References and Libraries

- [Fantasy Land Specification](https://github.com/fantasyland/fantasy-land)
- [pointfree-fantasy](https://github.com/DrBoolean/pointfree-fantasy)
- [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://www.gitbook.com/book/drboolean/mostly-adequate-guide/details)
- [Ramda](http://ramdajs.com/0.19.1/index.html)
- [ramda-fantasy](https://github.com/ramda/ramda-fantasy)
- [Sanctuary](https://github.com/plaid/sanctuary)

## License

MIT
