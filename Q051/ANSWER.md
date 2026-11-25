# Q051 — Your answer

## (d) HashMap order vs Python dict order

TODO(Q051): explain why printing `map.keySet()` from a `HashMap` may come out in a
different order on another machine or Java version, and what the Python `dict`
guarantees instead (and whether that guarantee is alphabetical).

**Answer.**

A `HashMap` does not store its keys in any order the programmer controls. It puts each
entry into a bucket chosen from the key's `hashCode()` (spread, then masked by the table
size), and `keySet()` simply walks the buckets from index 0 upwards. The `HashMap`
documentation explicitly makes no promise about the iteration order, not even that it stays
the same over time. So the order is a side effect of things that are not part of the
program's logic:

- the hash function and the bucket-spreading code, which the JDK is free to change
  between versions (Java 8 changed the internals of `HashMap`);
- the current table capacity, which depends on how many entries were added and when the
  table was resized;
- the `hashCode()` of the keys: for `String` it is fixed by the specification, but for
  keys that use the default identity hash code it differs from run to run, and the
  immutable `Set.of` / `Map.of` (Java 9+) deliberately randomise their order per JVM run.

In our program the `HashMap` printed `[SH, IR, WDI, WF]` while we inserted `WF, IR, WDI, SH`
and the `TreeMap` printed `[IR, SH, WDI, WF]`: three different orders from the same data.
A report that depends on it can therefore change when another machine or Java version
is used. The fix is to choose a map that has an order: `TreeMap` for sorted by code, or
`LinkedHashMap` for first-seen order (or sort the keys explicitly before printing).

A Python `dict` guarantees **insertion order**: the language specification (since Python
3.7) says iteration follows the order in which the keys were first inserted, on every
machine and every interpreter. That guarantee is **not alphabetical**: our `dict` lists
`['WF', 'IR', 'WDI', 'SH']` (first-seen order), and for alphabetical order we still have to
write `sorted(count)`.
