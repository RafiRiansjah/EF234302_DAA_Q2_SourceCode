# Q075 — Your answer

## (a) The trace

TODO(Q075): for each of the ten events, the rows shown, or "no update".

**Answer.** Row format: `id customer service status ready-time`. Default sort is by ready
time (ties by id).

| # | Event | Rows shown | View updated? |
|:-:|:--|:--|:-:|
| 0 | attach | A2 Budi WDI READY 11:00, A4 Dewi WF READY 12:00, A5 Rahmat WDI WASHING 13:00, A1 Sari WF WASHING 14:00, A6 Citra WF DRYING 15:00, A3 Ayu IR RECEIVED 16:00 | yes (1) |
| 1 | filter "ra" | A5 Rahmat WDI WASHING 13:00, A6 Citra WF DRYING 15:00 (only Rahmat and Citra contain "ra") | yes (2) |
| 2 | sort by customer | A6 Citra WF DRYING 15:00, A5 Rahmat WDI WASHING 13:00 | yes (3) |
| 3 | model receives A7 Budi | "Budi" does not contain "ra", so the rows are identical | **no update** |
| 4 | advance A5 (WASHING to DRYING) | A6 Citra WF DRYING 15:00, A5 Rahmat WDI **DRYING** 13:00 (same orders, A5's row text changed) | yes (4) |
| 5 | filter "RA" | the filter is normalised to "ra", the same as before | **no update** |
| 6 | only DRYING | A5 and A6 are both already DRYING, so the same rows | **no update** |
| 7 | advance A6 (DRYING to READY) | A5 Rahmat WDI DRYING 13:00 (A6 no longer matches DRYING) | yes (5) |
| 8 | filter cleared | with "only DRYING" still on, the only DRYING order is A5, so the same rows | **no update** |
| 9 | only READY | A2 Budi WDI READY 11:00, A6 Citra WF READY 15:00, A4 Dewi WF READY 12:00 (by customer: Budi, Citra, Dewi) | yes (6) |

Total: **6 view updates for the 10 events** (0 to 9), confirmed by running both versions.

## (c) Why compare

TODO(Q075): why compare the new rows with the old ones, and why is the view-model easy to
test?

**Answer.** *Why compare:* every event (a model change, a setting change) makes the
view-model recompute the rows, but many events do not change what the user sees: A7 does not
match the filter (event 3), `"RA"` is the same filter as `"ra"` (5), A5 and A6 are already
DRYING (6), and clearing the filter changes nothing while only DRYING is on (8). Redrawing the
widget for these would waste work and cause flicker, and in a real list could reset the scroll
position or the selection. So the view-model compares the new rows with the ones it showed
last (`rows.equals(shown)`) and calls the view only when they differ: 4 of the 10 events
were silent here.

*Why easy to test:* the view-model has no widgets and no window. The view is just a function
`Consumer<List<String>>` / a Python callable, so a test passes in a plain lambda that
collects the lists, drives the model and the three settings, and asserts on ordinary lists and
on the `updates` counter. It runs headless in milliseconds and is deterministic (ties are
broken by id); the `main` of this very file is such a test.

## (d) Chapter 9 ideas

TODO(Q075): which collection and generics ideas of Chapter 9 does the design use?

**Answer.**

- Collections: `List<Order>` / `ArrayList` for the book and the listeners (`List<Runnable>`),
  and `List.copyOf(...)` so `orders()` hands out an immutable snapshot (a defensive copy);
  Python uses a `tuple` for the same purpose.
- Comparators: a comparator chain (`comparingInt(...).thenComparing(id)`,
  `comparing(customer, String.CASE_INSENSITIVE_ORDER)`), where the id tie-break makes the sort
  deterministic. Python expresses the same as a tuple sort key, kept in a dict of key
  functions (`SORT_KEYS`).
- Generics: `Comparator<Order>`, `Consumer<List<String>>` as the view and `List<Runnable>`
  for the listeners, with the type parameters checked by the compiler; lambdas and method
  references (`this::recompute`) fill them in.
- Stream pipeline / comprehension: filter, filter, sorted, map, collect (`toList()`) in Java, and
  a list comprehension with `sorted(..., key=...)` in Python.
- Value equality of collections: `List.equals` / list `==` compares element by element, which
  is what makes the "did the rows really change" check possible.
- An enum (`Status`) for the closed set of statuses, with `null` / `None` meaning "every status".
