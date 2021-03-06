# API

| Function | Signature |
| -------- | --------- |
| [`all`](#all) | `[Promise a] -> Promise [a]` |
| [`backoff`](#backoff) | `Number -> Number -> (a... -> Promise b) -> a... -> Promise b` |
| [`convergeP`](#convergep) | `(b -> c -> Promise d) -> [(a -> Promise b), (a -> Promise c)] -> a -> Promise d` |
| [`copyProp`](#copyprop) | `String -> String -> { k: v } -> { k: v }` |
| [`evolveP`](#evolvep) | `{ k: (v -> Promise v) } -> { k: v } -> Promise { k: v }` |
| [`juxtP`](#juxtp) | `[a... -> Promise b] -> a... -> Promise [b]` |
| [`mapP`](#mapp) | `Functor f => (a -> Promise b) -> f a -> Promise f b` |
| [`move`](#move) | `Number -> Number -> [a] -> [a]` |
| [`normalizeBy`](#normalizeby) | `String -> [{ k: v }] -> { v: { k: v } }` |
| [`promisify`](#promisify) | `((a..., b -> ()) -> (), c) -> a... -> Promise b` |
| [`reject`](#reject) | `a -> Promise Error` |
| [`rename`](#rename) | `String -> String -> { k: v } -> { k: v }` |
| [`resolve`](#resolve) | `a -> Promise a` |
| [`tapP`](#tapp) | `(a -> Promise b) -> a -> Promise a` |
| [`validate`](#validate) | `Schema -> a -> Promise a` |

### all

```haskell
all : [Promise a] -> Promise [a]
```

Returns a single `Promise` that resolves when all of the promises in the list have resolved or when the iterable argument contains no promises. It rejects with the reason of the first promise that rejects.  Just a bound version of [`Promise.all`](http://devdocs.io/javascript/global_objects/promise/all).

See also [`reject`](#reject), [`resolve`](#resolve).

```js
all([ Promise.resolve('a') ]) //=> Promise ['a']
```

### backoff

```haskell
backoff : Number -> Number -> (a... -> Promise b) -> a... -> Promise b
```

Accepts a `base` delay in ms and max `tries`, and then wraps an async function with a [full jitter exponential backoff](https://www.awsarchitectureblog.com/2015/03/backoff.html) algorithm.  Useful for recovering from intermittent network failures.  Will retry for all caught errors until the number of `tries` is reached.

```js
const fetchImage = opts => { /* async, and might fail sometimes */ }

backoff(250, 5, fetchImage) //=> a new function that tries at most 5 times before rejecting
```

### convergeP

```haskell
convergeP : (b -> c -> Promise d) -> [(a -> Promise b), (a -> Promise c)] -> a -> Promise d
```

An async version of [`R.converge`](http://devdocs.io/ramda/index#converge) that accepts Promise-returning branching and converging functions.

Accepts a converging function and a list of branching async functions and returns a new async function. When invoked, this new function is applied to some arguments, and each branching function is applied to those same arguments. The resolved values of each branching function are passed as arguments to the converging function to produce the resolved value.

See also [`juxtP`](#juxtp).

```js
const getCourse = convergeP(assoc('course'), [ fetchByCourseId, identity ])

const addCourseLesson = composeP(addLesson, getCourse)
```

### copyProp

```haskell
copyProp : String -> String -> { k: v } -> { k: v }
```

Quickly copy one property on an object to another key.

See also [`rename`](#rename).

```js
copyProp('id', 'courseId', { id: 'abc' }) //=> { id: 'abc', courseId: 'abc' }
```

### evolveP

```haskell
evolveP : { k: (v -> Promise v) } -> { k: v } -> Promise { k: v }
```

An async version of [`R.evolve`](http://devdocs.io/ramda/index#evolve) that accepts Promise-returning transformation functions.

Creates a new object by recursively evolving a shallow copy of an object, according to the transformation functions. All non-primitive properties are copied by reference.  A transformation function will not be invoked if its corresponding key does not exist in the evolved object.

See also [`mapP`](#mapp).

```js
evolveP({ author: getProfile }, { author: 'abc' }) // Promise { author: { name: 'joey', ... } }
```

### juxtP

```haskell
juxtP : [a... -> Promise b] -> a... -> Promise [b]
```

An async version of [`R.juxt`](http://devdocs.io/ramda/index#juxt) that accepts Promise-returning branching functions.

Applies a list of functions to some values, and resolves with a list of their resolved values.

See also [`convergeP`](#convergep).

```js
const deleteCourseAndLessons = juxtP([ deleteCourse, deleteLessons ])
```

### mapP

```haskell
mapP : (a -> Promise b) -> [a] -> Promise [b]
```

An async version of [`R.map`]() that accepts a Promise-returning function.

Takes an async function and a list, applies the function to each of the list's values, and resolves with a list of the resolved values.

See also [`evolveP`](#evolvep).

```js
mapP(getProfile, ['abc','def']) //=> Promise [{ name: 'joey' }, { name: 'fella' }]
```

### move

```haskell
move : Number -> Number -> [a] -> [a]
```

Moves a list item from one position to another.

```js
move(3, 1, ['a','b','c','d']) //=> ['a','d','b','c']
```

### normalizeBy

```haskell
normalizeBy : String -> [{ k: v }] -> { v: { k: v } }
```

[Normalizes a list](http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html) by building an object, with the IDs of the list items as keys and the items themselves as the values.  List items will be normalized by the specified key, so make sure it is unique.

```js
normalizeBy('uid', [{ uid: 'abc' }, { uid: 'def' }]) //=> { abc: { uid: 'abc' }, def: { uid: 'def' }}
```

### promisify

```haskell
promisify : ((a..., b -> ()) -> (), c) -> a... -> Promise b
```

Takes a function which accepts a node-style callback and returns a new function that returns a `Promise` instead.  Will also bind it to an optional context object.

```js
const upload = promisify(s3.upload, s3)
```

### reject

```haskell
reject : a -> Promise Error
```

Returns a `Promise` object that is rejected with the given reason.  A bound version of [`Promise.reject`](http://devdocs.io/javascript/global_objects/promise/reject), but also wraps non-errors with `Error` for a consistent interface.

See also [`all`](#all), [`resolve`](#resolve).

```js
reject(new Error('bad guy')) //=> Promise Error('bad guy')
reject('bad guy')            //=> Promise Error('bad guy')
```

### rename

```haskell
rename : String -> String -> { k: v } -> { k: v }
```

Easily rename a property on an object to be a different key.

See also [`copyProp`](#copyprop).

```js
rename('id', 'courseId', { id: 'abc' }) //=> { courseId: 'abc' }
```

### resolve

```haskell
resolve : a -> Promise a
```

Lifts a value into a `Promise`.  Just a bound version of [`Promise.resolve`](http://devdocs.io/javascript/global_objects/promise/resolve).

See also [`all`](#all), [`reject`](#reject).

```js
resolve('a') //=> Promise 'a'
```

### tapP

```haskell
tapP : (a -> Promise b) -> a -> Promise a
```

An async version of [`R.tap`](http://devdocs.io/ramda/index#tap) that accepts a Promise-returning function.

Runs the given function with the supplied value, and then resolves with that value.

```js
tapP(a => Promise.resolve('b'), 'a') //=> Promise 'a'
```

### validate

```haskell
validate : Schema -> a -> Promise a
```

Validates a value against a [`Joi`](https://github.com/hapijs/joi) schema.  Curried and promisified for ease of use.

```js
const schema = Joi.object({
  id: Joi.string().required()
})

validate(schema, { id: 'abc' }) //=> Promise { id: 'abc' }
validate(schema, { id: 123 })   //=> Promise ValidationError
```
