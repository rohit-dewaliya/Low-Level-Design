
# Low-Level Design (LLD) — Complete Notes

## 1. What Is Low-Level Design?

Low-Level Design (LLD) describes the internal structure and implementation details of a software system.

It defines:
- Classes and objects
- Attributes and methods
- Interfaces and relationships
- Design patterns
- Data flow and interactions
- Error handling and edge cases
- How individual components collaborate

### High-Level Design vs Low-Level Design

| High-Level Design (HLD) | Low-Level Design (LLD) |
|---|---|
| Defines the overall system architecture | Defines internal component design |
| Focuses on services and infrastructure | Focuses on classes, objects, and methods |
| Discusses databases, APIs, and communication | Discusses OOP, interfaces, and design patterns |
| Answers: What components are needed? | Answers: How should components work? |
| Example: E-commerce architecture | Example: Shopping cart and payment classes |

### Why Is LLD Important?

- Improves code maintainability.
- Encourages reusable components.
- Reduces tight coupling.
- Makes unit testing easier.
- Supports extensibility.
- Helps manage complexity.
- Improves collaboration between developers.

---

## 2. Core Object-Oriented Programming Concepts

LLD commonly uses Object-Oriented Programming (OOP) to model real-world entities and software behavior.

### 2.1 Class

A class is a blueprint that defines the attributes and behavior of objects.

```python
class Car:
    def __init__(self, brand, color):
        self.brand = brand
        self.color = color

    def drive(self):
        print(f"{self.brand} is driving")


car = Car("Toyota", "Black")
car.drive()
```

### 2.2 Object

An object is an instance of a class.

```python
car1 = Car("Toyota", "Black")
car2 = Car("Honda", "White")
```

Each object has its own state and can use the behavior defined by its class.

### 2.3 Encapsulation

Encapsulation combines data and behavior in a class and controls how the internal state is accessed or modified.

```python
class BankAccount:
    def __init__(self, balance=0):
        self.__balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Amount must be positive")
        self.__balance += amount

    def get_balance(self):
        return self.__balance
```

Benefits:
- Protects object state.
- Centralizes validation.
- Reduces unintended modifications.
- Makes implementation easier to change.

Python's double underscore triggers name mangling; it does not provide absolute privacy.

### 2.4 Abstraction

Abstraction exposes essential operations while hiding implementation details.

```python
from abc import ABC, abstractmethod


class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount):
        pass


class CardPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Paid {amount} using card")
```

The caller uses `pay()` without needing to know the internal payment processing details.

### 2.5 Inheritance

Inheritance allows a class to reuse or specialize the behavior of another class.

```python
class Animal:
    def speak(self):
        print("Animal makes a sound")


class Dog(Animal):
    def speak(self):
        print("Dog barks")
```

Use inheritance when there is a genuine **is-a** relationship.

Avoid deep inheritance hierarchies when composition provides a simpler solution.

### 2.6 Polymorphism

Polymorphism allows different objects to be used through a common interface.

```python
class CardPayment:
    def pay(self, amount):
        print("Card payment")


class UpiPayment:
    def pay(self, amount):
        print("UPI payment")


def process_payment(payment_method, amount):
    payment_method.pay(amount)


process_payment(CardPayment(), 100)
process_payment(UpiPayment(), 200)
```

The function does not need to know the concrete payment class.

### 2.7 Composition

Composition builds a class by combining other objects.

```python
class Engine:
    def start(self):
        print("Engine started")


class Car:
    def __init__(self, engine):
        self.engine = engine

    def start(self):
        self.engine.start()
```

A car **has an** engine.

Composition generally makes it easier to replace components independently.

### 2.8 Association, Aggregation, and Composition

| Relationship | Meaning | Example |
|---|---|---|
| Association | Objects interact with each other | Teacher and Student |
| Aggregation | One object groups objects that can exist independently | Team and Player |
| Composition | One object owns the lifecycle of its parts | House and Room |

These relationships are conceptual; the exact lifecycle rules depend on the implementation.

---

## 3. Relationships Between Classes

### Is-a Relationship

Represents inheritance.

Example:
- Dog is an Animal.
- CardPayment is a PaymentMethod.

### Has-a Relationship

Represents a class containing or referring to another object.

Example:
- Car has an Engine.
- Order has OrderItems.

### Uses-a Relationship

Represents a temporary dependency or interaction.

