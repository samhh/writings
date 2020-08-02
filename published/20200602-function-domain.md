Functional programming is, funnily enough, all about functions. As such, it's good to refine how we write them. This post is all about the domains of our functions.

## What's the domain?

A function's domain is the set of all its possible argument values, for example:

```haskell
exclaim :: String -> String
exclaim x = x <> "!"
```

```typescript
const exclaim = (x: string): string => x + '!';
```

The domain of this function is a set of all possible strings. In other words, the input `x` could be any possible string; there are no constraints on this argument except the string type.

## Totality

Can you spot the problem with this function?

```haskell
head :: [a] -> a
head (x:_) = x
```

```typescript
const head = <A>(xs: A[]): A => xs[0];
```

It's _partial_, meaning its implementation is not defined for all possible inputs, specifically in the case of an empty list.

One way in which we can address this is to return `Maybe`/`Option`, thus making our function _total_ i.e. non-partial:

```haskell
head :: [a] -> Maybe a
head (x:_) = Just x
head _     = Nothing
```

```typescript
const head = <A>(xs: A[]): Option<A> => O.fromNullable(xs[0]);
```

Really, totality means only ensuring that you've handled all possible inputs such that no input could cause the function to throw or otherwise unexpectedly fail.

Note that property-based testing would be a good way to catch edge-case bugs such as that in our first, naive implementation, and could be used to ensure that this implementation isn't somehow flawed for some input we haven't considered.

## Constrained inputs

There is an alternative technique we can employ however, one that's often preferable. That is to limit our function's domain itself through the use of more restrictive types:

```haskell
head :: NonEmpty a -> a
head (x:|_) = x
```

```typescript
const head = <A>(xs: NonEmptyArray<A>): A => xs[0];
```

By constraining our function's domain we've been able to define a safe head function that's maximally ergonomic. If someone doesn't already have a decidedly non-empty list, they can _maybe_ make one and be no worse off than before. On the other hand, if the consumer's list is definitely non-empty, then they no longer have to deal with a false notion of nullability. Additionally, it's simplified our function implementation.

If you ever catch yourself widening your output type to satisfy a bad input, that's a sign that you might need to constrain your domain. A very common mistake from beginners to the `Maybe` type is to discover that they need to manipulate it in their business logic path, and write a function of type `Maybe a -> Maybe b`. Never do this! This is what functors are for.

## Codomain

To round up this post, a quick mention that just as `a` in `a -> b` is the domain, `b` is the _codomain_.

