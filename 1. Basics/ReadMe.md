
# Relationships Between Classes in Low-Level Design (LLD)

## 1. Introduction

In Object-Oriented Programming (OOP), classes interact with one another to build complex software systems.

Class relationships describe how classes are connected, how they use one another, and how objects depend on or contain other objects.

Understanding these relationships is essential for:
- Designing class diagrams.
- Applying SOLID principles.
- Choosing between inheritance and composition.
- Reducing coupling.
- Building maintainable software.
- Solving Low-Level Design (LLD) interview problems.

## 2. Types of Class Relationships

The main UML class relationships are:

1. Association
2. Aggregation
3. Composition
4. Inheritance (Generalization)
5. Dependency
6. Realization

---

## 3. Association

### Definition

**Association represents a structural relationship in which objects of one class are connected to or interact with objects of another class.**

One class may know about or hold a reference to another class.

### Real-World Examples

- A Teacher teaches Students.
- A Doctor treats Patients.
- A Customer places Orders.
- A Driver drives a Vehicle.

### Python Example

```python
class Student:
    def __init__(self, name):
        self.name = name


class Teacher:
    def __init__(self, name):
        self.name = name

    def teach(self, student):
        print(f"{self.name} teaches {student.name}")


student = Student("Alice")
teacher = Teacher("Mr. John")

teacher.teach(student)
```

### UML Representation

```text
+---------+        +---------+
| Teacher |--------| Student |
+---------+        +---------+
```

A solid line represents association in a UML class diagram.

### Characteristics

- Represents a connection between objects.
- Neither class necessarily owns the other.
- Objects may exist independently.
- Can be one-to-one, one-to-many, or many-to-many.

### Types of Association by Multiplicity

| Multiplicity | Meaning | Example |
|---|---|---|
| 1 to 1 | One object relates to one object | Person and Passport |
| 1 to many | One object relates to many objects | Manager and Employees |
| Many to many | Multiple objects relate to multiple objects | Students and Courses |

**Key point:** Association does not automatically imply ownership or lifecycle control.

---

## 4. Aggregation

### Definition

**Aggregation is a specialized form of association that represents a whole-part relationship in which the parts can exist independently of the whole.**

The whole groups or uses the parts, but does not exclusively own their lifecycle.

### Real-World Examples

- A Team has Players.
- A Department has Employees.
- A Library has Books.
- A Playlist contains Songs.

For example, a player can continue to exist even if a team is disbanded.

### Python Example

```python
class Player:
    def __init__(self, name):
        self.name = name


class Team:
    def __init__(self, name, players):
        self.name = name
        self.players = players

    def show_players(self):
        for player in self.players:
            print(player.name)


player1 = Player("Alice")
player2 = Player("Bob")

team = Team("Falcons", [player1, player2])
team.show_players()
```

The players are created independently and passed into the team.

### UML Representation

```text
+------+ ◇---------- +--------+
| Team |             | Player |
+------+             +--------+
```

The hollow diamond is placed on the **whole** side.

### Characteristics

- Represents a weak whole-part relationship.
- Parts can exist independently.
- Parts may be shared by multiple aggregates, depending on the domain.
- The whole does not necessarily control the lifecycle of its parts.

### When to Use Aggregation

Use aggregation when an object groups other objects without owning their lifecycles.

**Important:** In programming languages such as Python, aggregation and ordinary association may look similar in code. Their distinction is mainly about domain meaning and ownership.

---

## 5. Composition

### Definition

**Composition is a strong whole-part relationship in which the whole conceptually owns its parts and controls their lifecycle.**

The parts belong to the whole and are not normally shared as independently owned components.

### Real-World Examples

- A House has Rooms.
- An Order has OrderLineItems.
- A Document has Pages.
- A Computer has internally owned components in a simplified model.

The precise lifecycle relationship depends on the domain model.

### Python Example

```python
class Room:
    def __init__(self, name):
        self.name = name


class House:
    def __init__(self):
        self._rooms = [
            Room("Bedroom"),
            Room("Kitchen"),
            Room("Living Room"),
        ]

    def show_rooms(self):
        for room in self._rooms:
            print(room.name)


house = House()
house.show_rooms()
```

In this example, the house creates and manages its rooms internally.

### UML Representation

```text
+-------+ ◆---------- +------+
| House |             | Room |
+-------+             +------+
```

The filled diamond is placed on the **whole** side.

### Characteristics

- Represents strong ownership.
- The whole controls the lifecycle of the parts.
- Parts are normally not shared as independently owned components.
- The relationship expresses a stronger commitment than aggregation.

