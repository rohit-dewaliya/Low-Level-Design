
# SOLID Principles in Low-Level Design (LLD)

## 1. Introduction to SOLID

SOLID is a set of five object-oriented design principles that help developers build maintainable, flexible, reusable, and testable software.

SOLID stands for:

- **S — Single Responsibility Principle (SRP)**
- **O — Open/Closed Principle (OCP)**
- **L — Liskov Substitution Principle (LSP)**
- **I — Interface Segregation Principle (ISP)**
- **D — Dependency Inversion Principle (DIP)**

### Why Do We Need SOLID?

SOLID helps us:

- Reduce tight coupling between classes.
- Improve code readability and maintainability.
- Make code easier to test.
- Add new features with fewer changes to existing code.
- Reduce the risk of breaking existing functionality.
- Build systems that are easier to extend.

---

# 2. Single Responsibility Principle (SRP)

## Definition

**A class should have one responsibility and one primary reason to change.**

A class should focus on one cohesive responsibility rather than handling unrelated tasks.

## Bad Example

```python
class Invoice:
    def calculate_total(self, items):
        return sum(item["price"] for item in items)

    def save_to_database(self, invoice):
        print("Saving invoice to database")

    def print_invoice(self, invoice):
        print("Printing invoice")
```

### Problems

The `Invoice` class handles three different responsibilities:

- Business calculations
- Database persistence
- Presentation

A change to database storage or printing may require modifying the same class.

## Better Example

```python
class Invoice:
    def __init__(self, items):
        self.items = items

    def calculate_total(self):
        return sum(item["price"] for item in self.items)


class InvoiceRepository:
    def save(self, invoice):
        print("Saving invoice to database")


class InvoicePrinter:
    def print_invoice(self, invoice):
        print(f"Invoice total: {invoice.calculate_total()}")
```

### Benefits

- Each class has a clear purpose.
- Changes are isolated.
- Classes are easier to test.
- Responsibilities can evolve independently.

## When to Apply SRP

Apply SRP when a class handles unrelated operations or changes for different reasons.

**Important:** SRP does not mean every method must have its own class. Keep related behavior together.

---

# 3. Open/Closed Principle (OCP)

## Definition

**Software entities should be open for extension but closed for modification.**

You should be able to introduce new behavior without repeatedly changing stable, existing code.

## Bad Example

```python
class DiscountCalculator:
    def calculate(self, customer_type, price):
        if customer_type == "regular":
            return price * 0.05

        elif customer_type == "premium":
            return price * 0.15

        elif customer_type == "vip":
            return price * 0.25

        return 0
```

### Problems

Whenever a new customer type is introduced, the existing method must be modified.

As the number of discount types grows, the conditional logic becomes harder to maintain.

## Better Example

```python
from abc import ABC, abstractmethod


class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, price):
        pass


class RegularDiscount(DiscountStrategy):
    def calculate(self, price):
        return price * 0.05


class PremiumDiscount(DiscountStrategy):
    def calculate(self, price):
        return price * 0.15


class VIPDiscount(DiscountStrategy):
    def calculate(self, price):
        return price * 0.25


class DiscountCalculator:
    def calculate(self, price, strategy):
        return strategy.calculate(price)


calculator = DiscountCalculator()

print(calculator.calculate(1000, RegularDiscount()))
print(calculator.calculate(1000, PremiumDiscount()))
print(calculator.calculate(1000, VIPDiscount()))
```

### Benefits

- New discount policies can be added independently.
- Existing calculation logic stays unchanged.
- Each strategy can be tested separately.
- The design supports extension through polymorphism.

## When to Apply OCP

Apply OCP when a behavior is likely to have multiple implementations or change frequently.

**Related design pattern:** Strategy Pattern.

---

# 4. Liskov Substitution Principle (LSP)

## Definition

**Objects of a subtype should be usable wherever objects of the base type are expected without breaking the program's correctness.**

A subclass must honor the behavioral contract of its parent class.

## Bad Example

