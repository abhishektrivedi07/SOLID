# I — Interface Segregation Principle (ISP)

## The Idea

> Clients should not be forced to depend on interfaces they do not use.

In plain English: **don't create fat interfaces**. Split large interfaces into small, focused ones so that a class only implements what it actually needs.

---

## The Simplest Analogy

Imagine a **TV remote control**.

A **fat interface** remote has 50 buttons — volume, channel, Netflix, YouTube, game mode, sleep timer, subtitle language...

If you're programming a simple audio-only speaker that uses the remote interface, you still have to implement all 50 buttons — even though you only need 5. That's ISP violation.

Better: separate remotes for each purpose. The speaker only gets the audio interface.

---

## Bad Example — Fat Interface

```python
from abc import ABC, abstractmethod


# One giant interface — every class must implement all of it
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def eat(self):
        pass

    @abstractmethod
    def sleep(self):
        pass


class HumanWorker(Worker):
    def work(self):
        print("Human is working")

    def eat(self):
        print("Human is eating")

    def sleep(self):
        print("Human is sleeping")


class RobotWorker(Worker):
    def work(self):
        print("Robot is working")

    def eat(self):
        pass   # ← Robots don't eat. Forced to implement a no-op.

    def sleep(self):
        pass   # ← Robots don't sleep. Forced to implement a no-op.
```

**What's wrong?**

`RobotWorker` is forced to implement `eat()` and `sleep()` even though they make no sense for it. Empty `pass` methods are a classic ISP violation signal.

If `eat()` ever changes its signature — `RobotWorker` must be updated too, for no reason.

---

## Good Example — Segregated Interfaces

```python
from abc import ABC, abstractmethod


class Workable(ABC):
    @abstractmethod
    def work(self):
        pass


class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass


class Sleepable(ABC):
    @abstractmethod
    def sleep(self):
        pass


# Human needs all three → implements all three
class HumanWorker(Workable, Eatable, Sleepable):
    def work(self):
        print("Human is working")

    def eat(self):
        print("Human is eating")

    def sleep(self):
        print("Human is sleeping")


# Robot only needs work → implements only Workable
class RobotWorker(Workable):
    def work(self):
        print("Robot is working 24/7")
    # No eat(), no sleep() — doesn't need them, doesn't implement them


# Functions only declare what they actually need
def manage_work_shift(workers: list[Workable]):
    for w in workers:
        w.work()

def schedule_lunch(workers: list[Eatable]):
    for w in workers:
        w.eat()
```

Now `RobotWorker` is not burdened by things it doesn't need.

---

## Working Example — Printers and Scanners

The most famous ISP example. A multifunction device does everything; a basic printer does just one thing.

### ❌ Bad — One Fat Interface

```python
from abc import ABC, abstractmethod


class MultifunctionDevice(ABC):
    @abstractmethod
    def print_document(self, document: str):
        pass

    @abstractmethod
    def scan_document(self) -> str:
        pass

    @abstractmethod
    def fax_document(self, document: str, number: str):
        pass

    @abstractmethod
    def copy_document(self):
        pass


class OfficePrinter(MultifunctionDevice):
    def print_document(self, document: str):
        print(f"Printing: {document}")

    def scan_document(self) -> str:
        return "Scanned document"

    def fax_document(self, document: str, number: str):
        print(f"Faxing to {number}")

    def copy_document(self):
        print("Copying document")


class SimplePrinter(MultifunctionDevice):
    def print_document(self, document: str):
        print(f"Printing: {document}")

    def scan_document(self) -> str:
        raise NotImplementedError("This printer cannot scan!")  # ✗

    def fax_document(self, document: str, number: str):
        raise NotImplementedError("This printer cannot fax!")   # ✗

    def copy_document(self):
        raise NotImplementedError("This printer cannot copy!")  # ✗
```

`SimplePrinter` is forced to "implement" 3 features it doesn't have — by throwing exceptions.

### ✅ Good — Segregated Interfaces

