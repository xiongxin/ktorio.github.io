---
title: Velocity
caption: Using Velocity Templates
section: Features
permalink: /features/velocity.html
keywords: html
feature:
    artifact: io.ktor:ktor-velocity:$ktor_version
    class: io.ktor.velocity.Velocity
---

Ktor includes support for [Velocity](http://velocity.apache.org/) templates through the Velocity
feature.  Initialize the Velocity feature with a
[VelocityEngine](https://velocity.apache.org/engine/1.7/apidocs/org/apache/velocity/app/VelocityEngine.html):

{% include feature.html %}

## Installation
{: #installation}

You can install Velocity, and configure the `VelocityEngine`.

```kotlin
install(Velocity) { // this: VelocityEngine
    setProperty("resource.loader", "string");
    addProperty("string.resource.loader.class", StringResourceLoader::class.java.name)
    addProperty("string.resource.loader.repository.static", "false")
    init() // need to call `init` before trying to retrieve string repository
    
    (getApplicationAttribute(StringResourceLoader.REPOSITORY_NAME_DEFAULT) as StringResourceRepository).apply {
        putStringResource("test.vl", "<p>Hello, \$id</p><h1>\$title</h1>")
    }
}
```

## Usage
{: #usage}

When Velocity configured, you can call the `call.respond` method with a `VelocityContent` instance: 

```kotlin
routing {
    val model = mapOf("id" to 1, "title" to "Hello, World!")

    get("/") {
        call.respond(VelocityContent("test.vl", model, "e"))
    }
}
```