Example:
- ReportGenerator uses a DatabaseRepository.

### Prefer Composition Over Inheritance

Composition is often preferable when behavior needs to be changed or combined independently.

Use inheritance when subclasses truly satisfy the contract of the parent class.

---

## 4. SOLID Principles

SOLID is a set of five principles for designing maintainable object-oriented software.

### 4.1 Single Responsibility Principle (SRP)

**A class should have one responsibility and one primary reason to change.**

Bad design:

```python
class Invoice:
    def calculate_total(self):
        pass

    def save_to_database(self):
        pass

    def print_invoice(self):
        pass
```

This class handles calculation, persistence, and presentation.

Better design:

```python
class Invoice:
    def calculate_total(self):
        pass


class InvoiceRepository:
    def save(self, invoice):
        pass


class InvoicePrinter:
    def print_invoice(self, invoice):
        pass
```

Each class has a more focused responsibility.

### 4.2 Open/Closed Principle (OCP)

**Software entities should be open for extension but closed for modification.**

Bad design:

```python
def calculate_discount(customer_type, price):
    if customer_type == "regular":
        return price * 0.05
    elif customer_type == "premium":
        return price * 0.15
```

Every new customer type requires modifying the function.

Better design:

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
```

New discount policies can be added through new implementations.

### 4.3 Liskov Substitution Principle (LSP)

**A subtype should be usable wherever its base type is expected without breaking correctness.**

If a function expects a `Bird` that can fly, a `Penguin` should not inherit that contract and then fail when `fly()` is called.

A better model separates general bird behavior from flight capability.

```python
class Bird:
    def eat(self):
        print("Eating")


class FlyingBird(Bird):
    def fly(self):
        print("Flying")


class Sparrow(FlyingBird):
    pass


class Penguin(Bird):
    def swim(self):
        print("Swimming")
```

Subclasses should honor the behavioral promises of their base classes.

### 4.4 Interface Segregation Principle (ISP)

**Clients should not depend on methods they do not need.**

Instead of one large interface for every machine operation, split responsibilities into smaller interfaces.

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
```

A basic printer can implement `Printer` without being forced to implement scanning.

### 4.5 Dependency Inversion Principle (DIP)

**High-level modules should depend on abstractions, not concrete low-level implementations.**

```python
from abc import ABC, abstractmethod


class NotificationSender(ABC):
    @abstractmethod
    def send(self, message):
        pass


class EmailSender(NotificationSender):
    def send(self, message):
        print(f"Email: {message}")


class NotificationService:
    def __init__(self, sender: NotificationSender):
        self.sender = sender

    def notify(self, message):
        self.sender.send(message)
```

The notification service can work with different senders without depending directly on one implementation.

### SOLID Summary

| Principle | Main Idea |
|---|---|
| SRP | One focused responsibility per class |
| OCP | Extend behavior without repeatedly modifying existing code |
| LSP | Subtypes must honor their base contracts |
| ISP | Prefer small, focused interfaces |
| DIP | Depend on abstractions rather than concrete implementations |

---

## 5. Coupling and Cohesion

### Coupling

Coupling measures how strongly one component depends on another.

- **Tight coupling:** Components depend heavily on concrete implementations.
- **Loose coupling:** Components interact through stable interfaces or abstractions.

Prefer loose coupling because it makes changes and testing easier.

### Cohesion

Cohesion measures how closely related the responsibilities inside a module are.

- **High cohesion:** A module has a clear, focused purpose.
- **Low cohesion:** A module handles unrelated responsibilities.

### Design Goal

Aim for:

- High cohesion
- Low coupling
- Clear boundaries
- Small, understandable interfaces

---

## 6. UML Diagrams

UML stands for Unified Modeling Language. It provides a visual way to describe software structure and behavior.

### 6.1 Class Diagram

A class diagram shows classes, attributes, methods, and relationships.

```text
+---------------------------+
|         Customer          |
+---------------------------+
| - customer_id: int        |
| - name: str               |
+---------------------------+
| + place_order(): Order    |
+---------------------------+
             |
             | places
             v
+---------------------------+
|           Order           |
+---------------------------+
| - order_id: int           |
| - status: str             |
+---------------------------+
| + calculate_total(): float|
| + cancel(): None          |
+---------------------------+
```

Visibility notation:

- `+` Public
- `-` Private
- `#` Protected

### 6.2 Common UML Relationships

