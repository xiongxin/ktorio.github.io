---
title: Shutdown URL
caption: Add an URL for shutting down the server
section: Features
permalink: /features/shutdown-url.html
feature:
    artifact: io.ktor:ktor-server-host-common:$ktor_version
    class: io.ktor.server.engine.ShutDownUrl.ApplicationCallFeature
---

This feature enables a URL that when accessed, shutdowns the server.

There are two ways to use it: [Automatically using HOCON](#hocon) and [Installing the feature](#install)

{% include feature.html %}

## Automatically using HOCON
{: #hocon}

You can configure a shutdown URL using HOCON with the 
[ktor.deployment.shutdown.url](/servers/configuration.html#general) property.

```kotlin
ktor {
    deployment {
        shutdown.url = "/my/shutdown/path"
    }
}
```

## Installing the feature
{: #install}

You can manually install the feature, with `ShutDownUrl.ApplicationCallFeature` and set the `shutDownUrl` and an `exitCodeSupplier`:

```kotlin
install(ShutDownUrl.ApplicationCallFeature) {
    // The URL that will be intercepted
    shutDownUrl = "/ktor/application/shutdown"
    // A function that will be executed to get the exit code of the process
    exitCodeSupplier = { 0 } // ApplicationCall.() -> Int
}
```