```python
class Bird:
    def fly(self):
        print("Flying")


class Sparrow(Bird):
    def fly(self):
        print("Sparrow is flying")


class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("Penguins cannot fly")


def make_bird_fly(bird):
    bird.fly()


make_bird_fly(Sparrow())
make_bird_fly(Penguin())  # Fails
```

### Problems

The `Bird` class promises that every bird can fly, but `Penguin` cannot fulfill that promise.

The subclass violates the expected behavior of the parent class.

## Better Example

```python
class Bird:
    def eat(self):
        print("Bird is eating")


class FlyingBird(Bird):
    def fly(self):
        print("Bird is flying")


class Sparrow(FlyingBird):
    pass


class Penguin(Bird):
    def swim(self):
        print("Penguin is swimming")


def make_bird_eat(bird):
    bird.eat()


def make_bird_fly(bird: FlyingBird):
    bird.fly()


make_bird_eat(Penguin())
make_bird_fly(Sparrow())
```

### Benefits

- Subclasses honor their contracts.
- Code can safely use interchangeable implementations.
- Unexpected runtime errors are reduced.
- Inheritance becomes more reliable.

## Rules to Remember

A subtype should not:

- Strengthen preconditions unexpectedly.
- Weaken postconditions.
- Break important invariants.
- Replace expected behavior with incompatible behavior.

## When to Apply LSP

Apply LSP whenever you create inheritance hierarchies or interchangeable implementations.

**Key question:** Can the subclass replace the parent without surprising its callers?

---

# 5. Interface Segregation Principle (ISP)

## Definition

**Clients should not be forced to depend on methods they do not use.**

Prefer small, focused interfaces over one large interface that contains unrelated operations.

Python does not have a dedicated `interface` keyword, but abstract classes and protocols can express interface contracts.

## Bad Example

```python
from abc import ABC, abstractmethod


class Machine(ABC):
    @abstractmethod
    def print_document(self, document):
        pass

    @abstractmethod
    def scan_document(self):
        pass

    @abstractmethod
    def fax_document(self, document):
        pass


class BasicPrinter(Machine):
    def print_document(self, document):
        print("Printing document")

    def scan_document(self):
        raise NotImplementedError

    def fax_document(self, document):
        raise NotImplementedError
```

### Problems

A basic printer must implement scanning and faxing methods that it does not support.

This creates unnecessary dependencies and misleading contracts.

## Better Example

```python
from abc import ABC, abstractmethod


class Printer(ABC):
    @abstractmethod
    def print_document(self, document):
        pass


class Scanner(ABC):
    @abstractmethod
    def scan_document(self):
        pass


class Fax(ABC):
    @abstractmethod
    def fax_document(self, document):
        pass


class BasicPrinter(Printer):
    def print_document(self, document):
        print("Printing document")


class MultiFunctionPrinter(Printer, Scanner, Fax):
    def print_document(self, document):
        print("Printing document")

    def scan_document(self):
        print("Scanning document")

    def fax_document(self, document):
        print("Faxing document")
```

### Benefits

- Classes implement only relevant capabilities.
- Interfaces become easier to understand.
- Implementations have fewer unnecessary dependencies.
- New capabilities can be introduced independently.

## When to Apply ISP

Apply ISP when a class depends on a large interface but uses only a small part of it.

**Key idea:** Split interfaces by client needs, not merely by the number of methods.

---

# 6. Dependency Inversion Principle (DIP)

## Definition

**High-level modules should not depend directly on low-level modules. Both should depend on abstractions.**

Abstractions should not depend on implementation details. Implementation details should depend on abstractions.

- High-level modules contain important business logic.
- Low-level modules handle implementation details such as email, databases, and file storage.
- Abstractions define the contracts between them.

## Bad Example

```python
class EmailSender:
    def send(self, message):
        print(f"Sending email: {message}")


class NotificationService:
    def __init__(self):
        self.sender = EmailSender()

    def notify(self, message):
        self.sender.send(message)
```

### Problems

`NotificationService` directly creates an `EmailSender`.

Switching to SMS or another notification provider requires changing the high-level service.

Testing also becomes harder because the dependency is fixed.

## Better Example

