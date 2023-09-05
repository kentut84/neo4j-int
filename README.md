# neo4j-int

Starting Docker Desktop





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


