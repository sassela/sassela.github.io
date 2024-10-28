---
date: 2020-12-18
updated: 2023-12-26
state: 🪴
---
This post is an introduction to number conversion in Haskell; what it is, why it's needed and idioms for converting between different number types.

This post assumes some familiarity with value and type declaration in Haskell, can read type signatures, and are able to start Haskell's interactive environment, GHCi. 

## type conversion

One of the first things you might try when learning a new programming language is to evaluate simple mathematical **expressions**:

```haskell
λ> 0 + 1
1

λ> 1 * 2
2

λ> 2 / 3
0.6666666666666666

λ> 3 + (4/5)
3.8
```

In Haskell, and other **strongly-typed** programming languages, strict restrictions are enforced against mixing values with different **data types.** As you begin to declare more complex expressions using explicit data types, you may bump into type errors rather quickly. 

```haskell
λ> x = 0 :: Int -- define a variable `x` that has the type `Int`

λ> y = 1.0 :: Double -- define a variable `y` that has the type `Double`

λ> x + y

<interactive> error:
    • Couldn't match expected type ‘Int’ with actual type ‘Double’
    • In the second argument of ‘(+)’, namely ‘y’
      In the expression: x + y
      In an equation for ‘it’: it = x + y
```

In order to correct this expression, you'll need to **convert** the type of `x` or `y` to match the type of the other. This is because addition with `+` requires both inputs to be of the same type.

**Type conversion,** less commonly known as type casting, means to explicitly change the type of a value. In Haskell, type conversion is achieved using functions.

We can resolve the above error by either:

1. Converting `Int` to `Double` 
    
    We can convert the value `x` from type `Int` to `Double` using Haskell's `fromIntegral` function:
    
    ```haskell
    λ> fromIntegral x + y
    1.0
    ```
    
    - Disadvantage: [fromIntegral overflow](https://www.reddit.com/r/haskell/comments/i5kqid/cryptonite_fromintegral_overflow_causes_incorrect/)?
2. Converting `Double` to `Int`
    
    Alternatively, we can convert the value `y` from type `Double` to `Int`  using any of the following functions:
    
    ```haskell
    λ> x + round y -- rounds to the nearest integer
    1
    
    λ> x + ceiling y -- `ceiling` rounds up to the nearest integer
    1
    
    λ> x + floor y -- `floor` rounds down to the nearest integer
    1
    
    λ> x + truncate y -- `truncate` rounds towards 0 to the nearest integer
    1
    ```
    
    One disadvantage of using these rounding functions is that they are **lossy** and could lead to rounding errors.
    

## numeric types

Let's look at the type signatures of the above-mentioned functions and explore their numeric types.

`fromIntegral` has the following type signature:

```haskell
Prelude
λ> :t fromIntegral
fromIntegral :: (Integral a, Num b) => a -> b
```

This means that `fromIntegral` will convert from any `Integral` type into any `Num` (Numeric) type.

`ceiling` has the following type signature:

```haskell
Prelude
λ> :t ceiling
ceiling :: (RealFrac a, Integral b) => a -> b
```

This means that each of these functions will convert from any `RealFrac` type into any `Integral` type. `floor`, `truncate` and `round` all have the same type signature as `ceiling`.

What does it mean for a number to be an `Integral` , `RealFrac` or `Num` type?

`Integral` numbers are whole numbers; positive and negative. `Integral` types include:

- `Int` which represents integers within a finite range.
- `Integer` which can represent arbitrarily large integers, up to using all of the storage on your machine.
- (non-exhaustive, see further reading)

`RealFrac` numbers are not integers; for example **floating point** numbers or fractions. `RealFrac` types include:

- `Float` which represents a single-precision **floating point** number
- `Double` which represents a double-precision **floating point.** Double allows a larger range of values with more precision than `Float`.

`Num` is a general way to describe all numeric types. `Num` types include `Int`, `Integer`, `Float` and `Double`

- (non-exhaustive, see further reading)

## type inference

Let's revisit our original examples:

```haskell
Prelude
λ> 0 + 1
1
Prelude
λ> (0 :: Int) + (1 :: Double) -- explicitly define the numeric types

<interactive>:22:15: error:
    • Couldn't match expected type ‘Int’ with actual type ‘Double’
    • In the second argument of ‘(+)’, namely ‘(1 :: Double)’
      In the expression: (0 :: Int) + (1 :: Double)
      In an equation for ‘it’: it = (0 :: Int) + (1 :: Double)
Prelude
λ>
```

What's the difference between the first and second examples? First, let's look at the type signature for `+` :

```haskell
Prelude
λ> :t (+)
(+) :: Num a => a -> a -> a
```

This type signature tells us that `+` accepts any `Num` type but, importantly, both arguments must be of the same type `a`. The first example works as expected because inputs `0` and `1` are **inferred** to be of the same type. 

```haskell
Prelude
λ> 0 + 1
1
```

**Type inference** means that, in the absence of explicit type signatures, the type of values and expressions default to some default, usually general, type.

If you are curious as to what the inferred types are for these values, you can see this in GHCi:

```haskell
λ> :t 0
0 :: Num p => p
Prelude
λ> :t 1
1 :: Num p => p
```

In the second example, we explicitly define the types of inputs to be different to each other: 

```haskell
Prelude
λ> (0 :: Int) + (1 :: Double) -- explicitly define the numeric types

<interactive>:22:15: error:
    • Couldn't match expected type ‘Int’ with actual type ‘Double’
    • In the second argument of ‘(+)’, namely ‘(1 :: Double)’
      In the expression: (0 :: Int) + (1 :: Double)
      In an equation for ‘it’: it = (0 :: Int) + (1 :: Double)
```

This example does not work because the input types do not match the expectation that both inputs to `+` are of the same `Num` type. Another way of saying this is that the expression does not **type check**. To get this example to type check, you'll need to **convert** the input types:

```haskell
Prelude
λ> fromIntegral (0 :: Int) + (1 :: Double)
1.0
```

## type coercion

If you are familiar with other programming languages, this explicit type conversion might seem arduous. Why aren't types automatically converted in Haskell, as they are in languages like JavaScript?

This automatic, or implicit, conversion between types is known as **coercion**. That Haskell does not coerce types is one aspect of its **strong type system**, which guarantees against particular type-related errors at runtime. 
## resources

- [What's the difference between Integer and Int?](https://wiki.haskell.org/FAQ#What.27s_the_difference_between_Integer_and_Int.3F)
- [Converting numbers](https://wiki.haskell.org/Converting_numbers)
- [Generic number types](https://wiki.haskell.org/Generic_number_type)
- [Does Haskell have type casts?](https://wiki.haskell.org/FAQ#Does_Haskell_have_type_casts.3F)
- [Monomorphism restriction](https://wiki.haskell.org/Monomorphism_restriction)