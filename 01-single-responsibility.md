# S — Single Responsibility Principle (SRP)

## The Idea

> A class should have **only one reason to change**.

Another way to say it: **a class should do one thing and do it well**.

If a class is doing too many things, then:
- Changing the email logic also requires touching user logic
- Testing becomes hard (you can't test one thing in isolation)
- The class grows huge over time

---

## The Simplest Analogy

Think of a **chef** at a restaurant.

A **bad design**: one person cooks, takes orders, does accounting, cleans tables, and manages inventory.  
If you hire a better cleaner, you have to fire and rehire this one person entirely.

A **good design**: chef cooks, waiter takes orders, accountant handles money.  
Each person has one job. Changing one doesn't affect the others.

---

## Bad Example — Violating SRP

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def get_user_info(self):
        return f"{self.name} ({self.email})"

    def save_to_database(self):
        # Connects to DB and saves user
        print(f"Saving {self.name} to database...")

    def send_welcome_email(self):
        # Connects to email server and sends email
        print(f"Sending welcome email to {self.email}...")

    def generate_pdf_report(self):
        # Generates a PDF with user info
        print(f"Generating PDF report for {self.name}...")
```

**What's wrong?**

This `User` class has **4 reasons to change**:
1. User data structure changes (name/email fields)
2. Database logic changes (switch from SQL to MongoDB)
3. Email service changes (switch from SendGrid to Mailgun)
4. Report format changes (PDF → Excel)

Change the database → you have to touch the `User` class.  
Hire a new email vendor → you have to touch the `User` class.  
These have nothing to do with what a "user" IS.

---

## Good Example — Following SRP

Split into classes, each with one job:

```python
# Responsibility 1: Represent a user (data only)
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def get_user_info(self):
        return f"{self.name} ({self.email})"


# Responsibility 2: Persist users to storage
class UserRepository:
    def save(self, user: User):
        print(f"Saving {user.name} to database...")

    def find_by_email(self, email: str) -> User:
        print(f"Finding user with email {email}...")
        # return user from DB


# Responsibility 3: Send emails to users
class EmailService:
    def send_welcome_email(self, user: User):
        print(f"Sending welcome email to {user.email}...")


# Responsibility 4: Generate reports
class UserReportGenerator:
    def generate_pdf(self, user: User):
        print(f"Generating PDF report for {user.name}...")
```

Now:
- Database changes → only `UserRepository` changes
- Email vendor changes → only `EmailService` changes
- Report format changes → only `UserReportGenerator` changes
- None of these touch the `User` class

---

## Working Example — Order Processing System

Here's a real-world scenario: an e-commerce order.

### ❌ Bad — Everything in One Class

```python
class Order:
    def __init__(self, items, customer_email):
        self.items = items
        self.customer_email = customer_email

    def calculate_total(self):
        return sum(item['price'] * item['qty'] for item in self.items)

    def save_order(self):
        # Talks to database
        total = self.calculate_total()
        print(f"INSERT INTO orders (total) VALUES ({total})")

    def send_confirmation(self):
        # Talks to email server
        total = self.calculate_total()
        print(f"Email to {self.customer_email}: Your order total is ${total}")

    def print_receipt(self):
        # Formats and prints receipt
        total = self.calculate_total()
        print("--- RECEIPT ---")
        for item in self.items:
            print(f"  {item['name']}: ${item['price']} x {item['qty']}")
        print(f"  TOTAL: ${total}")
        print("---------------")
```

### ✅ Good — Each Class Has One Job

```python
class Order:
    """Knows only about order data and business logic."""
    def __init__(self, items, customer_email):
        self.items = items
        self.customer_email = customer_email

    def calculate_total(self):
        return sum(item['price'] * item['qty'] for item in self.items)


class OrderRepository:
    """Knows only about saving/loading orders from the database."""
    def save(self, order: Order):
        total = order.calculate_total()
        print(f"INSERT INTO orders (email, total) VALUES ('{order.customer_email}', {total})")


class OrderNotificationService:
    """Knows only about sending notifications."""
    def send_confirmation(self, order: Order):
        total = order.calculate_total()
        print(f"Email to {order.customer_email}: Your order total is ${total:.2f}")


class ReceiptPrinter:
    """Knows only about formatting and printing receipts."""
    def print_receipt(self, order: Order):
        print("--- RECEIPT ---")
        for item in order.items:
            print(f"  {item['name']}: ${item['price']:.2f} x {item['qty']}")
        print(f"  TOTAL: ${order.calculate_total():.2f}")
        print("---------------")


# --- Usage ---
order = Order(
    items=[
        {"name": "Laptop", "price": 999.99, "qty": 1},
        {"name": "Mouse",  "price": 29.99,  "qty": 2},
    ],
    customer_email="alice@example.com"
)

repo = OrderRepository()
notifier = OrderNotificationService()
printer = ReceiptPrinter()

repo.save(order)
notifier.send_confirmation(order)
printer.print_receipt(order)
```

**Output:**
```
INSERT INTO orders (email, total) VALUES ('alice@example.com', 1059.97)
Email to alice@example.com: Your order total is $1059.97
--- RECEIPT ---
  Laptop: $999.99 x 1
  Mouse: $29.99 x 2
  TOTAL: $1059.97
---------------
```

Now if you switch from SQL to MongoDB, only `OrderRepository` changes.  
If you add SMS notifications, you extend `OrderNotificationService` — nothing else changes.

---

## How to Spot an SRP Violation

Ask yourself: **"Why would this class need to change?"**

If you come up with more than one answer — it's doing too much.

Also watch for these warning signs:
- Class has more than ~200 lines
- Class imports 10+ other modules
- Class name contains "And" or "Manager" or "Handler" doing 5 different things
- Hard to write a unit test without setting up the entire world

---

## Summary

| | Bad | Good |
|-|-----|------|
| Class has | Multiple responsibilities | Exactly one responsibility |
| Reason to change | Multiple | One |
| Testing | Hard (dependencies everywhere) | Easy (test in isolation) |
| When requirement changes | Touches many files | Touches one file |

**Rule of thumb**: If you can't describe what a class does in one short sentence without using "and", it's doing too much.