```text
Inheritance:       Child --------|> Parent

Association:       ClassA -------- ClassB

Dependency:        ClassA - - - -> ClassB

Aggregation:       Whole o-------- Part

Composition:       Whole *-------- Part
```

### 6.3 Sequence Diagram

A sequence diagram shows how objects communicate over time.

```text
Customer       OrderService       PaymentService
   |                 |                   |
   | place_order()   |                   |
   |---------------->|                   |
   |                 | process_payment() |
   |                 |------------------>|
   |                 |   payment result  |
   |                 |<------------------|
   | order result    |                   |
   |<----------------|                   |
```

Use sequence diagrams to clarify interaction order and dependencies.

### 6.4 Use Case Diagram

A use case diagram shows actors and the goals they achieve through a system.

Example: An e-commerce customer can:
- Register
- Log in
- Browse products
- Add products to a cart
- Place an order
- Make a payment
- Track an order

### 6.5 Activity Diagram

An activity diagram represents a workflow.

Example:

```text
Start
  |
Select Product
  |
Add to Cart
  |
Checkout
  |
Make Payment
  |
Payment Successful?
  |           |
 Yes          No
  |           |
Create Order  Retry or Cancel
  |
End
```

---

## 7. Design Patterns

A design pattern is a reusable solution to a recurring software design problem.

Patterns are not complete applications. They are templates for organizing code and responsibilities.

### 7.1 Creational Patterns

These patterns concern object creation.

| Pattern | Purpose | Example |
|---|---|---|
| Singleton | Restricts a class to one shared instance | Shared configuration |
| Factory Method | Lets subclasses or implementations determine which object to create | Notification factory |
| Abstract Factory | Creates families of related objects | Cross-platform UI components |
| Builder | Constructs complex objects step by step | Complex report configuration |
| Prototype | Creates objects by cloning existing objects | Cloning a configured template |

### 7.2 Structural Patterns

These patterns concern how objects and classes are composed.

| Pattern | Purpose | Example |
|---|---|---|
| Adapter | Makes incompatible interfaces work together | Legacy API adapter |
| Bridge | Separates an abstraction from its implementation | Remote control and device |
| Composite | Treats individual objects and groups uniformly | File and folder tree |
| Decorator | Adds behavior without changing the original class | Logging or caching wrapper |
| Facade | Provides a simplified interface to a complex subsystem | Checkout service |
| Flyweight | Shares reusable state to reduce memory usage | Many similar game objects |
| Proxy | Controls access to another object | Lazy loading or access control |

### 7.3 Behavioral Patterns

These patterns concern communication and responsibilities between objects.

| Pattern | Purpose | Example |
|---|---|---|
| Strategy | Makes algorithms interchangeable | Payment methods |
| Observer | Notifies subscribers when state changes | Notification system |
| State | Changes behavior based on internal state | Order lifecycle |
| Command | Encapsulates an action as an object | Undoable editor action |
| Chain of Responsibility | Passes a request through handlers | Middleware pipeline |
| Template Method | Defines algorithm steps while allowing selected steps to vary | Data import workflow |
| Iterator | Traverses a collection | Custom collection iterator |
| Mediator | Centralizes communication between components | Chat room |
| Memento | Captures and restores previous state | Undo history |
| Visitor | Adds operations over object structures | Syntax-tree analysis |
| Interpreter | Evaluates a simple language or grammar | Rule expression evaluator |

### 7.4 Strategy Pattern Example

```python
class CreditCardPayment:
    def pay(self, amount):
        print(f"Paid {amount} using credit card")


class UpiPayment:
    def pay(self, amount):
        print(f"Paid {amount} using UPI")


class Checkout:
    def __init__(self, payment_strategy):
        self.payment_strategy = payment_strategy

    def complete_payment(self, amount):
        self.payment_strategy.pay(amount)


checkout = Checkout(UpiPayment())
checkout.complete_payment(500)
```

The checkout logic remains unchanged when the payment strategy changes.

### 7.5 Observer Pattern Example

```python
class NewsPublisher:
    def __init__(self):
        self.subscribers = []

    def subscribe(self, subscriber):
        self.subscribers.append(subscriber)

    def publish(self, news):
        for subscriber in self.subscribers:
            subscriber.update(news)


class NewsSubscriber:
    def update(self, news):
        print(f"Received news: {news}")


publisher = NewsPublisher()
publisher.subscribe(NewsSubscriber())
publisher.publish("New article published")
```

