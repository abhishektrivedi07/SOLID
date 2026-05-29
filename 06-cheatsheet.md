# SOLID Principles — Cheatsheet

## One-Line Summary of Each

| Principle | Rule | Violation Sign | Fix |
|-----------|------|----------------|-----|
| **S**ingle Responsibility | One class, one job | "Why would this change?" has 2+ answers | Split into focused classes |
| **O**pen/Closed | Extend without modifying | Adding a type means editing existing code | Use abstract class + new subclass |
| **L**iskov Substitution | Subclass = drop-in replacement | Subclass throws, returns None, or no-ops | Redesign hierarchy around behavior |
| **I**nterface Segregation | No forced irrelevant methods | `pass` or `raise NotImplementedError` in implementation | Split fat interface into small ones |
| **D**ependency Inversion | Depend on abstractions | `self.x = ConcreteClass()` hardcoded inside | Inject dependency via constructor |

---

## The Mental Test for Each

```
S — Can I describe what this class does in one sentence without "and"?
O — Can I add a new type without editing existing classes?
L — If I swap the subclass in, does everything still work correctly?
I — Does every method in this interface apply to every implementor?
D — Can I swap the concrete implementation without changing the high-level class?
```

If any answer is **No** — you're violating that principle.

---

## How They Work Together

```
S  →  Creates small, focused classes
O  →  Those classes extend via new subclasses (not edits)
L  →  Subclasses are truly interchangeable
I  →  Interfaces only contain what each implementor actually uses
D  →  High-level classes receive these interfaces, not concrete classes
```

They reinforce each other. A well-designed system usually satisfies all five.

---

## Common Anti-Patterns and Which Principle They Violate

| Anti-Pattern | Principle Violated |
|--------------|-------------------|
| "God class" that does everything | S |
| Long `if/elif` checking a type string | O |
| Subclass method throws `NotImplementedError` | L or I |
| Empty `pass` methods in a subclass | I |
| `self.db = MySQLDatabase()` hardcoded inside a service | D |
| Importing a concrete class inside a high-level module | D |
| Class named `UserManagerAndReportGeneratorAndEmailSender` | S |

---

## Quick Refactoring Guide

**Spotting SRP violation:**
- Class has more than one "section" of unrelated methods
- Class name has "And" in it

**Spotting OCP violation:**
- `if type == "x": ... elif type == "y": ...` chains
- "Every time we add a feature, we edit this file"

**Spotting LSP violation:**
- Subclass overrides method with empty body or exception
- Code has `if isinstance(x, SubClass): ...` checks

**Spotting ISP violation:**
- Interface has methods that some implementors always leave empty
- You implement an interface but only use 2 of its 10 methods

**Spotting DIP violation:**
- Can't unit test a class without setting up a real database
- Switching a dependency requires editing the class itself

---

## Files in This Directory

```
00-overview.md              ← Start here (you are here)
01-single-responsibility.md ← S: One class, one job
02-open-closed.md           ← O: Extend don't modify
03-liskov-substitution.md   ← L: Subclasses must behave
04-interface-segregation.md ← I: Small focused interfaces
05-dependency-inversion.md  ← D: Depend on abstractions
06-cheatsheet.md            ← This file
```
