---
title: 'Hmm Python @Property'
date: 2019-06-11 23:05:03
lastmod: 2019-06-12 15:42:18
draft: false
tags:
  - 'Functions'
  - 'Python'
categories:
  - 'Python'
aliases:
  - '/python-property/'
---

![Property decorator diagram](/images/property.png)

Hmm. I am thinking what is that? let's deep a dive with me. Before I start  I want to ask some questions myself.
1. What is <code>@property</code> in Python?
2. When should I use the <code>@Property</code> function?
3. How to use the <code>@property</code> function?
In this article, you will understand what exactly the Python <code>@Property</code> does.
<ol>
 	<li>What is the Property in Python?
In Python property is a built-in function. Property function is helping to modify class attributes and give backward compatibility. But you should have a basic concept of Python classes and method. Because of the <code>@property</code> is typically used inside one.</li>
 	<li>When should I use the <code>@Property</code> function?
Assume, You create a class that store user bank account information like a bank account number and balance.
Let's create the class

```
class Account:
    """ A basic class that store bank account information """

    def __init__(self, bank_account: str, balance: float):
        self.bank_account = bank_account
        self.balance = balance
        self.account_information = dict(bank_account=self.bank_account, balance=self.balance)

    def __str__(self):
        return f"Bank Account: {self.bank_account} - balance: {self.balance}"


account = Account("0004-0067894712", 1000.50)
print(f"account number: {account.bank_account}")
print(f"Current balance: {account.balance}")
print(f"Account information: {account.account_information}")

# Output should look like this 
account number: 0004-0067894712
Current balance: 1000.5
Account information: {'bank_account': '0004-0067894712', 'balance': 1000.5}

```

Now, from this account we transfer the fund what will we get after calling the account_information attribute? Let's transfer the balance and call the account_information attribute of the instance.
```
# Fund transfer process
account.balance = account.balance - 500.25

# Now call the account_information attribute
print("Calling the account_information attribute")
print(f"Account information: {account.account_information}")
# calling the balance attribute 
print(f"Current balance: {account.balance}")

# Output
Calling the account_information attribute
Account information: {'bank_account': '0004-0067894712', 'balance': 1000.5}
Current balance: 500.25
```

No change! But we transferred the balance. We expected that the balance should be 500.25. But it does not happen. Because Python does not change the account_information value.  If we call the balance attribute we will get the exact balance. I mean the Current balance is 500.25.

Assume, you are working with a team and you are the owner of this bank account module. Also, you share your bank account module with your colleagues. Let's your team members are working with a new module that is called Payment Gateway and they want to charge an amount from the bank account. When they call the account_information attribute. They will get the account balance is 1000.50. But we had already transferred 500.25 amount from that account. In this case, all team members are falling in trouble.

In this case, we can solve the problem by creating a new method of account class. Let's create the method.

```
class Account:
    """ A basic account information store class """

    def __init__(self, bank_account: str, balance: float):
        self.bank_account = bank_account
        self.balance = balance
        # self.account_information = dict(bank_account=self.bank_account, balance=self.balance)

    def account_information(self) -&gt; dict:
        """
        Get account information
        :return: {'bank_account': '0004-0067894712', 'balance': 1000.5}
        :rtype: dict
        """
        return dict(bank_account=self.bank_account, balance=self.balance)

    def __str__(self):
        return f"Bank Account: {self.bank_account} - balance: {self.balance}"


account = Account("0004-0067894712", 1000.50)
print(f"account number: {account.bank_account}")
print(f"Current balance: {account.balance}")
print(f"Account information: {account.account_information()}")

# Fund transfer process
account.balance = account.balance - 500.25

# Now call the account_information attribute
print("Calling the account_information attribute")
print(f"Account information: {account.account_information()}")
print(f"Current balance: {account.balance}")

#Output 
account number: 0004-0067894712
Current balance: 1000.5
Account information: {'bank_account': '0004-0067894712', 'balance': 1000.5}
Calling the account_information attribute
Account information: {'bank_account': '0004-0067894712', 'balance': 500.25}
Current balance: 500.25

```

Now we get the expected result. But Did you notice?? I commented out the account_information attribute and created a new method as the same name as the attribute. It has a backward compatibility issue. Because your teammates do not know the new method as the same name as the attribute and their codebase lines could be 1k or more than of their Payment Gateway module.

In this case,  The <code>@Property</code>  will help to solve these two types of issue. The balance amount change problem and backward compatibility.</li>
 	<li>How to use the <code>@property</code> function?
Let's solve the problem in a Pythonic way.
```
.......
    # Just declare the @property decorator and solve those problems.
    @property
    def account_information(self) -&gt; dict:
        """
        Get account information
        :return: {'bank_account': '0004-0067894712', 'balance': 1000.5}
        :rtype: dict
        """
        return dict(bank_account=self.bank_account, balance=self.balance)
.......
```
Both problems solved by using just <code>@property</code> decorator. But The <code>@property</code> function has more features. Such as you can set a new value, get the value, and delete the value.
Example:
```
class Account:
    """ A basic account information store class """

    def __init__(self, bank_account: str, balance: float):
        self.bank_account = bank_account
        self.balance = balance
        self._account_information = dict(bank_account=self.bank_account, balance=self.balance)

    # Just declare the @property decorator and solve those problems.
    @property
    def account_information(self):
        """
        Get account information
        :return: {'bank_account': '0004-0067894712', 'balance': 1000.5}
        :rtype: dict
        """
        return self._account_information

    @account_information.setter
    def account_information(self, kwargs):
        """
        :param kwargs:
        :return:
        """
        print("set a value")
        self._account_information = kwargs

    @account_information.deleter
    def account_information(self):
        """
        :return:
        """
        print("call deleter method")
        self._account_information = dict(bank_account=None, balance=None)

    def __str__(self):
        return f"Bank Account: {self.bank_account} - balance: {self.balance}"


account = Account("0004-0067894712", 1000.50)
print(f"account number: {account.bank_account}")
print(f"Current balance: {account.balance}")
print(f"Account information: {account.account_information}")

# Now set a new value in the _account_information attribute
account.account_information = {"bank_account": "0004-0067899870", "balance": 100.50}

# Output new account information
print(f"New account information: {account.account_information}")

# Output for deleter method
del account.account_information

# after deleting the information should print bank_account: None and balance=None
print(account.account_information)

# Output
account number: 0004-0067894712
Current balance: 1000.5
Account information: {'bank_account': '0004-0067894712', 'balance': 1000.5}
set a value
New account information: {'bank_account': '0004-0067899870', 'balance': 100.5}
call deleter method
{'bank_account': None, 'balance': None}
```
Hope the purpose of ```@Property``` is clear and you now know when and how to use it. If you did, congratulations! I will meet you in the next one.</li>
</ol>
