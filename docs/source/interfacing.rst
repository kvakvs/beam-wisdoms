Interfacing Erlang with the Outer World
=======================================

Native Libraries
----------------

There are plenty of different approaches on integrating NIFs written in other
languages:

C
````

*   `edtk, a port driver toolkit <http://www.snookles.com/erlang/edtk/>`_
*   `dryverl <http://dryverl.ow2.org/>`_ (license BSD)
*   `C/C++ Generic Erlang Port Driver <https://github.com/okeuday/erlang_py/>`_
    (license BSD)

C++
````

*   `EPI <https://github.com/bsmr-erlang/epi>`_ (license LGPL)
*   `eixx <https://github.com/saleyn/eixx>`_ (license Apache2)
*   `C/C++ Generic Erlang Port Driver <https://github.com/okeuday/erlang_py/>`_
    (license BSD)
*   `C++11 NIFPP <https://github.com/goertzenator/nifpp>`_ (license Boost)

Lua
````

*   erlua
*   `erlualib <https://github.com/Eonblast/Erlualib>`_ (license MIT)

Python
``````

*   `erlang_py <https://github.com/okeuday/erlang_py/>`_ (license BSD)
*   `py_interface <http://www.lysator.liu.se/~tab/erlang/py_interface/>`_,
    `Github <git://github.com/tomas-abrahamsson/py_interface.git>`_
    (license GPL)

Rust
````

*   `Rustler <https://github.com/hansihe/Rustler>`_ (license Apache, MIT)
*   `erlang-rust-nif <https://github.com/erszcz/erlang-rust-nif>`_

Ports and Network
-----------------

For the situations when you still need the speed and can tolerate some call
latency over, but don't want to risk crashing your node: choose interaction over
network or a port.

*   `Erlport <http://erlport.org/>`_
*   `C/C++, Erlang/Elixir, Java, JavaScript/node.js, Perl, PHP, Python and Ruby <http://cloudi.org/>`_
*   `Erlang Term Format (Ruby) <https://github.com/okeuday/erlang_rb>`_ (BSD)
*   `Erlang Term Format (Javascript) <https://github.com/okeuday/erlang_js>`_ (BSD)
*   `Erlang Term Format (Perl) <https://github.com/okeuday/erlang_pl>`_ (BSD)
*   `Erlang Term Format (PHP) <https://github.com/okeuday/erlang_php>`_ (BSD)