The publisher notifies registered subscribers when news is published.

### 7.6 Factory Method Example

```python
from abc import ABC, abstractmethod


class Notification(ABC):
    @abstractmethod
    def send(self, message):
        pass


class EmailNotification(Notification):
    def send(self, message):
        print(f"Email: {message}")


class SMSNotification(Notification):
    def send(self, message):
        print(f"SMS: {message}")


def create_notification(channel):
    if channel == "email":
        return EmailNotification()
    if channel == "sms":
        return SMSNotification()
    raise ValueError("Unsupported channel")


notification = create_notification("email")
notification.send("Hello")
```

This is a simple factory function. It demonstrates centralized creation, although it is not the formal GoF Factory Method pattern.

---

## 8. Important Object-Oriented Design Principles

### Program to an Interface

Depend on the behavior a component requires rather than its concrete implementation.

### Encapsulate What Varies

Identify behavior likely to change and isolate it behind an interface or strategy.

### Favor Composition Over Inheritance

Combine objects to build behavior when inheritance would create rigid hierarchies.

### Keep Classes Focused

A class should have a clear purpose and a manageable set of responsibilities.

### Separate Concerns

Keep business logic, persistence, presentation, and external communication separate where useful.

### Avoid Premature Abstraction

Do not introduce interfaces and patterns without a real need. Start with a clear design and refactor when variation or complexity justifies it.

### YAGNI

"You Aren't Gonna Need It."

Do not implement speculative features that are not currently required.

### DRY

"Don't Repeat Yourself."

Avoid duplicating the same knowledge or business rule across the codebase.

### KISS

"Keep It Simple, Stupid."

Prefer the simplest design that correctly meets the requirements.

---

## 9. Dependency Injection

Dependency Injection (DI) means providing an object with its dependencies rather than having it construct them internally.

### Without Dependency Injection

```python
class OrderService:
    def __init__(self):
        self.payment_service = PaymentService()
```

The class is tightly coupled to `PaymentService`.

### With Dependency Injection

```python
class OrderService:
    def __init__(self, payment_service):
        self.payment_service = payment_service
```

Usage:

```python
payment_service = PaymentService()
order_service = OrderService(payment_service)
```

Benefits:
- Easier unit testing
- Lower coupling
- More flexible implementations
- Clearer dependencies

Dependency injection can be performed through constructors, methods, or properties. Constructor injection is often a good default.

---

## 10. Interfaces and Abstract Classes

An interface defines a contract that implementations should satisfy.

An abstract class can define a contract and share common implementation.

Python supports abstract classes through the `abc` module.

```python
from abc import ABC, abstractmethod


class Storage(ABC):
    @abstractmethod
    def save(self, data):
        pass


class FileStorage(Storage):
    def save(self, data):
        print("Saving data to a file")


class DatabaseStorage(Storage):
    def save(self, data):
        print("Saving data to a database")
```

Both storage implementations satisfy the same contract.

Use an abstraction when multiple implementations need to be interchangeable or when it establishes a useful architectural boundary.

---

## 11. Common LLD Interview Questions

Practice designing these systems:

### Beginner Level

- Parking Lot
- Library Management System
- Tic-Tac-Toe
- Deck of Cards
- Vending Machine
- Elevator System

### Intermediate Level

- ATM
- Movie Ticket Booking System
- Hotel Booking System
- Splitwise
- Food Delivery System
- E-commerce Shopping Cart
- Notification Service

### Advanced Level

- Chess
- Ride-Sharing System
- Logging Framework
- Rate Limiter
- Task Scheduler
- In-Memory Cache
- File System
- Multiplayer Board Game

---

## 12. Step-by-Step Approach to Solving LLD Problems

### Step 1: Gather Requirements

Identify:
- Main use cases
- Actors
- Core business rules
- Constraints
- Expected scale of the object model
- Error cases

Ask clarifying questions rather than making too many assumptions.

### Step 2: Identify Core Entities

For a parking lot, possible entities include:

- ParkingLot
- ParkingFloor
- ParkingSpot
- Vehicle
- Ticket
- Payment
- ParkingManager

Do not create a class for every noun automatically. Include entities that have meaningful state, behavior, or responsibility.

### Step 3: Define Responsibilities

Decide what each class should do.

Example:

