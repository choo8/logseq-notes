---
title: State Monad
---

## Definition (Haskell)
### `newtype State s a = State {runState :: s -> (s, a)}`
###
```haskell
get :: State s s
get = (\s -> (s, s))
```
###
```haskell
put :: s -> State s ()
put = (\s -> (s, ()))
```
###
```haskell
modify :: (s -> s) - > State s ()
modify f = do
  s <- get
  put (f s)
```
## Purpose
### Haskell is a pure functional language, and hence does not naturally allow for "mutability" like other imperative languages
#### A straightforward way to maintain some kind of state in pure functional languages would be to modify all functions that would modify that state such that it takes in an additional argument and returns an additional value, which is the modified value of the state
#### e.g. `reverseWithCount :: Int -> [a] -> (Int, [a])` instead of `reverse :: [a] -> [a]`
### The State Monad is a mechanism that allows for mutability in pure functional languages that does not require much boilerplate or modifications to be done to functions that would be mutating states
## Intuition
### The State monad implementation does not allow one to directly access the state value, which is a little counter intuitive, but the handling of the "state threading" should be done by `(>>=)`
#### Instead of having to handle state mutation by other state mutating functions within the function, the monadic bind handles it for us
#### e.g. instead of 
```haskell
appendReversedWithCount :: Int -> [a] -> [a] -> (Int, [a])
appendReversedWithCount funcCount list1 list2 =
  let (funcCount', revList1) = reverseWithCount funcCount list1
      (funcCount'', revList2) = reverseWithCount funcCount' list2
  in (funcCount'' + 1, revList1 ++ revList2)
```
we now have
```haskell
appendReversedWithCount :: [a] -> [a] -> State Int [a]
appendReversedWithCount list1 list2 =
  reverseWithCount list1 >>= (\revList1 ->
    reverseWithCount list2 >>= (\revList2 ->
      State (\s -> (s + 1, revList1 ++ revList2))))
```
#### Notice that we no longer need manually keep track and pass around the function counts like in the non-monadic implementation
### Functions that mutate states previously now have a new signature
#### e.g. `reverseWithCount :: [a] -> State Int [a]`
#### The intuition is that the signature is almost identical to the original function, just that instead of returning the original type, replace some return type `a` with `State s a`
#### And instead of calling the functions one after the other, we can chain them together with the monadic bind
### We can make use of "primitive" State functions to modify and obtain the state and ultimately create an even cleaner implementation of the state mutating functions by using the do notation
#### e.g.
```haskell
appendReversedWithCount :: [a] -> [a] -> State Int [a]
appendReversedWithCount list1 list2 = do
  revList1 <- reverseWithCount list1
  revList2 <- reverseWithCount list2
  modify (+1)
  pure (revList1 ++ revList2)
```
## References
### https://williamyaoh.com/posts/2020-07-12-deriving-state-monad.html