```python
from abc import ABC, abstractmethod


# Each capability is its own small interface
class Printable(ABC):
    @abstractmethod
    def print_document(self, document: str):
        pass


class Scannable(ABC):
    @abstractmethod
    def scan_document(self) -> str:
        pass


class Faxable(ABC):
    @abstractmethod
    def fax_document(self, document: str, number: str):
        pass


class Copyable(ABC):
    @abstractmethod
    def copy_document(self):
        pass


# Basic printer — only prints
class SimplePrinter(Printable):
    def print_document(self, document: str):
        print(f"[SimplePrinter] Printing: {document}")


# Office machine — does everything
class OfficePrinter(Printable, Scannable, Faxable, Copyable):
    def print_document(self, document: str):
        print(f"[OfficePrinter] Printing: {document}")

    def scan_document(self) -> str:
        result = "Scanned document content"
        print(f"[OfficePrinter] Scanning... got: '{result}'")
        return result

    def fax_document(self, document: str, number: str):
        print(f"[OfficePrinter] Faxing '{document}' to {number}")

    def copy_document(self):
        print("[OfficePrinter] Copying document")


# Functions only require what they use
def print_report(printer: Printable, report: str):
    printer.print_document(report)

def scan_and_store(scanner: Scannable) -> str:
    return scanner.scan_document()

def send_invoice_by_fax(fax_machine: Faxable, invoice: str, number: str):
    fax_machine.fax_document(invoice, number)


# --- Usage ---
basic = SimplePrinter()
office = OfficePrinter()

print_report(basic, "Monthly Sales Report")      # ✓
print_report(office, "Q4 Financial Summary")     # ✓

scan_and_store(office)                            # ✓
# scan_and_store(basic)  ← type error ✓ caught at dev time

send_invoice_by_fax(office, "Invoice #1234", "555-0199")  # ✓
```

**Output:**
```
[SimplePrinter] Printing: Monthly Sales Report
[OfficePrinter] Printing: Q4 Financial Summary
[OfficePrinter] Scanning... got: 'Scanned document content'
[OfficePrinter] Faxing 'Invoice #1234' to 555-0199
```

---

## Working Example — User Roles / Permissions

A real-world web app scenario. Different users can do different things.

```python
from abc import ABC, abstractmethod


# Granular interfaces — each is one permission
class CanReadPosts(ABC):
    @abstractmethod
    def read_post(self, post_id: int) -> str:
        pass


class CanWritePosts(ABC):
    @abstractmethod
    def create_post(self, title: str, body: str) -> int:
        pass


class CanDeletePosts(ABC):
    @abstractmethod
    def delete_post(self, post_id: int):
        pass


class CanManageUsers(ABC):
    @abstractmethod
    def ban_user(self, user_id: int):
        pass


# Viewer — can only read
class Viewer(CanReadPosts):
    def read_post(self, post_id: int) -> str:
        return f"Reading post {post_id}"


# Author — can read and write
class Author(CanReadPosts, CanWritePosts):
    def read_post(self, post_id: int) -> str:
        return f"Reading post {post_id}"

    def create_post(self, title: str, body: str) -> int:
        print(f"Creating post: '{title}'")
        return 42  # new post id


# Admin — can do everything
class Admin(CanReadPosts, CanWritePosts, CanDeletePosts, CanManageUsers):
    def read_post(self, post_id: int) -> str:
        return f"Admin reading post {post_id}"

    def create_post(self, title: str, body: str) -> int:
        print(f"Admin creating post: '{title}'")
        return 99

    def delete_post(self, post_id: int):
        print(f"Admin deleted post {post_id}")

    def ban_user(self, user_id: int):
        print(f"Admin banned user {user_id}")


# Content moderation function — only needs delete capability
def moderate_content(moderator: CanDeletePosts, flagged_post_ids: list[int]):
    for post_id in flagged_post_ids:
        moderator.delete_post(post_id)
        print(f"  Removed flagged post {post_id}")


# --- Usage ---
viewer = Viewer()
author = Author()
admin = Admin()

print(viewer.read_post(1))       # Reading post 1
print(author.read_post(2))       # Reading post 2
author.create_post("Hello!", "My first post.")
admin.ban_user(999)

print("\n--- Content Moderation ---")
moderate_content(admin, [5, 12, 87])
# moderate_content(author, ...)  ← type error ✓ author can't delete
```

**Output:**
```
Reading post 1
Reading post 2
Creating post: 'Hello!'
Admin banned user 999

--- Content Moderation ---
Admin deleted post 5
  Removed flagged post 5
Admin deleted post 12
  Removed flagged post 12
Admin deleted post 87
  Removed flagged post 87
```

---

## How to Spot ISP Violations

Warning signs:
- A class implements an interface but some methods are just `pass` or `raise NotImplementedError`
- You find yourself saying "this method doesn't apply here"
- A large interface with 10+ methods that nothing fully uses
- Clients import an interface but only use 2 of its 8 methods

---

## Summary

| | Violation | Good Design |
|-|-----------|------------|
| Interface size | Large, "fat" | Small, focused |
| Classes forced to | Implement irrelevant methods | Only implement what they use |
| Change impact | Ripples to all implementors | Only affects related classes |
| Common sign | Empty `pass` or `raise NotImplementedError` | All methods genuinely implemented |

**The key insight**: prefer many small interfaces over one large one. Each interface should represent a coherent set of capabilities that go together naturally.
