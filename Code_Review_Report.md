# Static Verification Code Review Report

## Project
- **Project:** SVV Lab Task — Static Verification of C++ Code
- **Source file:** `game.cpp`
- **Review type:** Static code review of the supplied original source
- **Status:** Findings are based on source inspection. Run a static-analysis tool and compiler to confirm tool-specific diagnostics.

## Severity guide
- **Critical:** Likely crash, memory corruption, or undefined behavior in a common execution path.
- **High:** Serious correctness or memory-safety defect.
- **Medium:** Incorrect result or defect triggered by particular inputs/states.
- **Low:** Maintainability or less impactful issue.

## Findings

| Line(s) | Category | Defect type | Severity | Finding / impact | Proposed solution |
|---:|---|---|---|---|---|
| 41 | Arrays / pointers | Array capacity overflow | Critical | `spawnMonster` writes past the fixed array when five monsters already exist. The supplied `main` attempts to spawn six. | Check `monsterCount < MAX_MONSTERS` before inserting; report failure if full. |
| 47 | Conditional | Assignment instead of comparison | High | The condition overwrites a monster ID and is treated as true for nonzero IDs, so lookup is incorrect. | Use `monsters[i]->id == id`. |
| 54 (53) | Function calls / pointers | Possible null pointer dereference | Critical | `attackMonster` dereferences the result of `findMonster` without checking it. The supplied call searches for ID 99. | Check `m == nullptr` and return a documented failure value before accessing it. |
| 55 | Initialization / arithmetic | Unsafe signed-to-unsigned conversion | High | Negative damage converts to a large unsigned value, and mixed signed/unsigned arithmetic can produce unexpected results. | Reject negative damage and use a consistent signed integer type for validated health/damage calculations. |
| 64 | Loops / arrays | Off-by-one array access | Critical | The loop reads one element beyond the number of active monsters and may read beyond the array. | Use `i < monsterCount`. |
| 70 | Function return / lifetime | Returning reference to local variable | Critical | `label` is local; returning its reference leaves a dangling reference. | Return `std::string` by value. Also check that the monster was found before using it. |
| 77 | Initialization | Uninitialized variable | High | `sum` is read before it is initialized, making the result indeterminate. | Initialize it as `double sum = 0.0;`. |
| 80 | Arithmetic | Division by zero | High | Calling `averageAttack` when there are no monsters divides by zero. | Handle `monsterCount == 0` before dividing. |
| 85 | Pointers / memory | Use after free | Critical | `defeatMonster` deletes `m` and then reads `m->name`. It also leaves the pointer in the array, risking another delete in the destructor. | Save the name before deletion, remove the entry from the active array, shift remaining entries, and decrement the count. Check for a missing ID. |
| 98 (94) | Conditional / return | Missing return path | High | Scores of 100 or less reach the end of a non-void function without returning a value. | Add a defined return for every possible score. |
| 102 | Pointers / buffers | Unsafe fixed-size input buffer | Critical | Unbounded input into a 10-character allocation can overflow the buffer. | Use `std::string name` and read into it safely. |
| 108 | Memory management | Mismatched allocation/deallocation | High | The buffer was allocated with `new[]` but is released with scalar `delete`. | If retaining an array allocation, use `delete[]`; preferably use `std::string` and avoid manual allocation. |
| 91 | Arithmetic | Integer division / possible invalid values | Medium | Both operands are integers, so fractional damage is truncated before conversion to `double`. | Convert an operand to `double` before division and validate the allowed range of armor percentage. |
| 121 | Pointers / memory | Memory leak | Medium | The allocated `bonus` object is never deleted. | Use an automatic object or `std::unique_ptr`; ensure ownership is explicit. |
| 133 | Classes / object lifetime | Shallow copy of owning pointers | Critical | The implicit copy constructor copies raw pointers. Both `Game` objects then own the same monsters, risking double deletion and use-after-free. | Implement a deep-copy constructor/assignment operator or delete copying; consider storing smart pointers/objects. |
| 119 | Function call / capacity | Caller exceeds capacity | Critical | The source creates six monsters although `MAX_MONSTERS` is five. This exposes the capacity defect during normal startup. | Enforce capacity inside `spawnMonster` and adjust the caller or storage design. |
| 54 (83) | Function calls / pointers | Missing not-found handling | High | If the requested ID is absent, `m` is null; deletion and subsequent member access are unsafe. | Return safely when `findMonster` returns `nullptr`. |

## Category summary

| Category | Main issues |
|---|---|
| Initialization | Uninitialized `sum` |
| Pointers, allocation, deletion, arrays and buffers | Capacity overflow, null dereferences, dangling reference, unsafe buffer, mismatched delete, memory leak, use-after-free, shallow copy |
| Loops | Off-by-one loop in `totalMonsterHealth` |
| Arithmetic | Signed/unsigned conversion, division by zero, integer division |
| Conditional statements | Assignment in condition; incomplete rank return paths |
| Function calls | Missing not-found checks and capacity validation |
| Classes | Raw-pointer ownership and unsafe implicit copying |

## Recommended verification procedure

1. Preserve this original file as the baseline for the lab.
2. Run a compiler with warnings enabled, for example:
   ```bash
   g++ -std=c++17 -Wall -Wextra -Wpedantic -Wconversion -Wshadow game.cpp -o game
   ```
3. Run a static analyzer such as Cppcheck:
   ```bash
   cppcheck --enable=all --inconclusive --language=c++ game.cpp
   ```
4. Record the actual diagnostics, tool version, and screenshots in the submission. Static analysis may not report every defect listed here.
5. Fix defects on a separate branch only if the instructor asks for a corrected version. Keep this original source unchanged for evidence of the initial review.

## Conclusion

The source has multiple high-risk correctness and memory-safety defects. The most urgent are the fixed-array overflow, invalid lookup condition, null pointer dereferences, out-of-bounds loop, dangling reference, use-after-free, unsafe input buffer, and shallow copying of owned pointers. These should be addressed before the program is run as a corrected application.