- `Vehicle`: stores vehicle information.
- `ParkingSpot`: manages occupancy of a spot.
- `ParkingFloor`: organizes parking spots.
- `Ticket`: records parking session information.
- `Payment`: represents payment details and status.
- `ParkingLot`: coordinates parking operations.

### Step 4: Define Relationships

Determine:
- Which classes contain other objects?
- Which objects communicate?
- Which relationships require inheritance?
- Where should interfaces be used?

Prefer composition for flexible relationships.

### Step 5: Define Attributes and Methods

Example:

```python
class ParkingSpot:
    def __init__(self, spot_id, vehicle_type):
        self.spot_id = spot_id
        self.vehicle_type = vehicle_type
        self.vehicle = None

    def is_available(self):
        return self.vehicle is None

    def park_vehicle(self, vehicle):
        if not self.is_available():
            raise ValueError("Spot is occupied")

        self.vehicle = vehicle

    def remove_vehicle(self):
        vehicle = self.vehicle
        self.vehicle = None
        return vehicle
```

### Step 6: Apply SOLID Principles

Check whether:
- Every class has a focused responsibility.
- New behavior can be added without extensive modifications.
- Subtypes honor their contracts.
- Interfaces remain small.
- Dependencies point toward abstractions where appropriate.

### Step 7: Apply Design Patterns When Justified

Examples:
- Strategy for interchangeable pricing or payment rules.
- Factory for selecting object implementations.
- Observer for event notifications.
- State for lifecycle-dependent behavior.
- Decorator for optional behavior such as logging or extra features.

Do not force a pattern into the design just to use it.

### Step 8: Handle Edge Cases

For a parking lot:
- No spots available
- Vehicle type not supported
- Duplicate parking attempts
- Invalid ticket
- Repeated checkout
- Payment failure

### Step 9: Consider Concurrency

If multiple operations can happen simultaneously, consider:
- Race conditions
- Atomic state changes
- Locks or transactions
- Duplicate requests
- Consistency of shared resources

For example, two vehicles must not successfully reserve the same spot.

### Step 10: Write and Test the Code

Test:
- Normal flows
- Invalid inputs
- Boundary conditions
- State transitions
- Failure scenarios
- Interactions between components

Keep business logic testable without requiring every external service.

---

## 13. Example LLD: Parking Lot

### Requirements

- Support different vehicle types.
- Manage parking floors and spots.
- Assign available spots.
- Issue parking tickets.
- Process payment during exit.
- Release spots after successful checkout.

### Core Classes

```text
ParkingLot
    |
    +-- ParkingFloor
            |
            +-- ParkingSpot
                    |
                    +-- Vehicle

ParkingLot
    |
    +-- Ticket
    |
    +-- PaymentService
```

### Example Enums and Models

```python
from enum import Enum
from dataclasses import dataclass


class VehicleType(Enum):
    MOTORCYCLE = "motorcycle"
    CAR = "car"
    TRUCK = "truck"


@dataclass
class Vehicle:
    license_plate: str
    vehicle_type: VehicleType


class ParkingSpot:
    def __init__(self, spot_id, supported_type):
        self.spot_id = spot_id
        self.supported_type = supported_type
        self.vehicle = None

    def is_available(self):
        return self.vehicle is None

    def can_fit(self, vehicle):
        return (
            self.is_available()
            and vehicle.vehicle_type == self.supported_type
        )

    def park(self, vehicle):
        if not self.can_fit(vehicle):
            raise ValueError("Vehicle cannot use this spot")

        self.vehicle = vehicle

    def release(self):
        vehicle = self.vehicle
        self.vehicle = None
        return vehicle
```

This is a simplified model. A complete implementation would also define floors, ticket lifecycle, pricing, payment handling, and concurrency control.

### Design Considerations

- Should larger vehicles fit in smaller spots?
- How is the parking fee calculated?
- Can a ticket be paid twice?
- What happens if payment fails?
- How are spots reserved safely under concurrent requests?
- How should new pricing strategies be added?

Answer these questions before implementing a full system.

---

## 14. State Management and State Machines

Many LLD problems involve objects that change behavior according to their state.

Example: An order can move through these states:

```text
CREATED
   |
   v
CONFIRMED
   |
   v
SHIPPED
   |
   v
DELIVERED
```

Cancellation might be allowed only from selected states.

Instead of scattering state checks throughout the application, centralize and validate permitted transitions.

