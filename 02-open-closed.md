# O — Open/Closed Principle (OCP)

## The Idea

> Software entities should be **open for extension** but **closed for modification**.

In plain English: **add new behavior by writing new code, not by editing existing code**.

When you need a new feature:
- **Bad**: Edit the existing class → risk breaking what already works
- **Good**: Add a new class that extends behavior → existing code stays untouched

---

## The Simplest Analogy

Think of a **power strip**.

You don't crack open the wall socket and rewire it every time you want to plug in a new device. The socket is **closed for modification**. But you can plug in any device — **open for extension**.

Your code should work the same way. Design it so new behavior plugs in without rewiring the existing system.

---

## Bad Example — Violating OCP

```python
class DiscountCalculator:
    def calculate(self, customer_type: str, price: float) -> float:
        if customer_type == "regular":
            return price
        elif customer_type == "silver":
            return price * 0.9   # 10% off
        elif customer_type == "gold":
            return price * 0.8   # 20% off
        # ← Every new customer type requires editing this method
```

**What's wrong?**

Every time a new customer tier is added (e.g., `"platinum"`), you must:
1. Open this existing, working file
2. Add another `elif`
3. Risk introducing a bug in the existing tiers
4. Re-test everything

This violates OCP. The class is not closed for modification.

What if 5 different teams use this class? Now every team must re-test when you add a new tier.

---

## Good Example — Following OCP

Use **abstraction** (abstract class or interface). Each discount type is its own class.

```python
from abc import ABC, abstractmethod


# The abstraction — closed for modification, open for extension
class Discount(ABC):
    @abstractmethod
    def apply(self, price: float) -> float:
        pass


# Existing discount types — never need to change
class NoDiscount(Discount):
    def apply(self, price: float) -> float:
        return price


class SilverDiscount(Discount):
    def apply(self, price: float) -> float:
        return price * 0.9


class GoldDiscount(Discount):
    def apply(self, price: float) -> float:
        return price * 0.8


# Adding a NEW tier? Just add a new class. Nothing else changes.
class PlatinumDiscount(Discount):
    def apply(self, price: float) -> float:
        return price * 0.7


# The calculator never needs to change — it works with any Discount
class DiscountCalculator:
    def calculate(self, discount: Discount, price: float) -> float:
        return discount.apply(price)
```

Now:
- Adding `PlatinumDiscount` → write one new class, nothing else changes
- `DiscountCalculator` is closed for modification
- Existing discounts are untouched and don't need re-testing

---

## Working Example — Shape Area Calculator

A classic example. You have shapes and need to calculate their areas.

### ❌ Bad — Must Edit Every Time a New Shape is Added

```python
class AreaCalculator:
    def calculate_area(self, shape):
        if shape['type'] == 'circle':
            return 3.14159 * shape['radius'] ** 2
        elif shape['type'] == 'rectangle':
            return shape['width'] * shape['height']
        elif shape['type'] == 'triangle':
            return 0.5 * shape['base'] * shape['height']
        # Adding a hexagon? Must edit this method.
        # What if another team owns this file?
```

### ✅ Good — Extend Without Modifying

```python
import math
from abc import ABC, abstractmethod


# Abstract base — defines the contract, never changes
class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

    @abstractmethod
    def describe(self) -> str:
        pass


# Concrete shapes — each is its own closed unit
class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius

    def area(self) -> float:
        return math.pi * self.radius ** 2

    def describe(self) -> str:
        return f"Circle(radius={self.radius})"


class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

    def describe(self) -> str:
        return f"Rectangle({self.width}x{self.height})"


class Triangle(Shape):
    def __init__(self, base: float, height: float):
        self.base = base
        self.height = height

    def area(self) -> float:
        return 0.5 * self.base * self.height

    def describe(self) -> str:
        return f"Triangle(base={self.base}, height={self.height})"


# NEW shape added later — zero changes to anything above
class Hexagon(Shape):
    def __init__(self, side: float):
        self.side = side

    def area(self) -> float:
        return (3 * math.sqrt(3) / 2) * self.side ** 2

    def describe(self) -> str:
        return f"Hexagon(side={self.side})"


# Calculator works with ANY shape — past, present, and future
class AreaCalculator:
    def total_area(self, shapes: list[Shape]) -> float:
        return sum(s.area() for s in shapes)

    def print_report(self, shapes: list[Shape]):
        print("=== Area Report ===")
        for shape in shapes:
            print(f"  {shape.describe()}: {shape.area():.2f}")
        print(f"  TOTAL: {self.total_area(shapes):.2f}")
        print("===================")


# --- Usage ---
shapes = [
    Circle(radius=5),
    Rectangle(width=4, height=6),
    Triangle(base=3, height=8),
    Hexagon(side=4),   # ← added later, nothing else changed
]

calc = AreaCalculator()
calc.print_report(shapes)
```

**Output:**
```
=== Area Report ===
  Circle(radius=5): 78.54
  Rectangle(4x6): 24.00
  Triangle(base=3, height=8): 12.00
  Hexagon(side=4): 41.57
  TOTAL: 156.11
===================
```

`AreaCalculator` was written once and never touched. Adding `Hexagon` required zero changes to existing code.

---

## Real-World OCP: Payment Methods

```python
from abc import ABC, abstractmethod


class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount: float) -> str:
        pass


class CreditCardPayment(PaymentMethod):
    def __init__(self, card_number: str):
        self.card_number = card_number

    def pay(self, amount: float) -> str:
        return f"Charged ${amount:.2f} to card ending in {self.card_number[-4:]}"


class PayPalPayment(PaymentMethod):
    def __init__(self, email: str):
        self.email = email

    def pay(self, amount: float) -> str:
        return f"Sent ${amount:.2f} via PayPal to {self.email}"


# Added 6 months later — no existing code changed
class CryptoPayment(PaymentMethod):
    def __init__(self, wallet_address: str):
        self.wallet_address = wallet_address

    def pay(self, amount: float) -> str:
        return f"Transferred ${amount:.2f} in BTC to {self.wallet_address}"


# Checkout never needs to change
class Checkout:
    def process(self, amount: float, payment: PaymentMethod):
        result = payment.pay(amount)
        print(f"Payment successful: {result}")


# --- Usage ---
checkout = Checkout()
checkout.process(99.99, CreditCardPayment("4111111111111234"))
checkout.process(49.99, PayPalPayment("alice@example.com"))
checkout.process(199.99, CryptoPayment("1A2b3C4d5E6f..."))
```

**Output:**
```
Payment successful: Charged $99.99 to card ending in 1234
Payment successful: Sent $49.99 via PayPal to alice@example.com
Payment successful: Transferred $199.99 in BTC to 1A2b3C4d5E6f...
```

---

## How to Spot an OCP Violation

Look for:
- Long `if/elif` chains that check a type or string tag
- A class you have to edit every time a new "type" is added
- Comments like `# Add new type here`

The fix is almost always: **extract an abstraction, make each type its own class**.

---

## Summary

| | Bad | Good |
|-|-----|------|
| New feature requires | Editing existing class | Writing new class |
| Risk of regression | High (touching working code) | Low (existing code unchanged) |
| Testing | Re-test everything on change | Only test new class |
| Mechanism | Big `if/elif` on type | Abstract class / interface |

**The key insight**: design your abstractions upfront. The more stable the interface, the more you can extend without modifying.
