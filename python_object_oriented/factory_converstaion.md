# Key Insights from Factory Method Conversation
## Factory Method
### Obsercation Set 1

1. Factory -> creates object -> returns it -> caller manages it
2. Factory -> creates object -> keeps it internally -> exposes methods -> caller uses factory as the interface. 

#### Style 1 : Pure creation example

```python
from abs import ABC, abstractmethod

class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def on_click(self) -> str:
        pass

class WindowsButton(Button):
    def render(self) -> str:
        return "Rendering a Windows button"
    
    def on_click(self) -> str:
        return "Windows button clicked"

class MacButton(Button):
    def render(self) -> str:
        return "Rendering a Mac button"
    
    def on_click
        return "Windows button clicked"

# --- Pure creation factory ---
# Just create and return the object
# Caller is responisible for mamanging it. 

class ButtonFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: #Returns the object
        pass

class WindowsButtonFactory(ButtonFactory):
    def create_button(self) -> Button:
        return WindowsButton() # Just creates and returns 

class MacButtonFactory(ButtonFactory):
    def create_button(self) -> Button:
        return MacButton() # Just creates and returns

# --- Usage ---
# The CALLER manages object after creation
factory = WindowsButtonFactory()

button = factory.create_button() # Caller Takes ownership
print(button.render()) # Caller calls method directly
print(button.on_click()) # Caller is in control 
```

#### Style 2 : Factory as a manager example

```python
from abc import ABC, abstractmethod

class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def on_click(self) -> str:
        pass

class WindowsButton(Button):
    def render(self) -> str:
        return "Rendering a Windows button"
    
    def on_click(self) -> str:
        return "Windows button clicked"

class MacButton(Button):
    def render(self) -> str:
        return "Rendering a Mac button"
    
    def on_click
        return "Windows button clicked"

# --- Factory as a Manager ---
# Creates the object and manages it
# Caller never sees the created object directly
class ButtonManager(ABC):
    def __init__(self):
        # Factory creates and holds the object internally
        self._button() = self.create_button()
    
    @abstractmethod
    def createbutton(self) -> Button:
        pass

    #Manager exposes its own methods that deligates to the object. 
    def render_button(self) -> str:
        return self._button.render() # Delegated to internal object

# --- Usage ---
# The caller ONLY interacts with the manager
# The button object is hidden inside
manager = WindowsButtonManager()

print(manager.render_button()) # Caller never touches Button directly
print(manager.click_button()) # Manager handels everything
```


**TODO:** add the document example


### Observation set 2

#### Observation 1 - "Factory" is a Misleading Name

The following obsercations expand on style two. 

- In real life a factory makes a car and you dirve it. In style 2 a factory makes the car and drives it. So the term "Factory" is mis-leading. 
- In style two, its more accurately a 
    - Manager
    - Service
    - Handler
    - Coordinator 
    - Wrapper

#### Obsercation 2 - A factory Class Per Concrete Class is Wasteful.

Here is some absurd code
```python
# This is genuinely questionable code
class Dog:
    def speak(self):
        return "Woof"
    
# Why does this need to exist?
class DogFactory:
    def create(self):
        return Dog() # This adds Zero Value

# When you could just do:
dog = Dog() # Simpler, clearer, identical result

```

But there are some cases where factory earns its place by hiding complex creation logic.

