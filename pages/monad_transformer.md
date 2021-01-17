---
title: Monad Transformer
---

## Definition
###
## Purpose
### Monad transformers allows us to combine 2 monads together into one that shares the behaviour of both
#### e.g. if we were to use the `IO (Maybe a)` type, we would be force to perform pattern matching within the `IO` do-blocks, which is supposed to be eliminated by the `Maybe` monad
```haskell
getPassphrase :: MaybeT IO String
getPassphrase = do s <- lift getLine
                   guard (isValid s) -- Alternative provides guard.
                   return s
```
vs
```haskell
getPassphrase :: IO (Maybe String)
getPassphrase = do s <- getLine
                   if isValid s then return $ Just s
                                else return Nothing
```
## Intuition
## References
### https://en.wikibooks.org/wiki/Haskell/Monad_transformers
