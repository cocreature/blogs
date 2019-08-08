# DLint : Source code suggestions for DAML

We recently added an exciting new feature to the DAML IDE (you can download the IDE [here](https://docs.daml.com/)).

The DLint ("DAML lint") feature provides suggestions for improving your DAML source code as you type! You can get an idea of what this looks like from the image below.

![DLint in the IDE](img/lint-001.png)

DLint is based on [Neil Mitchell's](https://ndmitchell.com/) celebrated [HLint](http://hackage.haskell.org/package/hlint) package. HLint is a venerable program with over 15+ years of active development and it is heavily used in the Haskell community. We are able to leverage HLint for DAML since the DAML compiler uses the GHC API under the hood.

There is a delightfully symbiotic relationship between DLint and HLint. In order for us to be able to use HLint with DAML, it was necessary (actually this work is ongoing) to replace HLint's dependence on the [`haskell-src-exts`](http://hackage.haskell.org/package/haskell-src-exts) library with the [`ghc-lib-parser`](http://hackage.haskell.org/package/ghc-lib-parser) library that we developed. Coincidentally, as [recently announced](https://mail.haskell.org/pipermail/haskell-cafe/2019-May/131166.html), there are to be no more releases of `haskell-src-exts` so Hlint and thereby the Haskell community, get the benefit of this conversion too!
In particular, the switch to the GHC API will fix numerous long standing bugs in HLint caused by the use of [`haskell-src-exts`](https://github.com/ndmitchell/hlint/issues?q=is%3Aissue+is%3Aopen+label%3Adepends-on-HSE).

HLint is already integrated into `haskell-ide-engine` and will soon be integrated in our `hie-core` project (see … for details) so these changes will also soon be usable from your favorite Haskell IDE.
