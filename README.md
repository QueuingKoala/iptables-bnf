iptables-restore BNF Readme
====

This document is designed to be used along with the included BNF to describe the
`iptables-save(8)` syntax, usable as input to `iptables-restore(8)`. Consult the
BNF for the grammar rules.

Overview
----

The restore-file is composed of table definitions; each table definition
contains one or more chain definitions and rules. These rules are loaded into
the Netfilter state impacting the included tables. Note that tables not listed
in the restore-file are left alone; if not loaded into a running kernel they
will not be loaded, and if loaded will not be modified.

Combined with the BNF grammar description, this document explains how to define
a ruleset. Consult `iptables(8)` for general Netfilter information and
`iptables-extensions(8)` for specific match/target module operation. Both of
these topics are outside the scope of the restore-file description.

Modes of operation
----

Normally `iptables-restore` will flush each table listed in the ruleset before
applying the rules, but this can be avoided by adding the `-n` option. This will
leave the existing ruleset in place, running the commands in the ruleset on top
of the currently loaded Netfilter state.

It is possible to load only a specific table from a restore-file that may
contain other tables; pass the `-t table` option, where `table` is the table
name to load; all other tables defined in the restore-file are ignored.

Additionally, the hit counters are not restored unless the `-c` flag is used.

Syntax elements
----

Each line in a restore-file can be one of the following types; consult the BNF
for precise syntax requirements.

 * Comment-lines start with the `#` character and are ignored; leading
   whitespace is allowed
 * Lines that are blank or contain only whitespace are ignored
 * A table definition: see `<table-def>` in the BNF
 * A chain definition: see `<chain-def>` in the BNF
 * A command/rule: see `<rule>` in the BNF
 * The end of a table definition, which is the keyword `COMMIT`

Table definitions
----

A restore-file contains one or more tables, and each table is introduced with a
table definition. The table must be a valid type the kernel knows about;
provided kernel module auto-loading is enabled, the kernel will attempt to load
missing modules when a new table is referenced.

While it is possible to specify the same table twice, this will implicitly flush
the table both times unless the `-n` flag was passed (see above) and is best
avoided as a result.

Chain definitions
----

All chains used in a table must be created first, generally by a chain
definition; built-in chains are implicitly included, but when omitted from a
table definition the *existing* policy on the chain is preserved.

It is also possible to preserve the existing policy when defining a chain by
setting the policy to the character `-` (dash.) Otherwise a policy of `ACCEPT`
or `DROP` must be specified. Note that only the built-in chains may have target
policies; user-defined chains must have the dash character as the chain policy.

The hitcounter is optional, and ignored unless `-c` is passed to
`iptables-restore`.

Note that you can use the `-N` command to create a new chain, but this is best
avoided; calling `-N` on a chain that already exists is an error that will cause
the load to fail. It is best to use the chain definition syntax instead.

Rules, or table commands
----

Following the chain definitions, commands to apply to the table are given, each
on 1 line. Frequently, the only command used is the `-A` command, but the full
description of each command can be found in `iptables(8)`.
