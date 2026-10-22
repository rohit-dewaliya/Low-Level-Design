
# Behavioral Design Patterns in Low-Level Design (LLD)

## 1. What Are Behavioral Design Patterns?

Behavioral design patterns focus on **how objects communicate with one another and how responsibilities are distributed between them**.

They help reduce tight coupling and make object interactions easier to extend, maintain, and test.

Examples of problems they solve:

- Notifying multiple services when an order is shipped
- Switching between payment or routing algorithms
- Changing an object's behavior based on its state
- Passing requests through authentication and validation handlers
- Supporting undo and redo in an editor

The classic Gang of Four (GoF) catalog contains **11 behavioral design patterns**.

## 2. The 11 Behavioral Design Patterns

| Pattern | Main purpose | Example |
|---|---|---|
| Strategy | Make algorithms interchangeable | Payment methods |
| Observer | Notify subscribers when an event occurs | Order notifications |
| State | Change behavior according to internal state | Order lifecycle |
| Command | Represent a request as an object | Undo/redo, task queues |
| Chain of Responsibility | Pass a request through handlers | HTTP middleware |
| Template Method | Define a workflow with customizable steps | Data import |
| Iterator | Traverse a collection without exposing its internals | Custom collections |
| Mediator | Coordinate communication between objects | Chat room |
| Memento | Capture and restore previous state | Editor history |
| Visitor | Add operations across object types | Shape export or area calculations |
| Interpreter | Represent and evaluate a grammar | Simple expression evaluator |

---

## 3. Strategy Pattern

### Intent

Define a family of algorithms, encapsulate each one, and make them interchangeable.

### Problem

A payment system may support UPI, credit cards, and PayPal. A large `if/elif` block becomes harder to maintain whenever a new payment method is added.

### Structure

```text
PaymentService
      |
      v
PaymentStrategy
   /     |      \
  v      v       v
UPI    Card    PayPal
```

### Python example

```python
from abc import ABC, abstractmethod


class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount):
        pass


class UPIPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"Paid ₹{amount} using UPI")


class CardPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"Paid ₹{amount} using Card")


class PaymentService:
    def __init__(self, strategy):
        self.strategy = strategy

    def checkout(self, amount):
        self.strategy.pay(amount)


service = PaymentService(UPIPayment())
service.checkout(500)

service = PaymentService(CardPayment())
service.checkout(1000)
```

### When to use it

- Payment methods
- Pricing and discount rules
- Sorting algorithms
- Route-selection algorithms
- Authentication approaches

### SOLID connection

Supports the Open/Closed Principle and Dependency Inversion Principle when the service depends on an abstraction rather than concrete algorithms.

---

## 4. Observer Pattern

### Intent

Define a one-to-many dependency so that when one object changes or emits an event, its subscribers are notified.

### Problem

When an order ships, the application may need to notify email, SMS, and push-notification services. The order should not need to depend directly on every notification implementation.

### Structure

```text
Order / Publisher
       |
       | notifies
       v
  Subscribers
   /   |    \
  v    v     v
Email  SMS  Push
```

### Python example

```python
class Order:
    def __init__(self):
        self.subscribers = []

    def subscribe(self, subscriber):
        self.subscribers.append(subscriber)

    def unsubscribe(self, subscriber):
        self.subscribers.remove(subscriber)

    def ship(self):
        print("Order shipped!")

        for subscriber in self.subscribers:
            subscriber.update("Your order has shipped")


class EmailNotifier:
    def update(self, message):
        print(f"Email: {message}")


class SMSNotifier:
    def update(self, message):
        print(f"SMS: {message}")


order = Order()
order.subscribe(EmailNotifier())
order.subscribe(SMSNotifier())
order.ship()
```

### When to use it

- Event notification systems
- GUI event listeners
- Stock-price alerts
- Order status updates
- Real-time dashboards

### Key idea

The publisher communicates through a subscriber contract rather than knowing the concrete details of each subscriber.

---

## 5. State Pattern

### Intent

Allow an object to change its behavior when its internal state changes.

### Problem

An order can be pending, paid, shipped, or delivered. A class with many state-dependent conditions can become difficult to maintain.

### Structure

```text
Pending -> Paid -> Shipped -> Delivered
```