### When to Use Composition

Use composition when a component belongs to another object and its lifecycle is managed as part of that object.

**Important:** Composition does not mean a part must be destroyed immediately in every programming language when the whole disappears. UML describes a lifecycle and ownership relationship; actual memory management depends on the language and implementation.

---

## 6. Association vs Aggregation vs Composition

| Feature | Association | Aggregation | Composition |
|---|---|---|---|
| Relationship | General connection | Weak whole-part | Strong whole-part |
| Ownership | Not implied | Weak grouping | Strong ownership |
| Independent lifecycle | Usually possible | Yes | Usually tied to the whole |
| Shared parts | Possible | Possible | Normally not independently shared |
| UML symbol | Solid line | Hollow diamond | Filled diamond |
| Example | Teacher–Student | Team–Player | Order–OrderItem |

### Easy Way to Remember

- **Association:** Objects know about or interact with each other.
- **Aggregation:** A whole groups independently existing parts.
- **Composition:** A whole strongly owns its parts.

---

## 7. Inheritance (Generalization)

### Definition

**Inheritance represents an IS-A relationship. A subclass inherits or specializes the behavior and data of a superclass.**

### Real-World Examples

- Dog is an Animal.
- Car is a Vehicle.
- CardPayment is a PaymentMethod.
- AdminUser is a User.

### Python Example

```python
class Animal:
    def eat(self):
        print("Eating")


class Dog(Animal):
    def bark(self):
        print("Barking")


dog = Dog()
dog.eat()
dog.bark()
```

`Dog` inherits the `eat()` method from `Animal`.

### UML Representation

```text
+-----+
| Dog |
+-----+
   |
   |  (solid line with hollow triangle
   |   pointing toward the parent)
   ▼
+--------+
| Animal |
+--------+
```

The hollow triangle points toward the superclass.

### Characteristics

- Represents an IS-A relationship.
- Supports specialization and polymorphism.
- Reuses behavior from a parent class.
- Requires subclasses to respect the behavioral contract of the parent.

### When to Use Inheritance

Use inheritance when the subclass can genuinely substitute for the parent class.

Avoid inheritance simply to reuse a few methods. Composition may be more flexible.

---

## 8. Dependency

### Definition

**Dependency represents a relationship in which one class temporarily uses or relies on another class to perform an operation.**

A change in the depended-on class may affect the dependent class.

### Real-World Examples

- A ReportGenerator uses a Printer.
- An OrderService uses a PaymentService.
- A Controller uses a Validator.
- A NotificationService uses an EmailSender.

### Python Example

```python
class Printer:
    def print_document(self, document):
        print(document)


class ReportGenerator:
    def generate(self):
        return "Monthly Report"

    def print_report(self, printer):
        report = self.generate()
        printer.print_document(report)


printer = Printer()
report_generator = ReportGenerator()

report_generator.print_report(printer)
```

`ReportGenerator` uses `Printer` as a method parameter rather than owning it as a persistent part.

### UML Representation

```text
+-----------------+       +---------+
| ReportGenerator | - - ->| Printer |
+-----------------+       +---------+
```

A dashed arrow represents dependency. The arrow points toward the depended-on class.

### Characteristics

- Often represents temporary use.
- Can occur through method parameters, local variables, or method calls.
- Usually weaker than a persistent structural association.
- Excessive dependencies can make code difficult to change and test.

### When to Use Dependency

Use this concept when explaining how one class uses another without necessarily maintaining a long-lived relationship.

---

## 9. Realization

### Definition

**Realization represents a relationship in which a class implements a contract defined by an interface or specification.**

In Python, an abstract base class can represent such a contract.

### Python Example

```python
from abc import ABC, abstractmethod


class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount):
        pass


class CardPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Paid {amount} using a card")


class UpiPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Paid {amount} using UPI")
```

Both classes implement the `PaymentMethod` contract.

### UML Representation

```text
+-------------+        +----------------+
| CardPayment | - - -▷ | PaymentMethod  |
+-------------+        +----------------+
```

A dashed line with a hollow triangle points toward the interface or contract.

### Characteristics

- Represents implementation of a contract.
- Supports polymorphism.
- Allows implementations to be interchangeable.
- Helps reduce coupling between clients and concrete implementations.

---

## 10. Quick UML Symbol Reference

| Relationship | UML Representation | Meaning |
|---|---|---|
| Association | `A -------- B` | Structural connection |
| Aggregation | `A ◇------ B` | Weak whole-part relationship |
| Composition | `A ◆------ B` | Strong whole-part relationship |
| Inheritance | `Child ------▷ Parent` | IS-A relationship |
| Dependency | `A - - - -> B` | A uses B |
| Realization | `Class - - -▷ Interface` | Class implements a contract |

