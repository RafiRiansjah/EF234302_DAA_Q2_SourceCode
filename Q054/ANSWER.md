# Q054 — Your answer

## (a) The eight lines

TODO(Q054): for each of the eight lines, say whether it compiles and why
(totalKg(shirts); totalKg(anything); addSpareTowels(basket, 2); addSpareTowels(bathTowels, 1);
transfer(bathTowels, towels); transfer(towels, shirts); List<Garment> g = shirts;
view.add(new Shirt("x"))).

**Answer.** `Garment <- Shirt`, `Garment <- Towel <- BathTowel`; generics are invariant, so
`List<Shirt>` is **not** a `List<Garment>`; a wildcard is needed to relate them.

| # | Line | Compiles? | Why |
|:-:|:--|:-:|:--|
| 1 | `totalKg(shirts)` | yes | `List<Shirt>` is a `List<? extends Garment>`: every element is at least a `Garment`. |
| 2 | `totalKg(anything)` | **no** | `Object` is not a subtype of `Garment`, so `List<Object>` does not fit `? extends Garment`. |
| 3 | `addSpareTowels(basket, 2)` | yes | `Garment` is a supertype of `Towel`, so `List<Garment>` fits `? super Towel`; a `Towel` can be put in. |
| 4 | `addSpareTowels(bathTowels, 1)` | **no** | `BathTowel` is a *subtype* of `Towel`, not a supertype; the method would put a plain `Towel` into a `List<BathTowel>`. |
| 5 | `transfer(bathTowels, towels)` | yes | `T = Towel` works: `List<BathTowel>` is a `Collection<? extends Towel>` and `List<Towel>` is a `Collection<? super Towel>`. |
| 6 | `transfer(towels, shirts)` | **no** | We would need `Towel <: T` (source) and `T <: Shirt` (destination, `? super T` must accept `Shirt`'s list), and no type is both. |
| 7 | `List<Garment> g = shirts;` | **no** | Generics are invariant: `List<Shirt>` is not a `List<Garment>`, otherwise we could add a `Towel` through `g`. |
| 8 | `view.add(new Shirt("x"))` | **no** | `view` is `List<? extends Garment>`, really a list of some unknown subtype (here `Towel`); the compiler cannot know a `Shirt` is allowed, so only `null` can be added. Reading (`get`) and `clear()` are fine, as `main` shows. |

## (c) The rule

TODO(Q054): state the PECS rule and apply it to `transfer`.

**Answer.** **PECS: Producer Extends, Consumer Super.** If a structure only *produces* values
we read (`T` comes out), use `? extends T`; if it only *consumes* values we write (`T` goes
in), use `? super T`; if we do both, use the exact type with no wildcard. Inside the method the
wildcard enforces this: through `? extends` we may read but not add (except `null`); through
`? super` we may add a `T` but read only `Object`.

Applied to `transfer`: `src` only gives items away (producer), so it is
`Collection<? extends T>`; `dst` only receives them (consumer), so it is
`Collection<? super T>`. `T` is the common ground chosen by the compiler (`Towel` in call 5,
`Garment` for `transfer(shirts, basket)`); call 6 has no such `T`. `src.clear()` is allowed
because it does not involve the element type. Likewise `totalKg` only reads (`extends`)
and `addSpareTowels` only writes (`super`).

## (d) Python

TODO(Q054): which calls does mypy reject, and how do you express `? super Towel` in Python
(variance: covariant / contravariant / invariant)?

**Answer.** Checked with `mypy` (the file with the eight lines is not part of the submission;
its output is):

```python
total_kg(shirts)                       # (1) ok
total_kg(anything)                     # (2) rejected
add_spare_towels(basket, 2)            # (3) rejected (MutableSequence[Towel] is invariant)
add_spare_towels_anywhere(basket, 2)   # (3) ok, Sink[Towel] is contravariant
add_spare_towels(bath_towels, 1)       # (4) rejected
transfer(bath_towels, towels)          # (5) ok
transfer(towels, shirts)               # (6) rejected
g: List[Garment] = shirts              # (7) rejected
```

```
error: Argument 1 to "total_kg" has incompatible type "list[object]"; expected "Sequence[Garment]"  [arg-type]
error: Argument 1 to "add_spare_towels" has incompatible type "list[Garment]"; expected "MutableSequence[Towel]"  [arg-type]
error: Argument 1 to "add_spare_towels" has incompatible type "list[BathTowel]"; expected "MutableSequence[Towel]"  [arg-type]
error: Cannot infer value of type parameter "T" of "transfer"  [misc]
error: Incompatible types in assignment (expression has type "list[Shirt]", variable has type "list[Garment]")  [assignment]
note: "list" is invariant -- see https://mypy.readthedocs.io/en/stable/common_issues.html#variance
```

So `mypy` rejects the same lines as `javac` (2, 4, 6, 7; and 3 only when written with the
invariant `MutableSequence[Towel]`) and accepts 1 and 5. Line 8 corresponds to
`view.append(...)` on a `Sequence[Garment]`, which has no `append`, so mypy rejects it too.

`? super Towel` has no wildcard syntax in Python; it is expressed with **variance**:

- **covariant** (`? extends`): a read-only producer such as `Sequence[Garment]` or
  `Iterable[T]` accepts `Sequence[Shirt]`;
- **contravariant** (`? super`): a write-only consumer. We declare a `Protocol` with a
  contravariant `TypeVar("T_contra", contravariant=True)` that has only `append(item: T_contra)`;
  `Sink[Towel]` then accepts a `List[Garment]` (anything that can take a `Towel`);
- **invariant**: something both read and written, such as `List[T]`; `List[Shirt]` is
  not a `List[Garment]`.

At run time nothing is checked: `transfer([Towel("rogue")], only_shirts)` runs and puts a
`Towel` into a `List[Shirt]`, as `main` shows. The type hints help only when mypy is run.
