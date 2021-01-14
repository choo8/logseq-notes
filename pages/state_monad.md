---
title: State Monad
---

## Definition (Haskell)
### `newtype State s a = State {runState :: s -> (s, a)}`
### `get :: State s s`
### `put :: s -> State s ()`
## Purpose
### Haskell is a pure functional language, and hence does not naturally allow for "mutability" like other imperative languages
#### A straightforward way to maintain some kind of state in pure functional languages would be to modify all functions that would modify that state such that it takes in an additional argument and returns an additional value, which is the modified value of the state
#### e.g. `reverseWithCount :: Int -> [a] -> (Int, [a])` instead of `reverse :: [a] -> [a]`
### The State Monad is a mechanism that allows for mutability in pure functional languages that does not require much boilerplate or modifications to be done to functions that would be mutating states
## Intuition
### The State monad implementation does not allow one to directly access the state value, which is a little counter intuitive, but the handling of the "state threading" should be done by `(>>=)`
### Functions that mutate states previously now have a new signature
#### e.g. `reverseWithCount :: [a] -> State Int [a]`
#### The idea is that
## References
### https://williamyaoh.com/posts/2020-07-12-deriving-state-monad.html
