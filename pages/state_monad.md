---
title: State Monad
---

## Definition (Haskell)
### `newtype State s a = State {runState :: s -> (s, a)}`
### `get :: State s s`
### `put :: s -> State s ()`
## Purpose
### Haskell is a pure language, and hence does not naturally allow for "mutability"
### The State Monad is a way that allows for a functional style of mutability, like a Finite State Machine
###
## References
### https://williamyaoh.com/posts/2020-07-12-deriving-state-monad.html
