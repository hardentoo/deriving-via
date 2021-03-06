Deriving Via (let's get the skeleton up first)

# Abstract

Paper discussing how to capture "basic building blocks for a lot of
routine programming" ([Conor McBride](http://strictlypositive.org/Idiom.pdf))
in code.

# Example

> `Applicative`s, `Traversable`s, and `Monoid`s give us the *basic building blocks* for a lot of routine programming. Every `Applicative` can be used to lift monoids, as follows

```haskell
liftA0 = pure

instance (Applicative f, Monoid m) => Monoid (f m) where
  mempty = liftA0 mempty
  (<>)   = liftA2 (<>)
```

Many similar liftings exist and can be "derived" with CPP
([Conal](https://hackage.haskell.org/package/applicative-numbers))

```haskell
-- ApplicativeNumeric-inc.hs

instance (Num applicative_arg) => Num (APPLICATIVE applicative_arg) where
  negate        = liftA  negate
  (+)           = liftA2 (+)
  (*)           = liftA2 (*)
  fromInteger i = liftA0 (fromInteger i)
  abs           = liftA  abs
  signum        = liftA  signum
```

used as

```haskell
#define APPLICATIVE Vec2
#include "ApplicativeNumeric-inc.hs"
```

(yuck)

Our solution is to codify that pattern; to give this "basic building block" a name by attaching its behaviour `newtype` "adapters" (name taken from [The Essence of the Iterator Pattern](https://www.cs.ox.ac.uk/jeremy.gibbons/publications/iterator.pdf)), note that we never use constructor `App_` directly:

```haskell
newtype App f a = App_ (f a)
  deriving newtype
    (Functor, Applicative)

instance (Applicative f, Num a) => Num (App f a) where
  negate      = fmap negate
  (+)         = liftA2 (+)
  (*)         = liftA2 (*)
  fromInteger = pure . fromInteger
  abs         = fmap abs
  signum      = fmap signum
```

I definitely want to include stuff like

```haskell
newtype SameKleisli m a = SameKleisli (Join (Kleisli m) a)
  deriving newtype
    Monoid

instance Category m => Semigroup (Join m a) where
  Join a <> Join b = Join (a . b)

instance Category m => Monoid (Join m a) where
  mempty = Join id

  Join a `mappend` Join b = Join (a . b)
```

inspired by https://stackoverflow.com/questions/32935812/why-isnt-kleisli-an-instance-of-monoid and https://gist.github.com/Icelandjack/83e77449d2857f24a7c94fede6d7311b
