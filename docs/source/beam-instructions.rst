BEAM Instruction Codes
======================

BEAM file stores code in ``Code`` section, using 1 byte for opcode, and encoding
arguments one after one using :ref:`compact term encoding <beam-term-format>`.
Opcode table defines arity, or how many arguments each opcode can take.

The table is not complete, for details on remaining opcodes please refer to
`LING VM opcode table <http://erlangonxen.org/more/beam>`_ or to the source
``beam_emu.c``. Also see the opcode rules table ``ops.tab``, which defines
special opcodes and rules to to create superinstructions.
Superinstructions bundle several often used instructions into one and try to
optimize for specific argument types.

As of R19 there are 158 base opcodes and several hundreds of superinstructions.

Opcode Table
------------

Search for "#N" to find opcode N.

.. |op-no-update-cp| replace:: Does not update the CP register with a return
    address, making return bypass current code location.

.. |op-save-cp| replace:: Return address is saved in CP.

.. |op-mfarity-def| replace:: mfarity (a tuple ``{Mod, Fun, Arity}``)
    which can point to an exported function or a BIF

.. |op-mfarity-def-short| replace:: mfarity (a tuple ``{Mod, Fun, Arity}``)

#2 func_info
````````````

``Func_info`` is located at the beginning of every function, but execution
always begins at the next label.
If all function clauses have failed guard checks, VM will jump to this
instruction.
It raises ``function_clause`` error with appropriate line number.

#4 call
```````

Spec: ``call Arity Label``

Saves current IP to CP (return address) and jumps to the ``Label``,
which usually is a function entry point.

#5 call_last
````````````

Spec: ``call_last Arity Label Deallocate``

Deallocates ``Deallocate`` words on stack and does a tail recursive call (jump)
to the function at ``Label``.
|op-no-update-cp|

#6 call_only
````````````

Spec: ``call_only Arity Label``

Performs tail recursive call of a function at ``Label``.
|op-no-update-cp|

#7 call_ext
```````````

Spec: ``call_ext Arity Destination::mfarity()``

Performs call to ``Destination`` |op-mfarity-def|.
|op-save-cp|

#8 call_ext_last
````````````````

Spec: ``call_ext_last Arity Destination::mfarity() Deallocate``

Deallocates ``Deallocate`` words from stack and performs a tail recursive
call to ``Destination`` |op-mfarity-def|.
|op-no-update-cp|

#9 bif0
```````

Spec: ``bif0 ImportIndex Dst::mfarity()`` (note no fail label)

Looks up BIF with zero arguments by ``Dst`` |op-mfarity-def-short| and calls it.
Cannot silently fail by jumping to a label and will always throw an exception.

#10 bif1 #11 bif2 #152 bif3
```````````````````````````

Spec: ``bif1 Fail::label() ImportIndex Arg1 Dst::mfarity()``

Spec: ``bif2 Fail::label() ImportIndex Arg1 Arg2 Dst::mfarity()``

Spec: ``bif3 Fail::label() ImportIndex Arg1 Arg2 Arg3 Dst::mfarity()``

Looks up BIF with 1, 2 or 3 arguments by ``Dst`` |op-mfarity-def-short|
and calls it.
If ``Fail`` label is valid (not 0?), VM will jump to the label on error,
otherwise will throw an exception.

#12 allocate
````````````

Spec: ``allocate StackNeed Live``

Allocates ``StackNeed`` words on the stack.
``Live`` defines how many X registers are currently in use (for GC call).
Current CP value is saved on top of the stack.
