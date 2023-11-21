# Personal To-Do List Manager

## Classes 

### `Task` class
- Represents a task with a description, due date, and tags.
- Tags are optional and default to an empty list.
- Due date is optional and default to None.

### `TaskBuilder` class
- A builder class to help construct `Task` objects.
- This includes description of the task. 
- Allows setting the due date and adding tags before building the final task.

### `TaskMemento` class
- Represents the state of tasks at a specific point in time.
- Used to store the state of tasks for undo and redo operations.

### `TaskCaretaker` class
- Manages the history of task states.
- Supports undo and redo operations by restoring tasks from the history.

### `User` class
- Represents a user with a username and a list of tasks.
- Utilizes a `TaskCaretaker` to manage the history of task states.
- Provides methods to add, remove tasks, and display tasks.
- Can restore its state from a `TaskMemento`.

## Program Flow

1. **Task Creation and Modification:**
   - Users can create tasks by providing a description, due date, and tags.
   - Tasks are added to the user's task list, and each addition or removal triggers a state save to the history.

2. **Undo and Redo:**
   - The `TaskCaretaker` manages a history stack containing task states.
   - `undo` and `redo` methods allow users to move backward and forward through the task history.
   - The `TaskCaretaker` creates a new `TaskMemento` for each save to the history stack.

3. **Menu-Driven Interaction:**
   - Users interact with the program through a menu-driven interface.
   - Options include adding tasks, removing tasks, displaying tasks, undoing, redoing, and exiting the program.

4. **User Input Handling:**
   - User input is taken for each menu option, and corresponding actions are performed based on the input.

## Source Code
```python
# Represents a task
class Task:
    def __init__(self, description, due_date=None, tags=None):
        self.description = description
        self.due_date = due_date
        self.tags = tags if tags is not None else []

    def __repr__(self):
        return f"Task('{self.description}', due_date={self.due_date}, tags={self.tags})"

# Used to create tasks
class TaskBuilder:
    def __init__(self, description):
        self.task = Task(description)

    def set_due_date(self, due_date):
        self.task.due_date = due_date
        return self

    def add_tag(self, tag):
        self.task.tags.append(tag)
        return self

    def build(self):
        return self.task

# Class used to save the state of each task created by user
class TaskMemento:
    def __init__(self, tasks):
        self.tasks = tasks

    def get_state(self):
        return self.tasks.copy()

# Class used to maintain the history of tasks created 
class TaskCaretaker:
    def __init__(self, user):
        self.user = user
        self.history = []  # Single stack to store both undo and redo states
        self.index = -1    # Index to keep track of the current state in the history

    def save_to_history(self):
        # Save the current state to history
        self.history = self.history[:self.index + 1]
        self.history.append([task.__dict__ for task in self.user.tasks])
        self.index += 1

    def undo(self):
        if self.index > 0:
            self.index -= 1
            self.user.restore_from_memento(TaskMemento(self.history[self.index]))
 
    def redo(self):
        if self.index < len(self.history) - 1:
            self.index += 1
            self.user.restore_from_memento(TaskMemento(self.history[self.index]))

class User:
    def __init__(self, username):
        self.username = username
        self.tasks = []
        self.caretaker = TaskCaretaker(self)

    def add_task(self, task):
        # When adding a task, create a new state and push it onto the history stack
        self.tasks.append(task)
        self.caretaker.save_to_history()

    def remove_task(self, task):
        # When removing a task, create a new state and push it onto the history stack
        self.tasks.remove(task)
        self.caretaker.save_to_history()

    def restore_from_memento(self, memento):
        # Restore the state from the given Memento
        self.tasks = [Task(**task_data) for task_data in memento.get_state()]

    def display_tasks(self):
        print(f"Tasks for {self.username}:")
        for task in self.tasks:
            print(task)

# Function to display the menu
def display_menu():
    print("\nMENU:")
    print("1. Add Task")
    print("2. Remove Task")
    print("3. Display Tasks")
    print("4. Undo")
    print("5. Redo")
    print("6. Exit")

# Example usage with a menu-driven program
user = User("User")

while True:
    display_menu()
    choice = input("Enter your choice (1-6): ")

    if choice == '1':
        description = input("Enter task description: ")
        task_builder = TaskBuilder(description)
        due_date = input("Enter due date (optional, press Enter to skip): ")
        if due_date:
            task_builder.set_due_date(due_date)
        tags = input("Enter tags (optional, press Enter to skip): ")
        if tags:
            tags_list = [tag.strip() for tag in tags.split(',')]
            for tag in tags_list:
                task_builder.add_tag(tag)

        user.add_task(task_builder.build())
        print("Task added successfully.")

    elif choice == '2':
        user.display_tasks()
        index_to_remove = int(input("Enter the index of the task to remove: "))
        if 0 <= index_to_remove < len(user.tasks):
            user.remove_task(user.tasks[index_to_remove])
            print("Task removed successfully.")
        else:
            print("Invalid index.")

    elif choice == '3':
        user.display_tasks()

    elif choice == '4':
        user.caretaker.undo()
        print("Undo successful.")

    elif choice == '5':
        user.caretaker.redo()
        print("Redo successful.")

    elif choice == '6':
        print("Exiting the program.")
        break

    else:
        print("Invalid choice. Please enter a number between 1 and 6.")

