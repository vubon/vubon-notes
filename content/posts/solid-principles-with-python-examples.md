---
title: "SOLID Principles with Python Examples"
date: 2026-01-03T12:00:00Z
draft: false
tags:
  - "SOLID"
  - "Python"
  - "OOP"
  - "Design Principles"
categories:
  - "Programming"
  - "Python"
---

![SOLID Principles Diagram](/images/solid-principles.png)

Let’s deep dive into SOLID principles. SOLID is a short form. It stands for

> 1. Single Responsibility Principle
> 2. Open and Closed Principle
> 3. Lisvok Sub situation Principle
> 4. Interface Segregation Principle
> 5. Dependency Inversion Principle

This is the meaning of the SOLID. In this article, I’m going to try to explain SOLID Principles in the simplest way so that it’s easy for beginners to understand.

The SOLID principles were defined in the early 2000s by [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob). Uncle Bob elaborated some of these and identified others already existing and said that these principles should be used to get good management of dependencies in our code.

### Why do you need to know?
> 1. Easy to understand the codebase
> 2. Easy to extend
> 3. Easy to maintain the codebase
> 4. Robust code
> 5. Minimum changing existing codebase or not at all.

Let’s go through each principle one by one:

### Single Responsibility Principle(SRP):
A class should have only one responsibility and only one reason to change. That means a class does not perform multiple jobs.

**Example:**  
**Violation of SRP**

```python
class Account:
   """Demo bank account class"""

   def __init__(self, account_no: str):
       self.account_no = account_no

   def get_account_number(self):
       """Get account number"""
       return self.account_no

   def save(self):
     """Save account number into DB"""
     pass 
```

### How does it violate the SRP?
In the account class, I am performing two tasks. One is stored data and another one gets account number. So it violates the SRP.

**Solution**: A common solution to this problem is to apply the facade pattern. Let’s create another class and this class will handle database management job and the account class will only handle his properties.

```python
class AccountDB:
   """Account DB management class """
   def get_account_number(self, _id):
       """Get account number"""
       pass

   def account_save(self, obj):
       """Save account number into DB"""
       pass


class Account:
   """Demo bank account class """
   def __init__(self, account_no: str):
       self.account_no = account_no
       self._db = AccountDB()

   def get_account_number(self):
       """Get account number"""
       return self.account_no

   def get(self, _id):
       """
       :param _id:
       :return:
       """
       return self._db.get_account_number(_id=_id)

   def save(self):
       """account save"""
       self._db.account_save(obj=self)
```

### Open and Closed Principle(OCP):
*Software entities (classes, function, module) open for extension, but not for modification (or closed for modification)*

**Example:**  
**Violation of OCP**

```python
class Discount:
   """Demo customer discount class"""
   def __init__(self, customer, price):
       self.customer = customer
       self.price = price

   def give_discount(self):
       """A discount method"""
       if self.customer == 'normal':
           return self.price * 0.2
       elif self.customer == 'vip':
           return self.price * 0.4
```
This example is failed to pass the Open and Close Principle(OCP). Assume, we have a super VIP customer and we want to give a discount of 0.8 percentage. What would we do in this case? Maybe we will solve the problem this way

```python
.......
    def give_discount(self):
       """A discount method"""
       if self.customer == 'normal':
           return self.price * 0.2
       elif self.customer == 'vip':
           return self.price * 0.4
       elif self.customer ==  'supvip':
           return self.price * 0.8
```
But this solution violates the OCP. Because we can’t modify the give_discount method. Only we can extend the method.

**Solution:**

```python
class Discount:
   """Demo customer discount class"""
   def __init__(self, customer, price):
       self.customer = customer
       self.price = price

   def get_discount(self):
       """A discount method"""
       return self.price * 0.2

class VIPDiscount(Discount):
   """Demo VIP customer discount class"""
   def get_discount(self):
       """A discount method"""
       return super().get_discount() * 2

class SuperVIPDiscount(VIPDiscount):
   """Demo super vip customer discount class"""
   def get_discount(self):
       """A discount method"""
       return super().get_discount() * 2
```

### Liskov Substitution Principle(LSP):
*Let φ(x) be a property provable about objects x of type T. Then φ(y) should be true for objects y of type S where S is a subtype of T.
More formally, this is the original definition (LISKOV 01) of Liskov’s substitution principle: if S is a subtype of T, then objects of type T may be replaced by objects of type S, without breaking the program.*

Liskov Substitution Principle was introduced by Barbara Liskov in her conference keynote “Data Abstraction” in 1987.

**Example**  
**Violation of LSP**

```python
class Vehicle:
   """A demo Vehicle class"""
   def __init__(self, name: str, speed: float):
       self.name = name
       self.speed = speed

   def get_name(self) -> str:
       """Get vehicle name"""
       return f"The vehicle name {self.name}"

   def get_speed(self) -> str:
       """Get vehicle speed"""
       return f"The vehicle speed {self.speed}"

   def engine(self):
       """A vehicle engine"""
       pass

   def start_engine(self):
       """A vehicle engine start"""
       self.engine()


class Car(Vehicle):
   """A demo Car Vehicle class"""
   def start_engine(self):
       pass


class Bicycle(Vehicle):
   """A demo Bicycle Vehicle class"""
   def start_engine(self):
       pass
```

In Bicycle class violates the LSP. Cause in the Vehicle class has an engine method. But naturally, a bicycle has no engine. So we could not start any engine.

Refactor the code and make a solution for this problem.

**Solution:**

