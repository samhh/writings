---
slug: "monoids-semigroups"
date: "2020-08-30"
title: "Monoids (and semigroups)"
---

This post contains examples written in Haskell and TypeScript ([fp-ts](https://gcanti.github.io/fp-ts/modules/)).

[Semigroups](https://en.wikipedia.org/wiki/Semigroup) and [monoids](https://en.wikipedia.org/wiki/Monoid) are mathematical structures that capture a very common programmatic operation, the reduction of multiple elements into one. Formalisms like this enable us to create and utilise otherwise unobtainable abstractions, and signal to other developers our intent with common language.

But, to understand monoids, we must first begin with semigroups.

## Concatenation

Semigroups formally define how to concatenate together two items of the same type. For example, arrays are semigroups:

```haskell
[1, 2] <> [3, 4] -- [1, 2, 3, 4]
```

```typescript
[1, 2].concat([3, 4]) // [1, 2, 3, 4]
```

You could equally define a semigroup instance for numbers under addition and multiplication, or for booleans under conjunction and disjunction. If the underlying mathematics interest you, you can read more about semigroups on [Wikipedia](https://en.wikipedia.org/wiki/Semigroup).

What’s more interesting is the ability to define semigroups for arbitrary types that we’ve defined. Let's imagine we have the type `Cocktail`, and we want to be able to combine any two of them together. Given a definition for the type as follows:

```haskell
data Cocktail = Cocktail
    { name :: String
    , ingredients :: [String]
    }
```

```typescript
type Cocktail = {
    name: string;
    ingredients: string[];
};
```

We can then define a formal semigroup instance which will allow us to combine any pair of cocktails together:

```haskell
instance Semigroup Cocktail where
    a <> b = Cocktail (name a <> " " <> name b) (ingredients a <> ingredients b)

mojito = Cocktail "Mojito" ["rum", "mint"]
robroy = Cocktail "Rob Roy" ["scotch", "bitters"]

combined = mojito <> robroy
-- Cocktail { name = "Mojito Rob Roy", ingredients = ["rum", "mint", "scotch", "bitters"] }
```

```typescript
const semigroupCocktail: Semigroup<Cocktail> = {
    concat: (a, b) => ({
        name: a.name + ' ' + b.name,
        ingredients: a.ingredients.concat(b.ingredients),
    }),
};

const mojito: Cocktail = { name: 'Mojito', ingredients: ['rum', 'mint'] };
const robroy: Cocktail = { name: 'Rob Roy', ingredients: ['scotch', 'bitters'] };

// { name: 'Mojito Rob Roy', ingredients: ['rum', 'mint', 'scotch', 'bitters'] }
const combined = semigroupCocktail.concat(mojito, robroy);
```

And just like that we can combine our types together! Of course, we could just define a function for concatenating these two particular types together without the semigroup formalism, but that’d be both less obvious at a glance to developers familiar with this mathematical terminology, and less likely to be reusable in common abstractions; you’d have to rely upon duck typing and couldn’t rely upon adherence to any laws. Speaking of which, utilising the semigroup type class implies adhering to the law of [associativity](https://en.wikipedia.org/wiki/Associative_property), something common in mathematical and functional abstractions that allows the consumer to make useful assumptions.

_Fun fact_: If your (attempted) semigroup instance isn’t associative, then it is in actual fact a [magma](https://en.wikipedia.org/wiki/Magma_(algebra))! Although it does exist in the type class hierarchy, it’s exceptionally rare to define a magma that’s not associative thus not also a semigroup.

## Identity

We now know how to combine things, but what about if we mightn’t have anything? For such circumstances we can define a monoid, which is simply a semigroup with an [identity element](https://en.wikipedia.org/wiki/Identity_element).

An identity element is a value for a type which, combined with another of its type, results in no change to said other. For example, the identity for strings is `''`, for numbers under addition `0`, for numbers under multiplication `1`, for booleans under conjunction `true`, and so on.

The identity element for arrays is an empty array. See how when we apply this identity element to any given array we always get said array back again unchanged:

```haskell
[1, 2] <> [] -- [1, 2]
[] <> [1, 2] -- [1, 2]
```

```typescript
[1, 2].concat([]) // [1, 2]
[].concat([1, 2]) // [1, 2]
```

What about our `Cocktail` type from earlier? Given the two fields are each already monoids, this will be quite simple:

```haskell
instance Monoid Cocktail where
    mempty = Cocktail mempty mempty
```

```typescript
const monoidCocktail: Monoid<Cocktail> = getStructMonoid({
    name: monoidString,
    ingredients: A.getMonoid<string>(),
});
```

And just like this we can combine any arbitrary number of cocktails:

```haskell
mconcat []               -- Cocktail { name = "", ingredients = [] }
mconcat [mojito]         -- Cocktail { name = "Mojito", ingredients = ["rum", "mint"] }
mconcat [mojito, robroy] -- Cocktail { name = "Mojito Rob Roy", ingredients = ["rum", "mint", "scotch", "bitters"] }
```

```typescript
fold(monoidCocktail)([])               // { name: '', ingredients: [] }
fold(monoidCocktail)([mojito])         // { name: 'Mojito', ingredients: ['rum', 'mint'] }
fold(monoidCocktail)([mojito, robroy]) // { name: 'Mojito Rob Roy', ingredients: ['rum', 'mint', 'scotch', 'bitters'] }
```

This is equivalent to reducing over an array of items using the semigroup concatenation operation as the function and the monoidal identity element as the starting value.

