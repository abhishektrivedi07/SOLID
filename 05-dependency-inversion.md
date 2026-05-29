# D — Dependency Inversion Principle (DIP)

## The Idea

> 1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
> 2. Abstractions should not depend on details. Details should depend on abstractions.

In plain English: **depend on interfaces, not on concrete implementations**.

---

## The Simplest Analogy

Think of a **lamp and a wall socket**.

The lamp doesn't depend on a specific power plant (coal, solar, nuclear). It depends on the **socket interface** — a standard plug shape and voltage.

The power plant also doesn't know about your specific lamp. It just provides power through the socket standard.

Both depend on the **abstraction** (the socket standard). Neither depends on the other directly.

Your code should work the same way.

---

## What "Inversion" Means

In traditional procedural code, high-level logic calls low-level utilities directly:

```
High-level → calls → Low-level

OrderService → calls → MySQLDatabase (concrete)
```

DIP inverts this: both point at an abstraction:

```
High-level → depends on → Abstraction ← depends on ← Low-level
OrderService → depends on → Database (interface) ← MySQLDatabase, MongoDB
```

The dependency arrow is "inverted" — the low-level module now depends on the abstraction, not the high-level module depending on the low-level module.

---

## Bad Example — Tight Coupling

```python
class MySQLDatabase:
    def save(self, data: dict):
        print(f"Saving to MySQL: {data}")

    def find(self, id: int) -> dict:
        print(f"Finding in MySQL: id={id}")
        return {"id": id, "name": "Alice"}


class UserService:
    def __init__(self):
        # ← Hardcoded! UserService is tightly coupled to MySQL.
        self.db = MySQLDatabase()

    def create_user(self, name: str, email: str):
        self.db.save({"name": name, "email": email})

    def get_user(self, id: int) -> dict:
        return self.db.find(id)
```

**What's wrong?**

- You can't switch to PostgreSQL without editing `UserService`
- You can't unit test `UserService` without a real MySQL connection
- `UserService` (high-level) is locked to `MySQLDatabase` (low-level)

---

## Good Example — Depend on Abstraction

```python
from abc import ABC, abstractmethod


# The abstraction — both sides depend on this
class Database(ABC):
    @abstractmethod
    def save(self, data: dict):
        pass

    @abstractmethod
    def find(self, id: int) -> dict:
        pass


# Low-level detail — depends on the abstraction
class MySQLDatabase(Database):
    def save(self, data: dict):
        print(f"[MySQL] Saving: {data}")

    def find(self, id: int) -> dict:
        print(f"[MySQL] Finding id={id}")
        return {"id": id, "name": "Alice (from MySQL)"}


# Another low-level detail — also depends on the abstraction
class MongoDatabase(Database):
    def save(self, data: dict):
        print(f"[MongoDB] Inserting: {data}")

    def find(self, id: int) -> dict:
        print(f"[MongoDB] Querying _id={id}")
        return {"id": id, "name": "Alice (from MongoDB)"}


# High-level module — depends only on the abstraction
class UserService:
    def __init__(self, db: Database):   # ← Receives abstraction, not concrete class
        self.db = db

    def create_user(self, name: str, email: str):
        self.db.save({"name": name, "email": email})

    def get_user(self, id: int) -> dict:
        return self.db.find(id)


# --- Usage: inject whatever DB you want ---
mysql_service = UserService(db=MySQLDatabase())
mysql_service.create_user("Alice", "alice@example.com")

mongo_service = UserService(db=MongoDatabase())
mongo_service.create_user("Bob", "bob@example.com")
```

**Output:**
```
[MySQL] Saving: {'name': 'Alice', 'email': 'alice@example.com'}
[MongoDB] Inserting: {'name': 'Bob', 'email': 'bob@example.com'}
```

`UserService` doesn't know or care which database is used. You decide at runtime.

---

## Dependency Injection (DI)

DIP is the *principle*. Dependency Injection is the *technique* used to implement it.