```python
class Vehicle:
   """A demo Vehicle class"""
   def __init__(self, name: str, speed: float):
       self.name = name
       self.speed = speed

   def get_name(self) -> str:
       """Get vehicle name"""
       return f"The vehicle name {self.name}"

   def get_speed(self) -> str:
       """Get vehicle speed"""
       return f"The vehicle speed {self.speed}"

class VehicleWithoutEngine(Vehicle):
   """A demo Vehicle without engine class"""
   def start_moving(self):
      """Moving"""
      raise NotImplemented

class VehicleWithEngine(Vehicle):
   """A demo Vehicle engine class"""
   def engine(self):
      """A vehicle engine"""
      pass

   def start_engine(self):
      """A vehicle engine start"""
      self.engine()

class Car(VehicleWithEngine):
   """A demo Car Vehicle class"""
   def start_engine(self):
       pass

class Bicycle(VehicleWithoutEngine):
   """A demo Bicycle Vehicle class""" 
   def start_moving(self):
       pass

```

Actually, LSP is a concept that applies to all kinds of polymorphism. Only if you don’t use polymorphism of all you don’t need to care about the LSP.

### Interface Segregation Principle(ISP):
*Actually, This principle suggests that “A client should not be forced to implement an interface that it does not use”*

**Example:**  
**Violation of ISP**

```python
class Shape:
   """A demo shape class"""
   def draw_circle(self):
       """Draw a circle"""
       raise NotImplemented

   def draw_square(self):
       """ Draw a square"""
       raise NotImplemented

class Circle(Shape):
    """A demo circle class"""
   def draw_circle(self):
       """Draw a circle"""
       pass

   def draw_square(self):
       """ Draw a square"""
       pass
```
In the above example, we need to call an unnecessary method in the Circle class. Hence the example violated the Interface Segregation Principle.

**Solution:**

```python
class Shape:
   """A demo shape class"""
   def draw(self):
      """Draw a shape"""
      raise NotImplemented

class Circle(Shape):
   """A demo circle class"""
   def draw(self):
      """Draw a circle"""
      pass

class Square(Shape):
   """A demo square class"""
   def draw(self):
      """Draw a square"""
      pass
```
Another example

```python
class BankAccount:
   """A demo Bank Account class"""
   def __init__(self, balance: float, account: str):
       self.account = {f"{account}": balance}

   def balance(self, account: str):
       """Get current balance"""
       raise NotImplemented

class Deposit(BankAccount):
   """A demo circle class"""
   def balance(self, account: str):
      """Get current balance"""
      return self.account.get(account)

   def deposit(self, amount: float, account: str):
       """Deposit a new amount"""
       current = self.balance(account)
       new_amount = current + amount
       self.account.update({account: new_amount})
```

### Dependency Inversion Principle(DIP):
This principle suggests that below two points.  
a. High-level modules should not depend on low-level modules. Both should depend on abstractions.  
b. Abstractions should not depend on details. Details should depend on abstractions.

**Example:**  
**Violation of DIP**
```python
class BackendDeveloper:
    """This is a low-level module"""
    @staticmethod
    def python():
        print("Writing Python code")

class FrontendDeveloper:
    """This is a low-level module"""
    @staticmethod
    def javascript():
        print("Writing JavaScript code")

class Project:
    """This is a high-level module"""
    def __init__(self):
        self.backend = BackendDeveloper()
        self.frontend = FrontendDeveloper()

    def develop(self):
        self.backend.python()
        self.frontend.javascript()
        return "Develop codebase"

project = Project()
print(project.develop())
```

Another example

```python
class NewsPerson:
    """This is a high-level module"""
    @staticmethod
    def publish(news: str) -> None:
        """
        :param news:
        :return:
        """
        print(NewsPaper().publish(news=news))

class NewsPaper:
    """This is a low-level module"""
    @staticmethod
    def publish(news: str) -> None:
        """
        :param news:
        :return:
        """
        print(f"{news} Hello newspaper")

person = NewsPerson()
print(person.publish("News Paper"))

```
The project class is a high-level module and backend & frontend are the low-level modules. In this example, we found that the high-level module depends on the low-level module. Hence this example are violated the Dependency Inversion Principle. Let’s solve the problem according to the definition of DIP.

**Solution:**
Example 1

```python
class BackendDeveloper:
   """This is a low-level module"""
   def develop(self):
       self.__python_code()

   @staticmethod
   def __python_code():
       print("Writing Python code")

    
class FrontendDeveloper:
   """This is a low-level module"""
   def develop(self):
       self.__javascript()

   @staticmethod
   def __javascript():
       print("Writing JavaScript code")


class Developers:
   """An Abstract module"""
   def __init__(self):
       self.backend = BackendDeveloper()
       self.frontend = FrontendDeveloper()

   def develop(self):
       self.backend.develop()
       self.frontend.develop()

class Project:
   """This is a high-level module"""
   def __init__(self):
       self.__developers = Developers()

   def develops(self):
       return self.__developers.develop()

project = Project()
print(project.develops())
```

Example 2:

```python
class NewsPerson:
   """This is a high-level module"""
   @staticmethod
   def publish(news: str, publisher=None) -> None:
       print(publisher.publish(news=news))

class NewsPaper:
   """This is a low-level module"""
   @staticmethod
   def publish(news: str) -> None:
       print("{} news paper".format(news))

class Facebook:
   """This is a low-level module"""
   @staticmethod
   def publish(news: str) -> None:
       print(f"{news} - share this post on {news}")

person = NewsPerson()
person.publish("hello", NewsPaper())
person.publish("facebook", Facebook())
```
I hope the purpose of this article is clear and you now know when and how to use it. If you did, congratulations! I will meet you in the next one.  
That all for today. Thank you for reading my article. 

[Source Code](https://github.com/vubon/SOLID-Principles)

**Reference:**
1. https://raygun.com/blog/solid-design-principles/
2. https://learning.oreilly.com/library/view/clean-code/9780136083238/