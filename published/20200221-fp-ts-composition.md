---
slug: "fp-ts-composition"
date: "2020-02-21"
title: "Composition in fp-ts"
---

Preface: All functions should strive to be pure, meaning they perform no side effects - all this means is that you take some data and return some data, and you don’t do anything outside the confines of said data. This post is written with this context in mind.

The most fundamental aspect of FP that brings it all together is the idea of function composition - composing functions together like LEGO. Firstly, let’s look at it quickly in abstract:

```haskell
ab :: a -> b

bc :: b -> c

ac :: a -> c
ac = bc . ab
```

For the sake of understanding composition, we don’t need to think about what specifically `a`, `b`, and `c` represent, they could be any concrete type. Think abstract, think algebraic, think generic.

We have a function from `a` to `b` (`a -> b`), and a second function from `b` to `c` (`b -> c`) (implementations irrelevant). We then _compose_ these two functions together, creating a third new function from `a` to `c` (`a -> c`) (Haskell’s function composition operator, `.`, is right-to-left - Haskell is largely derived from mathematics).

Here’s how we’d write this in vanilla JavaScript without composition:

```javascript
const ab = (a) => /* impl returns b */;

const bc = (b) => /* impl returns c */;

const ac = (a) => bc(ab(a));
```

This works, but it’s a bit hard to read with the nesting. And what happens if we add a third intermediary function `c -> d`? It becomes yet worse. This is where composition can come into play. Here’s what `ac` looks like rewritten with the [flow](https://gcanti.github.io/fp-ts/modules/function.ts.html#flow) composition function from [fp-ts](https://gcanti.github.io/fp-ts/) (thankfully left-to-right!):

```typescript
const ac = flow(ab, bc);
```

As alluded to this is far friendlier to extension, and it is - once you understand how composition works - objectively far more readable; you can read it from left to right like prose.

Here’s precisely what happens when we call `ac`: We pass it an `a` as an argument, and `flow` passes that onto `ab`. `ab` returns a `b`, which is then passed onto the next function in the list, `bc`. `bc` returns a `c`, and as it’s the final function in the composition, this return value is what we finally get back. So, as described earlier, this function takes an `a` and returns a `c`!

It might also help to think about the composition in terms of its functions: `(a -> b), (b -> c)`

When we compose, we essentially just provide whatever the first function wants and get back whatever the last function returns. In this case, that’s `a` and `c` respectively, hence `a -> c`. Were our composition `(string[] -> number), (number -> string), (string -> boolean)`, then our new function signature having composed these together would be `string[] -> boolean`. This is all `flow` is - it’s for composing functions together.

Let’s move onto a more tangible example and think about how we could improve it with composition. Here, we have an array of numbers, and (for some business logic reason) we want to map over the array, adding five to each number, then doubling it, and then finally converting it to a string. Here’s a barebones, non-functional approach:

```typescript
const xs = [5, 10, 25];
const ys = xs.map(x => String((x + 5) * 2));
```

What’s wrong with this? Well, for starters, I’m finding the logic inside of the map callback really hard to read. I have to pause and think about it for a minute. So let’s improve this by writing some small functions that can encapsulate what’s happening here:

```typescript
const double = (x: number): number => x * 2;
const plus = (x: number) => (y: number): number => x + y;
const toString = (x: number): string => String(x);

const xs = [5, 10, 25];
const ys = xs.map(x => toString(double(plus(5)(x))));
```

We’re not really seeing the fruits of our labour yet, particularly in this trivial example. But, if we introduce function composition, it starts to make some sense and look a lot better than what we had before:

```typescript
const double = (x: number): number => x * 2;
const plus = (x: number) => (y: number): number => x + y;
const toString = (x: number): string => String(x);

const xs = [5, 10, 25];
const ys = xs.map(flow(plus(5), double, toString));
```

We can now read this in words from left-to-right - we plus five, we double, we (convert) to a string. There can be no ambiguity that we take a number, perform these operations as described, and finally get back a string. And, if we want to, we can easily abstract this logic out - it’s just an expression:

```typescript
const double = (x: number): number => x * 2;
const plus = (x: number) => (y: number): number => x + y;
const toString = (x: number): string => String(x);

const myBusinessLogic = flow(plus(5), double, toString);

const xs = [5, 10, 25];
const ys = xs.map(myBusinessLogic);
```

This readability delta over the imperative approach only increases as you add extra steps and/or extra complexity (read: real-world business logic).

Anyway, why am I harping on about readability so much? Why am I obsessed with type safety? Why have I been naturally drawn to pure functional programming?

In my opinion, our minds are quite limited in what they can think about at any given time, and the extent to which they can think about anything infallibly. Writing code in this style allows you to take mental shortcuts as a developer that you can’t take with imperative, mutable code. It also provides inherent guardrails against common sources of bugs, be they type or logic-related (and where possible encoding logic errors into the type system, hence the existence of types like `NonEmptyArray`).

Back to TypeScript, there is a second type that fp-ts provides for function composition that is only subtly different from `flow` - [pipe](https://gcanti.github.io/fp-ts/modules/pipeable.ts.html#pipe). This function is identical to `flow`, except that it takes the value to be piped through our composition immediately, whereas `flow` took it afterwards.

Whilst `flow` made sense for the above example we wrote, let’s rewrite it with `pipe` so that we can build an intuition for the difference between them:

```typescript
xs.map((x) => pipe(x, plus(5), double, toString))
```

If it’s still not quite clicking, here’s flow again, also rewritten more explicitly/verbosely:

```typescript
xs.map(flow(plus(5), double, toString))

xs.map((x) => flow(plus(5), double, toString)(x))
```

So, whereas `flow` creates a new composed function that takes its first argument after-the-fact, `pipe` needs to be provided with said argument immediately as its first argument before proceeding through the pipeline.

I hope this explanation helped. As with many things, I think sitting down for an hour and just playing around with it is sometimes to best way to learn something new, but that differs from person to person. Please comment if there’s anything that remains unclear.

Oh, and as of time of writing, there is a [stage 1 proposal](https://github.com/tc39/proposal-pipeline-operator) for a pipeline operator in JavaScript that'd effectively replace our usage of `pipe`. Yay!

