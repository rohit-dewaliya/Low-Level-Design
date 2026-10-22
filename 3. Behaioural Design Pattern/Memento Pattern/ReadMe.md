
# Memento Design Pattern — LLD Notes

## 1. What Is the Memento Pattern?

The **Memento Pattern** is a behavioral design pattern that allows an object to save its internal state and restore it later without exposing its internal implementation details.

It is useful when we need to implement:

- Undo and redo functionality
- Save and restore functionality
- Checkpoints in applications
- Game progress saving
- Editor history
- Transaction rollback in in-memory workflows

### Real-World Examples

- A text editor restoring the previous document content.
- A game restoring a player's health, score, and position.
- A drawing application undoing the last shape modification.
- A form restoring previously saved values.

---

## 2. Main Components

The Memento Pattern typically has three components.

### 1. Originator

The object whose state needs to be saved and restored.

Responsibilities:
- Maintains the current state.
- Creates a memento containing its state.
- Restores its state from a memento.

### 2. Memento

Stores a snapshot of the originator's state.

Responsibilities:
- Holds the saved state.
- Keeps the originator's internal representation encapsulated.
- Prevents external objects from modifying the saved state.

### 3. Caretaker

Manages the saved mementos.

Responsibilities:
- Stores snapshots.
- Selects a snapshot to restore.
- Maintains history when implementing undo functionality.

The caretaker should not need to understand the internal details of the saved state.

---

## 3. How the Memento Pattern Works

Example: A text editor.

1. The user writes some text.
2. The editor creates a snapshot of its current state.
3. The user modifies the text.
4. The editor creates another snapshot.
5. The user selects Undo.
6. The caretaker retrieves the previous snapshot.
7. The editor restores the saved state.

### Class Relationship

```text
+---------------------+
|      Caretaker      |
+---------------------+
| - history           |
+---------------------+
| + save()            |
| + undo()            |
+---------------------+
          |
          | stores
          v
+---------------------+
|      Memento        |
+---------------------+
| - saved_state       |
+---------------------+
          ^
          | creates / restores
          |
+---------------------+
|      Originator     |
+---------------------+
| - current_state     |
+---------------------+
| + save_state()      |
| + restore_state()   |
+---------------------+
```

---

## 4. Basic Python Implementation

### Example: Text Editor with Undo

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Memento:
    content: str


class TextEditor:
    def __init__(self):
        self.content = ""

    def write(self, text):
        self.content += text

    def save(self):
        return Memento(self.content)

    def restore(self, memento):
        self.content = memento.content

    def show(self):
        print(self.content)


class History:
    def __init__(self):
        self._snapshots = []

    def save(self, editor):
        self._snapshots.append(editor.save())

    def undo(self, editor):
        if not self._snapshots:
            print("Nothing to undo")
            return

        memento = self._snapshots.pop()
        editor.restore(memento)


# Usage
editor = TextEditor()
history = History()

editor.write("Hello")
history.save(editor)

editor.write(" World")
history.save(editor)

editor.write("!")
editor.show()
# Hello World!

history.undo(editor)
editor.show()
# Hello World

history.undo(editor)
editor.show()
# Hello
```

### Explanation

- `TextEditor` is the Originator.
- `Memento` stores the document content.
- `History` is the Caretaker.
- `save()` creates a snapshot.
- `restore()` restores the saved content.
- `undo()` retrieves a previous snapshot.

**Note:** In this simple implementation, Undo restores the last saved snapshot. A production editor may use a different history convention, such as storing the state before every edit.

---

## 5. Improved Implementation with Undo and Redo

A practical editor often needs both Undo and Redo.

- Undo restores the previous state.
- Redo reapplies a state that was undone.
- A new edit after Undo clears the redo history.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class EditorMemento:
    content: str


class TextEditor:
    def __init__(self):
        self._content = ""

    def write(self, text):
        self._content += text

    def set_content(self, content):
        self._content = content

    def get_content(self):
        return self._content

    def save(self):
        return EditorMemento(self._content)

    def restore(self, memento):
        self._content = memento.content


class EditorHistory:
    def __init__(self):
        self._undo_stack = []
        self._redo_stack = []

    def save(self, editor):
        self._undo_stack.append(editor.save())
        self._redo_stack.clear()

    def undo(self, editor):
        if not self._undo_stack:
            return

        self._redo_stack.append(editor.save())
        previous_state = self._undo_stack.pop()
        editor.restore(previous_state)

    def redo(self, editor):
        if not self._redo_stack:
            return

        self._undo_stack.append(editor.save())
        next_state = self._redo_stack.pop()
        editor.restore(next_state)


# Usage
editor = TextEditor()
history = EditorHistory()

editor.write("Hello")
history.save(editor)

editor.write(" World")
history.save(editor)

editor.write("!")
print(editor.get_content())
# Hello World!

history.undo(editor)
print(editor.get_content())
# Hello World

history.redo(editor)
print(editor.get_content())
# Hello World!
```

### Important Design Detail

This version demonstrates the basic idea, but its stack convention can be confusing if saving happens only at selected times. A real editor should consistently define when snapshots are recorded.

One common approach is to store the current state on the undo stack before every edit, and move the current state to the opposite stack during Undo or Redo.

---

## 6. Correct Undo/Redo Implementation

The following implementation uses a consistent history convention.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Memento:
    content: str


class TextEditor:
    def __init__(self):
        self._content = ""

    def get_content(self):
        return self._content

    def write(self, text):
        self._content += text

    def restore(self, memento):
        self._content = memento.content

    def save(self):
        return Memento(self._content)


