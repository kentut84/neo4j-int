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

## Using FastAPI
### Prerequisite

<br>pip3 install fastapi
<br>pip3 install uvicorn
<br>
<br>create an .env file with the uri, user, and pwd

### Writing the API
Customer Count
<br>main.py
```python
from fastapi import FastAPI
from pydantic import BaseModel
from neo4j import GraphDatabase
import os
from dotenv import load_dotenv

load_dotenv()

uri=os.getenv("uri")
user=os.getenv("user")
pwd=os.getenv("pwd")

class nodemodel(BaseModel):
        name:str
        cust_id:int

def connection():
        driver=GraphDatabase.driver(uri=uri,auth=(user,pwd))
        return (driver)

app=FastAPI()
@app.get("/")
def default():
    return {"response":"this is my default test"}


#GET API FUNCTION
@app.get("/count")
def countnode(label):
    driver_neo4j=connection()
    session=driver_neo4j.session()
    q1="""
    match(n) where labels(n) in [[$a]] return n.Country as Country, count(n) as count
    """

    x={"a":label}
    results=session.run(q1,x)
    return {"response":[{"Country":row["Country"],"Count":row["count"]}for row in results]}

#API TO CREATE
@app.post("/create")
def createnode(node:nodemodel):
    driver_neo4j=connection()
    session=driver_neo4j.session()
    q2="""
    create(n:mycustomer{name:$name,cust_id:$cust_id}) return n.name as name
    """
 
    y={"name":node.name,"cust_id":node.cust_id}
    results=session.run(q2,y)
    data=[{"Name":row["name"]} for row in results][0]["Name"]
    return{"response":"node created with customer name as: "+data}

#API TO UPDATE INFORMATION
@app.put("/update")
def update(node:nodemodel,inputname):
    driver_neo4j=connection()
    session=driver_neo4j.session()
    q3="""
    match(n:mycustomer{name:$inputname}) set n.name=$name, n.cust_id=$cust_id return n.name as name
    """
    z={"inputname":inputname,"name":node.name,"cust_id":node.cust_id}
    results=session.run(q3,z)
    data=[{"Name":row["name"]} for row in results]
    if (len(data)>0):
       data=data[0]["Name"]
       return {"response":"node updated with new name: "+data}
    else:
       return {"response":"name could not be found"}

#API TO DELETE INFORMATION
@app.delete("/delete/{cust_id}")
def delete(cust_id:int):
    driver_neo4j=connection()
    session=driver_neo4j.session()
    q4="""
    match(n:mycustomer{cust_id:$cust_id}) delete n
    """
    a={"cust_id":cust_id}
    session.run(q4,a)
    return "customer id deleted"
```

### Starting the API
uvicorn main:app --port 8081
<br>![image](https://github.com/kentut84/neo4j-int/assets/16041392/aca196e2-b042-4f77-837d-72a8a78de601)
<br>

### FastAPI Swagger - Test The API
<img width="1421" alt="image" src="https://github.com/kentut84/neo4j-int/assets/16041392/54fe3f94-b93b-48d6-abab-088d6db78905">