Real systems should also define valid transitions, such as whether cancellation is permitted from a given state.

### Python example

```python
from abc import ABC, abstractmethod


class OrderState(ABC):
    @abstractmethod
    def next_state(self, order):
        pass


class PendingState(OrderState):
    def next_state(self, order):
        order.state = PaidState()


class PaidState(OrderState):
    def next_state(self, order):
        order.state = ShippedState()


class ShippedState(OrderState):
    def next_state(self, order):
        order.state = DeliveredState()


class DeliveredState(OrderState):
    def next_state(self, order):
        print("Order already delivered")


class Order:
    def __init__(self):
        self.state = PendingState()

    def advance(self):
        self.state.next_state(self)


order = Order()
order.advance()  # Pending -> Paid
order.advance()  # Paid -> Shipped
order.advance()  # Shipped -> Delivered
```

This is a simplified example. A production implementation should validate transitions and handle invalid operations explicitly.

### When to use it

- Order lifecycles
- Vending machines
- Media players
- Document approval workflows
- Connection states

### Strategy vs State

| Strategy | State |
|---|---|
| Selects an algorithm | Represents the object's current condition |
| Often selected by a client or configuration | Often changes through internal transitions |
| Example: UPI vs card payment | Example: pending vs shipped order |

---

## 6. Command Pattern

### Intent

Encapsulate a request as an object.

### Problem

A remote control should operate different devices without containing the implementation details of every device action.

### Structure

```text
RemoteControl -> Command -> Device
```

### Python example

```python
class Light:
    def turn_on(self):
        print("Light ON")

    def turn_off(self):
        print("Light OFF")


class TurnOnCommand:
    def __init__(self, light):
        self.light = light

    def execute(self):
        self.light.turn_on()


class TurnOffCommand:
    def __init__(self, light):
        self.light = light

    def execute(self):
        self.light.turn_off()


class RemoteControl:
    def press(self, command):
        command.execute()


light = Light()
remote = RemoteControl()

remote.press(TurnOnCommand(light))
remote.press(TurnOffCommand(light))
```

### When to use it

- Undo/redo operations
- Task queues
- Background jobs
- Smart-home controls
- Command history

### Interview tip

Command is useful when requests must be queued, logged, retried, scheduled, or reversed. Undo generally requires commands to retain enough information to reverse their effects.

---

## 7. Chain of Responsibility Pattern

### Intent

Pass a request through a chain of handlers until it is processed or rejected.

### Problem

An HTTP request may need authentication, validation, rate limiting, and business processing. Hardcoding every step into one function can make the pipeline difficult to extend.

### Structure

```text
Request
   |
   v
AuthenticationHandler
   |
   v
ValidationHandler
   |
   v
RateLimitHandler
   |
   v
BusinessHandler
```

### Python example

```python
class Handler:
    def __init__(self, next_handler=None):
        self.next_handler = next_handler

    def handle(self, request):
        if self.next_handler:
            return self.next_handler.handle(request)
        return True


class AuthenticationHandler(Handler):
    def handle(self, request):
        if not request.get("authenticated"):
            print("Authentication failed")
            return False

        return super().handle(request)


class ValidationHandler(Handler):
    def handle(self, request):
        if not request.get("data"):
            print("Invalid request")
            return False

        return super().handle(request)


pipeline = AuthenticationHandler(ValidationHandler())

pipeline.handle({
    "authenticated": True,
    "data": {"name": "Alice"}
})
```

### When to use it

- HTTP middleware
- Logging pipelines
- Validation workflows
- Support-ticket escalation
- Request-processing pipelines

---

## 8. Template Method Pattern

### Intent

Define the skeleton of an algorithm in a base class while allowing subclasses to customize particular steps.

### Problem

CSV and JSON imports may follow the same workflow, but the data-reading logic differs.

### Structure

```text
import_data()
   |
   +-- read_data()    [customizable]
   +-- transform()    [optional override]
   +-- save()         [shared/default]
```

### Python example