For aggregation and composition, the diamond is placed next to the whole.

For inheritance and realization, the hollow triangle points toward the parent or interface.

---

## 11. Relationship Examples in an E-Commerce System

Consider an e-commerce application.

### Customer and Order — Association

A customer places orders. The customer and order are modeled as connected domain objects.

```text
Customer 1 -------- 0..* Order
```

One customer can place multiple orders. Each order normally belongs to one customer.

### Order and OrderItem — Composition

An order owns its line items as part of its structure.

```text
Order 1 ◆-------- 1..* OrderItem
```

The order manages its order items in this domain model.

### Team and Player — Aggregation

A team groups players, while players can exist independently.

```text
Team 1 ◇-------- 0..* Player
```

### CardPayment and PaymentMethod — Realization

Card payment implements a common payment contract.

```text
CardPayment - - -▷ PaymentMethod
```

### OrderService and PaymentService — Dependency

An order service uses a payment service to process checkout.

```text
OrderService - - -> PaymentService
```

These examples are domain-dependent. A real system may model some relationships differently based on its lifecycle and business rules.

---

## 12. Multiplicity in UML

Multiplicity describes how many objects can participate in a relationship.

| Multiplicity | Meaning |
|---|---|
| `1` | Exactly one |
| `0..1` | Zero or one |
| `0..*` | Zero or many |
| `1..*` | One or many |
| `2..5` | Between two and five |

### Example

```text
Customer 1 -------- 0..* Order
```

This means one customer may have zero or more orders, while each order is associated with exactly one customer in this model.

Multiplicity should reflect actual business rules rather than assumptions.

---

## 13. How to Choose the Right Relationship

Ask the following questions when designing classes.

### Question 1: Is it an IS-A relationship?

Use **Inheritance** if a subclass can substitute for its parent.

Example: `Dog` is an `Animal`.

### Question 2: Does one object own another object's lifecycle?

Use **Composition** when the whole strongly owns the part.

Example: `Order` owns its `OrderItem` objects.

### Question 3: Does one object group independent objects?

Use **Aggregation** when the parts exist independently of the whole.

Example: `Team` groups `Player` objects.

### Question 4: Do two objects maintain a structural connection?

Use **Association**.

Example: `Customer` and `Order`.

### Question 5: Does one class temporarily use another?

Use **Dependency**.

Example: A report generator uses a printer.

### Question 6: Does a class implement a contract?

Use **Realization**.

Example: `CardPayment` implements `PaymentMethod`.

---

## 14. Common LLD Mistakes

- Using inheritance when composition is more appropriate.
- Confusing aggregation with composition.
- Assuming every object reference represents composition.
- Forgetting multiplicity in class diagrams.
- Making classes tightly coupled to concrete implementations.
- Using inheritance without checking the Liskov Substitution Principle.
- Assuming UML relationships automatically determine memory management.
- Creating relationships that do not represent actual business requirements.

---

## 15. Interview Questions

### Q1. What is the difference between association and dependency?

Association generally represents a structural connection between objects. Dependency represents a usage relationship where one class relies on another.

### Q2. What is the difference between aggregation and composition?

Aggregation represents a weak whole-part relationship in which parts can exist independently. Composition represents strong ownership of parts by the whole.

### Q3. What is the difference between inheritance and composition?

Inheritance represents an IS-A relationship. Composition represents a HAS-A relationship and allows behavior to be assembled from other objects.

### Q4. Which relationship is represented by a hollow diamond?

Aggregation.

### Q5. Which relationship is represented by a filled diamond?

Composition.

### Q6. What is realization?

Realization represents a class implementing an interface or contract.

### Q7. Why is composition often preferred over inheritance?

Composition allows components to be replaced or combined independently and can reduce rigid inheritance hierarchies.

### Q8. Can aggregation and association look the same in Python?

Yes. Both may use ordinary object references. The difference lies in the intended domain semantics and ownership.

---

## 16. Final Takeaways

- Association represents a general structural connection.
- Aggregation represents a weak whole-part relationship.
- Composition represents strong ownership of parts.
- Inheritance represents an IS-A relationship.
- Dependency represents usage of another class.
- Realization represents implementation of a contract.
- Multiplicity specifies how many objects participate in a relationship.
- Use composition when it models the domain better than inheritance.
- Always choose relationships based on actual requirements and lifecycle rules.
