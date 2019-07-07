# 4 Simple Ways To Improve Haskell

In the course of our work implementing [DAML](https://daml.com/), we felt there were some things Haskell could do better and so we went ahead and did them! The ideas in this post are intentionally small but cumulatively we judge their effect to be large. There are more changes that we made to suit DAML; these are the ones that we decided just had to be tackled first.

So, in no particular order and without further ado, here they are!

## 1. The "new colon convention"

This extension reverses the roles of `:` and `::`. That is, `:` is read "type of" and `::` as the list "cons" operator. Here's an example.
```haskell
append : a -> List a -> List a
append x xs = x :: xs
```
Yes, it's been [proposed before]( https://github.com/ghc-proposals/ghc-proposals/pull/118). Our implementation builds on that proposal and addresses shortcomings with respect to warning and error messages. The proposal sadly was withdrawn though. We really think that was a mistake and it ought to be revisited. For the rest of this blog we'll assume this extension is in effect.

## 2. Record "with" syntax

We didn't get around to proposing this one yet although its been part of DAML since inception. It is trivially implemented in just the parser but its impact on readability is profound! The idea is to introduce a new keyword `with` that can be used for record type declarations and updates. For example, instead of
```haskell
data T = T {foo : Integer, bar : String}
```
you can write,
```haskell
data T = T with foo : Integer; bar : String
```
Or, since `with` is "layout inducing", that can also be written,
```haskell
data T =
  T with
    foo : Integer
    bar : String
```
or variations on that theme as appropriate. Similarly, for record updates, instead of
```haskell
setFooTwo : T -> T
setFooTwo r = r {foo = 2}
```
one can write
```haskell
setFooTwo : T -> T
setFooTwo r = r with foo = 2
```
This syntax places very nicely with the extensions `RecordPuns` and `RecordWildcards` but we'll not go on about that...

## 3. Inline function return type annotations

[This proposal](https://github.com/ghc-proposals/ghc-proposals/pull/185) augments the `ScopedTypeVariables` extension to permit inline function return type signatures. For example, one can write
```haskell
fact (n : Int) : Int
  | n  <= 1   = 1
  | otherwise = n * fact (n - 1)
```
Currently, the GHC parser recognizes these return types but rejects them. Our implementation gives them meaning. The discussions so far indicate that there is appetite for adding this extension but some favor generalization beyond what the proposal currently offers. Perhaps this simple start is enough for now?

## 4. Module qualified syntax

To import a qualified module you must specify `qualified` in prepositive position : `import qualified M`. In the Haskeller's eternal quest for alignment, this often leads to a "hanging indent". For example,
```haskell
import qualified A
import           B
import           C
```
The syntax `import M qualified` is suggested in [this proposal](https://github.com/ghc-proposals/ghc-proposals/pull/190) solving the hanging indent issue. The future of this one looks favorable and is currently "pending committee review".

## Wrap-up

As mentioned in the opening to this post, there are more things we'd like to see upstreamed. In particular, we'd love to see the "no top-level record field selectors" [proposal](https://github.com/ghc-proposals/ghc-proposals/pull/160) accepted. Implementation of that has eluded us so far however and our focus here is on the simple stuff.

We think the things listed above help close the gap on perfect and fervently hope they can make their way into Haskell.
