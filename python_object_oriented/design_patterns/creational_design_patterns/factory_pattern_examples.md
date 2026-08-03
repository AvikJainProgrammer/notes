# Factory Method Pattern Examples

## Overview
The Factory Method Pattern is a creational design pattern that provides an interface for creating objects, but lets subclasses decide which class to instantiate. It promotes loose coupling and makes code more flexible and maintainable.

---

## Example 1: Document Creation System

A document processing application that creates different types of documents (PDF, Word, Excel).

```python
from abc import ABC, abstractmethod

# Product Interface
class Document(ABC):
    @abstractmethod
    def create(self):
        pass
    
    @abstractmethod
    def save(self, filename):
        pass

# Concrete Products
class PDFDocument(Document):
    def create(self):
        return "Creating PDF document..."
    
    def save(self, filename):
        return f"Saving as PDF: {filename}.pdf"

class WordDocument(Document):
    def create(self):
        return "Creating Word document..."
    
    def save(self, filename):
        return f"Saving as Word: {filename}.docx"

class ExcelDocument(Document):
    def create(self):
        return "Creating Excel spreadsheet..."
    
    def save(self, filename):
        return f"Saving as Excel: {filename}.xlsx"

# Creator (Abstract Factory)
class DocumentFactory(ABC):
    @abstractmethod
    def create_document(self):
        pass
    
    def process_document(self, filename):
        doc = self.create_document()
        print(doc.create())
        print(doc.save(filename))

# Concrete Creators
class PDFDocumentFactory(DocumentFactory):
    def create_document(self):
        return PDFDocument()

class WordDocumentFactory(DocumentFactory):
    def create_document(self):
        return WordDocument()

class ExcelDocumentFactory(DocumentFactory):
    def create_document(self):
        return ExcelDocument()

# Usage
pdf_factory = PDFDocumentFactory()
pdf_factory.process_document("report")

word_factory = WordDocumentFactory()
word_factory.process_document("letter")

excel_factory = ExcelDocumentFactory()
excel_factory.process_document("budget")
```

---

## Example 2: Database Connection Factory

Creates different types of database connections (MySQL, PostgreSQL, SQLite).

```python
from abc import ABC, abstractmethod

# Product Interface
class DatabaseConnection(ABC):
    @abstractmethod
    def connect(self):
        pass
    
    @abstractmethod
    def disconnect(self):
        pass
    
    @abstractmethod
    def query(self, sql):
        pass

# Concrete Products
class MySQLConnection(DatabaseConnection):
    def connect(self):
        return "Connected to MySQL database"
    
    def disconnect(self):
        return "Disconnected from MySQL"
    
    def query(self, sql):
        return f"Executing MySQL query: {sql}"

class PostgreSQLConnection(DatabaseConnection):
    def connect(self):
        return "Connected to PostgreSQL database"
    
    def disconnect(self):
        return "Disconnected from PostgreSQL"
    
    def query(self, sql):
        return f"Executing PostgreSQL query: {sql}"

class SQLiteConnection(DatabaseConnection):
    def connect(self):
        return "Connected to SQLite database"
    
    def disconnect(self):
        return "Disconnected from SQLite"
    
    def query(self, sql):
        return f"Executing SQLite query: {sql}"

# Creator
class DatabaseFactory:
    @staticmethod
    def create_connection(db_type):
        if db_type == "mysql":
            return MySQLConnection()
        elif db_type == "postgresql":
            return PostgreSQLConnection()
        elif db_type == "sqlite":
            return SQLiteConnection()
        else:
            raise ValueError(f"Unknown database type: {db_type}")

# Usage
mysql_conn = DatabaseFactory.create_connection("mysql")
print(mysql_conn.connect())
print(mysql_conn.query("SELECT * FROM users"))
print(mysql_conn.disconnect())

postgres_conn = DatabaseFactory.create_connection("postgresql")
print(postgres_conn.connect())
print(postgres_conn.query("SELECT * FROM products"))
```

---

## Example 3: Payment Processing System

Creates different payment processors (Credit Card, PayPal, Bitcoin).

