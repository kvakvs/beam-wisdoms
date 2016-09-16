Input/Output and Ports ELI5
===========================

A main reason why people run computer programs is to have side effects. A pure
program without side effects will consume watts of power, warm up
your room and do nothing else. To connect programs to the outer world, there are
input and output features in every language. For example: reading and writing
files, accessing hardware ports or interacting with OS drivers, drawing on
screen and so on.

Erlang does this with ports, driven by small C modules, called *port drivers*.
Some drivers are built into Erlang, and some you will have to create yourself,
or pay someone to do it for you. For many port problems there may already be a
solution on Github (Unix pipes as example).

When you connect to a resource to start using it, you receive a
:ref:`Port <def-port>` value back. It behaves similar to a process: it consumes
CPU time to process your data, you can send messages to it, also it can send
messages back to you. You can link to a port or monitor a port (monitors added
since v.19).

Port Driver
-----------

A port driver is a C module with several functions defined.
Define an ``erl_drv_entry`` variable (a struct) and fill it with pointers to
callback functions.
You will need to cover things such as starting your driver,
stopping it, opening a port, sending commands and receiving data, and few other.

.. seealso::
    BEAM Wisdoms: :doc:`interfacing`

    `Driver HowTo <http://erlang.org/doc/man/erl_driver.html>`_

    `Port Driver and erl_drv_entry documentation <http://erlang.org/doc/man/driver_entry.html>`_

Load your port driver and register it in the system under a name
(``add_driver_entry``).

.. seealso::
    `Erl Dynamic Driver Loader Linker <http://erlang.org/doc/man/erl_ddll.html>`_
