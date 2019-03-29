# Query Language Reference Guide
A reference for knowing how to perform equivalent queries in different languages/paradigms. 

A lot of times, you have a database that you need to get some information out of. You know how to write a SQL query to get that information, but you aren't sure how to do the equivalent in something like SQLAlchemy. Well this guide is here to show you how to perform the equivalent query for common use cases.

First lets define our database `shopDB` that has two tables, `customer` and `purchase`:

# shopDB

## customer
| customerId | firstName | lastName | 
| ---------- | --------- | -------- |
| 1 | Peter | Tran |
| 2 | Ash | Ketchum |
| 3 | Samus | Aran |
| 4 | Big | Boss |
| 5 | Gordon | Freeman |

## purchase
| purchaseId | itemName | price | customerId | date |
| ---------- | -------- | ----- | ---------- | ---- |
| 1 | Burger | 5.00 | 1 | 01-01-1970 |
| 2 | Crowbar | 20.99 | 5 | 06-28-2007 |
| 3 | USP45 | 300.00 | 4 | 11-13-1965 |
| 4 | French Fries | 1.99 | 1 | 07-23-2018 |
| 5 | Headcrab Repellent | 5.49 | 5 | 07-15-2007 |
| 6 | Missles | 99.99 | 3 | 02-23-2080 |
| 7 | Varia Suit | 1049.99 | 3 | 02-20-2080 |
| 8 | Master Ball | 999.99 | 2 | 12-23-1998 |
| 9 | Cardboard Box | 0.99 | 4 | 08-01-1964 |
| 10 | Chocolate | 5.00 | 1 | 04-24-2002 |

Ok, now we can start.

### SQLAlchemy setup
Before we can start querying with SQLAlchemy, there's some setup we have to do to reflect the tables.
Here's the boilerplate for `shopDB`.
```python
import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, func

engine = create_engine("sqlite:///shopDB.sqlite")

Base = automap_base()
Base.prepare(engine, reflect=True)

Customer = Base.classes.customers
Purchase = Base.classes.purchases

session = Session(engine)
```

Now we can use the `session` object to make our queries.

### Selecting everything from a table

SQL:
```sql
SELECT * FROM customers;
```

SQLAlchemy:
```python
customers = session.query(Customer.customerId, Customer.firstName, Customer.lastName).\
  all()
  
for customer in customers:
  print(customer)
```

| customerId | firstName | lastName | 
| ---------- | --------- | -------- |
| 1 | Peter | Tran |
| 2 | Ash | Ketchum |
| 3 | Samus | Aran |
| 4 | Big | Boss |
| 5 | Gordon | Freeman |

### Getting all purchases made over $50

SQL:
```sql
SELECT itemName, price
FROM purchases
WHERE price > 50;
```

SQLAlchemy:
```python
items = session.query(Purchase.itemName, Purchase.price).\
  filter(Purchase.price > 50).\
  all()
  
for item in items:
  print(item)
```

| itemName | price |
| -------- | ----- |
| USP45 | 300.00 |
| Missles | 99.99 | 3 
| Varia Suit | 1049.99 |
| Master Ball | 999.99 |

### Finding all purchases made by Peter Tran

SQL:
```sql
SELECT itemName
FROM purchases p
INNER JOIN customers c
ON p.customerId = c.customerId;
```

SQLAlchemy:
```python
items = session.query(Purchase.itemName).\
  filter(Purchase.customerId == Customer.customerId).\
  all()

for item in items:
  print(item)
```

| itemName |
| -------- |
| Burger |
| French Fries |
| Chocolate |

Doesn't seem like I have a healthy diet...

### Figuring out the number of customers

SQL:
```sql
SELECT COUNT(cusomterId) AS 'customerCount'
FROM customers;
```

SQLAlchemy:
```python
customer_count = session.query(func.count(Customer.customerId)).\
  all()

print(customer_count)
```

| customerCount |
| ------------- |
| 5 |

### Finding how much money each customer has spent

SQL:
```sql
SELECT c.firstName, SUM(p.price) AS 'totalSpent'
FROM customers c
INNER JOIN purchases p
ON c.customerId = p.customerId
GROUP BY c.customerId;
```

SQLAlchemy:
```python
customer_spending = session.query(Customer.firstName, func.sum(Purchase.price)).\
  filter(Customer.customerId == Purchase.customerId).\
  group_by(Customer.customerId).\
  all()

for customer in customer_spending:
  print(customer)
```

| firstName | totalSpent |
| --------- | ---------- |
| Peter | 11.99 |
| Ash | 999.99 |
| Samus | 1149.98 |
| Big | 300.99 |
| Gordon | 26.48 |

### Figuring out who made the most purchases (yes I know it's me)

SQL:
```sql
SELECT c.firstName, COUNT(p.priceId) AS 'numberOfPurchases'
FROM customers c
INNER JOIN purchases p
ON c.customerId = p.customerId
GROUP BY c.customerId
ORDER BY numberOfPurchases DESC
LIMIT 1;
```

SQLAlchemy:
```python
customer_purchase_count = session.query(Customer.firstName, func.count(Purchase.purchaseId)).\
  filter(Customer.customerId == Purchase.customerId).\
  group_by(Customer.customerId).\
  order_by(func.count(Purchase.purchaseId)).\
  first()

print(customer_purchase_count)
```

| firstName | numberOfPurchases |
| --------- | ----------------- |
| Peter | 3 |

### Finding which items were bought in between 1900 and 2000

SQL:
```sql
SELECT itemName, date
FROM items
WHERE date >= '01-01-1900'
AND date <= '12-31-2000'
```

SQLAlchemy:
```python
items = session.Query(Purchase.itemName, Purchase.date).\
  filter(Purchase.date >= '01-01-1990').\
  filter(Purchase.date <= '12-31-2000').\
  all()

for item in items:
  print(item)
```

| itemName | date |
| -------- | ---- |
| Burger | 01-01-1970 |
| USP45 | 11-13-1965 |
| Master Ball | 12-23-1998 |
| Cardboard Box | 08-01-1964 |