```python
from abc import ABC, abstractmethod
from datetime import datetime

# Product Interface
class PaymentProcessor(ABC):
    @abstractmethod
    def validate_payment(self, amount):
        pass
    
    @abstractmethod
    def process_payment(self, amount):
        pass
    
    @abstractmethod
    def refund(self, transaction_id):
        pass

# Concrete Products
class CreditCardPayment(PaymentProcessor):
    def validate_payment(self, amount):
        return amount > 0 and amount < 100000
    
    def process_payment(self, amount):
        if self.validate_payment(amount):
            return f"Credit card payment of ${amount} processed at {datetime.now()}"
        return "Invalid payment amount"
    
    def refund(self, transaction_id):
        return f"Refunding credit card transaction {transaction_id}"

class PayPalPayment(PaymentProcessor):
    def validate_payment(self, amount):
        return amount > 0 and amount < 50000
    
    def process_payment(self, amount):
        if self.validate_payment(amount):
            return f"PayPal payment of ${amount} processed at {datetime.now()}"
        return "Invalid payment amount for PayPal"
    
    def refund(self, transaction_id):
        return f"Refunding PayPal transaction {transaction_id}"

class BitcoinPayment(PaymentProcessor):
    def validate_payment(self, amount):
        return amount > 0 and amount < 10
    
    def process_payment(self, amount):
        if self.validate_payment(amount):
            return f"Bitcoin payment of {amount} BTC processed at {datetime.now()}"
        return "Invalid Bitcoin amount"
    
    def refund(self, transaction_id):
        return f"Refunding Bitcoin transaction {transaction_id}"

# Creator
class PaymentFactory:
    @staticmethod
    def create_payment_processor(payment_method):
        payment_methods = {
            "credit_card": CreditCardPayment,
            "paypal": PayPalPayment,
            "bitcoin": BitcoinPayment
        }
        
        processor_class = payment_methods.get(payment_method)
        if processor_class is None:
            raise ValueError(f"Unknown payment method: {payment_method}")
        return processor_class()

# Usage
credit_card = PaymentFactory.create_payment_processor("credit_card")
print(credit_card.process_payment(99.99))
print(credit_card.refund("TXN123"))

paypal = PaymentFactory.create_payment_processor("paypal")
print(paypal.process_payment(49.99))

bitcoin = PaymentFactory.create_payment_processor("bitcoin")
print(bitcoin.process_payment(0.5))
```

---

## Example 4: Logging System

Creates different types of loggers (File, Console, Database).

```python
from abc import ABC, abstractmethod
import datetime

# Product Interface
class Logger(ABC):
    @abstractmethod
    def log(self, message):
        pass

# Concrete Products
class ConsoleLogger(Logger):
    def log(self, message):
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[{timestamp}] CONSOLE: {message}")

class FileLogger(Logger):
    def __init__(self, filename="app.log"):
        self.filename = filename
    
    def log(self, message):
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with open(self.filename, "a") as f:
            f.write(f"[{timestamp}] FILE: {message}\n")

class DatabaseLogger(Logger):
    def __init__(self, db_name="logs.db"):
        self.db_name = db_name
    
    def log(self, message):
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        # Simulating database insert
        print(f"[{timestamp}] DATABASE({self.db_name}): {message}")

# Creator
class LoggerFactory:
    @staticmethod
    def create_logger(logger_type, **kwargs):
        loggers = {
            "console": ConsoleLogger,
            "file": FileLogger,
            "database": DatabaseLogger
        }
        
        logger_class = loggers.get(logger_type)
        if logger_class is None:
            raise ValueError(f"Unknown logger type: {logger_type}")
        return logger_class(**kwargs)

# Usage
console_logger = LoggerFactory.create_logger("console")
console_logger.log("Application started")

file_logger = LoggerFactory.create_logger("file", filename="myapp.log")
file_logger.log("User logged in")

db_logger = LoggerFactory.create_logger("database", db_name="production.db")
db_logger.log("Payment processed")
```

---

## Example 5: Vehicle Factory

Creates different types of vehicles (Car, Truck, Motorcycle).

