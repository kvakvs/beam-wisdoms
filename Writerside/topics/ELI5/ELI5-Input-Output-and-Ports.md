# Input/Output and Ports ELI5

A main reason why people run computer programs is to have side effects. A pure program without side effects will consume
watts of power, warm up your room and do nothing else. To connect programs to the outer world, there are input and
output features in every language. For example: reading and writing files, accessing hardware ports or interacting with
OS drivers, drawing on screen and so on.

Erlang does this with ports, driven by small C modules, called *port drivers*. Some drivers are built into Erlang, and
some you will have to create yourself, or pay someone to do it for you. For many port problems there may already be a
solution on GitHub (Unix pipes as example).

When you connect to a resource to start using it, you receive a [](BEAM-Definitions.md#port) value back. It behaves
similar to a process: it consumes CPU time to process your data, you can send messages to it, also it can send messages
back to you. You can link to a port or monitor a port (monitors added since v.19).

## Port Tasks

Each port similar to processes gets assigned to one scheduler. Every scheduler on each CPU core will periodically check
assigned ports and perform polling and maintenance (this is called running port tasks). This gives CPU time to port
drivers to perform actual IO and deliver results to the processes who are waiting.

To improve this situation and to separate port tasks from schedulers, so that they don't affect main code, Async Threads
have been invented. VM creates extra CPU threads which have only one primary goal — to serve IO tasks.

This can be controlled with `+A<number>` command line flag. Default value is 10 async threads.

## Port Driver

A C module can be registered by name as a port driver. You can specify, which driver to invoke, when opening a port.
Port driver performs several basic commands as opening, closing a port, sending data to it or reading from it. Socket
interface, OS process spawning -- for example they are implemented as port drivers.

> See Also: [](IO-in-Erlang-and-BEAM.md)
