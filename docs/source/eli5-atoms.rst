Atoms ELI5
==========

Atom table is a global table which maps internal atom id (an integer) to a string,
and the opposite (string can be searched and internal id is retrieved. Atom
table has a hard limit (``+t`` flag for ``erl``, default is 1'048'576), if it
is ever reached, the node will crash -- this is why creating atoms dynamically
is a risky idea.

Atom is a symbol which cannot be changed. When a new atom is added to atom table,
a new unique id is assigned to it which becomes atom's hidden value on this node.
Internally atoms are just integer values which refer to atom table, this is why
manipulating atoms is cheap.

During BEAM module loading atom values are read and looked up in atom table,
atom names are replaced with their integer values, tagged as Atom
:ref:`immediate <def-immed>`, and henceforth atom integer value is used
everywhere in the code.

Externally (over network and on disk) internal values cannot be used, because
another node may have different order of atoms creation and they will gain
different numeric ids, thus externally atoms are always stored as a string.
This affects BEAM compiled file data, external pids, external ports, atoms in
Mnesia/DETS and so on. This is the reason why sometimes developers prefer very
short atom names for Mnesia record values and field names -- they will appear
as strings in database data.
