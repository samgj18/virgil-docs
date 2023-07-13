---
title: "Quick Start"
description: "One page summary of how to start with Virgil."
lead: "One page summary of how to start with Virgil."
date: 2020-11-16T13:59:39+01:00
lastmod: 2020-11-16T13:59:39+01:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 110
toc: true
---

## Introduction

You can follow along by checking out this repository and running docker-compose up which will bring up Datastax Enterprise Cassandra node along with Datastax Studio which provides a nice UI to interact with Cassandra. You have to create a new connection in Datastax Studio where you point to the container (since this is running in the same Docker network, we utilize the Docker DNS to resolve the hostname to the container IP and the hostname of the Cassandra cluster is datastax-enterprise):

![Datastax Studio Connection](/dsc.png "Datastax Studio Connection")
![Datastax Notebook](/notebook.png "Datastax Notebook")
![Datastax Query](/queries.png "Datastax Query")

{{< alert icon="👉" text="Please keep reading if you want to follow along" />}}

### Keyspace setup

```sql
CREATE KEYSPACE IF NOT EXISTS virgil
  WITH REPLICATION = {
    'class': 'SimpleStrategy',
    'replication_factor': 1
}
```

### Table setup

And the following Casandra table along with its User Defined Types (UDTs) (Make sure we are using the keyspace with USE virgil):

```sql
CREATE TYPE info (
  favorite BOOLEAN,
  comment TEXT
);

CREATE TYPE address (
  street TEXT,
  city TEXT,
  state TEXT,
  zip INT,
  data frozen<list<info>>
);

CREATE TABLE IF NOT EXISTS persons (
  id TEXT,
  name TEXT,
  age INT,
  past_addresses frozen<set<address>>,
  PRIMARY KEY ((id), age)
);
```

### Scala data-types

If we want to read and write data to this table, we create case classes that mirror the table and UDTs in Scala:

```scala
import io.kaizensolutions.virgil.annotations.CqlColumn

final case class Info(favorite: Boolean, comment: String)
final case class Address(street: String, city: String, state: String, zip: Int, data: List[Info])
final case class Person(
  id: String,
  name: String,
  age: Int,
  @CqlColumn("past_addresses") addresses: Set[Address]
)
```

{{< alert icon="👉" text="Note that the CqlColumn annotation can be used if the column/field name in the Cassandra table is different from the Scala representation. This can also be used inside User Defined Types as well." />}}

### Scala 3 caveats

If you are using Scala 3.3.x, you will need to use semi-automatic derivation as I have not yet figured out how to enable fully automatic derivation like Scala 2.x has.

```scala
final case class Info(favorite: Boolean, comment: String)
object Info:
    given cqlUdtValueEncoderForInfo: CqlUdtValueEncoder.Object[Info] = CqlUdtValueEncoder.derive[Info]
    given cqlUdtValueDecoderForInfo: CqlUdtValueDecoder.Object[Info] = CqlUdtValueDecoder.derive[Info]

final case class Address(street: String, city: String, state: String, zip: Int, data: List[Info])
object Address:
    given cqlUdtValueEncoderForAddress: CqlUdtValueEncoder.Object[Address] = CqlUdtValueEncoder.derive[Address]
    given cqlUdtValueDecoderForAddress: CqlUdtValueDecoder.Object[Address] = CqlUdtValueDecoder.derive[Address]

final case class Person(
    id: String,
    name: String,
    age: Int,
    @CqlColumn("past_addresses") addresses: Set[Address]
)
object Person:
    given cqlRowDecoderForPersonForPerson: CqlRowDecoder.Object[Person] = CqlRowDecoder.derive[Person]
```

### Writing data

Now that all the data-types are in place, we can write some data:

```scala
import io.kaizensolutions.virgil._
import io.kaizensolutions.virgil.dsl._

def insert(p: Person): CQL[MutationResult] =
  InsertBuilder("persons")
    .value("id", p.id)
    .value("name", p.name)
    .value("age", p.age)
    .value("past_addresses", p.addresses)
    .build

def setAddress(personId: String, personAge: Int, address: Address): CQL[MutationResult] =
  UpdateBuilder("persons")
    .set("past_addresses" := Set(address))
    .where("id" === personId)
    .and("age" === personAge)
    .build
```

We can also read data:

```scala
def select(personId: String, personAge: Int): CQL[Person] =
  SelectBuilder
    .from("persons")
    .columns("id", "name", "age", "past_addresses")
    .where("id" === personId)
    .and("age" === personAge)
    .build[Person]
    .take(1)
```


