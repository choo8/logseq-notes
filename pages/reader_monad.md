---
title: Reader Monad
---

## Definition
### `newtype Reader cfg a = Reader { runReader :: cfg -> a}`
###
```haskell
ask :: Reader cfg cfg
```
