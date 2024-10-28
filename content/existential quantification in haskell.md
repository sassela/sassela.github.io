---
date: 2021-03-16
updated: 2023-12-26
state: 🪴
---
Existential quantification is a way of encapsulating type information so that different data types can be handled in a uniform way based on their common properties.

You will need to have the `ExistentialQuantification` [GHC extension enabled](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/guide-to-ghc-extensions/how-to-enable-extensions) to run the following examples.

Let’s look at an example in [GHC base package](https://hackage.haskell.org/package/base):

```haskell
data Opaque =
  forall a. O a
```

`Opaque` is a data type that encapsulates the existential type, `a`. `O` is the constructor function of `Opaque`, which allows us to create values of type `Opaque`.

In Haskell, the typical indicator of an existential type is `forall a.` on the right-, rather than the left-, hand side of the data type declaration.

## encapsulation

What do we mean when we talk about *encapsulation*? One or both of two things: 

1. restricting direct access to some components of a data type. 
2. exposing functions that operate on that data type through a common interface.

### restricting direct access

To illustrate the first concept, let’s create values of the `Opaque` type:

```haskell
x :: Opaque
x = O (1 :: Int)

y :: Opaque
y = O "a"

z :: Opaque
z = O True
```

What exactly can we do with values `x`, `y` or `z`?

We can’t print them…

```haskell
λ> x

<interactive> error:
    • No instance for (Show Opaque) arising from a use of ‘print’
    • In a stmt of an interactive GHCi command: print it
```

…nor can we extract their underlying values:

```haskell
λ> extract (O value) = value

<interactive> error:
    • Couldn't match expected type ‘p’ with actual type ‘a’
      ‘a’ is a rigid type variable bound by
        a pattern with constructor: O :: forall a. a -> Opaque,
        in an equation for ‘extract’
      ‘p’ is a rigid type variable bound by
        the inferred type of extract :: Opaque -> p
    • In the expression: value
      In an equation for ‘extract’: extract (O value) = value
```

The only use for an existential type such as `Opaque` is to *hide* information about its underlying `Int`, `String` and `Bool` data types, and restrict access to their values.

Another real-world example is `Fold` from the `[foldl](https://hackage.haskell.org/package/foldl)` package:

```haskell
data Fold a b =
  forall x. Fold (x -> a -> x)
                 x
                 (x -> b)
```

In this example, `x` is the existential type. Once a `Fold` is constructed, the type of `x` is discarded.

```haskell
--            Fold (x -> a ->  x)   x  (x -> b)
λ> example1 = Fold (\x   a -> a:x) []    show
λ> :t example1
--                    Fold a   b
example1 :: Show a => Fold a String
```

So, how do we access `x` in this example? We can pattern match on the `Fold` constructor:

```haskell
fold :: Foldable f => Fold a b -> f a -> b
fold (Fold step begin done) as = F.foldr cons done as begin
  where
    cons a k x = k $! step x a
```

Note that we still cannot extract the value of `x` to return it. However, we do not need to know the value of `x` in order to be able to use it in the `fold` function; `x` is an *intermediate value*, which means that it is only ever used internally in `fold` and not exposed in its inputs or outputs. For example:

```haskell
λ> fold example1 [1,2,3]
"[3,2,1]"
-- the intermediate type, Int, is never exposed

λ> example2 = Fold (\x a -> x + (floor a)) 0 even
λ> fold example2 [1.0, 2.5, 3.9]
True
-- the intermediate type, Double, is never exposed
```

### expose common functions

Next, what does it mean for existential types to expose common functions? Let’s look at a different example:

```haskell
data Showable =
  forall a. Show a => S a
```

`Showable` encapsulates an existential type `a`, with a key difference in the added `Show` constraint. This takes us from an `Opaque` type, that we can’t do much with, to one that allows us to treat values of different types in a uniform way:

```haskell
instance Show Showable where
  show (S value) = show value

x' :: Showable
x' = S (1 :: Int)

y' :: Showable
y' = S "a"

z' :: Showable
z' = S True
```

Thanks to the `Show` constraint on the existential type, we can print each `Showable` value…

```haskell
λ> x'
1

λ> show <$> [x', y', z']
["1","\"a\"","True"]
```

…however, we still can’t extract their underlying values and return them:

```haskell
λ> extract (S value) = value

<interactive> error:
    • Couldn't match expected type ‘p’ with actual type ‘a’
      ‘a’ is a rigid type variable bound by
        a pattern with constructor: S :: forall a. Show a => a -> Showable,
        in an equation for ‘extract’
        at <interactive>
      ‘p’ is a rigid type variable bound by
        the inferred type of extract :: Showable -> p
        at <interactive>
    • In the expression: value
      In an equation for ‘extract’: extract (S value) = value
```

That’s O.K. — we don’t ever need to know the underlying types. It’s enough to know that because `x`, `y` and `z` are instances of `Show` we can use a common function, `show` on them.

This example illustrates how constraints allow us to work with existential types despite not knowing anything about the underlying type. We can also see from this example that existential types allow us to work with different (heterogeneous) types in a uniform way.

However, this contrived example isn’t a good use case in practice. You might notice that defining a `Showable` existential is a long-winded way of writing:

```haskell
λ> [show 1, show "a", show True]
["1","\"a\"","True"]
```

A more practical real-world use-case is in Haskell [Exceptions](http://hackage.haskell.org/package/base-4.7.0.1/docs/Control-Exception-Base.html#t:SomeException).

```haskell
data SomeException =
  forall e. Exception e =>
            SomeException e
```

`SomeException` encapsulates an existential type `e` along with an `Exception` constraint. This constraint effectively “packs” several common functions and makes them available to us. For example:

```haskell
toException :: SomeException -> SomeException

fromException :: SomeException -> Maybe SomeException 

displayException :: SomeException -> String
```

This goes further than the previous `Showable` example; we can show exceptions of any type using `displayException`, but we can also define an extensible hierarchy of exceptions with the use of `toException` and `fromException` instances for them. For example:

```haskell
---------------------------------------------------------------------
 -- Make the root exception type for all the exceptions in a compiler

 data SomeCompilerException = forall e . Exception e => SomeCompilerException e

 instance Show SomeCompilerException where
     show (SomeCompilerException e) = show e

 instance Exception SomeCompilerException

 compilerExceptionToException :: Exception e => e -> SomeException
 compilerExceptionToException = toException . SomeCompilerException

 compilerExceptionFromException :: Exception e => SomeException -> Maybe e
 compilerExceptionFromException x = do
     SomeCompilerException a <- fromException x
     cast a

 ---------------------------------------------------------------------
 -- Make a subhierarchy for exceptions in the frontend of the compiler

 data SomeFrontendException = forall e . Exception e => SomeFrontendException e

 instance Show SomeFrontendException where
     show (SomeFrontendException e) = show e

 instance Exception SomeFrontendException where
     toException = compilerExceptionToException
     fromException = compilerExceptionFromException

 frontendExceptionToException :: Exception e => e -> SomeException
 frontendExceptionToException = toException . SomeFrontendException

 frontendExceptionFromException :: Exception e => SomeException -> Maybe e
 frontendExceptionFromException x = do
     SomeFrontendException a <- fromException x
     cast a

 ---------------------------------------------------------------------
 -- Make an exception type for a particular frontend compiler exception

 data MismatchedParentheses = MismatchedParentheses
     deriving Show

 instance Exception MismatchedParentheses where
     toException   = frontendExceptionToException
     fromException = frontendExceptionFromException
```

This is a powerful example of how existentials can allow us to handle different types in a uniform way.

## other examples in Haskell libraries

- The `[tomland` TOML parsing and pretty-printing library](https://github.com/kowainik/tomland/blob/main/src/Toml/Type/AnyValue.hs) uses existential types to give compile-time guarantees about the types of parsed values.
- The `[parser-combinators` library](https://hackage.haskell.org/package/parser-combinators) uses existentials in its implementation of permutation parsers.
- The [ST monad](https://en.wikibooks.org/wiki/Haskell/Existentially_quantified_types#Example:_runST) uses existential types to hide the internal environment of an ST computation.

## summary

As we’ve seen from the above examples, existential types are a way to create opaque data types that expose only the information required to operate on them.

## resources

- [Existential Quantification: Patterns and Antipatterns — Jonathan Fischoff](https://medium.com/@jonathangfischoff/existential-quantification-patterns-and-antipatterns-3b7b683b7d71)
- [24 Days of GHC Extensions: Existential Quantification — Roman Cheplyaka](https://ocharles.org.uk/guest-posts/2014-12-19-existential-quantification.html)
- [Haskell Report Wiki: ExistentialQuantification — Ben Gamari](https://gitlab.haskell.org/haskell/prime/-/wikis/ExistentialQuantification)
- [Haskell Wiki: Existential type](https://wiki.haskell.org/Existential_type)