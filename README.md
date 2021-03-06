<img align="right" src="https://avatars0.githubusercontent.com/u/21360882?v=3&s=200">

# Hareactive

A pure FRP library for JavaScript and TypeScript with the following
features/goals:

* Simple and precise semantics similar to classic FRP. This makes the
  library simpler. (the semantics are WIP see [here](./semantics.md))
* Support for continuous time for performant and expressive
  declaration of time-dependent behavior and motions.
* Splendid performance.
* Support for declarative of side-effects in a way that is pure,
  testable and utilizes FRP for powerful handling of asynchronous
  operations.

[![Build Status](https://travis-ci.org/Funkia/hareactive.svg?branch=master)](https://travis-ci.org/Funkia/hareactive)
[![codecov](https://codecov.io/gh/Funkia/hareactive/branch/master/graph/badge.svg)](https://codecov.io/gh/Funkia/hareactive)

## Contributing

Install dependencies.
```
npm install
```

Run tests.
```
npm test
```
Running the tests will generate an HTML coverage report in `./coverage/`.

Continuously run the tests with
```
npm run test-watch
```

## Benchmark

Get set up to running the benchmarks:

```
npm run build
./benchmark/prepare-benchmarks.sh
```

Run a single benchmark with:
```
node benchmark/<name-of-benchmark>
```

Example
```
node benchmark/scan.suite
```

To run all benchmarks:
```
npm run bench
```

## Documentation

Hareactive contains four key concepts: Future, stream, behavior and
now. These are explained below.

### Future

A future is a value belonging to a certain point in time. For
instance, the result of a http request could be a future since it
occurs at a specific time (when the response is received) and contains
a value (the response itself).

Future has much in common with JavaScript's Promises. However, they
are simpler, because a future has no notion of resolution or
rejection. That is a specific future can be understood simply as a
time and a value. Something like this:

```js
{time: 22, value: "Foo"}
```

A promise however contains a third value:

```js
{resolvedOrRejected: "resolved", time: 22, value: "foo"}
```

### Stream

A `Stream` is a list of futures. That is, a list of values where the
values are each associated with a point in time.

An example could be a stream of keypresses that a user makes. Each
keypress happens at a specific moment in time and has an associated
with the specific key that was pressed.

The relationship between `Future` and `Stream` is the same as the
relationship between having a variable that is a string and a variable
that is a list of strings. You wouldn't store a username as `["foo"]`
because there is always exactly one username.

Similarly in Hareactive we don't use `Stream` to express the result of
a http request since a http request only delivers a response exactly
once. Use a `Future` for things where there is exactly one occurrence
and `Stream` where there may be zero or more.

### Behavior

A behavior represents a value that changes over time. For instance,
the current position of the mouse or the value of an input field is a
behavior.

The difference between a stream and a behavior is that a behavior has
a value at all points in time where a stream is a series of events
that happens at specific moments in time.

### Future, stream or behavior?

At first the difference between the three things may be tricky to
understand. Especially if you're used to other libraries where all the
three are represented as a single structure called "stream" or
"observable". The key is to understand that the three types represent
things that are fundamentally different. And that expressing
expressing different things with different structures is beneficial.

You could forget about future and use a stream where you'd otherwise
use a future. Because stream is more powerful than future. In the same
way you could always use arrays of values instead of just single
values. But you don't do that because `username = "foo"` expresses
that only one username exists whereas `username = ["foo"]` gives the
impression that a user can have more than one username. Similarly one
could forget about numbers and just use strings instead. But saying
`amount = 22` is obviously better than `amount = "22"`.

This is how to figure out if a certain thing is a future, a stream or
a behavior:

1. Ask the question: "does the thing always have a current value?". If
   yes, you're done, the thing should be represented as a behavior.
2. Ask the question: "does the thing always happen once?". If yes, the
   thing should be represented as a future. If no, you should use a
   stream.

Below are some examples:

* The time remaining before an alarm goes off: The remaining time
  always have a current value, therefore it is a behavior.
* The moment where the alarm goes off: This has no current value. And
  since the alarm only goes off a single time this is a future.
* User clicking on a specific button: This has no notion of a current
  value. And the user may press the button more than once. This is a
  stream.
* Whether or not a button is currently pressed: This always has a
  current value. The button is always pressed or not pressed. This
  should be represented as a behavior.
* The tenth time a button is pressed: This happens once at a specific
  moment in time. Use a future.

### Now

## API

### Future

#### `Future#listen(o: Consumer<A>): void`

Adds a consumer as listener to a future. If the future has already
occurred the consumer is immediately pushed to.

#### `fromPromise<A>(p: Promise<A>): Future<A>`

Converts a promise to a future.

### Stream

#### `filter<A>(predicate: (a: A) => boolean, s: Stream<A>): Stream<A>`

Returns a stream with all the occurrences from `s` for which
`predicate` returns `true`.

#### `scanS<A, B>(fn: (a: A, b: B) => B, startingValue: B, stream: Stream<A>): Behavior<Stream<B>>`

A stateful scan.

#### `snapshot<B>(b: Behavior<B>, s: Stream<any>): Stream<B>`

Returns a stream that occurs whenever `s` occurs. The value of the
occurrence is `b`s value at the time.

#### `snapshotWith<A, B, C>(f: (a: A, b: B) => C, b: Behavior<B>, s: Stream<A>): Stream<C>`

Returns a stream that occurs whenever `s` occurs. At each occurrence
the value from `s` and the value from `b` is passed to `f` and the
return value is the value of the returned streams occurrence.

#### `switchStream<A>(b: Behavior<Stream<A>>): Stream<A>`

Takes a stream valued behavior and returns a stream that emits values
from the current stream at the behavior. I.e. the returned stream
always "switches" to the current stream at the behavior.

#### `changes<A>(b: Behavior<A>): Stream<A>`

Takes a behavior and returns a stream that has an occurrence whenever
the behavior changes.

#### `combine<A, B>(a: Stream<A>, b: Stream<B>): Stream<(A|B)>`

Combines two streams into a single stream that contains the occurrences
of both `a` and `b`.

#### `isStream(obj: any): boolean`

Returns `true` if `obj` is a stream and `false` otherwise.

### Behavior

#### `when(b: Behavior<boolean>): Behavior<Future<{}>>`

Takes a boolean valued behavior an returns a behavior that at any
point in time contains a future that occurs in the next moment where
`b` is `true`.

#### `snapshot<A>(b: Behavior<A>, f: Future<any>): Behavior<Future<A>>`

Creates a future than on occurence samples the current value of the
behavior and occurs with that value. That is, the original value of
the future is overwritten with the behavior value at the time when the
future occurs.

#### `switcher<A>(init: Behavior<A>, next: Future<Behavior<A>>): Behavior<A>`

From an initial behavior and a future of a behavior `switcher` creates
a new behavior that acts exactly like `initial` until `next` occurs
after which it acts like the behavior it contains.

#### `stepper<B>(initial: B, steps: Stream<B>): Behavior<B>`

Creates a behavior whose value is the last occurrence in the stream.

#### `scan<A, B>(fn: (a: A, b: B) => B, init: B, source: Stream<A>): Behavior<Behavior<B>>`

The returned behavior initially has the initial value, on each
occurrence in `source` the function is applied to the current value of
the behaviour and the value of the occurrence, the returned value
becomes the next value of the behavior.

#### `fromFunction<B>(fn: () => B): Behavior<B>`

This takes an impure function that varies over time and returns a
pull-driven behavior. This is particularly useful if the function is
contionusly changing, like `Date.now`.

#### `isBehavior(b: any): b is Behavior<any>`

Returns `true` if `b` is a behavior and `false` otherwise.

#### `time: Behavior<Time>`

A behavior whose value is the number of milliseconds elapsed win UNIX
epoch. I.e. its current value is equal to the value got by calling
`Date.now`.

#### `timeFrom: Behavior<Behavior<Time>>`

A behavior giving access to continous time. When sampled the outer
behavior gives a behavior with values that contain the difference
between the current sample time and the time at which the outer
behavior was sampled.

### Now

The Now monad represents a computation that takes place in a given
moment and where the moment will always be now when the computation is
run.

#### `async<A>(comp: IO<A>): Now<Future<A>>`

Run an asynchronous IO action and return a future in the Now monad
that resolves with the eventual result of the IO action once it
completes. This function is what allows the Now monad to execute
imperative actions in a way that is pure and integrated with FRP.

#### `sample<A>(b: Behavior<A>): Now<A>`

Returns the current value of a behavior in the Now monad. This is
possible because computations in the Now monad have an associated
point in time.

#### `performStream<A>(s: Stream<IO<A>>): Now<Stream<A>>`

Takes a stream of `IO` actions and return a stream in a now
computation. When run the now computation executes each `IO` action
and delivers their result into the created stream.

#### `plan<A>(future: Future<Now<A>>): Now<Future<A>>`

Convert a future now computation into a now computation of a future.
This function is what allows a Now-computation to reach beyond the
current moment that it is running in.

#### `runNow<A>(now: Now<Future<A>>): Promise<A>`

Run the given Now-computation. The returned promise resolves once the
future that is the result of running the now computation occurs. This
is an impure function and should not be used in normal application
code.
