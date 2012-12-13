cakefile-skeleton (WIP)
=======================

My starting point of working with Cakefile.

Getting started
---------------

Download `Cakefile` into your project root directory:

    curl -O https://raw.github.com/japboy/cakefile-skeleton/master/Cakefile

If you don't have preprocessor packages, download `package.json` and run `npm install`:

    curl -O https://raw.github.com/japboy/cakefile-skeleton/master/package.json

Convention
----------

This `Cakefile` looks for supported files recursively under the project root directory, and compiles them.

Supported extensions are:

* `.coffee` as CoffeeScript
* `.jade` as Jade
* `.styl` as Stylus

For my convenience, files started with `_` are ignored. This should be useful to take advantage of Stylus `@import` or Jade `extends`.

License
-------

The original [cakefile-template][1] is made by [@twilson63][2], and
this is a fork project made by [@japboy][3] in 2012.

Distributed under the [Public Domain][4].

[1]: https://github.com/twilson63/cakefile-template
[2]: https://github.com/twilson63
[3]: https://github.com/japboy
[4]: http://creativecommons.org/licenses/publicdomain/
