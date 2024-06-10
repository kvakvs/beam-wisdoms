# BEAM VM ELI5

BEAM got its name from Bogdan/Björn Erlang Abstract machine. This is register VM, but it also has a stack.

## Registers and Calls

Being a register VM means that stack is not used to pass arguments
between calls, but registers are. They are not same registers as one would
expect to see in a CPU architecture, but rather just an array of
[](BEAM-Definitions.md#term) values. There are 1024 registers but functions of such
arity are rare, so usually VM only uses first 3 to 10 registers.

For example, when a call to a function of arity 3 is made: registers x0, x1,
and x2 are set to function arguments. If we ever need to make a recursive
call, we will have to overwrite registers and lose the original values in
them. In this situation, we will need to save old values on the stack. There
is no dynamic stack allocation other than stack frame, determined by the
compiler.

Each process has own set of registers.

## Instructions

BEAM instruction set contains approximately 158 basic instructions, most of them
can be visible if you dump assembly by calling ``erlc -S yourfile.erl``. Each
instruction has 0 or more arguments. Different tricks are used to encode
arguments in a portable and compact way. For example locations in code (so-called
labels) are encoded by their index, and code loader later translates them to
memory addresses.

During BEAM code loading, some combinations of opcodes are replaced with a
faster opcode. This is optimization trick called *super-instruction*.
For each opcode, there is a piece of C code in `beam_emu.c` which has a
C label. An array of C labels is stored at the end of the same VM loop routine
and is used as the lookup table.

> See Also: [](BEAM-File-Format.md)

> See Also: [](BEAM-Instruction-Codes.md)

## Threaded VM Loop

VM emulator loop in `emulator/beam/beam_emu.s` contains a lot of small
pieces of code, each having a label and handling one BEAM instruction.
They all belong to one very long function.
A table of labels is stored in the same function which is used as lookup table.

After the loading opcodes are replaced with such label addresses, followed by
arguments. For example, for opcode #1, an element with index 1 from labels
array is placed in the code memory.

This way it is easy to jump to a location in C code which handles next opcode.
Just read a `void*` pointer and do a `goto *p`. This feature is an
extension to C and C++ compilers. This type of VM loop is called
*direct-threaded dispatch* virtual machine loop.

Other types of VM loops are:

*   *switch dispatch* VM (this one is used by BEAM VM source if the C compiler
    refuses to support labels extension). It is also 20 to 30% slower than direct
    threading;
*   *direct call threading*
*   and a few more exotic
    http://www.cs.toronto.edu/~matz/dissertation/matzDissertation-latex2html/node6.html