class History:
    def __init__(self):
        self._undo_stack = []
        self._redo_stack = []

    def execute_edit(self, editor, text):
        self._undo_stack.append(editor.save())
        self._redo_stack.clear()
        editor.write(text)

    def undo(self, editor):
        if not self._undo_stack:
            return

        self._redo_stack.append(editor.save())
        editor.restore(self._undo_stack.pop())

    def redo(self, editor):
        if not self._redo_stack:
            return

        self._undo_stack.append(editor.save())
        editor.restore(self._redo_stack.pop())


editor = TextEditor()
history = History()

history.execute_edit(editor, "Hello")
history.execute_edit(editor, " World")
history.execute_edit(editor, "!")

print(editor.get_content())
# Hello World!

history.undo(editor)
print(editor.get_content())
# Hello World

history.undo(editor)
print(editor.get_content())
# Hello

history.redo(editor)
print(editor.get_content())
# Hello World
```

### Why This Version Is Better

- Every edit saves the state that existed before the edit.
- Undo moves the current state to the redo stack.
- Redo moves the current state to the undo stack.
- A new edit clears redo history.
- Empty stacks are handled safely.

---

## 7. Memento Pattern in a Game

Imagine a game where the player has health, score, and a position.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class GameMemento:
    health: int
    score: int
    position: tuple


class Game:
    def __init__(self):
        self.health = 100
        self.score = 0
        self.position = (0, 0)

    def save(self):
        return GameMemento(
            health=self.health,
            score=self.score,
            position=self.position,
        )

    def restore(self, memento):
        self.health = memento.health
        self.score = memento.score
        self.position = memento.position


game = Game()

checkpoint = game.save()

game.health = 20
game.score = 500
game.position = (10, 15)

game.restore(checkpoint)

print(game.health)    # 100
print(game.score)     # 0
print(game.position)  # (0, 0)
```

The game can restore the saved state without exposing all its internal state-management logic to the caretaker.

---

## 8. Advantages

- Supports Undo and Redo.
- Preserves encapsulation.
- Separates state management from the originator's main logic.
- Allows checkpoints and restoration.
- Makes state recovery straightforward.

## 9. Disadvantages

- Many snapshots can consume substantial memory.
- Saving large objects can be expensive.
- State consistency can become complicated.
- Deeply nested mutable objects require careful snapshot handling.
- Managing history limits and expiration adds complexity.

### Performance Improvements

For large documents or game states, consider:

- Limiting the number of stored snapshots.
- Using incremental snapshots or command-based history.
- Sharing immutable data between snapshots.
- Using copy-on-write techniques.
- Compressing snapshots when appropriate.

A frozen dataclass does not automatically make nested mutable objects immutable. Use immutable data or defensive copies when necessary.

---

## 10. Memento vs Command Pattern

| Memento Pattern | Command Pattern |
|---|---|
| Stores a snapshot of state | Encapsulates an operation |
| Undo restores an earlier state | Undo often applies a reverse operation |
| Useful for complex state restoration | Useful for reversible actions and action queues |
| May use more memory for snapshots | Can be memory-efficient for small operations |
| Example: restore document contents | Example: undo inserting a character |

The patterns can also be combined. A Command can use a Memento to restore state.

---

## 11. Memento vs Database Transactions

| Memento | Database Transaction |
|---|---|
| Usually restores in-memory object state | Manages changes to persistent database data |
| Common in Undo/Redo and checkpoints | Common in atomic data operations |
| Implemented by the application | Typically provided by the database system |
| Does not automatically provide concurrency control | Transactions can provide isolation and consistency guarantees |

Memento is not a substitute for database transactions or durable recovery mechanisms.

---

## 12. Common Use Cases

- Text editors
- Drawing applications
- Games and checkpoints
- Form editing
- Configuration editors
- Workflow state restoration
- In-memory simulation rollback

---

## 13. Common Mistakes

- Exposing internal mutable state directly.
- Saving references instead of snapshots.
- Forgetting to clear redo history after a new edit.
- Mixing up the order of Undo and Redo stack operations.
- Storing too many large snapshots.
- Assuming a shallow copy safely captures every nested object.
- Using Memento when a simple reversible command would be more efficient.

---

## 14. Interview Questions

### Q1. What problem does Memento solve?

It allows an object's previous state to be saved and restored without exposing its internal implementation.

### Q2. What are its three main components?

Originator, Memento, and Caretaker.

### Q3. How does it preserve encapsulation?

The originator creates and restores its snapshots, while the caretaker stores them without needing to understand the internal state representation.

### Q4. What is the difference between Undo and Redo?

Undo restores a previous state. Redo restores a state that was undone.

### Q5. What are the main drawbacks?

Memory consumption, snapshot creation cost, and the complexity of correctly restoring mutable state.

### Q6. When should you prefer Command over Memento?

Use Command when actions can be represented and reversed efficiently. Use Memento when restoring a snapshot of complex state is simpler or safer.

### Q7. Which design pattern category contains Memento?

Memento is a **behavioral design pattern**.

---

## 15. Key Takeaways

- Memento captures and restores an object's state.
- Originator creates and restores snapshots.
- Memento stores the snapshot.
- Caretaker manages snapshot history.
- It is widely used for Undo/Redo, checkpoints, and state recovery.
- Snapshot copying and memory usage require careful consideration.
- Correct history-stack management is essential for reliable Undo/Redo behavior.
