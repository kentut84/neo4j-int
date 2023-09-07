# neo4j-int

## Using Neo4j and Docker
Starting Docker Desktop and downloading the Neo4j:Enterprise package 
<img width="1259" alt="2023-09-05_23-49-08" src="https://github.com/kentut84/neo4j-int/assets/16041392/6b05bba6-3cad-46cf-b54d-549acda51c8f">
<br>
<br>
In the images tab --> Play 
<img width="992" alt="image" src="https://github.com/kentut84/neo4j-int/assets/16041392/6c9048f4-c0c1-4eb6-9e74-1e7695ace9f2">
<br>
<br>
Container settings
<br>
<img width="530" alt="image" src="https://github.com/kentut84/neo4j-int/assets/16041392/66332489-2c93-4d72-bdf2-b8d0d3f3068d">

```
NEO4J_ACCEPT_LICENSE_AGREEMENT=yes
```

## Loading Customers data from Python IDLE

```python
from neo4j import GraphDatabase
graphdb=GraphDatabase.driver(uri="bolt://localhost:7687", auth=("neo4j","password"))
session=graphdb.session()
query="""LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/maruthiprithivi/f11bf40b558879aca0c30ce76e7dec98/raw/f6800464bf4125b8dd218bc6168447a129205fdc/customers.csv' AS f1
CREATE (N:CUSTOMERS{CIF:f1.CIF,Age:f1.Age, EmailAddress:f1.EmailAddress, FirstName:f1.FirstName, LastName:f1.LastName, PhoneNumber:f1.PhoneNumber, Gender:f1.Gender, Address:f1.Address, Country:f1.Country, JobTitle:f1.JobTitle, CardNumber:f1.CardNumber, AccountNumber:f1.AccountNumber})"""
record=session.run(query)
```

## Loading Purchases data from Python IDLE
```python
from neo4j import GraphDatabase
graphdb=GraphDatabase.driver(uri="bolt://localhost:7687", auth=("neo4j","password"))
session=graphdb.session()
query="""LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/maruthiprithivi/f11bf40b558879aca0c30ce76e7dec98/raw/f6800464bf4125b8dd218bc6168447a129205fdc/purchases.csv' AS f1
CREATE (N:TRANSACTIONS{TransactionID:f1.TransactionID, CardNumber:f1.CardNumber, Merchant:f1.Merchant, Amount:f1.Amount, PurchaseDateTime:f1.PurchaseDateTime, CardIssuer:f1.CardIssuer})"""
record=session.run(query)
```

## Loading Transfers data from Python IDLE
```python
from neo4j import GraphDatabase
graphdb=GraphDatabase.driver(uri="bolt://localhost:7687", auth=("neo4j","password"))
session=graphdb.session()
query="""LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/maruthiprithivi/f11bf40b558879aca0c30ce76e7dec98/raw/f6800464bf4125b8dd218bc6168447a129205fdc/transfers.csv' AS f1
CREATE (N:TRANSFERS{TransactionID:f1.TransactionID, SenderAccountNumber:f1.SenderAccountNumber, ReceiverAccountNumber:f1.ReceiverAccountNumber, Amount:f1.Amount, TransferDatetime:f1.TransferDatetime})"""
record=session.run(query)
```

## Creating Relationships with Nodes
### Customers -> Transactions
```cypher
MATCH (c:CUSTOMERS), (t:TRANSACTIONS)
WHERE c.CardNumber = t.CardNumber
CREATE (c) - [:PURCHASES_FROM] -> (t)
```
### Customers -> Transfers (sending)
```cypher
MATCH (c:CUSTOMERS), (tr:TRANSFERS)
WHERE c.AccountNumber = tr.SenderAccountNumber
CREATE (c) - [:SENDS_TO] -> (tr)
```
### Customers -> Transfers (receiving)
```cypher
MATCH (c:CUSTOMERS), (tr:TRANSFERS)
WHERE c.AccountNumber = tr. ReceiverAccountNumber
CREATE (tr) - [:RECEIVES_FROM] -> (c)
```

## Explore the Data

Customer whose name is Greta that has made purchases with the Maestro card 
```cypher
MATCH (c:CUSTOMERS {FirstName: "Greta"}) – [rel:PURCHASES_FROM] -> (tr:TRANSACTIONS {CardIssuer: "Maestro"})
RETURN c, tr
```

The most popular credit card used for varous Merchants
```cypher
MATCH (t:TRANSACTIONS)
RETURN t.Merchant, t.CardIssuer, count(t.TransactionID) ORDER BY t.Merchant, count(t.TransactionID) DESC
```

The number of transfers each Customer has made 
```cypher
MATCH (c:CUSTOMERS) – [:SENDS_TO] -> (tr:TRANSFERS) 
RETURN c.FirstName + " " + c.LastName as CustName, c. AccountNumber as AccountNumber, count(distinct tr. ReceiverAccountNumber) as CountSend
```



