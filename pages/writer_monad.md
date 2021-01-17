---
title: Writer Monad
---

## Definition
### `newtype Writer log a = Writer { runWriter :: (log, a) }`
### `tell` allows you to add in a log
```haskell
tell :: log -> Writer log ()
tell msg = Writer (msg, ())
```
### `censor` allows you to perform an arbitrary transformation on the log
```haskell
censor :: (log -> log) -> Writer log a -> Writer log a
censor f (Writer (log, x)) = Writer (f log, x)
```
### `listen` allows you to inspect the value of the log
```haskell
listen :: Writer log a -> Writer log (a, log)
listen (Writer (log, x)) = Writer (log, (x, log))
```
## Purpose
### The Writer Monad is a special case of the [[State Monad]], that facilitates the writing to and passing along of a write-only state between functions (e.g. a logfile)
### The [[State Monad]] could be used as well but we do not require its ability to get the value of the state, hence a simpler design could be achieved
## Intuition
### The Writer Monad abstracts away the handling of the log accumulation of the functions via the monadic bind, `(>>=)`
#### This reduction in boilerplate is most significant when multiple functions that write to the log are called
```haskell
augmentAndStringify :: Int -> Int -> ([String], String)
augmentAndStringify x y =
  let (xLog, x') = addTwo x
      (yLog, y') = addTwo y
  in (["augmenting..."] ++ xLog ++ yLog ++ ["stringifying..."], show (x' + y'))
```
vs
```haskell
augmentAndStringify :: Int -> Int -> Logger String
augmentAndStringify x y = do
  log "augmenting..."
  x' <- addTwo x
  y' <- addTwo y
  log "stringifying..."
  pure (show (x' + y'))
```
### The Writer Monad can actually use a more general type for `log` so that it is not restricted to only logging `[String]`, `log` can have the monoid typeclass to increase its generality
#### This increases the generality of the Writer Monad because now we can make use of `mempty` and `mappend` functions of the monoid typeclass in the monadic bind to combine the logs between function passes. Hence, as long as the `log` type is in the monoid typeclass, the Writer Monad will be able to correctly accumulate the logs without us having to reimplement any of the Writer Monad's functions
#### The benefit here is really making use of the monoid typeclass
```haskell
instance Functor (Writer log) where
  fmap f (Writer (log, x)) = Writer (log, f x)

instance Monoid log => Applicative (Writer log) where
  pure x = Writer (mempty, x)
  (<*>) (Writer (log1, fx)) (Writer (log2, x)) =
    Writer (mappend log1 log2, fx x)

instance Monoid log => Monad (Writer log) where
  return = pure
  (>>=) (Writer (log1, x)) nextFn =
    let Writer (log2, y) = nextFn x
    in Writer (mappend log1 log2, y)
```
vs
```haskell
instance Functor Logger where
  fmap f (Logger (log, x)) = Logger (log, f x)

instance Applicative Logger where
  pure x = Logger ([], x)
  (<*>) (Logger (log1, fx)) (Logger (log2, x)) =
    Logger (log1 ++ log2, fx x)

instance Monad Logger where
  return = pure
  (>>=) (Logger (log1, x)) nextFn =
    let Logger (log2, y) = nextFn x
```
## References
### https://williamyaoh.com/posts/2020-07-26-deriving-writer-monad.html
