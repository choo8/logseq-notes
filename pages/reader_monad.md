---
title: Reader Monad
---

## Definition
### `newtype Reader cfg a = Reader { runReader :: cfg -> a}`
###
```haskell
ask :: Reader cfg cfg
```
###
```haskell
asks :: (cfg -> a) -> Reader cfg a
```
## Purpose
### The Reader Monad is a special case of the [[State Monad]], where it allows for the passing along of some kind of read-only state between functions (e.g. a global runtime config)
### The [[State Monad]] can be used as well, but since the state can be modified, it could potentially create problems in the code
