---
title: Monad Transformer
---

## Definition
### For example, a monad transformer that gives `IO` monad some characteristic of the `Maybe` monad (`m` is `IO` in this example)
`newtype MaybeT m a = MaybeT { runMaybeT :: m (Maybe a) }`
### `lift` lifts a computation from the argument or precursor monad to the constructed monad transformer
`lift :: Monad m => m a -> t m a`
#### all these 3 ways are equivalent
```haskell
monadicValue >>= \x -> return (f x)
```
vs
```haskell
do x <- monadicValue
   return (f x)
```
vs
```haskell
lift f monadicValue
```
## Purpose
### Monad transformers allows us to combine 2 monads together into one that shares the behaviour of both monads
#### e.g. if we were to use the `IO (Maybe a)` type, we would be force to perform pattern matching within the `IO` do-blocks, which is supposed to be eliminated by the `Maybe` monad. `guard` will reduce the do-block to `empty` if the predicate is `False`, which results in the same effect as not using the Monad Transformer, but with less boilerplate and pattern matching
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
### Monad transformers transform monads into monads, so `MaybeT m` needs to be an instance of the `Monad` class
####
```haskell
instance Monad m => Monad (MaybeT m) where
  return  = MaybeT . return . Just

  -- The signature of (>>=), specialized to MaybeT m:
  -- (>>=) :: MaybeT m a -> (a -> MaybeT m b) -> MaybeT m b
  x >>= f = MaybeT $ do maybe_value <- runMaybeT x
                        case maybe_value of
                           Nothing    -> return Nothing
                           Just value -> runMaybeT $ f value
```
#### In this example, `MaybeT m` is also made an instance of the `Alternative` typeclass to let us make use of the `guard` function. This also makes sense since `Maybe` is an instance of `Alternative`. The `(<|>)` function defined for `MaybeT m` is essentially returning the only `Just x` value when one of the arguments is `Nothing`, or choosing the first argument when both are not `Nothing` 
```haskell
instance Monad m => Alternative (MaybeT m) where
    empty   = MaybeT $ return Nothing
    x <|> y = MaybeT $ do maybe_value <- runMaybeT x
                          case maybe_value of
                               Nothing    -> runMaybeT y
                               Just _     -> return maybe_value
```
#### `MaybeT` is also an instance of `MonadTrans`, which implements `lift`. In this example, `lift` is useful for "lifting" functions from `String` into `MaybeT IO String`, as seen in the `getPassphrase` function
```haskell
class MonadTrans t where
  lift :: Monad m => m a -> t m a

instance MonadTrans MaybeT where
    lift = MaybeT . (liftM Just)
```
##### If we do not have the `lift` function, it would be inconvenient to work with values in the base monad and require us to wrap values up in the data constructor
```haskell
modify' :: (s -> Either e s) -> EitherT e (State s) ()
modify' f = EitherT $ do
  s0 <- get
  case f s0 of
    Left e -> return $ Left e
    Right s1 -> do
      put $! s1
      return $ Right ()
```
vs
```haskell
modify' :: (s -> Either e s) -> EitherT e (State s) ()
modify' f = do
  s0 <- lift get
  case f s0 of
    Left e -> exitEarly e
    Right s1 -> lift $ put $! s1
```
#### The monadic bind from `MaybeT m` also eliminates the checking of the values of `Maybe`. This allows us to make full use of the monadic binds from both `IO` and `Maybe` monads instead of only one of them and then writing boilerplate code for the other
```haskell
askPassphrase :: MaybeT IO ()
askPassphrase = do lift $ putStrLn "Insert your new passphrase:"
                   value <- getPassphrase
                   lift $ putStrLn "Storing in database..."
```
vs
```haskell
askPassphrase :: IO ()
askPassphrase = do putStrLn "Insert your new passphrase:"
                   maybe_value <- getPassphrase
                   case maybe_value of
                       Just value -> do putStrLn "Storing in database..."  -- do stuff
                       Nothing -> putStrLn "Passphrase invalid."
```
#### Not all monad transformers are related to their precursor monad in the same way, because not all precursor monads in monad transformers would have a constructor
:PROPERTIES:
:later: 1611152312230
:END:
```haskell
runListT :: ListT m a -> m [a]
```
vs
```haskell
runWriterT :: WriterT w m a-> m (a, w)
```
## References
### https://en.wikibooks.org/wiki/Haskell/Monad_transformers
### https://en.wikibooks.org/wiki/Haskell/Alternative_and_MonadPlus
### http://hackage.haskell.org/package/transformers-0.5.6.2/docs/Control-Monad-Trans-Class.html
### https://www.fpcomplete.com/haskell/tutorial/monad-transformers/
