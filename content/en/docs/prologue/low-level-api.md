---
title: "API Reference"
description: "Low Level API reference for Virgil."
lead: ""
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

## Low Level API

If you find that you have a complex query that cannot be expressed with the DSL yet, then you can use the lower level cql interpolator to express your query or mutation:

```scala
import io.kaizensolutions.virgil.cql._
def selectAll: CQL[Person] =
  cql"SELECT id, name, age, addresses FROM persons".query[Person]

def insertLowLevel(p: Person): CQL[MutationResult] =
  cql"INSERT INTO persons (id, name, age, addresses) VALUES (${p.id}, ${p.name}, ${p.age}, ${p.addresses}) USING TTL 10".mutation
```

**Note that the lower-level API will turn the CQL into a string along with bind markers for each parameter and use bound statements under the hood, so you do not have to worry about CQL injection attacks.**

If you want to string interpolate some part of the query because you may not know your table name up front (i.e. its passed through configuration, then you can use `s"I am a String ${forExample}".appendCql(cql"continuing the cassandra query") or cql"SELECT * FROM ".appendString(s"$myTable")`). Doing interpolation in cql is different from string interpolation as it will cause bind markers to be created.


### Cassandra batches

You can also batch (i.e. Cassandra's definition of the word) mutations together by using +:

```scala
val batch: CQL[MutationResult]         = insert(p1) + update(p2.id, newPInfo) + insert(p3)
val unloggedBatch: CQL[MutationResult] = CQL.unlogged(batch)
```

**Note that you cannot batch together queries and mutations as this is not allowed by Cassandra.**

### Compiling CQL queries and mutations into streaming effects

Now that we have built our CQL queries and mutations, we can execute them:

```scala
import zio._
import zio.stream._

// A single element stream is returned
val insertResult: ZStream[Has[CQLExecutor], Throwable, MutationResult] = insert(person).execute

// A stream of results is returned
val queryResult: ZStream[Has[CQLExecutor], Throwable, Person] = selectAll.execute
```

### Executing queries and mutations

Running CQL queries and mutations is done through the CQLExecutor, which produces a ZStream that contains the results. You can obtain a CQLExecutor layer provided you have a CqlSessionBuilder from the Datastax Java Driver:

```scala
val dependencies: ULayer[Has[CQLExecutor]] = {
  val cqlSessionBuilderLayer: ULayer[Has[CqlSessionBuilder]] =
    ZLayer.succeed(
      CqlSession
        .builder()
        .withKeyspace("virgil")
        .withLocalDatacenter("dc1")
        .addContactPoint(InetSocketAddress.createUnresolved("localhost", 9042))
        .withApplicationName("virgil-tester")
  )
  val executor: ZLayer[Any, Throwable, Has[CQLExecutor]] = cqlSessionBuilderLayer >>> CQLExecutor.live
  executor.orDie
}

val insertResultReady: Stream[Throwable, MutationResult] = insertResult.provideLayer(dependencies)
```

### Adding support for custom data types

Virgil provides all the default primitive data-types supported by the Datastax Java Driver. However, you can add support for your own primitive data-types. For example, if you want to add support for `java.time.LocalDateTime`, you can do so in the following manner (make sure to be extra careful when managing timezones):

```scala
import io.kaizensolutions.virgil.codecs._
import java.time.{LocalDateTime, ZoneOffset}

implicit val javaTimeInstantEncoder: CqlPrimitiveEncoder[LocalDateTime] =
  CqlPrimitiveEncoder[java.time.Instant].contramap[java.time.LocalDateTime](_.toInstant(ZoneOffset.UTC))

implicit val javaTimeInstantDecoder: CqlPrimitiveDecoder[LocalDateTime] =
  CqlPrimitiveDecoder[java.time.Instant].map[LocalDateTime](instant =>
    LocalDateTime.ofInstant(instant, ZoneOffset.UTC)
  )
```

### Underlying driver configuration

This library is built on the Datastax Java Driver, please see the [Datastax Java Driver documentation](https://docs.datastax.com/en/developer/java-driver/4.15/) if you would like to configure the driver.