```python

class DatabaseConnection:
    def __init__(
        self,
        host: str,
        port: int,
        username: str,
        password: str,
        pool_size: int,
        timeout: int,
        ssl_enabled: bool
    ):
        self.host = host
        self.port = port
        self.username = username
        self.password = password
        self.pool_size = poolsize
        self.timeout = timeout
        self.ssl_enabled = ssl_enabled

# Now a factory earns its place - it hides complex creation logic 
class DatabaseConnectionFactory
    @staticmethod
    def create_development() -> DatabaseConnection:
        return DatabaseConnection(
            host="localhost",
            port=5432,
            username="dev_user",
            password="dev_pass",
            pool_size=2,
            timeout=30,
            ssl_enabled=False
        )
    
    @staticmethod
    def create_production() -> DatabaseConnection:
        return DatabaseConnection(
            host="prod.myserver.com",
            port=5432,
            username="prod_user",
            password="ultra_secure_pass",
            pool_size=20,
            timeout=10,
            ssl_enabled=True
        )

    @staticmethod
    def create_testing() -> DatabaseConnection:
        return DatabaseConnection(
            host="localhost",
            port=5432,
            username="test_user",
            password="test_pass",
            pool_size=1,
            timeout=5,
            ssl_enabled=False
        )

# --- Usage ---
# The caller doesn't need to know ANY of those details
import os
env = os.getenv("APP_ENV", "development")

factory = DatabaseConnectionFactory()

if env == "production":
    conn = factory.create_production()
elif env == "testing":
    conn = factory.create_testing()
else
    conn = factory.create_development()

print(conn.query("SELECT * FROM users"))
```

Here the factory earns its place because

- It hides complex configuration
- It centralises environment specific logic 
- Caller just says. "give me a production connection" withoug knowing the details 

More accurate then textbook defination fo Style 2


> " A way of extending the functionality of a set of classes sharing the same interface, and managing the instance of the object - not an object returning object" 

**TODO:** Understand why it describes Service Layer or Wrapper pattern and type the example


## Abstract Factory
Its a single factory that can create multiple related objects that belong together. 

The objects created by one factory are designed to work together. 

```python
from abc import ABC, abstractmethod

# ABSTRACT PRODUCTS

"""
Abstract Factory Design Pattern - Button Example

Goal: create UI elements (buttons and checkboxes) that match a theme
(Windows or Mac) WITHOUT the client code knowing which concrete classes
it is using.
"""

from abc import ABC, abstractmethod


# ---------------------------------------------------------------
# 1. Abstract Products: the interfaces every product must follow
# ---------------------------------------------------------------
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass

    @abstractmethod
    def on_click(self) -> str:
        pass


class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str:
        pass


# ---------------------------------------------------------------
# 2. Concrete Products: one family per operating system
# ---------------------------------------------------------------
class WindowsButton(Button):
    def render(self) -> str:
        return "[ Windows Button ]  (square corners, blue)"

    def on_click(self) -> str:
        return "Windows button clicked -> plays Windows 'ding'"


class MacButton(Button):
    def render(self) -> str:
        return "( Mac Button )  (rounded corners, gray)"

    def on_click(self) -> str:
        return "Mac button clicked -> plays Mac 'pop'"


class WindowsCheckbox(Checkbox):
    def render(self) -> str:
        return "[x] Windows Checkbox"


class MacCheckbox(Checkbox):
    def render(self) -> str:
        return "(✓) Mac Checkbox"


# ---------------------------------------------------------------
# 3. Abstract Factory: declares how to create each product
# ---------------------------------------------------------------
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass

    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass


# ---------------------------------------------------------------
# 4. Concrete Factories: each creates a matching family of products
# ---------------------------------------------------------------
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()

    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()


class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()

    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()


# ---------------------------------------------------------------
# 5. Client: works ONLY with the abstract interfaces
# ---------------------------------------------------------------
class Application:
    def __init__(self, factory: GUIFactory):
        # The app never says "WindowsButton" or "MacButton".
        self.button = factory.create_button()
        self.checkbox = factory.create_checkbox()

    def paint(self) -> None:
        print(self.button.render())
        print(self.checkbox.render())
        print(self.button.on_click())


# ---------------------------------------------------------------
# 6. Configuration: the ONLY place a concrete factory is chosen
# ---------------------------------------------------------------
def get_factory(os_name: str) -> GUIFactory:
    factories = {"windows": WindowsFactory, "mac": MacFactory}
    try:
        return factories[os_name.lower()]()
    except KeyError:
        raise ValueError(f"Unsupported OS: {os_name}")


if __name__ == "__main__":
    for os_name in ("Windows", "Mac"):
        print(f"--- Running on {os_name} ---")
        app = Application(get_factory(os_name))
        app.paint()
        print()
```

