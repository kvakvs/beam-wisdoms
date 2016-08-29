BEAM VM ELI5
============

BEAM got it's name from Bogdan/Björn Erlang Abstract machine. This is register
VM, but it also has a stack.

Registers ELI5
--------------

Being a register VM means that stack is not used to pass arguments
between calls, but registers are. They are not same registers as one would
expect to see in a CPU architecture, but rather just an array of
:ref:`Term <def-term>` values (there are 1024 of them but functions of arity 1024
are fairly rare, so usually first 3-10 registers are used).

When a call to a function, for example, of arity 3, is made: registers x0, x1
and x2 are set to function arguments. If we ever need to make a recursive call
and will need old register values later, they are pushed onto stack (which has
cells named y0, y1... and so on, counting back from stack top, and then registers
are filled with arguments for next call. There is no dynamic stack allocation and
compiler knows exactly how many words on stack given function needs.


Bytecodes and VM Loop ELI5
---------------------------

When BEAM code is loaded, each opcode from BEAM file is processed (there are
currently about 158 of them) and some combinations of opcodes are replaced with
one faster opcode (there are about 3x as many of those). For each opcode there
is a piece of C code in ``beam_emu.c`` which has a C label. An array of C labels
is stored in the end of the same VM loop routine and is used as lookup table.

After loading opcodes are replaced with such label addresses, followed by
arguments. So for example, for opcode 1, an element with index 1 is taken from
labels array and placed in code memory, then arguments are placed.

This way jumping to C code which handles next opcode is as easy, as
reading a ``void*`` pointer and doing a ``goto *p`` (there is a C and C++
language and compiler extension which allows to do it). This type of loop is
called *direct-threaded dispatch* virtual machine loop.

Other types of VM are *switch dispatch* VM (this one is used in BEAM VM if
compiler refuses to support labels as pointers extension and is 20-30% slower)
also *direct call threading* and few more exotic.
http://www.cs.toronto.edu/~matz/dissertation/matzDissertation-latex2html/node6.html
