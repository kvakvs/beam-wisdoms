Tracing ELI5
============

How does Erlang VM perform tracing of calls, messages, processes spawning and
passing away?

Tracing is a VM mode, that can be toggled on and off, and starts producing a
stream of events.
A call to ``dbg:tracer()`` starts a process which will receive this stream.
You can create your own tracer with its own state and feed events to it.

Tracing is able to produce overwhelming amount of irrelevant data.
To limit this data a trace filter is applied with ``dbg:tp/4`` (and similar).

When everything is prepared: a tracer and a filter, it is time to open the2
valve. A call to ``dbg:p/2`` (and similar) sets the trace target (a process,
a port, spawn and exit events, everything, and so on).
It will start sending everything that matches trace target and the filter
you've set to the tracer process.


Inner Workings of the Tracing
-----------------------------

Simple things, like process lifetime events or messages have tracer checks
in place everywhere in the virtual machine's C code. If tracing is enabled,
then a message is sent to the current tracer.

Tracing calls and returns from BIF functions is a bit more complex. Because BIF
are not real Erlang code, they have to be wrapped in a tracing code somehow.
This is done by replacing BIF table with BIF entry function addresses with
another table.
Each entry in this new table is a simple call to ``erts_bif_trace`` with a
function name and arguments.
This function performs the real call and sends trace messages.

At certain moments, when we want to trace a BIF trapping and returning to
execution later, another trick is used.
Address to a special BEAM opcode is pushed onto the stack before the BIF is
finished. This allows to catch up when trap continues execution and will send
the trace event correctly when the BIF was finished.
