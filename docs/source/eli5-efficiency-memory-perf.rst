Memory Performance ELI5
=======================

Data Locality
-------------

In recent decades performance of computer memory did not grow as fast as
performance of CPUs has improved.
To offset impact of memory low performance several levels of CPU caches
are used which store recently accessed data closer to the CPU.
When you access any byte in memory, larger blocks of data are loaded to
cache or flushed at once (these are called cache rows).

The general rule of gaining best performance is to store relevant data close
together. From Erlang point of view it means also creating all
relevant data together which will place them together in process heap.

If the data is large, it should be grouped to allow sequential
processing. And the cost of memory access can be so high that some values
are cheaper to calculate instead of loading them from some cold
(uncached and rarely read) memory location.

Hot (often accessed) memory is a welcome place to store
your variables and working data, for C and C++ programs this is program stack
and for C++ this would be fields of your current class object, because it
always was accessed recently. It is harder to say which memory is hot for
Erlang, as we don't control much of its behaviour, but there are some tricks
that affect data placement on heap and how it will perform after.


Cache Miss
----------

A "cache miss" is a special situation which occurs when CPU accesses some data
and the data wasn't in cache. A cache row (64 bytes) of data are requested
from main RAM. CPU is forced to wait for the data. If possible the CPU
will try and do some other work by reordering machine instructions, but
this does not always happen and just assume that it is waiting for the data.

Typical cost of a cache miss is ~100 CPU cycles. This is a lot of time,
many simpler values can be calculated without accessing memory and save
significant time with not reading the main RAM.

Even worse if your program runs on a large production server with hundreds
GB of memory, then most likely it uses non-uniform memory architecture (NUMA)
and the memory is divided between CPU cores. The cost of a cache miss on
such machines is even higher, especially if accessing part of RAM "owned"
by another CPU core. Erlang and BEAM VM is aware about this and does its
best to preserve the core locality of used memory.

So when you are optimizing your awesome algorithm which now is perfect and
doesn't run any better no matter the effort, have a look at how it works with
memory. The reward for using memory efficiently is your code running
much faster!


Data Types
----------

.. note::
    Major garbage collection of a process will order its data in
    memory by placing all list cells together, and tuple values will follow the
    tuple in memory. This is very good for memory locality although moving all
    this data might be super expensive.

.. note::

    **For memory-heavy operations**
    consider owning the data in the process which does the work instead of
    requesting it from ETS or database while doing the work.
    Keeping at most one memory-intensive process per available CPU core is
    welcome, let the BEAM VM do the rest for you.
    Also consider doing a major GC on your worker process before running a
    big calculation (see ``erlang:garbage_colect``).

Immediate Values
````````````````

These values are smallest, they always fit into a CPU register and there is
not much you can do to make them work any faster. They are atoms,
small integers, local pids etc.

Lists
`````

Lists in BEAM are represented as single-linked lists of value pairs (cells).
If a list is created in a complex algorithm slowly, its cells might be sparsely
distributed in memory interleaved with other data created at the same time.
This may lead to inefficient CPU cache usage. A way to offset this
is to create your large working lists in tight loops while not creating other
unnecessary data. This will place list cells tightly together with their values.
When it is not possible, you can always invoke a major garbage collection and
have everything sorted for you (at a cost).


Tuples
``````

As tuples are stored as arrays, they are cache friendly. If a tuple contains
other data structures such as lists or tuples, the data will be stored somewhere
else on heap and the tuple will only contain pointers to it.
Even better if a major garbage collection was performed recently which sorts
your data in memory and groups it together.


Floats
``````

Even if a floating point value fits into a CPU register and has same size as
an immediate value (small integer or atom etc), actually they are
stored on-heap with a header word marking their presence. This makes floats
consume double the memory and each access to them will dereference a pointer
to a possibly cold memory location where no other floats are stored.

Best you can do here is to create your arrays or lists of floating point values
in tight loop together, which will maximize how many floats are cached when
a single cache row of 64 bytes is loaded.
Or run a major GC to group them automatically.


ETS Tables
``````````

There is not much you can do to affect ETS table memory behaviour, so here
you are at the mercy of BEAM VM developers.