```python
class Order:
    ALLOWED_TRANSITIONS = {
        "created": {"confirmed", "cancelled"},
        "confirmed": {"shipped", "cancelled"},
        "shipped": {"delivered"},
        "delivered": set(),
        "cancelled": set(),
    }

    def __init__(self):
        self.status = "created"

    def transition_to(self, new_status):
        allowed = self.ALLOWED_TRANSITIONS[self.status]

        if new_status not in allowed:
            raise ValueError(
                f"Cannot transition from {self.status} to {new_status}"
            )

        self.status = new_status
```

For more complex workflows, the State pattern may make each state's behavior easier to maintain.

---

## 15. Error Handling and Validation

A good design defines how invalid operations are handled.

Examples:
- Reject invalid input.
- Raise meaningful exceptions.
- Validate state transitions.
- Avoid silently swallowing errors.
- Distinguish business errors from infrastructure failures.
- Ensure failed operations do not leave inconsistent state.

Example:

```python
class InsufficientBalanceError(Exception):
    pass


class Wallet:
    def __init__(self, balance=0):
        self.balance = balance

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Amount must be positive")

        if amount > self.balance:
            raise InsufficientBalanceError("Insufficient balance")

        self.balance -= amount
```

---

## 16. Unit Testing in LLD

A well-designed system should make individual classes easy to test.

```python
def test_wallet_withdrawal():
    wallet = Wallet(100)

    wallet.withdraw(30)

    assert wallet.balance == 70
```

Important testing categories:

- Unit tests: test one class or function.
- Integration tests: test cooperating components.
- Edge-case tests: test unusual inputs and boundaries.
- State-transition tests: validate allowed and forbidden transitions.
- Failure tests: verify behavior when dependencies fail.

Inject dependencies so they can be replaced with test doubles when needed.

---

## 17. Common LLD Mistakes

- Creating too many classes too early.
- Using inheritance for every relationship.
- Applying design patterns unnecessarily.
- Building large classes with unrelated responsibilities.
- Exposing mutable internal state without validation.
- Depending on concrete classes everywhere.
- Ignoring failure scenarios.
- Ignoring concurrency in shared resources.
- Overengineering hypothetical future requirements.
- Writing code before clarifying requirements.
- Focusing only on class diagrams and ignoring behavior.

A good design is not the one with the most classes or patterns. It is the simplest design that satisfies the requirements and remains easy to change.

---

## 18. LLD Practice Checklist

Before considering a design complete, ask:

- [ ] Have I clarified the functional requirements?
- [ ] Have I identified the important entities?
- [ ] Does each class have a clear responsibility?
- [ ] Are relationships modeled correctly?
- [ ] Have I used composition where appropriate?
- [ ] Are interfaces and abstractions meaningful?
- [ ] Does the design follow SOLID where relevant?
- [ ] Are changing behaviors isolated?
- [ ] Are error cases handled?
- [ ] Are object states and transitions valid?
- [ ] Have I considered concurrency if required?
- [ ] Can I unit-test the important business logic?
- [ ] Have I avoided unnecessary abstractions?
- [ ] Can I explain why I chose this design?

---

## 19. Recommended Learning Order

1. OOP fundamentals
2. Classes, objects, inheritance, and polymorphism
3. Association, aggregation, and composition
4. Encapsulation and abstraction
5. SOLID principles
6. Coupling and cohesion
7. UML class and sequence diagrams
8. Dependency injection
9. Creational design patterns
10. Structural design patterns
11. Behavioral design patterns
12. State machines and error handling
13. LLD practice problems
14. Testing and refactoring

### Suggested Practice Routine

For each design problem:

1. Write the requirements.
2. Identify entities and responsibilities.
3. Draw a class diagram.
4. Describe important interactions.
5. Implement the core classes.
6. Test normal and failure scenarios.
7. Review SOLID principles.
8. Refactor only where there is a clear improvement.

---

## 20. Final Takeaways

- LLD focuses on the internal design of software components.
- OOP provides the building blocks for modeling state and behavior.
- SOLID helps make object-oriented designs easier to maintain.
- UML helps communicate structure and interactions.
- Design patterns provide reusable solutions to recurring problems.
- Dependency injection makes components easier to replace and test.
- Composition is often more flexible than inheritance.
- Good designs handle edge cases and invalid states.
- Patterns should solve real problems rather than add unnecessary complexity.
- Consistent practice is the best way to improve at LLD.
