# Interfacing Erlang with the Outer World


## Native Libraries

> 👴 Outdated Information Note:
> This page was created between 2016 and 2018, more changes _might have_ happened since.

There are plenty of different approaches on integrating NIFs written in other
languages:

### C Language

*   [edtk, a port driver toolkit](http://www.snookles.com/erlang/edtk/)
*   [dryverl](http://dryverl.ow2.org/) (license BSD)
*   [C/C++ Generic Erlang Port Driver](https://github.com/okeuday/GEPD/)
    (license MIT)
*   [Nifty, a NIF wrapper generator](http://parapluu.github.io/nifty/)

### C++ 

*   [EPI](https://github.com/bsmr-erlang/epi) (license LGPL)
*   [eixx](https://github.com/saleyn/eixx) (license Apache2)
*   [C/C++ Generic Erlang Port Driver](https://github.com/okeuday/GEPD/)
    (license MIT)
*   [C++11 NIFPP](https://github.com/goertzenator/nifpp) (license Boost)

### Lua

*   erlua
*   [erlualib](https://github.com/Eonblast/Erlualib) (license MIT)

### Python

*   [Pyrlang](https://github.com/esl/Pyrlang) (Erlang node written in asyncio) (license Apache2)
*   ErlPort (see below in "Ports")
*   [erlang_py](https://github.com/okeuday/erlang_py/) (license MIT)
*   py_interface
    [Website](http://www.lysator.liu.se/~tab/erlang/py_interface/),
    [GitHub](https://github.com/tomas-abrahamsson/py_interface.git) (licence LGPL)
*   PyErl [Github](https://github.com/hamano/python-erlang-interface) (license MPL)

### Ruby

*   ErlPort (see below in "Ports")
*   [Erlang Term Format (Ruby)](https://github.com/okeuday/erlang_rb) (MIT)

### PHP

*   [Erlang Term Format (PHP)](https://github.com/okeuday/erlang_php) (MIT)


### Rust

*   [Rustler](https://github.com/hansihe/Rustler) (license Apache, MIT)
*   [erlang-rust-nif](https://github.com/erszcz/erlang-rust-nif)

### Nim

*   [nimler](https://github.com/wltsmrz/nimler) (MIT)

### Javascript

*   [Erlang Term Format (Javascript)](https://github.com/okeuday/erlang_js) (MIT)

### Perl

*   [Erlang Term Format (Perl)](https://github.com/okeuday/erlang_pl) (MIT)

## Ports and Network

For the situations when you still need the speed and can tolerate some call
latency over, but don't want to risk crashing your node: choose interaction over
network or a port.

*   [Erlport](http://erlport.org/)
*   [CloudI, A private cloud engine](http://cloudi.org/)
    (C/C++, Erlang/Elixir, Java, JavaScript/node.js, Perl, PHP, Python and Ruby)