### Insight

The factory should not just be a grouping mechanism - it should also be the place that understand how its own components interact with each other. 

```text
What we had before:
    Factory -> creates compatible objects -> hands them out -> done

Insight
    Factory -> creates compatible objects -> ALSO ddefines how they interact with each other -> manages their relationships 
```

```python
"""
Abstract Factory Design Pattern - Extended Button Example

Before:
    Factory -> creates compatible objects -> hands them out -> done

Now:
    Factory -> creates compatible objects
            -> ALSO defines how they interact with each other
            -> manages their relationships

Scenario: a "Terms & Conditions" form with a checkbox ("I agree") and a
button ("Submit"). Each theme creates its own widgets AND decides how
they behave together:

    Windows: unchecked checkbox -> button is DISABLED (greyed out)
             clicking Submit     -> checkbox is RESET (agree again next time)

    Mac:     unchecked checkbox -> button is HIDDEN
             clicking Submit     -> checkbox is LOCKED (cannot be unticked)
"""

from abc import ABC, abstractmethod
from typing import Callable, List


# ---------------------------------------------------------------
# 1. Abstract Products
#    They expose "hooks" (events + state setters) but have NO idea
#    who they are connected to. Relationships come from the factory.
# ---------------------------------------------------------------
class Checkbox(ABC):
    def __init__(self, label: str):
        self.label = label
        self.checked = False
        self.locked = False
        self._listeners: List[Callable[[bool], None]] = []

    def on_change(self, listener: Callable[[bool], None]) -> None:
        self._listeners.append(listener)

    def toggle(self) -> None:
        if self.locked:
            print(f"  {self.label}: locked, cannot change")
            return
        self.set_checked(not self.checked)

    def set_checked(self, value: bool) -> None:
        self.checked = value
        for listener in self._listeners:
            listener(value)

    @abstractmethod
    def render(self) -> str:
        pass


class Button(ABC):
    def __init__(self, label: str):
        self.label = label
        self.enabled = True
        self.visible = True
        self._listeners: List[Callable[[], None]] = []

    def on_click(self, listener: Callable[[], None]) -> None:
        self._listeners.append(listener)

    def click(self) -> None:
        if not self.visible:
            print(f"  {self.label}: not visible, nothing to click")
            return
        if not self.enabled:
            print(f"  {self.label}: disabled, click ignored")
            return
        print(f"  {self.label}: clicked!")
        for listener in self._listeners:
            listener()

    @abstractmethod
    def render(self) -> str:
        pass


# ---------------------------------------------------------------
# 2. Concrete Products (appearance only)
# ---------------------------------------------------------------
class WindowsCheckbox(Checkbox):
    def render(self) -> str:
        return f"[{'x' if self.checked else ' '}] {self.label}"


class MacCheckbox(Checkbox):
    def render(self) -> str:
        mark = "✓" if self.checked else " "
        lock = " 🔒" if self.locked else ""
        return f"({mark}) {self.label}{lock}"


class WindowsButton(Button):
    def render(self) -> str:
        state = "" if self.enabled else "  (greyed out)"
        return f"[ {self.label} ]{state}"


class MacButton(Button):
    def render(self) -> str:
        return f"( {self.label} )" if self.visible else "(button hidden)"


# ---------------------------------------------------------------
# 3. The composed result the client receives
# ---------------------------------------------------------------
class Form:
    def __init__(self, checkbox: Checkbox, button: Button):
        self.checkbox = checkbox
        self.button = button

    def render(self) -> None:
        print("    " + self.checkbox.render())
        print("    " + self.button.render())


# ---------------------------------------------------------------
# 4. Abstract Factory
#    Creates the products AND owns the rules for how they relate.
# ---------------------------------------------------------------
class GUIFactory(ABC):
    @abstractmethod
    def create_checkbox(self, label: str) -> Checkbox:
        pass

    @abstractmethod
    def create_button(self, label: str) -> Button:
        pass

    @abstractmethod
    def connect(self, checkbox: Checkbox, button: Button) -> None:
        """Define how this family's widgets interact."""
        pass

    def create_form(self, checkbox_label: str, button_label: str) -> Form:
        """Template method: create -> wire -> hand out a ready-made form."""
        checkbox = self.create_checkbox(checkbox_label)
        button = self.create_button(button_label)
        self.connect(checkbox, button)
        return Form(checkbox, button)


# ---------------------------------------------------------------
# 5. Concrete Factories: same products, DIFFERENT relationships
# ---------------------------------------------------------------
class WindowsFactory(GUIFactory):
    def create_checkbox(self, label: str) -> Checkbox:
        return WindowsCheckbox(label)

    def create_button(self, label: str) -> Button:
        return WindowsButton(label)

    def connect(self, checkbox: Checkbox, button: Button) -> None:
        # Rule 1: checkbox state enables/disables the button
        button.enabled = checkbox.checked
        checkbox.on_change(lambda checked: setattr(button, "enabled", checked))

        # Rule 2: submitting resets the checkbox
        button.on_click(lambda: checkbox.set_checked(False))


class MacFactory(GUIFactory):
    def create_checkbox(self, label: str) -> Checkbox:
        return MacCheckbox(label)

    def create_button(self, label: str) -> Button:
        return MacButton(label)

    def connect(self, checkbox: Checkbox, button: Button) -> None:
        # Rule 1: checkbox state shows/hides the button
        button.visible = checkbox.checked
        checkbox.on_change(lambda checked: setattr(button, "visible", checked))

        # Rule 2: submitting locks the checkbox
        button.on_click(lambda: setattr(checkbox, "locked", True))


# ---------------------------------------------------------------
# 6. Client: never wires anything itself, just uses the form
# ---------------------------------------------------------------
def run_scenario(factory: GUIFactory) -> None:
    form = factory.create_form("I agree to the terms", "Submit")

    print("  Initial state:")
    form.render()

    print("  User clicks Submit without agreeing:")
    form.button.click()

    print("  User ticks the checkbox:")
    form.checkbox.toggle()
    form.render()

    print("  User clicks Submit:")
    form.button.click()
    form.render()

    print("  User tries to toggle the checkbox again:")
    form.checkbox.toggle()
    form.render()


def get_factory(os_name: str) -> GUIFactory:
    factories = {"windows": WindowsFactory, "mac": MacFactory}
    try:
        return factories[os_name.lower()]()
    except KeyError:
        raise ValueError(f"Unsupported OS: {os_name}")


if __name__ == "__main__":
    for os_name in ("Windows", "Mac"):
        print(f"=== {os_name} ===")
        run_scenario(get_factory(os_name))
        print()
```

## Key findings after studying Factory Method and Abstract Factory:

**Factory Method:**

- A way to extend the functionality of interfaces of concrete classes.
  The factory wraps a class (or family of related behaviour) and lets you:
  - Add concrete methods to the abstract factory -> all factories get the behaviour
  - Add abstract methods to the abstract factory -> each concrete factory implements it differently

**Abstract Factory**

- Extends functionality of classes when objects might work together
  The key addition of Abstract factory over Factory Method is:
  - It manages multiple related objects instead of one
  - It is the natural home for interaction logic between those objects
  - Adding a concrete method to the abstract factory extends all families.
  - Adding a concrete method to a concrete factory is family-specific behaviour.

### Sharpening the above.

**Factory Method:**

- Concerned with ONE product and Extending its behaviour
- the factory IS the thing you work with.
- One dimension of varioation (which implemention)

**Abstract Factory:**

- Concerned with MULTIPLE products that belong together
- The factory COORDINATES the things you work with
- Two dimensions of variation:
  1. Which family (Windows vs Mac)
  2. Which product within the family (Button vs Label vs Input)
