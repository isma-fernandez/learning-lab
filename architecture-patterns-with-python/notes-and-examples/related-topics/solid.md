# SOLID
SOLID is a set of five object-oriented design principles that help developers create maintainable, flexible, and scalable software on well-designed, cleanly structured classes. 

The principles are:
1. [**S**ingle Responsibility Principle (SRP)](#single-responsibility-principle-srp): _A class should have only one reason to change_, meaning it should have only one job or responsibility.
2. [**O**pen-Closed Principle (OCP)](#open-closed-principle-ocp): _Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification_.
3. [**L**iskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp): _Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program_.
4. [**I**nterface Segregation Principle (ISP)](#interface-segregation-principle-isp): _Clients should not be forced to depend on interfaces they do not use_. This means that larger interfaces should be split into smaller, more specific ones.
5. [**D**ependency Inversion Principle (DIP)](#dependency-inversion-principle-dip): _High-level modules should not depend on low-level modules_. Both should depend on abstractions. Additionally, abstractions should not depend on details. Details should depend on abstractions.

## Single Responsibility Principle (SRP)
The Single Responsibility Principle (SRP) introduced by [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) in 2003 states that:

> A class should have only one reason to change.

This means that a class should have only one job or responsability.

To illustrate the single-responsability principle, consider the following example of a class that violates SRP:

```python
from pathlib import Path
from zipfile import ZipFile

class FileManager:
    # This class has two responsibilities: handling file I/O and handling ZIP compression/decompression.
    def __init__(self, file_path: Path):
        self.file_path = file_path

    def read_file(self) -> str:
        with open(self.file_path, 'r') as file:
            return file.read()

    def write_file(self, content: str) -> None:
        with open(self.file_path, 'w') as file:
            file.write(content)

    def compress_file(self, zip_path: Path) -> None:
        with ZipFile(zip_path, 'w') as zip_file:
            zip_file.write(self.file_path)

    def decompress_file(self, zip_path: Path) -> None:
        with ZipFile(zip_path, 'r') as zip_file:
            zip_file.extractall(self.file_path.parent)
```

In this example, the `FileManager` class has two different responsibilities: reading and writing files, as well as compressing and decompressing files.

This class violates the single-responsibility principle because there is more than one reason for changing it (file I/O and ZIP handling).

To adhere to the single-responsibility principle, we can refactor the code into two separate classes:

```python
from pathlib import Path
from zipfile import ZipFile

class FileHandler:
    # This class is responsible for handling file I/O operations.
    def __init__(self, file_path: Path):
        self.file_path = file_path

    def read_file(self) -> str:
        with open(self.file_path, 'r') as file:
            return file.read()

    def write_file(self, content: str) -> None:
        with open(self.file_path, 'w') as file:
            file.write(content)

class ZipHandler:
    # This class is responsible for handling ZIP compression and decompression.
    def compress_file(self, file_path: Path, zip_path: Path) -> None:
        with ZipFile(zip_path, 'w') as zip_file:
            zip_file.write(file_path)

    def decompress_file(self, zip_path: Path, extract_path: Path) -> None:
        with ZipFile(zip_path, 'r') as zip_file:
            zip_file.extractall(extract_path)
```

In this refactored version, we have two separate classes: `FileHandler` for handling file I/O operations and `ZipHandler` for handling ZIP compression and decompression. Each class has a single responsibility, adhering to the single-responsibility principle.

## Open-Closed Principle (OCP)

The Open-Closed Principle (OCP) introduced by [Bertrand Meyer](https://en.wikipedia.org/wiki/Bertrand_Meyer) in 1988 states that:
> Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. 

This means that you should be able to add new functionality without changing existing code.

Consider the following example of a class that violates OCP:

```python
class Shape:
    def __init__(self, shape_type, **kwargs):
        self.shape_type = shape_type
        if self.shape_type == "rectangle":
            self.width = kwargs["width"]
            self.height = kwargs["height"]
        elif self.shape_type == "circle":
            self.radius = kwargs["radius"]
        else:
            raise TypeError("Unsupported shape type")

    def calculate_area(self):
        if self.shape_type == "rectangle":
            return self.width * self.height
        elif self.shape_type == "circle":
            return pi * self.radius**2
        else:
            raise TypeError("Unsupported shape type")
```

In this example, the `Shape` class has a method `calculate_area` that calculates the area of different shapes based on the `shape_type`. If we want to add a new shape type (e.g., triangle), we would need to modify the existing code, which violates the Open-Closed Principle.

> **NOTE:** OCP doesn't mean your classes should never be modified. It means that you should minimize the need to modify existing code when adding new functionality.

To adhere to the Open-Closed Principle, we can refactor the code using inheritance and polymorphism:

```python
from abc import ABC, abstractmethod
from math import pi

class Shape(ABC):
    def __init__(self, shape_type):
        self.shape_type = shape_type

    @abstractmethod
    def calculate_area(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        super().__init__("circle")
        self.radius = radius

    def calculate_area(self):
        return pi * self.radius**2

class Rectangle(Shape):
    def __init__(self, width, height):
        super().__init__("rectangle")
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        super().__init__("square")
        self.side = side

    def calculate_area(self):
        return self.side**2
```

In this refactored version, we have an abstract base class `Shape` with an abstract method `calculate_area`. Each specific shape (Circle, Rectangle, Square) is implemented as a subclass of `Shape` and provides its own implementation of the `calculate_area` method. This way, we can add new shapes without modifying existing code, adhering to the Open-Closed Principle.

## Liskov Substitution Principle (LSP)
The Liskov Substitution Principle (LSP) introduced by [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov) at an OOPSLA conference in 1987 states that:
> Subtypes must be substitutable for their base types.

If you have a piece of code that works with a `Shape` class, then you should be able to replace it with any subclass of `Shape` (e.g., `Circle`, `Rectangle`, `Square`) without breaking the code.

In practice, this principle is about making your subclasses behave like their base classes when they call the same methods.

This one is pretty much self-explanatory. If you have a function that takes a `Shape` as an argument, it should work with any subclass of `Shape` without needing to know the specific type of shape. Every subclass should have the same methods and properties as the base class, and they should behave in a way that is consistent with the expectations set by the base class.


## Interface Segregation Principle (ISP)
The Interface Segregation Principle (ISP) introduced by [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) in 2003 states that:
> Clients should not be forced to depend on interfaces they do not use. Interfaces should belong to clients, not to hierarchies.

In this definition, clients are classes and subclasses, and interfaces consist of methods and attributes. In other words, if a class doesn't use particular methods or attributes, then those methods and attributes should not be part of the class's interface.

Consider the following example of a class that violates ISP:

```python
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

    @abstractmethod
    def fax(self, document):
        pass

    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

    def fax(self, document):
        raise NotImplementedError("Fax functionality not supported")

    def scan(self, document):
        raise NotImplementedError("Scan functionality not supported")

class ModernPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```

This one is evident, the `OldPrinter` class is forced to implement the `fax` and `scan` methods, even though it doesn't support those functionalities. This violates the Interface Segregation Principle because clients (in this case, the `OldPrinter` class) are forced to depend on methods they do not use.

To adhere to the Interface Segregation Principle, we can refactor the code by splitting the `Printer` interface into smaller, more specific interfaces:

```python
from abc import ABC, abstractmethod
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

class Fax(ABC):
    @abstractmethod
    def fax(self, document):
        pass

class Scanner(ABC):
    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

class NewPrinter(Printer, Fax, Scanner):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```

In Python, you'll rarely define many abstract base classes like this. You may instead rely on duck typing or mixin classes.

## Dependency Inversion Principle (DIP)
The Dependency Inversion Principle (DIP) introduced by [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) in 2003 states that:
> Abstractions should not depend on details. Details should depend on abstractions. High-level modules should not depend on low-level modules. Both should depend on abstractions.

This is already explained in the [introduction notes](../introduction/notes.md#dependency-inversion-principle-dip) DIP section.

Nonetheless, the main idea is that you should depend on abstractions (e.g., interfaces or abstract classes) rather than concrete implementations. This allows for more flexible and maintainable code, as you can easily swap out implementations without affecting the high-level logic of your application.

To illustrate the Dependency Inversion Principle, consider the following example of a class that violates DIP:

```python
class MySQLDatabase:
    def connect(self):
        print("Connecting to MySQL database...")

class UserRepository:
    def __init__(self):
        self.database = MySQLDatabase()  # This creates a tight coupling between UserRepository and MySQLDatabase

    def get_user(self, user_id):
        self.database.connect()
        print(f"Getting user with ID {user_id} from MySQL database...")
```

In this example, the `UserRepository` class is tightly coupled to the `MySQLDatabase` class. If we want to switch to a different database (e.g., PostgreSQL), we would need to modify the `UserRepository` class, which violates the Dependency Inversion Principle.

To adhere to DIP, we can introduce an abstraction for the database:

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def connect(self):
        pass

class MySQLDatabase(Database):
    def connect(self):
        print("Connecting to MySQL database...")

class PostgreSQLDatabase(Database):
    def connect(self):
        print("Connecting to PostgreSQL database...")

class UserRepository:
    def __init__(self, database: Database):
        self.database = database

    def get_user(self, user_id):
        self.database.connect()
        print(f"Getting user with ID {user_id} from the database...")
```

## Sources
All the code examples and content are from  [this article](https://realpython.com/solid-principles-python/)
