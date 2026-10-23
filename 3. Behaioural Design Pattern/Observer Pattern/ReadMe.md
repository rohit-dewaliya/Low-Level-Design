
# Observer Design Pattern — LLD Notes

## 1. What Is the Observer Pattern?

The **Observer Pattern** is a behavioral design pattern that establishes a one-to-many relationship between objects.

When one object changes its state or publishes an event, all registered observers are notified automatically.

It helps objects communicate without being tightly coupled to one another.

### Real-World Examples

- YouTube subscribers receiving notifications when a channel uploads a video.
- Users receiving notifications when an order status changes.
- Stock market applications updating subscribers when prices change.
- News applications notifying subscribers about new articles.
- Event listeners responding to button clicks in a GUI.
- Chat applications notifying users when a new message arrives.

### Why Use the Observer Pattern?

- Reduces tight coupling between components.
- Supports one-to-many communication.
- Allows observers to subscribe and unsubscribe dynamically.
- Makes it easier to add new notification receivers.
- Separates event publishing from event handling.

---

## 2. Main Components

The Observer Pattern typically consists of three main components.

### 1. Subject (Publisher)

The object that maintains a list of observers.

Responsibilities:
- Register observers.
- Remove observers.
- Notify registered observers when an event occurs.

### 2. Observer (Subscriber)

An object that receives notifications from the subject.

Responsibilities:
- Define a common notification interface.
- React when notified by the subject.

### 3. Concrete Observer

A specific implementation of the observer interface.

Examples:
- EmailNotifier
- SMSNotifier
- MobileAppNotifier

### Class Diagram

```text
             +----------------------+
             |        Subject       |
             +----------------------+
             | - observers          |
             +----------------------+
             | + subscribe()        |
             | + unsubscribe()      |
             | + notify()            |
             +----------------------+
                        |
                        | notifies
                        v
             +----------------------+
             |   Observer Interface |
             +----------------------+
             | + update(event)      |
             +----------------------+
                  ^       ^       ^
                  |       |       |
             Email     SMS      Mobile
             Observer  Observer Observer
```

---

## 3. How the Observer Pattern Works

Example: A YouTube notification system.

1. A user subscribes to a channel.
2. The channel stores the user's observer.
3. The channel uploads a new video.
4. The subject notifies all registered observers.
5. Each observer receives the notification.
6. A user can unsubscribe and stop receiving future notifications.

### Interaction Flow

```text
User subscribes
      |
      v
Channel.subscribe(user)
      |
      v
Channel uploads video
      |
      v
Channel.notify_observers()
      |
      +-------------------+
      |                   |
      v                   v
Observer A notified   Observer B notified
      |                   |
      v                   v
Receive update        Receive update
```

---

## 4. Basic Python Implementation

### Example: YouTube Notification System

```python
from abc import ABC, abstractmethod


class Observer(ABC):
    @abstractmethod
    def update(self, message):
        pass


class Subscriber(Observer):
    def __init__(self, name):
        self.name = name

    def update(self, message):
        print(f"{self.name} received: {message}")


class YouTubeChannel:
    def __init__(self, channel_name):
        self.channel_name = channel_name
        self._observers = []

    def subscribe(self, observer):
        if observer not in self._observers:
            self._observers.append(observer)

    def unsubscribe(self, observer):
        if observer in self._observers:
            self._observers.remove(observer)

    def notify_observers(self, message):
        for observer in self._observers.copy():
            observer.update(message)

    def upload_video(self, video_title):
        print(f"\n{self.channel_name} uploaded: {video_title}")

        self.notify_observers(
            f"New video uploaded: {video_title}"
        )


# Usage
channel = YouTubeChannel("Tech Academy")

subscriber1 = Subscriber("Alice")
subscriber2 = Subscriber("Bob")

channel.subscribe(subscriber1)
channel.subscribe(subscriber2)

channel.upload_video("Learn Python")

channel.unsubscribe(subscriber1)

channel.upload_video("Learn Design Patterns")
```

### Expected Output

```text
Tech Academy uploaded: Learn Python
Alice received: New video uploaded: Learn Python
Bob received: New video uploaded: Learn Python

Tech Academy uploaded: Learn Design Patterns
Bob received: New video uploaded: Learn Design Patterns
```

### Explanation

