+++
title = 'Log of a Type Is Information'
date = 2018-11-17T19:24:12+05:30
draft = true
tags = ["tags"]
description = "Desc"

# For twitter cards, see https://github.com/mtn/cocoa-eh-hugo-theme/wiki/Twitter-cards
meta_img = "/images/image.jpg"

# For hacker news and lobsters builtin links, see github.com/mtn/cocoa-eh-hugo-theme/wiki/Social-Links
hacker_news_id = ""
lobsters_id = ""
+++


# The `log` of a type is the information content in it.

This is easy to see, if one has some experience with information theory. This is meant
to be an extention to the general "sum, product, differentiation" story of types.

Consider the grammar of types generated by

```
Type := 0 | 1 | Type + Type | Type * Type | Type -> Type
```

Let's look at the familiar haskell counterparts for these beasts

```hs
-- unconstructible, also called `Void`
data Zero

-- single constructor
data One = One

-- holds either a or b
data Plus a b = Left a | Right b

-- holds both a and b
data Prod a b = Prod a b
```

We can construct booleans using
```hs
data Bool' = Plus One One
```

Now, applying the rule of `log` to this, we arrive at:
##### `log` of `1`:
```
log(1) = 0 // reasonable, the number of bits of information held in a single constructor is zero

```

##### sanity check: `log` of `2`:
```
log(2) = 1
```
This is reasonable, since `2 ~= Bool`, and we need one bit to express a `Bool`.

###### log of product
```
log(ab) = log a + log b
```
 If storing `a` needs log a bits, and `b` needs `log b` bits, then storing
 the tuple `(a, b)` needs `log a` + `log b` bits.

 ##### log of function

 ```hs
 log (a -> b) = log (b^a) = a log b
 ```
For each `a` we store `n` bits of information telling us which `b` to provide
as output for taht particular `a`. Hence, total space needed to encode a function
`a -> b` is `a log b`.


## Relationship to `Deltas`

I'm trying to construct a type theoretic theory of patches. What do I mean by this?
For example, consider the humble `Bool`. I want a pair of functions:

```hs
takeDelta :: Bool -> Bool -> DeltaBool
applyDelta :: Bool -> DeltaBool -> Bool
```

which satisfies:

```
applyDelta (takeDelta b1 b2) = b2
```

We can try to impose more structure on `DeltaBool`, ask that it should be invertible, should have
a composition operator to compose deltas, etc. But all that's later. Let's first ask, what is
the correct implementation of `DeltaBool`, `takeDelta` and `applyDelta` for this case?


The answer is this: If we have a boolean `b`, its delta from another boolean `b'` was
whether `b` was toggled or not. So, the implementation would be

```hs
data DeltaBool = Toggled | Same
takeDelta :: Bool -> Bool -> DeltaBool
takeDelta b1 b2 = if b1 == b2 then Same else Toggled

applyDelta b Same = b
applyDelta True Toggled = False
applyDelta False Toggled = True
```

These clearly satisfy the laws.

Now, looking at this algebraically, we can say that

```hs
Delta :: Type -> Type -> Type
Delta(Bool, Bool) = Bool
Delta(2, 2) = 2
```

That is, we ask for a type-level function `Delta`, which given two types, tells us the representation
of the type that encodes the delta between two types. In this case, we just figured that the delta
between a `Bool`and a `Bool` is another `Bool`.


##### `Delta(1, 1)`

Next, we can also conclude that
```hs
Delta(1, 1) = 1
```

The reasoning is that looking at it information theoretically, there is no information needed to distinguish
`One` from `One`, so it has `0` bits of information. Hence, our type for this should be `2^0 = One`

##### Linearity of `Delta`?

We next ask the question, now tat we know `Delta` for some values, can we provide some rules as to
how `Delta` interacts with `+` and `*`?

##### `Delta` and `*`:
A natural definition would be this:
```hs
Delta(x, y * z) = Delta(x, y) * Delta(x, z)
```

For us to tell how to change `x` into `y * z`, we need to store the difference between `x, y` as
well as `x, z`.

##### `Delta` and `+`:
This is where things get tricky.

```hs
Delta(x, y + z) = ?
```

One tempting definition is this:

```hs
Delta(x, y + z) = Delta x y + Delta x z
Delta(z, x + y) = Delta z x + Delta z y
```
##### The problem:
However, if we use these rules to calculate `Delta(Bool, Bool)`, we arrive at:

```hs
Delta(Bool, Bool)
= Delta(2, 1 + 1) = Delta(2, 1) + Delta(2, 1)
= 2 * Delta(2, 1)
= 2 * Delta(1 + 1 , 1)
= 2 * (Delta (1, 1) + Delta (1, 1))
= 2 * (1 + 1)
= 4
```

So, this tells us that `Delta(2, 2)` is 4, while we had previously
concluded that `Delta(2, 2) = 2`!

Clearly, something is amiss, and I can't tell why. Can you?