```python
from abc import ABC, abstractmethod


class NotificationSender(ABC):
    @abstractmethod
    def send(self, message):
        pass


class EmailSender(NotificationSender):
    def send(self, message):
        print(f"Sending email: {message}")


class SMSSender(NotificationSender):
    def send(self, message):
        print(f"Sending SMS: {message}")


class NotificationService:
    def __init__(self, sender: NotificationSender):
        self.sender = sender

    def notify(self, message):
        self.sender.send(message)


email_sender = EmailSender()
sms_sender = SMSSender()

email_service = NotificationService(email_sender)
sms_service = NotificationService(sms_sender)

email_service.notify("Welcome!")
sms_service.notify("Your OTP is 123456")
```

### Benefits

- The high-level service depends on an abstraction.
- Different implementations can be injected.
- Unit testing becomes easier.
- Infrastructure can change without rewriting business logic.

## Dependency Inversion vs Dependency Injection

These concepts are related but different.

| Dependency Inversion | Dependency Injection |
|---|---|
| A design principle | A technique |
| Encourages dependencies on abstractions | Supplies dependencies from outside |
| Focuses on dependency direction | Focuses on how dependencies are provided |
| Example: service depends on `NotificationSender` | Example: passing `EmailSender` into the constructor |

**When to Apply DIP:** Use it when high-level business logic should work with replaceable infrastructure or multiple implementations.

---

# 7. SOLID Principles Summary

| Principle | Full Form | Main Idea |
|---|---|---|
| S | Single Responsibility Principle | One focused responsibility per class |
| O | Open/Closed Principle | Extend behavior without repeatedly modifying stable code |
| L | Liskov Substitution Principle | Subtypes must honor base-type contracts |
| I | Interface Segregation Principle | Keep interfaces focused |
| D | Dependency Inversion Principle | Depend on abstractions, not concrete implementations |

---

# 8. How SOLID Principles Work Together

Consider an e-commerce checkout system.

- **SRP:** Separate order management, payment processing, and invoice generation.
- **OCP:** Add new payment methods without rewriting checkout logic.
- **LSP:** Ensure every payment implementation honors the payment contract.
- **ISP:** Separate payment, refund, and reporting capabilities when clients need them independently.
- **DIP:** Make checkout depend on a payment abstraction instead of a specific provider.

Together, these principles help keep the system modular and maintainable.

---

# 9. SOLID Principles in LLD Interviews

When designing a system such as Parking Lot, ATM, Splitwise, or E-commerce, use the following checklist:

1. Identify classes with too many responsibilities.
2. Find conditionals that grow whenever new behaviors are added.
3. Check whether subclasses honor parent contracts.
4. Split oversized interfaces into focused capabilities.
5. Inject dependencies instead of constructing concrete services everywhere.
6. Explain the trade-offs of your design.
7. Avoid introducing abstractions that do not solve a real problem.

## Common Interview Questions

### Q1. What is SOLID?

SOLID is a collection of five object-oriented design principles that improve maintainability, extensibility, and testability.

### Q2. What is the difference between SRP and ISP?

SRP focuses on keeping a class's responsibilities cohesive. ISP focuses on preventing clients from depending on methods they do not need.

### Q3. What is the difference between OCP and DIP?

OCP focuses on extending behavior without repeatedly modifying stable code. DIP focuses on making high-level and low-level modules depend on abstractions.

### Q4. Can inheritance violate LSP?

Yes. A subclass violates LSP when it breaks the behavior or guarantees expected from its parent class.

### Q5. Is SOLID always necessary?

SOLID provides useful guidelines, not rigid rules. Apply the principles where they reduce complexity; avoid unnecessary abstractions in simple code.

### Q6. Which design pattern is associated with OCP?

The Strategy Pattern is a common example because it allows algorithms to be added as separate implementations.

---

# 10. Final Takeaways

- **SRP:** Keep responsibilities focused.
- **OCP:** Make changing behavior extensible.
- **LSP:** Preserve behavioral contracts.
- **ISP:** Keep interfaces small and relevant.
- **DIP:** Depend on abstractions and inject implementations.
- Apply SOLID to solve real design problems rather than mechanically creating more classes.
- A good design balances flexibility, simplicity, and maintainability.
