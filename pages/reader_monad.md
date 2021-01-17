---
title: Reader Monad
---

## Definition
### `newtype Reader cfg a = Reader { runReader :: cfg -> a}`
### `ask` allows you to get the configuration
```haskell
ask :: Reader cfg cfg
ask = Reader id
```
### `asks` allows you to transform the configuration or project out specific fields in it
```haskell
asks :: (cfg -> a) -> Reader cfg a
asks f = Reader f
```
or
```haskell
asks :: (cfg -> a) -> Reader cfg a
asks f = do
  cfg <- ask
  pure (f cfg)
```
or even
```haskell
asks :: Reader 
```
### `local` allows functions to run with a different configuration, like the idea of a "local environment"
```haskell
local :: (cfg -> cfg') -> Reader cfg' a -> Reader cfg a
local transform (Reader f) = Reader (f . transform) 
```
## Purpose
### The Reader Monad is a special case of the [[State Monad]], that facilitates the passing along of some kind of read-only state between functions (e.g. a global runtime config)
### The [[State Monad]] can be used as well, but since the state can be mutated between functions, it could potentially create problems in the code (e.g. unintentional configuration mutations)
## Intuition
### If we are to look at the types of the functions, we are actually abstracting the passing on of the config parameter with the `(>>=)` function of the Reader Monad, hence removing the boilerplate previously required (the passing around of the config as an additional parameter for all functions that need to read from this configuration)
#### e.g. `(>>=) :: m a -> (a -> m b) -> m b`
#### This is most apparent where functions that required the configuration parameter also has to call other functions that also require the same configuration parameter
```haskell
welcomeMessage :: ABConfig -> String -> String -> String
welcomeMessage cfg motd username =
  "Welcome, " ++
  toUpperStr cfg username ++
  "! Message of the day: " ++
  toUpperStr cfg motd
```
vs
```haskell
welcomeMessage :: String -> String -> Reader ABConfig String
welcomeMessage motd username = do
  upperMOTD <- toUpperStr motd
  upperUsername <- toUpperStr username
  pure ("Welcome, " ++ upperUsername ++ "! Message of the day: " ++ upperMOTD)
```
## References
### https://williamyaoh.com/posts/2020-07-19-deriving-reader-monad.html
