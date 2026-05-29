# L — Liskov Substitution Principle (LSP)

## The Idea

> If S is a subclass of T, you should be able to use S anywhere T is used — **without breaking the program**.

In plain English: **a subclass must behave like its parent class**. If you swap in a subclass, the program should not break, crash, or behave unexpectedly.

Named after Barbara Liskov, who introduced it in 1987.

---

## The Simplest Analogy

Think of a **penguin and a bird**.

All birds can `fly()`, right? So `Penguin extends Bird` should work.  
But if your program calls `bird.fly()` and it suddenly crashes because it got a penguin — you've violated LSP.

A penguin **IS** a bird biologically, but it's not a substitutable bird in code if your program depends on flying.

The fix: model what's actually substitutable, not what's biologically true.

---

## The Classic Violation: Square extends Rectangle

This is the most famous LSP violation.

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def set_width(self, width):
        self.width = width

    def set_height(self, height):
        self.height = height

    def area(self):
        return self.width * self.height


class Square(Rectangle):
    # A square must always have equal sides
    def set_width(self, width):
        self.width = width
        self.height = width   # ← forces height to match

    def set_height(self, height):
        self.height = height
        self.width = height   # ← forces width to match
```

This seems logical — a square IS a rectangle geometrically.

**But watch what happens:**

```python
def stretch_rectangle(rect: Rectangle):
    rect.set_width(10)
    rect.set_height(5)
    print(f"Area: {rect.area()}")   # Expected: 50

r = Rectangle(3, 3)
stretch_rectangle(r)    # Area: 50  ✓

s = Square(3)
stretch_rectangle(s)    # Area: 25  ✗ ← WRONG! Should be 50, got 25
```

`Square` broke the behavior `Rectangle` promised. Substituting a `Square` where a `Rectangle` is expected broke the program.

**LSP violated.**

---

## What LSP Actually Requires

A subclass must:
1. **Keep all the promises** the parent made (postconditions)
2. **Not add stricter demands** the parent didn't have (preconditions)
3. **Not throw exceptions** the parent doesn't throw
4. **Not return unexpected values** that callers don't expect

---

## Bad Example — Penguin Can't Fly

```python
class Bird:
    def fly(self):
        print("I'm flying!")

    def eat(self):
        print("Eating...")


class Eagle(Bird):
    def fly(self):
        print("Eagle soaring!")   # ✓ works fine


class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins can't fly!")   # ✗ BREAKS callers

# Code that uses Bird:
def make_bird_fly(bird: Bird):
    bird.fly()   # ← CRASHES if bird is a Penguin

make_bird_fly(Eagle())    # Eagle soaring!  ✓
make_bird_fly(Penguin())  # Exception!      ✗
```

`Penguin` cannot substitute for `Bird` when flying is involved.

---

## Good Example — Fix by Redesigning the Hierarchy

Don't force a shared interface that not all subclasses can honor. Split it:

```python
from abc import ABC, abstractmethod


class Bird(ABC):
    @abstractmethod
    def eat(self) -> str:
        pass

    @abstractmethod
    def move(self) -> str:
        pass


class FlyingBird(Bird):
    def move(self) -> str:
        return self.fly()

    @abstractmethod
    def fly(self) -> str:
        pass


class SwimmingBird(Bird):
    def move(self) -> str:
        return self.swim()

    @abstractmethod
    def swim(self) -> str:
        pass


class Eagle(FlyingBird):
    def fly(self) -> str:
        return "Eagle soaring high!"

    def eat(self) -> str:
        return "Eagle eating fish."


class Penguin(SwimmingBird):
    def swim(self) -> str:
        return "Penguin swimming fast!"

    def eat(self) -> str:
        return "Penguin eating krill."


# Now any Bird can safely be asked to move() and eat()
def observe_bird(bird: Bird):
    print(bird.eat())
    print(bird.move())

observe_bird(Eagle())    # ✓ No crashes
observe_bird(Penguin())  # ✓ No crashes
```

---

## Working Example — Document Export

Suppose you have a document system that exports files:

### ❌ Bad — Subclass Breaks Parent's Contract

```python
class Document:
    def __init__(self, content: str):
        self.content = content

    def export(self) -> str:
        return self.content

    def set_content(self, content: str):
        self.content = content


class ReadOnlyDocument(Document):
    def set_content(self, content: str):
        raise PermissionError("This document is read-only!")  # ← breaks LSP