```python
from abc import ABC, abstractmethod

# Product Interface
class Vehicle(ABC):
    @abstractmethod
    def start(self):
        pass
    
    @abstractmethod
    def drive(self):
        pass
    
    @abstractmethod
    def stop(self):
        pass
    
    @abstractmethod
    def get_info(self):
        pass

# Concrete Products
class Car(Vehicle):
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model
    
    def start(self):
        return f"{self.brand} {self.model} engine started"
    
    def drive(self):
        return "Car is driving smoothly on the road"
    
    def stop(self):
        return "Car has stopped"
    
    def get_info(self):
        return f"Car: {self.brand} {self.model}"

class Truck(Vehicle):
    def __init__(self, brand, model, cargo_capacity):
        self.brand = brand
        self.model = model
        self.cargo_capacity = cargo_capacity
    
    def start(self):
        return f"{self.brand} {self.model} diesel engine started"
    
    def drive(self):
        return f"Truck is driving with {self.cargo_capacity}kg cargo"
    
    def stop(self):
        return "Truck has stopped and cargo secured"
    
    def get_info(self):
        return f"Truck: {self.brand} {self.model} (Capacity: {self.cargo_capacity}kg)"

class Motorcycle(Vehicle):
    def __init__(self, brand, model, engine_cc):
        self.brand = brand
        self.model = model
        self.engine_cc = engine_cc
    
    def start(self):
        return f"{self.brand} {self.model} motorcycle started ({self.engine_cc}cc)"
    
    def drive(self):
        return "Motorcycle is speeding through traffic"
    
    def stop(self):
        return "Motorcycle has stopped"
    
    def get_info(self):
        return f"Motorcycle: {self.brand} {self.model} ({self.engine_cc}cc)"

# Creator
class VehicleFactory:
    @staticmethod
    def create_vehicle(vehicle_type, **kwargs):
        vehicles = {
            "car": Car,
            "truck": Truck,
            "motorcycle": Motorcycle
        }
        
        vehicle_class = vehicles.get(vehicle_type)
        if vehicle_class is None:
            raise ValueError(f"Unknown vehicle type: {vehicle_type}")
        return vehicle_class(**kwargs)

# Usage
car = VehicleFactory.create_vehicle("car", brand="Toyota", model="Camry")
print(car.start())
print(car.drive())
print(car.get_info())

truck = VehicleFactory.create_vehicle("truck", brand="Volvo", model="FH16", cargo_capacity=5000)
print(truck.start())
print(truck.drive())

motorcycle = VehicleFactory.create_vehicle("motorcycle", brand="Harley", model="Street 750", engine_cc=750)
print(motorcycle.start())
print(motorcycle.drive())
```

---

## Example 6: Notification System

Creates different notification types (Email, SMS, Push Notification).

```python
from abc import ABC, abstractmethod

# Product Interface
class Notification(ABC):
    @abstractmethod
    def send(self, recipient, message):
        pass

# Concrete Products
class EmailNotification(Notification):
    def send(self, recipient, message):
        return f"Email sent to {recipient}: {message}"

class SMSNotification(Notification):
    def send(self, recipient, message):
        return f"SMS sent to {recipient}: {message}"

class PushNotification(Notification):
    def send(self, recipient, message):
        return f"Push notification sent to {recipient}: {message}"

# Creator
class NotificationFactory:
    @staticmethod
    def create_notification(notification_type):
        notifications = {
            "email": EmailNotification,
            "sms": SMSNotification,
            "push": PushNotification
        }
        
        notification_class = notifications.get(notification_type)
        if notification_class is None:
            raise ValueError(f"Unknown notification type: {notification_type}")
        return notification_class()

# Usage
email = NotificationFactory.create_notification("email")
print(email.send("user@example.com", "Welcome to our app!"))

sms = NotificationFactory.create_notification("sms")
print(sms.send("+1234567890", "Your OTP is 123456"))

push = NotificationFactory.create_notification("push")
print(push.send("user_device_id", "You have a new message"))
```

---

## Key Benefits of Factory Method Pattern

1. **Loose Coupling**: The client code doesn't need to know about concrete classes
2. **Single Responsibility**: Object creation logic is centralized
3. **Open/Closed Principle**: Easy to add new types without modifying existing code
4. **Flexibility**: Easy to switch implementations
5. **Testability**: Easier to mock and test with different implementations

---

## When to Use Factory Method Pattern

- When a class can't anticipate the type of objects it needs to create
- When you want to delegate the instantiation logic to subclasses
- When you need to support multiple product families
- When you want to encapsulate object creation logic
- When dealing with complex object creation logic
