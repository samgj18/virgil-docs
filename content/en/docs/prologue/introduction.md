---
title: "Introduction"
description: "Virgil is a functional Cassandra client built using ZIO 2.x, Cats Effect 3.x, Magnolia and the Datastax 4.x Java drivers"
lead: "Virgil is a functional Cassandra client built using ZIO 2.x, Cats Effect 3.x, Magnolia and the Datastax 4.x Java drivers"
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 100
toc: true
---

## Get started

There are two main ways to get started with Virgil depending on your Effect System of choice:

### ZIO 2.x

```sbt
libraryDependencies += "io.kaizen-solutions" %% "virgil-zio" % "1.0.2"
```

### Cats Effect 3.x

```sbt
libraryDependencies += "io.kaizen-solutions" %% "virgil-cats-effect" % "1.0.2"
```

### Alternative

To integrate another effect system (or runtime), depend on the core module and reference the implementations for ZIO & Cats Effect

```sbt
libraryDependencies += "io.kaizen-solutions" %% "virgil-core" % "1.0.2"
```
{{< alert icon="👉" text="Please note that Virgil is built for Scala 2.12.x, 2.13.x and 3.3.x but fully-automatic derivation is not present for 3.3.x." />}}

### Quick Start

One page summary of how to start a new Doks project. [Quick Start →]({{< relref "quick-start" >}})

### Low Level API

Low Level API reference for Virgil. [Low Level API →]({{< relref "low-level-api" >}})

## Contributing

Find out how to contribute to Virgil. [Contributing →](https://getdoks.org/docs/contributing/how-to-contribute/)