def update_and_export(doc: Document, new_content: str) -> str:
    doc.set_content(new_content)    # ← CRASHES on ReadOnlyDocument
    return doc.export()

doc = Document("Original")
print(update_and_export(doc, "Updated"))   # Works ✓

readonly = ReadOnlyDocument("Fixed text")
print(update_and_export(readonly, "..."))  # PermissionError ✗
```

### ✅ Good — Hierarchy Reflects Actual Behavior

```python
from abc import ABC, abstractmethod


class BaseDocument(ABC):
    def __init__(self, content: str):
        self._content = content

    @abstractmethod
    def export(self) -> str:
        pass


class EditableDocument(BaseDocument):
    def export(self) -> str:
        return self._content

    def set_content(self, content: str):
        self._content = content


class ReadOnlyDocument(BaseDocument):
    def export(self) -> str:
        return self._content
    # No set_content — it simply doesn't support it, and callers know that


# Functions declare what they actually need
def export_any_document(doc: BaseDocument) -> str:
    return doc.export()   # ✓ works for both — LSP safe

def update_document(doc: EditableDocument, content: str) -> str:
    doc.set_content(content)
    return doc.export()   # ✓ only called with editable documents


# --- Usage ---
editable = EditableDocument("Draft content")
readonly = ReadOnlyDocument("Published content")

print(export_any_document(editable))   # Draft content
print(export_any_document(readonly))   # Published content

print(update_document(editable, "Updated draft"))  # Updated draft
# update_document(readonly, "...")  ← type error at development time ✓
```

The type system now protects you. You can't accidentally call `update_document` with a `ReadOnlyDocument`.

---

## Working Example — Notification System

```python
from abc import ABC, abstractmethod


class Notifier(ABC):
    """All notifiers can send a message to a user."""
    @abstractmethod
    def send(self, user_email: str, message: str) -> bool:
        """Returns True if sent successfully, False otherwise."""
        pass


class EmailNotifier(Notifier):
    def send(self, user_email: str, message: str) -> bool:
        print(f"[EMAIL] To: {user_email} | {message}")
        return True


class SMSNotifier(Notifier):
    def send(self, user_email: str, message: str) -> bool:
        # SMS uses the email to look up the phone number
        print(f"[SMS] Looked up phone for {user_email} | {message}")
        return True


class PushNotifier(Notifier):
    def send(self, user_email: str, message: str) -> bool:
        print(f"[PUSH] To device of {user_email} | {message}")
        return True


# Works with ANY notifier — completely substitutable
def notify_all_users(notifiers: list[Notifier], users: list[str], message: str):
    for notifier in notifiers:
        for user in users:
            success = notifier.send(user, message)
            if not success:
                print(f"  Warning: failed to notify {user}")


# --- Usage ---
users = ["alice@example.com", "bob@example.com"]
channels = [EmailNotifier(), SMSNotifier(), PushNotifier()]

notify_all_users(channels, users, "Your order has shipped!")
```

**Output:**
```
[EMAIL] To: alice@example.com | Your order has shipped!
[EMAIL] To: bob@example.com | Your order has shipped!
[SMS] Looked up phone for alice@example.com | Your order has shipped!
[SMS] Looked up phone for bob@example.com | Your order has shipped!
[PUSH] To device of alice@example.com | Your order has shipped!
[PUSH] To device of bob@example.com | Your order has shipped!
```

Every `Notifier` is fully substitutable. Adding `SlackNotifier` later requires zero changes to `notify_all_users`.

---

## LSP Violation Checklist

You're breaking LSP if a subclass:
- Throws an exception the parent didn't throw
- Returns `None` when the parent always returns a value
- Ignores a method call that the parent would have acted on
- Requires stricter inputs than the parent accepts
- Requires overriding a method with a no-op (`pass`) because it doesn't apply

---

## Summary

| | Violation | Good Design |
|-|-----------|------------|
| Subclass behavior | Breaks or surprises callers | Works exactly like parent |
| Promises (postconditions) | Weakened (e.g., returns None sometimes) | Maintained or strengthened |
| Demands (preconditions) | Stricter than parent | Same or looser |
| Result of substitution | Crash or wrong behavior | Correct behavior |

**The key insight**: inheritance is not just "is-a" in biology. In code, it's "is-substitutable-for". Design your hierarchy around *behavior*, not real-world taxonomy.
