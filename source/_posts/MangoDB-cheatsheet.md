---
title: MangoDB-cheatsheet
date: 2016-11-02 17:27:38
updated: 2016-11-02 17:27:38
tags:
categories:
---
#### Start Mongod && Command

```

mongod
ps aux | grep mongo

-- Start the Mongo Shell
mongo
show dbs
use meals-development
show collections
show users
show roles
show profile
show databases
```
<!-- more -->
```
db.auth()
db.help()
-- collection 为集合名称
db.collection.help()
db.collection.help()
db.collection.find().help()
db.collection.find().toArray

db.collection.find()
db.collection.insert()
db.collection.update()
db.collection.save()
db.collection.remove()
db.collection.drop()
db.collection.createIndex()
db.getSiblingDB()

db.system.users.find().pretty()
db.createCollection('customers')  

-- 创建用户
db.createUser({  
    "user": "chrisaiv",
    "pwd": "password",
    "roles": [
        { role: "clusterAdmin", db: "admin" },
        { role: "readAnyDatabase", db: "admin" },
        "readWrite"
        ]
    },
    { w: "majority", wtimeout: 5000 }
)

```
#### Data Types
[参考](https://docs.mongodb.com/v3.2/core/shell-types/#types)
##### Date
- Date() method which returns the current date as a string.
- new Date() constructor which returns a Date object using the ISODate() wrapper.
- ISODate() constructor which returns a Date object using the ISODate() wrapper.

##### ObjectId
new ObjectId

##### NumberLong &&  NumberInt
NumberLong("2090845886852")。demo：
> db.collection.insert( { _id: 10, calc: NumberLong("2090845886852") } )
db.collection.update( { _id: 10 },
                      { $set:  { calc: NumberLong("2555555000000") } } )
db.collection.update( { _id: 10 },
                      { $inc: { calc: NumberLong(5) } } )

#### Check Types

- instanceof -- mydoc._id instanceof ObjectId
- typeof -- typeof mydoc._id



#### SQL &&  MongoDB Mapping Chart
[sql与nosql的不同于区别参考](https://gist.github.com/aponxi/4380516)
##### Executables
|MySQL/Oracle	 | MongoDB
---|---|---
Database Server | mysqld/oracle| mongod
Database Client | mysql/sqlplus| mongo
##### Terminology and Concepts
SQL Terms/Concepts	 | MongoDB Terms/Concepts
---|---|
database |	database
table |	collection
row	| document or BSON document
column	| field
index |	index
table joins | embedded documents and linking
primary key,Specify any unique column or column combination as primary key.|primary key,In MongoDB, the primary key is automatically set to the _id field.

##### Create and Alter and CURD


```
db.users.insert( {
    user_id: "abc123",
    age: 55,
    status: "A"
 } )
 
 -- However, you can also explicitly create a collection:
 db.createCollection("users")
 
 db.users.ensureIndex( { user_id: 1 } )
 db.users.ensureIndex( { user_id: 1, age: -1 } )
 db.users.drop()
 
 
 db.users.find()
 db.users.find(
    { },
    { user_id: 1, status: 1 }
)

db.users.find(
    { },
    { user_id: 1, status: 1, _id: 0 }
)

db.users.find(
    { status: "A" },
    { user_id: 1, status: 1, _id: 0 }
)
db.users.find(
   { age: { $gt: 25, $lte: 50 } }
)
db.users.find(
   { user_id: /^bc/ }
)
db.users.find( { status: "A" } ).sort( { user_id: 1 } )

db.users.count()
db.users.find().count()

db.users.count( { user_id: { $exists: true } } )
db.users.find( { user_id: { $exists: true } } ).count()

db.users.count( { age: { $gt: 30 } } )
db.users.find( { age: { $gt: 30 } } ).count()

db.users.distinct( "status" )

db.users.findOne()
db.users.find().limit(1)

db.users.find().limit(5).skip(10)
db.users.find( { status: "A" } ).explain()

db.users.update(
   { age: { $gt: 25 } },
   { $set: { status: "C" } },
   { multi: true }
)

db.users.update(
   { status: "A" } ,
   { $inc: { age: 3 } },
   { multi: true }
)


db.users.remove( { status: "D" } )
db.users.remove( )
```
##### Example
##### INSERTING DATA

```
db.customers.insert({  
    first_name: "chris",
    last_name:  "aiv" 
})

--Append a lot of data into a customer
db.customers.insert({  
    first_name: "Peter",
    age: 32,
    address: {
        street: "120 Main St",
        city: "Chicago",
        state: "Illinois",
        zip: "38475"
    },
    phone: {
        home: "5555555555",
        work: "4444444444",
        mobile: "3333333333"
    },
    services: [
        {
            service_id: "time warner"
        },
        {
            service_id: "pge"
        },
        {
            service_id: "moviepass"
        }
    ],
    services_count: 3
});

-- Append data to a customer

db.customers.insert({  
    first_name: "Billy",
    last_name:  "Corgan",
    gender: "m"
})

-- Insert multiple customers in one query

db.customers.insert([  
    {
    first_name: "Jimmy",
    last_name:  "Hendrix"
    },
    {
    first_name: "Jimmy",
    last_name:  "Page"
    },
    {
    first_name: "Kurt",
    last_name:  "Cobain"
    },
    {
    first_name: "Adrian",
    last_name:  "Belew"
    },
    {
    first_name: "Billy",
    last_name:  "Corgan"
    }
])


```
##### FINDING DATA

```
db.customers.find({ first_name: "Peter"}, { services: 1})  
db.customers.find({ first_name: "Peter"}, { "services.service_id": 1})  
db.customers.findOne(  
    { first_name: /^billy$/i },
    { first_name: 1 }
)

db.customers.find({  
    gender: "male"
})

db.customers.find({  
    gender: /(m|male)/i, first_name: /^billy$/i 
})

```
##### UPDATING DATA

```
--慎用，破坏式的更新
db.customers.update(  
    { first_name: "Jimmy" },
    { last_name:  "Hendrix"}
)
-- Gentel Update 
db.customers.update(  
    { last_name: /^hendrix$/i },
    { $set: { first_name: "Jimmy" } }
)

-- Increment a value in a field
db.customers.update(  
    { first_name: "Billy" },
    { $inc: { age: 1 }
    }
)

-- Update or Insert a field using an object ID
db.customers.update(  
    { _id: ObjectId("5669f625a0a8005d8c7aae31") },
    {
        $set: {
            gender: "male",
            age: 50,
            birthdate: new Date("Aug 20, 1985")
        }
    }
)
-- Update a field using someones first name
db.customers.update(  
    { first_name: "Jimmy" },
    {
        $set: {
            gender: "male",
            age: 50,
            birthdate: new Date("Aug 20, 1985")
        }
    },
    { upsert: true }
)
-- Add to an existing document
db.customers.update(  
    { first_name: "Jimmy" },
    { $push: {
        services: {
            service_id: 'hosting windows',
            service_name: "windows hosting"
        }
        }
    }
)

```

##### REMOVING DATA

```
--Remove a field
db.customers.update(  
    { last_name: "Page" },
    { 
        $unset: { age: 1 }
    }
)

--Remove a customer
db.customers.remove(  
    //!!! DO NOT US THIS
    { first_name: "Billy"}, true 
)
-- Remove any customer above the age of 31
db.customers.remove(  
    { age: { $gt: 31 } }
    , true
)
```
##### DELETING DATA

```
-- Delete a collection
db.customers.drop()
```

##### SEARCH

###### How to search an array within an object

```
{
  _id: 1,
  name: { first: 'John', last: 'Backus' },
  birth: new Date('Dec 03, 1924'),
  death: new Date('Mar 17, 2007'),
  contribs: [ 'Fortran', 'ALGOL', 'Backus-Naur Form', 'FP' ],
  awards: [
            { award: 'National Medal',
              year: 1975,
              by: 'NSF' },
            { award: 'Turing Award',
              year: 1977,
              by: 'ACM' }
          ]
}
db.users.find({awards: {$elemMatch: {award:'National Medal', year:1975}}})
```

- [命令参考与更新](https://docs.mongodb.com/v3.2/tutorial/access-mongo-shell-help/)
- [参考](http://www.cnblogs.com/huangxincheng/archive/2012/02/21/2361205.html)