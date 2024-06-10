# In Depth: IO in Erlang

## How a Driver Looks Like

A port driver is a C module with several functions defined (see `ErlDrvEntry`
in `emulator/beam/erl_driver.h`:

* Start/stop, finish, process exit/emergency close functions
* IO events (async/input/output)
* Timeout handler
* Call function, which passes commands to the port
* Control function, which changes options in the port
* Flush function, which ensures that all data is written

Making a Port Driver
--------------------

Define an `erl_drv_entry` variable (a struct) and fill it with pointers to
callback functions.
You will need to cover things such as starting your driver,
stopping it, opening a port, sending commands and receiving data, and few others.

> See Also: [](Interfacing-with-the-Outer-World.md)
> * [Driver HowTo](http://erlang.org/doc/man/erl_driver.html)
> * [Port Driver and erl_drv_entry documentation](http://erlang.org/doc/man/driver_entry.html)

Load your port driver and register it in the system under a name
(`add_driver_entry`).

> See also [Erl Dynamic Driver Loader Linker](http://erlang.org/doc/man/erl_ddll.html)
