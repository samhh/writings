If you're looking into functional programming for the first time, the terminology can be really overwhelming. I think one of the easiest ways to learn is to try and map the terms to concepts that you likely already know and then branch out from there.

All of these terms have _laws_ that express limitations which ensure that all instances behave reasonably. We shan't go over them here, but it's good to know that - even if we're not ready to look into them yet - they exist, that there is a rich mathematical backing to these concepts. If this piques your curiosity at all, the best resource is probably [Typeclassopedia on HaskellWiki](https://wiki.haskell.org/Typeclassopedia).

All of these examples will be written in both Haskell and TypeScript. The latter will be written with the [fp-ts](https://gcanti.github.io/fp-ts) library.

For some reason, different languages sometimes call the same concepts different things. For example, Haskell has the `Maybe` type, whilst Rust and fp-ts have the identical `Option` type. Equally, Haskell and fp-ts have the `Either` type, whilst Rust has opted to call it `Result`. Don't let this discrepancy throw you off, they're otherwise identical.

Without any further ado let's get started!

## Functor

A functor is some sort of container that allows you to map its contents. Arrays are the prototypical functor:

```haskell
(*2) <$> [1, 2, 3] -- [2, 4, 6]
```

```typescript
[1, 2, 3].map(x => x * 2) // [2, 4, 6]
```

Here we've taken each item in our array and applied our function to it. The very same concept applies to types like `Option`:

```haskell
(*2) <$> (Just 5) -- Just 10
(*2) <$> Nothing  -- Nothing
```

```typescript
option.map(some(5), x => x * 2) // Some 10
option.map(none, x => x * 2)    // None
```

If the value is `Some`, then we map the inner value, else if it's `None` then we short-circuit and essentially do nothing.

There's nothing that technically says that functors have to map over `Some` in the case of `Option`, or `Right` in the case of `Either`, except it's universally expected behavior and to do otherwise would be very odd.

## Bifunctor

For types with (at least) two variants which you might want to map, for example tuples, or `Either` with its `Left` and `Right` variants, there is the concept of a _bifunctor_. This is the very same as functor, except as the name implies you can map "the other side" as well:

```haskell
first (*2) (Left 5)   -- Left 10
first (*2) (Right 5)  -- Right 5
second (*2) (Left 5)  -- Left 5
second (*2) (Right 5) -- Right 10
```

```typescript
either.mapLeft(left(5), x => x * 2)  // Left 10
either.mapLeft(right(5), x => x * 2) // Right 5
either.map(left(5), x => x * 2)      // Left 5
either.map(right(5), x => x * 2)     // Right 10
```

## Monad

Ah, the scary sounding one, the monad! Monads build atop functors with one important addition, the idea of _joining_ or flattening. As with the functor, we'll start by demonstrating how arrays are also monads:

```haskell
join [[1, 2], [3, 4]] -- [1, 2, 3, 4]
```

```typescript
[[1, 2], [3, 4]].flat() // [1, 2, 3, 4]
```

And likewise with nested `Option`s:

```haskell
join (Just (Just 5))  -- Just 5
join (Just (Nothing)) -- Nothing
join Nothing          -- Nothing
```

With this newfound ability to flatten things, we can now also _bind_ or chain things.

Let's imagine we have a function `parse` which takes a `string`, tries to parse it as a `number`, and returns `Option<number>`, and to start with we have an `Option<string>`. Thus far, the only way we could make this work would be to map with a functor, giving us back `Option<Option<number>>`, and then join down to `Option<number>`. That works, but is a bit tedious and we can imagine needing to perform this combination of operations quite often.

This is what bind is for!

```haskell
Just "5" >>= parse -- Just 5
Just "x" >>= parse -- Nothing
Nothing  >>= parse -- Nothing
```

```typescript
option.chain(some('5'), parse) // Some 5
option.chain(some('x'), parse) // None
option.chain(none, parse)      // None
```

What else do we know in JavaScript-land which is monad-like? The promise! A promise is - imprecisely - a functor, a bifunctor, and a monad, among other things. When we `.then`, we're either functor mapping or monad binding depending upon whether we're returning another promise (JavaScript handles this implicitly), and when we `.catch` we're either bifunctor mapping or sort of monad binding over the left side. Promises aren't _really_ monads because of these slightly different behaviours, but they absolutely are analagous.

Further, async/await is like a specialised form of Haskell's [do notation](https://wiki.haskell.org/Typeclassopedia#do_notation). In this example in Haskell, `IO` is just another monad, but _any_ monad supports this syntax:

```haskell
f :: String -> IO Int
f x = do
    res <- getData x
    res * 2
```

```typescript
const f = async (x: string): Promise<number> => {
    const res = await getData(x);
    return res * 2;
};
```

Before we move on, if you were wondering why JavaScript's promise isn't a proper functor or monad, here's the legacy of that unfortunate decision:

{% github https://github.com/promises-aplus/promises-spec/issues/94#issuecomment-16176966 %}

It hasn't aged particularly well. This also happens to be whence the [fantasy-land](https://github.com/fantasyland/fantasy-land) specification derived its name.

## Semigroup

Semigroups define how to concatenate two items of the same type. For example, arrays are semigroups:

```haskell
[1, 2] <> [3, 4] -- [1, 2, 3, 4]
```

```typescript
[1, 2].concat([3, 4]) // [1, 2, 3, 4]
```

You could equally define a semigroup instance for numbers under addition and multiplication, or for booleans under conjunction and disjunction. If the underlying mathematics interest you, you can read more about semigroups on [Wikipedia](https://en.wikipedia.org/wiki/Semigroup).

We can also define semigroups for arbitrary types! Let's imagine we have the type `Cocktail`, and we want to be able to combine any two of them together. Given a definition for the type as follows:

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

combined = mojito <> robroy -- Cocktail { name = "Mojito Rob Roy", ingredients = ["rum", "mint", "scotch", "bitters"] }
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

const combined = semigroupCocktail.concat(mojito, robroy); // { name: 'Mojito Rob Roy', ingredients: ['rum', 'mint', 'scotch', 'bitters'] }
```

## Monoid

Like how the monad derives most of its abilities from the functor, as does the monoid from the semigroup. A monoid is a semigroup with one extra thing - an _identity_ element, which means essentially a sort of "default" element which, when concatenated with others of its type, will result in the same output.

Here are some example identity elements in mathematics:

- Addition/subtraction: `0`, `5 + 0 == 5` & `5 - 0 == 5`
- Multiplication/division: `1`, `5 * 1 == 5` & `5 / 1 == 5`

See how when we apply the identity element to an operation alongside _n_ we always get said _n_ back again. We can do the same thing with types when we're programming. Once again, let's start with arrays:

```haskell
[1, 2] <> [] -- [1, 2]
```

```typescript
[1, 2].concat([]) // [1, 2]
```

If we concatenate an empty array with any other array, we'll get said other array back. The same goes for strings which can be thought of conceptually as arrays of characters, which happens to be exactly what they are in Haskell.

What about our `Cocktail` type from earlier? Given the two fields are each already monoids, or easy to treat as monoids - a string and an array - this will be quite simple:

```haskell
instance Monoid Cocktail where
    mempty = Cocktail mempty mempty
```

```typescript
const monoidCocktail: Monoid<Cocktail> = {
    ...semigroupCocktail,
    empty: { name: '', ingredients: [] },
};
```

This is cool, but truth be told it's relatively rare that we need to concatenate only two items of an arbitrary type. What I find myself wanting to do far more regularly is fold over an array of said items, which is trivially possible using our monoid instance. Here we'll just fold over small arrays, but this can work for arrays of any size at all:

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

## Sequence

Here's one that's super useful but you mightn't have heard of. Sequencing is the act of inverting the relationship between two types:

```haskell
sequenceA [Just 5, Just 10] -- Just [5, 10]
sequenceA [Just 5, Nothing] -- Nothing
```
```typescript
const seqOptArr = array.sequence(option);

seqOptArr([some(5), some(10)]) // some([5, 10])
seqOptArr([some(5), none])     // none
```

This is something you've probably done plenty of times but never knew that this is what it was - this is what you're doing when you call `Promise.all` in JavaScript! Think in terms of types: We take an array of promises, and we convert it to a promise of an array. We inverted the relationship or, as we now know to call it, we sequenced!

As with `Promise.all`, the sequence will short-circuit to the fail-case if anything fails.

## Traverse

Hot on the heels of sequencing is traversal, which is essentially just a combination of sequencing with a functor map after-the-fact. You'll find that operations which are very common like this often have functions predefined in the likes of Haskell.

```haskell
traverse (fmap (*2)) [Just 5, Just 10] -- Just [10, 20]
traverse (fmap (*2)) [Just 5, Nothing] -- Nothing
```

```typescript
const traverseOptArr = array.traverse(option);

traverseOptArr([some(5), some(10)], option.map(x => x * 2)) // some([10, 20])
traverseOptArr([some(5), none],     option.map(x => x * 2)) // none
```

Just like with sequencing, this will short-circuit if the type we're inverting is already in its failure state.

---

I hope that's helpful to some of you reading. Let me know in the comments if there's anything in particular you'd like to see addressed that's missing.