Instead of a class creating its own dependencies, **they are passed (injected) from outside**.

Three forms of injection:

### 1. Constructor Injection (most common)
```python
class OrderService:
    def __init__(self, db: Database, notifier: Notifier):
        self.db = db
        self.notifier = notifier
```

### 2. Method Injection
```python
class OrderService:
    def process(self, order: Order, db: Database):
        db.save(order)
```

### 3. Property Injection
```python
class OrderService:
    def __init__(self):
        self.db: Database = None  # set after construction
```

Constructor injection is preferred — dependencies are visible and required.

---

## Working Example — Notification System

```python
from abc import ABC, abstractmethod


# Abstractions
class MessageSender(ABC):
    @abstractmethod
    def send(self, recipient: str, message: str) -> bool:
        pass


class Logger(ABC):
    @abstractmethod
    def log(self, message: str):
        pass


# Concrete implementations
class EmailSender(MessageSender):
    def send(self, recipient: str, message: str) -> bool:
        print(f"[Email] To: {recipient} → {message}")
        return True


class SMSSender(MessageSender):
    def send(self, recipient: str, message: str) -> bool:
        print(f"[SMS]   To: {recipient} → {message}")
        return True


class ConsoleLogger(Logger):
    def log(self, message: str):
        print(f"[LOG] {message}")


class FileLogger(Logger):
    def log(self, message: str):
        print(f"[FILE] Writing to log file: {message}")


# High-level module — knows nothing about Email, SMS, files, or console
class OrderNotificationService:
    def __init__(self, sender: MessageSender, logger: Logger):
        self.sender = sender
        self.logger = logger

    def notify_shipped(self, customer_email: str, order_id: str):
        message = f"Your order #{order_id} has shipped!"
        success = self.sender.send(customer_email, message)
        if success:
            self.logger.log(f"Notified {customer_email} about order {order_id}")
        else:
            self.logger.log(f"FAILED to notify {customer_email} about order {order_id}")


# --- Usage: compose any combination ---
print("=== Production (Email + File Log) ===")
service = OrderNotificationService(
    sender=EmailSender(),
    logger=FileLogger()
)
service.notify_shipped("alice@example.com", "ORD-001")

print("\n=== Alternate (SMS + Console Log) ===")
service2 = OrderNotificationService(
    sender=SMSSender(),
    logger=ConsoleLogger()
)
service2.notify_shipped("bob@example.com", "ORD-002")
```

**Output:**
```
=== Production (Email + File Log) ===
[Email] To: alice@example.com → Your order #ORD-001 has shipped!
[FILE] Writing to log file: Notified alice@example.com about order ORD-001

=== Alternate (SMS + Console Log) ===
[SMS]   To: bob@example.com → Your order #ORD-002 has shipped!
[LOG] Notified bob@example.com about order ORD-002
```

`OrderNotificationService` never changes. You swap delivery channel and logger by passing different implementations.

---

## DIP Makes Testing Easy

The biggest benefit of DIP: you can inject a **fake (mock)** implementation in tests.

```python
# In production: use real DB
service = UserService(db=MySQLDatabase())

# In tests: use a fake in-memory DB — no real DB needed!
class InMemoryDatabase(Database):
    def __init__(self):
        self._store = {}
        self._next_id = 1

    def save(self, data: dict):
        data['id'] = self._next_id
        self._store[self._next_id] = data
        self._next_id += 1

    def find(self, id: int) -> dict:
        return self._store.get(id, {})


# Unit test — fast, no database, no network
def test_create_user():
    fake_db = InMemoryDatabase()
    service = UserService(db=fake_db)

    service.create_user("Alice", "alice@example.com")

    user = fake_db.find(1)
    assert user['name'] == 'Alice'
    assert user['email'] == 'alice@example.com'
    print("Test passed ✓")

test_create_user()
```

**Output:**
```
Test passed ✓
```

No MySQL. No network. No setup. Pure, fast unit test.

---

## Full Working Example — Payment Processing

Combining everything: high-level business logic that works with any payment provider and any notification method.