- `Observer` defines the notification contract.
- `Subscriber` implements the observer behavior.
- `YouTubeChannel` acts as the subject.
- `subscribe()` registers observers.
- `unsubscribe()` removes observers.
- `notify_observers()` sends notifications.
- `upload_video()` triggers the notification process.

The channel does not need to know how each subscriber handles the notification.

---

## 5. Observer Pattern with Multiple Notification Types

Suppose an e-commerce application needs to notify users through email, SMS, and mobile push notifications.

The subject should not depend directly on concrete notification implementations.

```python
from abc import ABC, abstractmethod


class Observer(ABC):
    @abstractmethod
    def update(self, event):
        pass


class EmailNotifier(Observer):
    def update(self, event):
        print(f"Email notification: {event}")


class SMSNotifier(Observer):
    def update(self, event):
        print(f"SMS notification: {event}")


class MobileNotifier(Observer):
    def update(self, event):
        print(f"Mobile notification: {event}")


class Order:
    def __init__(self, order_id):
        self.order_id = order_id
        self.status = "CREATED"
        self._observers = []

    def subscribe(self, observer):
        if observer not in self._observers:
            self._observers.append(observer)

    def unsubscribe(self, observer):
        if observer in self._observers:
            self._observers.remove(observer)

    def _notify(self):
        event = {
            "order_id": self.order_id,
            "status": self.status,
        }

        for observer in self._observers.copy():
            observer.update(event)

    def update_status(self, new_status):
        if new_status == self.status:
            return

        self.status = new_status
        self._notify()


# Usage
order = Order("ORD-101")

order.subscribe(EmailNotifier())
order.subscribe(SMSNotifier())
order.subscribe(MobileNotifier())

order.update_status("CONFIRMED")
order.update_status("SHIPPED")
```

### Benefits

- New notification types can be introduced independently.
- The order class does not need to know notification implementation details.
- Subscribers can be registered dynamically.
- The system is easier to extend and test.

**Design principle:** Program to an abstraction rather than a concrete implementation.

---

## 6. Push Model vs Pull Model

The Observer Pattern commonly uses two notification models.

### Push Model

The subject sends the event data directly to observers.

```python
observer.update({
    "order_id": "ORD-101",
    "status": "SHIPPED",
})
```

Advantages:
- Observers immediately receive the necessary information.
- Simple for small event-driven systems.

Disadvantages:
- The subject may send data an observer does not need.
- Event payloads can become large.

### Pull Model

The subject notifies observers that something changed. Observers retrieve the necessary information from the subject.

```python
class OrderObserver:
    def update(self, order):
        print(f"Current status: {order.status}")
```

Advantages:
- Observers choose which information to retrieve.
- The notification itself can remain lightweight.

Disadvantages:
- Observers become dependent on accessing the subject.
- They may retrieve the state after it has changed again.

### Comparison

| Push Model | Pull Model |
|---|---|
| Subject sends event data | Observer retrieves data |
| Often simpler for event notifications | Useful when observers need different information |
| Can send unnecessary information | Requires access to the subject or a query interface |

---

## 7. Observer Pattern and SOLID Principles

### Single Responsibility Principle (SRP)

The subject manages subscriptions and publishes events. Observers handle their own notification behavior.

### Open/Closed Principle (OCP)

New observer implementations can be added without modifying the subject's core notification logic.

### Liskov Substitution Principle (LSP)

Any observer implementing the expected contract should be usable wherever an observer is required.

### Interface Segregation Principle (ISP)

A focused observer interface should contain only the methods observers actually need.

### Dependency Inversion Principle (DIP)

The subject should depend on the observer abstraction instead of specific concrete observers.

---

## 8. Advantages

- Loose coupling between publishers and subscribers.
- Dynamic subscription and unsubscription.
- Easy extension with new observer types.
- Supports event-driven architecture.
- Encourages separation of concerns.
- Useful for broadcasting state changes to multiple components.

## 9. Disadvantages

- Notifications can become difficult to trace when many observers are registered.
- One slow observer can delay synchronous notifications.
- Circular dependencies may arise if observers trigger more events.
- Observer order can matter in some implementations.
- Poor subscription management can cause memory leaks.
- Exceptions from one observer can interrupt notification of others unless handled.

### Practical Improvements

For production systems, consider:

- Copying the observer collection before iterating.
- Preventing duplicate subscriptions.
- Providing explicit unsubscribe operations.
- Defining exception-handling behavior.
- Using asynchronous delivery for slow observers.
- Adding retry mechanisms when notifications must be reliable.
- Using weak references when appropriate to reduce retention of objects.

