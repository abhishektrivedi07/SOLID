# SOLID Principles — Overview

SOLID is a set of 5 design principles that make software easier to understand, maintain, and extend.

Each letter stands for one principle:

| Letter | Principle | One-Line Summary |
|--------|-----------|-----------------|
| **S** | Single Responsibility | A class should do **one thing** |
| **O** | Open/Closed | Open for **extension**, closed for **modification** |
| **L** | Liskov Substitution | Subclasses should **behave like** their parent |
| **I** | Interface Segregation | Don't force classes to implement what they **don't need** |
| **D** | Dependency Inversion | Depend on **abstractions**, not concrete classes |

## Why Should You Care?

Bad code that ignores SOLID looks like this after 6 months:
- Adding a feature breaks 3 other things
- Changing one class requires editing 10 files
- Unit testing is impossible because everything is coupled
- New engineers are afraid to touch the code

SOLID code is:
- Easy to test (each piece is isolated)
- Easy to change (changes stay local)
- Easy to extend (add without breaking existing code)
- Easy to understand (each piece has one clear job)

## Files in This Directory

```
01-single-responsibility.md
02-open-closed.md
03-liskov-substitution.md
04-interface-segregation.md
05-dependency-inversion.md
```

Start with `01-single-responsibility.md`.