```python
from abc import ABC, abstractmethod


class DataImporter(ABC):
    def import_data(self):
        data = self.read_data()
        data = self.transform(data)
        self.save(data)

    @abstractmethod
    def read_data(self):
        pass

    def transform(self, data):
        return data

    def save(self, data):
        print("Saving:", data)


class CSVImporter(DataImporter):
    def read_data(self):
        return ["CSV row 1", "CSV row 2"]


class JSONImporter(DataImporter):
    def read_data(self):
        return [{"name": "Alice"}]


CSVImporter().import_data()
JSONImporter().import_data()
```

### When to use it

- Data import pipelines
- Report generation
- Common testing workflows
- File processing
- Framework lifecycle hooks

### Strategy vs Template Method

Strategy usually delegates an algorithm to a separate object through composition. Template Method defines a fixed workflow in a base class and customizes selected steps through inheritance.

---

## 9. Iterator Pattern

### Intent

Traverse a collection sequentially without exposing its internal representation.

Python supports the iterator protocol directly.

### Python example

```python
numbers = [10, 20, 30]

iterator = iter(numbers)

print(next(iterator))  # 10
print(next(iterator))  # 20
print(next(iterator))  # 30
```

A custom iterator:

```python
class CountUp:
    def __init__(self, limit):
        self.limit = limit
        self.current = 1

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.limit:
            raise StopIteration

        value = self.current
        self.current += 1
        return value


for number in CountUp(3):
    print(number)
```

### When to use it

- Custom data structures
- Lazy data processing
- Streaming large datasets
- Pagination
- Tree traversal

---

## 10. Mediator Pattern

### Intent

Centralize communication between collaborating objects to reduce direct dependencies.

### Problem

In a chat application, users should not need to know every other user to send messages.

### Structure

```text
Alice ----\
Bob -------+----> ChatRoom (Mediator)
Charlie --/                |
                    Distributes messages
```

### Python example

```python
class ChatRoom:
    def __init__(self):
        self.users = []

    def join(self, user):
        self.users.append(user)

    def send_message(self, sender, message):
        for user in self.users:
            if user != sender:
                user.receive(sender.name, message)


class User:
    def __init__(self, name, room):
        self.name = name
        self.room = room

    def send(self, message):
        self.room.send_message(self, message)

    def receive(self, sender, message):
        print(f"{self.name} received from {sender}: {message}")


room = ChatRoom()
alice = User("Alice", room)
bob = User("Bob", room)

room.join(alice)
room.join(bob)

alice.send("Hello!")
```

### When to use it

- Chat rooms
- Complex UI components
- Workflow coordination
- Systems where many objects interact

### Observer vs Mediator

Observer distributes notifications from a publisher to subscribers. Mediator coordinates interactions between collaborating objects.

---

## 11. Memento Pattern

### Intent

Capture and restore an object's previous state without exposing its internal implementation.

### Problem

A text editor needs to restore earlier text when the user selects Undo.

### Python example

```python
class Editor:
    def __init__(self):
        self.text = ""

    def type_text(self, text):
        self.text += text

    def save(self):
        return self.text

    def restore(self, snapshot):
        self.text = snapshot


editor = Editor()

editor.type_text("Hello")
snapshot = editor.save()

editor.type_text(" World")
print(editor.text)  # Hello World

editor.restore(snapshot)
print(editor.text)  # Hello
```

This simplified example uses an immutable string as the snapshot. Larger applications can use dedicated memento objects and a history stack.

### When to use it

- Undo/redo systems
- Game save states
- Draft recovery
- Restoring previous application state

---

## 12. Visitor Pattern

### Intent

Add operations across a family of object types without placing every operation inside those classes.

### Problem

A shape model may need area calculation, exporting, and rendering. Adding all these operations to every shape class can make the classes harder to maintain.

### Structure

```text
Shape
  +-- Circle
  +-- Rectangle

Visitor
  +-- AreaVisitor
  +-- ExportVisitor
```

### Python example

```python
from abc import ABC, abstractmethod


class ShapeVisitor(ABC):
    @abstractmethod
    def visit_circle(self, circle):
        pass

    @abstractmethod
    def visit_rectangle(self, rectangle):
        pass


class Circle:
    def __init__(self, radius):
        self.radius = radius

    def accept(self, visitor):
        return visitor.visit_circle(self)


class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def accept(self, visitor):
        return visitor.visit_rectangle(self)


class AreaVisitor(ShapeVisitor):
    def visit_circle(self, circle):
        return 3.14159 * circle.radius ** 2

    def visit_rectangle(self, rectangle):
        return rectangle.width * rectangle.height


visitor = AreaVisitor()

print(Circle(2).accept(visitor))
print(Rectangle(3, 4).accept(visitor))
```