```python
from abc import ABC, abstractmethod


# Abstractions
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: float, card_token: str) -> str:
        """Returns a transaction ID on success, raises on failure."""
        pass


class ReceiptSender(ABC):
    @abstractmethod
    def send_receipt(self, email: str, amount: float, tx_id: str):
        pass


# Concrete implementations
class StripeGateway(PaymentGateway):
    def charge(self, amount: float, card_token: str) -> str:
        tx_id = f"stripe_tx_{hash(card_token) % 10000:04d}"
        print(f"[Stripe] Charged ${amount:.2f} → tx: {tx_id}")
        return tx_id


class PayPalGateway(PaymentGateway):
    def charge(self, amount: float, card_token: str) -> str:
        tx_id = f"paypal_tx_{hash(card_token) % 10000:04d}"
        print(f"[PayPal] Charged ${amount:.2f} → tx: {tx_id}")
        return tx_id


class EmailReceiptSender(ReceiptSender):
    def send_receipt(self, email: str, amount: float, tx_id: str):
        print(f"[Email Receipt] To: {email} | ${amount:.2f} | Ref: {tx_id}")


class SMSReceiptSender(ReceiptSender):
    def send_receipt(self, email: str, amount: float, tx_id: str):
        print(f"[SMS Receipt]   To: {email} | ${amount:.2f} | Ref: {tx_id}")


# High-level business logic — depends ONLY on abstractions
class PaymentProcessor:
    def __init__(self, gateway: PaymentGateway, receipt_sender: ReceiptSender):
        self.gateway = gateway
        self.receipt_sender = receipt_sender

    def process_payment(self, customer_email: str, amount: float, card_token: str):
        print(f"\nProcessing ${amount:.2f} for {customer_email}...")
        tx_id = self.gateway.charge(amount, card_token)
        self.receipt_sender.send_receipt(customer_email, amount, tx_id)
        print(f"Done. Transaction: {tx_id}")


# --- Wire it all together ---
stripe_processor = PaymentProcessor(
    gateway=StripeGateway(),
    receipt_sender=EmailReceiptSender()
)
stripe_processor.process_payment("alice@example.com", 99.99, "tok_visa_4242")

paypal_processor = PaymentProcessor(
    gateway=PayPalGateway(),
    receipt_sender=SMSReceiptSender()
)
paypal_processor.process_payment("bob@example.com", 49.99, "tok_pp_bob")
```

**Output:**
```
Processing $99.99 for alice@example.com...
[Stripe] Charged $99.99 → tx: stripe_tx_2516
[Email Receipt] To: alice@example.com | $99.99 | Ref: stripe_tx_2516
Done. Transaction: stripe_tx_2516

Processing $49.99 for bob@example.com...
[PayPal] Charged $49.99 → tx: paypal_tx_7890
[SMS Receipt]   To: bob@example.com | $49.99 | Ref: paypal_tx_7890
Done. Transaction: paypal_tx_7890
```

`PaymentProcessor` never needs to change. Swap Stripe for PayPal, Email for SMS — it's just a constructor argument.

---

## How to Spot DIP Violations

Warning signs:
- `self.db = MySQLDatabase()` inside a class (hardcoded)
- `import` statements for concrete classes inside high-level modules
- "I can't test this without a real database/network/file system"
- Adding a new DB/email/payment provider requires editing multiple service classes

---

## Summary

| | Violation | Good Design |
|-|-----------|------------|
| Dependency direction | High-level → concrete low-level | Both → abstraction |
| Coupling | Tight (locked to one implementation) | Loose (any implementation works) |
| Testability | Hard (need real services) | Easy (inject fakes/mocks) |
| Switching providers | Edit multiple files | Change one constructor argument |
| Mechanism | `self.dep = ConcreteClass()` | `self.dep = dep` (injected) |

**The key insight**: the things that change most often (which DB, which email provider, which payment processor) should be the things your high-level logic knows the least about. Abstract them away and inject them.
