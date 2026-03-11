# PhantomOperator

High-performance absent-aware n-dimensional arrays powered by [NumPy](https://numpy.org/) and [PhantomTrace](https://pypi.org/project/PhantomTrace/).

## Install

```bash
pip install phantomoperator
```

This automatically installs NumPy and PhantomTrace as dependencies.

## What It Does

PhantomOperator brings NumPy-level performance to PhantomTrace's absence calculus. Instead of looping through Python lists of individual `AbsentNumber` objects, operations run on compact NumPy arrays — two arrays side by side: one for values, one for states (present and absent).

## Important: Void Is Not a State

Void is **not** a state. Present is a state. Absent is a state. Void is **absolute nothingness** — it means there is nothing at that position. No number, no state, no calculation, no meaning. It is the complete absence of anything to talk about.

When you subtract 5 by 5, nothing is left. That's not zero — zero is a real number (`1(0)`, one unit of absence). That's void. Subtraction removes the value and forgets it — there is simply nothing there anymore. Erasure is different: erasing 5 by 5 gives `5(0)` (calculable absence). The value is still there, just flipped to the other state. Erasure remembers what it removed.

PhantomOperator allows void inside arrays and handles it in operations so that **everything works and there are no errors**. But void carrying through an operation does not mean anything is being computed. It means: "there was nothing here, so there is still nothing here." If you can avoid operations involving void, avoid them. They don't produce meaningful results — they just keep things from breaking.

Think of it this way:
- `void + 5 = 5` — Adding nothing to 5 is just 5. Void didn't contribute anything.
- `void × 5 = void` — Multiplying by nothing produces nothing. There was no quantity to scale.
- `void - void = void` — Nothing minus nothing is still nothing.

Void exists in PhantomOperator as a practical necessity, not as a mathematical participant. The real math happens between present and absent numbers. Void is what's left when the math is done and nothing remains.

### Void vs Zero vs Absence

These three are fundamentally different things:

| | What it is | Example | Has a state? | Has magnitude? |
|---|---|---|---|---|
| **Void** | Absolute nothingness. No number exists here. | Result of subtracting 5 by 5 | No | No |
| **Zero** (`0`) | One unit of absence. A real number: `1(0)` | The number zero on a number line | Yes (absent) | Yes (1) |
| **`3(0)`** | Three units of absence. A real quantity in the absent state. | Three absent things | Yes (absent) | Yes (3) |

Zero is not void. Zero is a real, meaningful number with a state and a magnitude. Void is nothing at all.

## Quick Start

```python
from phantom_operator import AbsentArray, absent_array, arange, ones
from phantom_operator import arr_add, arr_subtract, arr_multiply, arr_divide, arr_erase, arr_toggle
from phantom_operator import arr_combine, arr_compare, arr_trace

a = absent_array([1, 2, 3, 4, 5])                          # all present
b = absent_array([1, 2, 3, 4, 5], states=[0, 1, 0, 1, 0])  # mixed states

c = arange(1, 10)          # [1, 2, 3, ..., 10] all present
d = arange(1, 10, state=1) # [1, 2, 3, ..., 10] all absent
e = ones(1000)             # 1000 ones, all present
```

## Function Naming

PhantomOperator uses `arr_` prefixed function names (`arr_add`, `arr_multiply`, etc.) so they don't collide with PhantomTrace's `add`, `multiply`, etc. This way you can use both libraries together without conflicts:

```python
from absence_calculator import add, multiply, n          # scalar operations
from phantom_operator import arr_add, arr_multiply       # array operations

scalar_result = add(n(5), n(3))                           # → 8
array_result = arr_add(absent_array([5, 3]), absent_array([2, 4]))  # → [7, 7]
```

## Operations

All five PhantomTrace operations work element-wise on arrays.

Void is handled in every operation so nothing breaks, but remember: void is not a real operand. Operations involving void don't compute anything meaningful — they just propagate nothingness so you don't get errors.

### Addition & Subtraction

Same-state elements combine magnitudes. When void appears, it acts as an identity for addition (adding nothing changes nothing), but this is not a real calculation:

```python
a = absent_array([5, 3, 7], states=[0, 0, 1])
b = absent_array([2, 3, 3], states=[0, 0, 1])

arr_add(a, b)       # → [7, 6, 10(0)]
arr_subtract(a, b)  # → [3, void, 4(0)]   (3 - 3: nothing left, that's void)
```

### Multiplication & Division

State combination rule: present × present = present, present × absent = absent, absent × absent = present.

When void appears in multiplication, the result is void — you can't scale nothingness. This isn't a mathematical rule; it's just what "nothing" means:

```python
a = absent_array([5, 4, 6], states=[0, 1, 1])
b = absent_array([3, 3, 2], states=[0, 0, 1])

arr_multiply(a, b)  # → [15, 12(0), 12]  (absent × absent = present)
arr_divide(a, b)    # → [1, 1(0), 3]
```

### Erasure

Erasure flips the state of the erased portion. Returns a dict with `remainder`, `erased`, and `excess`.

Unlike subtraction, erasure does not produce void when the values are equal. Erasing 5 by 5 gives `5(0)` — the value flips state but is still there. The remainder is void (there is nothing left un-erased), but the erased portion is calculable absence, not void. When there's no over-erasure, the excess is void — there is no excess to report.

The erasure identity: **y erased x erased x(0) = y**. Erasing x from y, then erasing the flipped x from the result, gives you back y. The erasure undoes itself.

```python
a = absent_array([7, 5, 3])
b = absent_array([3, 5, 1])

result = arr_erase(a, b)
result['remainder']  # → [4, void, 2]         4 left, nothing un-erased, 2 left
result['erased']     # → [3(0), 5(0), 1(0)]   flipped state — calculable absence
result['excess']     # → [void, void, void]    no over-erasure at any position
```

### Toggle

`arr_toggle()` flips the state of every element. Void elements stay void — you can't flip the state of something that doesn't exist:

```python
a = absent_array([1, 2, 3], states=[0, 1, 0])
arr_toggle(a)  # → [1(0), 2, 3(0)]
```

Toggle is closely related to erasure. Toggling an element is equivalent to fully erasing it by its own value — the remainder is void (nothing left un-erased), but the erased portion is `5(0)` (calculable absence, not void). The value is still there, just in the opposite state. So toggling `5` (present) gives `5(0)` (absent): same value, opposite state. This is why erasure and subtraction are different — subtraction would give void (forgotten), but erasure gives calculable absence (remembered in the other state).

PhantomOperator's `arr_toggle()` does this for every element at once across the entire array using vectorized NumPy operations.

### Combine

`arr_combine()` analyzes the state relationship between two arrays element-wise:

```python
from absence_calculator import n

a = absent_array([n(1), n(2)], states=[0, 0])
b = absent_array([n(3), n(4)(0)], states=[0, 1])

arr_combine(a, b)  # → [2, 1]   same state: 2, different state: 1
```

When void appears, combine returns the state info of the non-void element. But this isn't a real combination — there was nothing on the void side to combine with.

### Compare

`arr_compare()` compares how magnitude is distributed between present and absent states. **It only works on two arrays that have the same total magnitude at each position.** This makes it ideal for use with combined results where the total is guaranteed to be equal — the compare then tells you the shift in present content.

If the magnitudes don't match, compare raises an error. If one position is void and the other isn't, that's also an error — void has no magnitude to compare.

When magnitudes match, compare returns:
- The difference as **present** (y has more present content)
- The difference as **absent** (x has more present content)
- **Void** when the present content is equal (no difference to report)

```python
a = absent_array([5, 3, 7], states=[0, 1, 0])
b = absent_array([5, 3, 7], states=[1, 0, 0])

arr_compare(a, b)  # → [5(0), 3, void]
# Position 0: same magnitude (5), x is present, y is absent → x has 5 more present → 5(0)
# Position 1: same magnitude (3), x is absent, y is present → y has 3 more present → 3
# Position 2: both present with same value → no difference → void
```

Mismatched magnitudes are rejected:

```python
arr_compare(absent_array([5]), absent_array([3]))
# → ValueError: total magnitudes differ (5 vs 3)
```

## Trace

`arr_trace()` evaluates a function over a range of AbsentNumbers and builds the results directly into an AbsentArray. This is the array version of PhantomTrace's `trace()`.

Same-state ranges iterate by magnitude. Cross-state ranges require an explicit ordering (present and absent are different states of the same magnitude — they aren't ordered against each other).

```python
from absence_calculator import n, trace, present, absent, largest, ordering
from phantom_operator import arr_trace

result = arr_trace(lambda x: x, n(1), n(5))
# → AbsentArray([1, 2, 3, 4, 5])

result = arr_trace(lambda x: x, n(3)(0), n(1)(0))
# → AbsentArray([3(0), 2(0), 1(0)])

order = ordering(largest(present()), largest(absent()))
result = arr_trace(lambda x: x, n(3), n(3)(0), order=order)
# → AbsentArray([3, 2, 1, 1(0), 2(0), 3(0)])
```

The function receives each AbsentNumber in the range and can return any AbsentNumber or Void result:

```python
from absence_calculator import erase

result = arr_trace(lambda x: erase(x, n(2)), n(1), n(5))
# Each element is erased by 2 — results collected into an AbsentArray
```

## Array Features

### Shapes & Dimensions

```python
a = absent_array([[1, 2, 3], [4, 5, 6]])
a.shape   # (2, 3)
a.ndim    # 2
a.size    # 6
```

### Indexing

```python
a = absent_array([10, 20, 30, 40, 50])
a[0]      # AbsentNumber: 10
a[1:3]    # AbsentArray([20, 30])
```

Indexing a void position returns PhantomTrace's `Void` object — because there is nothing there:

```python
result = arr_subtract(absent_array([5]), absent_array([5]))
result[0]  # Void — subtraction forgot the value entirely
```

### Conversion

```python
from absence_calculator import n

a = absent_array([n(5), n(3)(0), n(7)])
a.to_list()  # → [AbsentNumber(5), AbsentNumber(3, 1), AbsentNumber(7)]
```

Void positions in `to_list()` return PhantomTrace's `Void` object.

### Querying State

```python
from phantom_operator import arr_count_present, arr_present_mask, arr_absent_mask, arr_void_mask

a = absent_array([n(1), n(2)(0), VOID, n(4), n(5)(0)])

arr_count_present(a)  # 2
arr_present_mask(a)   # [True, False, False, True, False]
arr_absent_mask(a)    # [False, True, False, False, True]
arr_void_mask(a)      # [False, False, True, False, False]
```

`arr_void_mask()` tells you which positions are void — which positions have nothing. This is useful for knowing where real data exists and where there is nothing to work with.

## Performance

PhantomOperator is built on NumPy's C arrays, so operations on large arrays are orders of magnitude faster than looping through individual AbsentNumbers:

```python
from phantom_operator import arange, arr_multiply
import time

a = arange(1, 100000)
b = arange(1, 100000)

start = time.time()
result = arr_multiply(a, b)  # 100,000 multiplications
elapsed = time.time() - start
# Typically < 2ms vs seconds with plain Python lists
```

## Dependencies

- [NumPy](https://numpy.org/) — array computation engine
- [PhantomTrace](https://pypi.org/project/PhantomTrace/) >= 0.8.0 — absence calculus framework

## License

MIT
