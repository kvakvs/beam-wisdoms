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

**Slots** are special values marking stack cell or a register. They can be
used to point at the data source or move destination. There are several
values of secondary tag which denote them.

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

Performs call to an external ``Destination`` |op-mfarity-def|.
|op-save-cp|

#8 call_ext_last
````````````````

Spec: ``call_ext_last Arity Destination::mfarity() Deallocate``

Deallocates ``Deallocate`` words from stack and performs a tail recursive
call to an external ``Destination`` |op-mfarity-def|.
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

#12 allocate #14 allocate_zero
``````````````````````````````

Spec: ``allocate StackNeed Live``

Spec: ``allocate_zero StackNeed Live``

Allocates ``StackNeed`` words on the stack (a stack frame).
``Live`` defines how many X registers are currently in use (for GC call).
Current CP value is saved on top of the stack.
``Allocate_zero`` writes NIL to all new stack cells, while ``allocate`` may
or may not do this.

#13 allocate_heap #15 allocate_heap_zero
````````````````````````````````````````

Spec: ``allocate_heap StackNeed HeapNeed Live``

Spec: ``allocate_heap_zero StackNeed HeapNeed Live``

Allocates ``StackNeed`` words on the stack and ensures that there are
``HeapNeed`` available words on the heap.
``Live`` defines how many X registers are currently in use (for GC call).
Current CP value is saved on top of the stack.
``Allocate_heap_zero`` writes NIL to all new stack (possibly heap too?) cells,
while ``allocate_heap`` may or may not do this.

#16 test_heap
`````````````

Spec: ``test_heap Need Live``

Ensures there are at least ``Need`` words on the heap. If needed, GC is
performed using ``Live`` values from registers array.

#17 init
````````

Spec: ``init Y``

Clears ``Y+1``-th stack word by writing ``NIL``. Offset by one is there because
CP is also stored on stack but it is not considered an ``Y`` cell.

#18 deallocate
``````````````

Spec: ``deallocate N``

Restores CP from stack and deallocates ``N+1`` words from stack (one extra for
the CP).

#19 return
``````````

Jumps to current value of CP register. Sets CP to 0.

#20 send
````````

Sends value ``X[1]`` to the process specified by ``X[0]``. ``X[1]`` becomes
the result of the operation (is moved to ``X[0]``).

#21 remove_message
``````````````````

Unlinks current message from the message queue. Message is moved to ``X[0]``.
*Current* means that there is a movable pointer to a message in the linked list.

#23 loop_rec
````````````

Spec: ``loop_rec Label Source``

Picks up next message in the queue and places it into ``X[0]``. If there is no
message, jumps to ``Label`` which points to a ``wait`` or ``wait_timeout``
instruction.

#24 loop_rec_end
````````````````

Spec: ``loop_rec_end Label``

Advances message pointer to the next message, then jump to ``Label`` which
points to a ``loop_rec`` instruction.

#25 wait
````````

Spec: ``wait Label``

Jumps to the ``Label`` and immediately suspends the process (wait for event).

Comparisons
```````````

Spec: ``<opcode> Label Arg1 Arg2``

Performs a comparison and jumps to ``Label`` if false.

*   #39 ``is_lt`` - is less
*   #40 ``is_ge`` - is greater or equal
*   #41 ``is_eq`` - equal ``==``
*   #42 ``is_ne`` - not equal ``/=``
*   #43 ``is_eq_exact`` - exactly equal ``=:=``
*   #44 ``is_ne_exact`` - exactly not equal ``=/=``
*   #58 ``test_arity`` - checks if ``Arg1`` is a tuple of size ``Arg2``
*   #115 ``is_function2`` - checks if ``Arg1`` is a fun of arity ``Arg2``