### When to use it

- Compilers and abstract syntax trees
- Operations over structured documents
- Exporting complex object models
- Reporting over stable object hierarchies

### Trade-off

Adding a new operation is relatively easy, but adding a new element type often requires updating every visitor.

---

## 13. Interpreter Pattern

### Intent

Represent a grammar and provide a way to evaluate expressions in that grammar.

### Example

A simple rule engine might evaluate expressions such as:

```text
age > 18 AND has_membership = true
```

An interpreter represents expressions as objects and evaluates them against data.

### Python example

```python
class Number:
    def __init__(self, value):
        self.value = value

    def interpret(self):
        return self.value


class Add:
    def __init__(self, left, right):
        self.left = left
        self.right = right

    def interpret(self):
        return self.left.interpret() + self.right.interpret()


expression = Add(Number(10), Number(20))
print(expression.interpret())  # 30
```

This is a tiny arithmetic expression tree, not a full parser or language implementation.

### When to use it

- Simple expression evaluators
- Domain-specific languages
- Rule engines
- Mathematical expression parsers

For complex grammars, a parser generator or dedicated parsing library is often more practical.

---

## 14. Quick Comparison for LLD Interviews

| Pattern | Main question it answers | Example |
|---|---|---|
| Strategy | Which algorithm should be used? | Payment method |
| Observer | Who needs to know that an event occurred? | Order notifications |
| State | How does behavior depend on current state? | Order lifecycle |
| Command | How can an action be represented as an object? | Undo/redo |
| Chain of Responsibility | Which handler should process this request? | HTTP middleware |
| Template Method | Which steps are fixed and which can vary? | Data import |
| Iterator | How do we traverse a collection? | Custom collection |
| Mediator | How should collaborating objects communicate? | Chat room |
| Memento | How can an earlier state be restored? | Editor history |
| Visitor | How can operations be added across object types? | Shape export |
| Interpreter | How can a grammar or expression be evaluated? | Arithmetic expression |

## 15. How to Choose a Pattern in an LLD Interview

1. **Identify the changing requirement.** What behavior, algorithm, or interaction varies?
2. **Identify the responsibilities.** Which classes are doing too much or depending on too many other classes?
3. **Find the design problem.** Is it object creation, communication, state transitions, or algorithm selection?
4. **Choose the simplest suitable pattern.** Do not introduce a pattern just to make the design look sophisticated.
5. **Explain the trade-offs.** Patterns can improve extensibility but also introduce extra classes and indirection.
6. **Test the design.** Check how easy it is to add a new behavior, handle invalid states, and write unit tests.

## 16. Recommended Learning Order

### Phase 1 — Learn first

- [ ] Strategy
- [ ] Observer
- [ ] State
- [ ] Command
- [ ] Chain of Responsibility
- [ ] Adapter
- [ ] Decorator

### Phase 2 — Learn next

- [ ] Factory Method
- [ ] Builder
- [ ] Template Method
- [ ] Facade
- [ ] Composite
- [ ] Proxy
- [ ] Mediator
- [ ] Iterator
- [ ] Memento

### Phase 3 — Broaden your knowledge

- [ ] Abstract Factory
- [ ] Prototype
- [ ] Singleton
- [ ] Bridge
- [ ] Flyweight
- [ ] Visitor
- [ ] Interpreter

## 17. Key Takeaways

- Behavioral patterns organize communication and responsibilities between objects.
- Strategy makes algorithms interchangeable.
- Observer notifies subscribers about events.
- State changes behavior based on internal state.
- Command encapsulates an action as an object.
- Chain of Responsibility passes requests through handlers.
- Template Method fixes a workflow while allowing selected steps to vary.
- Mediator centralizes communication between collaborating objects.
- Memento supports restoring previous state.
- Visitor adds operations over a family of object types.
- Iterator provides a standard way to traverse collections.
- Interpreter evaluates expressions defined by a grammar.

**Most important:** Understand the problem a pattern solves, recognize when that problem appears, and implement the pattern in a small project before using it in a larger LLD design.
