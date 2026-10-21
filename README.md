# SVV Lab Task: Static Verification of C++ Code

This repository contains the original `game.cpp` source for a text-based monster battle game and a static code review report.

## Files

- `game.cpp` — original source supplied for the static-verification exercise. It intentionally retains the defects for review.
- `Code_Review_Report.md` — review findings with source line numbers, categories, severity, impact, and proposed solutions.

## Review scope

The review checks initialization, pointers and memory management, arrays and buffers, loops, arithmetic, conditional statements, function calls, and class/object-lifetime behavior.

## Suggested tools

- A C++ compiler such as GCC
- Compiler warning flags
- Cppcheck
- GitHub Copilot for explaining diagnostics and proposing fixes

## Run static checks

With GCC installed:

```bash
g++ -std=c++17 -Wall -Wextra -Wpedantic -Wconversion -Wshadow game.cpp -o game
```

With Cppcheck installed:

```bash
cppcheck --enable=all --inconclusive --language=c++ game.cpp
```

The supplied program is intentionally defective and may crash or exhibit undefined behavior. Do not treat successful compilation as proof of correctness. Record actual tool output and screenshots for the lab submission; the report is based on source inspection and does not claim that the tools have already been run.

## Workflow

1. Review the original source and report.
2. Run static-analysis tools and record their actual findings.
3. If required by the instructor, create a separate branch for fixes.
4. Re-run checks after changes and document the results.
