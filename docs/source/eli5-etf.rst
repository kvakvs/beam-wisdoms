External Term Format ELI5
=========================

To communicate with the outer world and other Erlang nodes, also to save things
on disk, Erlang uses special encoding --- External Term Format.

The main requirements to the format:

*   Easy for human and machine to see an encoded term and what it is. An external
    term always starts with byte 131 (``0x83``).
*   Compact with optional compression.
    A tag 131 (``0x83``), 80 (``0x50``), U32/big length marks a compressed term.
*   Cross-platform, all integers are encoded as
    `big endian <https://en.wikipedia.org/wiki/Endianness>`_ (from most
    significant bytes to least),
    floats use big endian
    `IEEE encoding <https://en.wikipedia.org/wiki/IEEE_floating_point>`_,
    text uses UTF-8 or ISO-8859-1 (ASCII superset).

To encode a term see ``erlang:term_to_binary`` and the opposite operation is
``erlang:binary_to_term``.
Decoding handcrafted binary also allows constructing terms which are
impossible to make otherwise, like ports and pids.

Usually atoms are encoded as strings.
In distributed mode over the network, to further reduce the protocol size,
a distribution header can be added.
It contains table of atom names which are included in the encoded message.
Then atoms in the message are replaced with indexes in this table.

It is impossible to encode internal VM value types which never appear in
Erlang programs, such as ``THE_NON_VALUE``, references to registers, CP etc.

See detailed format description in the
`ETF documentation <http://erlang.org/doc/apps/erts/erl_ext_dist.html>`_.