Guards as Opcodes
`````````````````

Spec: ``<opcode> Label Arg``

Some guards are implemented as opcodes.
Performs a check and jumps to ``Label`` if false.

*   #45 ``is_integer`` - is small or bignum
*   #46 ``is_float``
*   #47 ``is_number`` - is small or bignum or float
*   #48 ``is_atom``
*   #49 ``is_pid``
*   #50 ``is_reference``
*   #51 ``is_port``
*   #52 ``is_nil``
*   #53 ``is_binary``
*   #54 ``is_constant`` - possibly checks if term belongs to literal area of a module?
*   #55 ``is_list`` - term is a NIL or points to a cons cell
*   #56 ``is_nonempty_list`` - term points to a cons cell
*   #57 ``is_tuple``
*   #77 ``is_function``
*   #114 ``is_boolean``

#59 select_val
``````````````

Spec: ``select_val Arg FailLabel Destinations``

Scans ``Destinations`` even elements (0, 2, 4...) and compares with ``Arg``.
If match is found, jumps to the label in the next odd element (1, 3, 5...)
otherwise jumpts to ``FailLabel``.
By "match" naive compare is meant.

#60 select_tuple_arity
``````````````````````

Spec: ``select_tuple_arity Tuple FailLabel Destinations``

Check the arity of the ``Tuple`` and jump to the corresponding ``Destination``
label, if no arity matches, jump to ``FailLabel``.

#61 jump
````````

Spec: ``jump Address``

#62 catch
`````````

Spec: ``catch Y Label``

Saves a resumption address &CatchEnd in the local frame at position
``Y``. Increments the process catch counter. The instruction is followed by a
``catch_end`` instruction. By followed we mean that the ``catch_end``
instruction is put after corresponding Erlang expression that is protected
from errors by the catch.

#63 catch_end
`````````````

Spec: ``catch_end Y``

Clears a resumption address stored in the local frame at position ``Y``.
Decrements the process catch counter.
This instruction is preceded by a matching ``catch`` instruction.

#64 move
````````

Spec: ``move From To``

Moves value ``From`` to a slot ``To``.
``From`` can be a value or a slot.
``To`` must be a slot.

#65 get_list
````````````

Spec: ``get_list Source Head Tail``

Gets the head and tail of a list (splits its cons cell) from ``Source``
and puts values into the registers ``Head`` and ``Tail``.

#66 get_tuple_element
`````````````````````

Spec ``get_tuple_element Source Element Destination::slot()``

Gets ``Element``-th item from the tuple denoted by ``Source`` and puts
it into the ``Destination`` slot.

#69 put_list
````````````

Spec: ``put_list H T Dst::slot()``

Creates a cons cell with ``[H|T]`` and places the value into ``Dst``.

#70 put_tuple #71 put
`````````````````````

Spec ``put_tuple Arity Dst``

This opcode is followed by ``Arity`` repeated ``put Value`` opcodes.
Creates an empty tuple of ``Arity`` and places pointer to it into ``Dst``.
Then moves instruction pointer forward, while next opcode is ``put``,
reads argument for every ``put`` and places it into the next tuple element.
Stops after ``Arity`` steps.

#72 badmatch
````````````
Produces an error.

#73 if_clause
`````````````

Produces an error.

#74 case_clause
```````````````

Produces an error.

#75 call_fun
````````````

Spec: ``call_fun Arity``

Calls a fun or export.
Arguments are in ``X[0..Arity-1]``.
Function object is in ``X[Arity]``.
|op-save-cp|

Raises ``badarity`` if arity does not match the function object.
Raises ``badfun`` if object is not callable (not a fun or export).

#76 make_fun
````````````

Seems to be deprecated, so compiler always generates ``make_fun2``.

#78 call_ext_only
`````````````````

Spec: ``call_ext_only Arity Destination::mfarity()``

Performs a tail recursive call to a ``Destination`` |op-mfarity-def|.
|op-no-update-cp|

#103 make_fun2
``````````````

Spec: ``make_fun2 Lambda``

Produces a callable fun object. ``Lambda`` should be resolved at load time
to a function entry. Creates a callable box object on the heap which points
to this fun object and also has space to store frozen values (Free variables).

#112 apply #113 apply_last
``````````````````````````

Spec ``apply Arity``

Calls function at ``X[Arity+1]`` in module ``X[Arity]``
with arguments ``X[0..Arity-1]``.
``Apply`` saves current instruction pointer into CP, while ``apply_last`` does not
and performs a tail recursive call.

#136 trim
`````````

Spec: ``trim N _Remaining``

Drops ``N`` words on stack after saved CP, moving it ``N`` words up.