The basic pattern does not automatically provide asynchronous execution, guaranteed delivery, or thread safety.

---

## 10. Observer Pattern vs Mediator Pattern

| Observer Pattern | Mediator Pattern |
|---|---|
| Notifies multiple subscribers about events | Coordinates interactions between components |
| Usually one-to-many communication | Often many-to-many coordination through a central object |
| Components subscribe to notifications | Components communicate through the mediator |
| Example: order status notifications | Example: chat room coordinating users |

Use Observer when components need to react to published events.

Use Mediator when interactions between components are becoming complex and should be coordinated centrally.

---

## 11. Observer Pattern vs Pub/Sub

These concepts are related but not identical.

| Observer Pattern | Publish/Subscribe |
|---|---|
| Observers commonly register directly with a subject | Publishers and subscribers commonly communicate through a broker or event bus |
| Often used within one application process | Frequently used across services or processes |
| Usually synchronous in a basic implementation | Often supports asynchronous message delivery |
| Example: GUI event listeners | Example: a message broker distributing order events |

A publish/subscribe system may use the Observer Pattern's underlying idea, but adds infrastructure and delivery mechanisms.

---

## 12. Common Use Cases

- Notification systems
- Stock price updates
- GUI event listeners
- News subscriptions
- Order status updates
- Chat applications
- Monitoring dashboards
- Event-driven applications
- Cache invalidation
- Real-time data displays

---

## 13. Common Mistakes

- Forgetting to unsubscribe observers.
- Allowing duplicate subscriptions.
- Coupling the subject to concrete observer classes.
- Modifying the observer list unsafely while iterating.
- Assuming notifications are delivered asynchronously.
- Assuming every observer will receive an event successfully.
- Ignoring exceptions raised by observers.
- Using Observer when a simple direct method call is sufficient.

---

## 14. Interview Questions

### Q1. What is the Observer Pattern?

It is a behavioral design pattern that allows one subject to notify multiple registered observers when an event occurs or its state changes.

### Q2. What are the main components?

- Subject
- Observer interface
- Concrete observers

### Q3. Why is Observer useful?

It reduces coupling between the object publishing an event and the objects reacting to it.

### Q4. What happens when an observer unsubscribes?

It should stop receiving notifications from that subject after unsubscription takes effect.

### Q5. Is Observer synchronous or asynchronous?

The basic pattern can be either. A simple implementation usually calls observers synchronously; asynchronous behavior must be implemented separately.

### Q6. How does Observer support OCP?

New observer implementations can be added without changing the subject's notification mechanism.

### Q7. What is the difference between Observer and Strategy?

Observer broadcasts events to interested objects. Strategy selects an interchangeable algorithm or behavior for a particular task.

### Q8. How can Observer cause memory leaks?

If a long-lived subject retains references to observers that are no longer needed, those observer objects may remain in memory.

### Q9. Can one observer affect another observer?

Yes. In synchronous implementations, an exception or slow operation in one observer may affect subsequent notifications unless the subject handles it.

---

## 15. Practice Exercise

### Problem: Stock Market Notification System

Design a stock price monitoring system.

Requirements:

- A stock maintains its current price.
- Users can subscribe to stock updates.
- Users receive notifications when the price changes.
- Users can unsubscribe at any time.
- Multiple users can observe the same stock.
- Different observers can handle updates differently.

### Suggested Classes

- `Stock` — Subject
- `Observer` — Observer abstraction
- `MobileAppObserver` — Concrete observer
- `EmailObserver` — Concrete observer
- `StockMarket` — Optional coordinator

### Implementation Checklist

- [ ] Define the observer interface.
- [ ] Implement subscription.
- [ ] Implement unsubscription.
- [ ] Notify observers when the price changes.
- [ ] Avoid duplicate subscriptions.
- [ ] Test multiple observers.
- [ ] Test unsubscription.
- [ ] Test repeated price values.
- [ ] Consider what happens when one observer fails.

---

## 16. Key Takeaways

- Observer is a behavioral design pattern.
- It establishes a one-to-many relationship between a subject and its observers.
- The subject manages subscriptions and notifications.
- Observers react to events without requiring the subject to know their implementation details.
- It is useful for notifications, event listeners, and state-change propagation.
- Push and pull models define how observers receive event information.
- Reliable production systems may need explicit error handling, asynchronous delivery, and subscription lifecycle management.
