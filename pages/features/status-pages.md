---
title: Status Pages
caption: Handle Exceptions and Customize Status Pages
section: Features
permalink: /features/status-pages.html
feature:
    artifact: io.ktor:ktor-server-core:$ktor_version
    class: io.ktor.features.StatusPages
---

The `StatusPages` feature allows Ktor applications to respond appropriately to any failure state. 
This feature is installed using the standard application configuration:

```kotlin
fun Application.main() {
    install(StatusPages)
}
```

There are three main configuration options provided to StatusPages:

1. `exceptions` - Configures response based on mapped exception classes 
2. `status` - Configures response to status code value
3. `statusFile` - Configures standard file response from classpath

{% include feature.html %}

### Exceptions 

The exception configuration can provide simple interception patterns for calls that result in a thrown exception. In the most basic case, a 500 HTTP status code can be configured for any exception.

```kotlin
install(StatusPages){
    exception<Throwable> { cause ->
        call.respond(HttpStatusCode.InternalServerError)
    }
}
```

More specific responses can allow for more complex user interactions.

```kotlin
install(StatusPages){
    exception<AuthenticationException> { cause ->
        call.respond(HttpStatusCode.Unauthorized)
    }
    exception<AuthorizationException> { cause ->
        call.respond(HttpStatusCode.Forbidden)
    }
```

These customizations can work well when paired with custom status code responses, e.g. providing a login page when a user has not authenticated.

Each call is only caught by a single exception handler, the closest exception up the object graph from the thrown exception. When multiple exceptions within the same object hierarchy are handled, only a single one will be executed.

```kotlin
install(StatusPages) {
    exception<IllegalStateException> { cause ->
        fail("will not reach here")
    }
    exception<ClosedFileSystemException> {
        throw IllegalStateException()
    }
}
intercept(ApplicationCallPipeline.Fallback) {
    throw ClosedFileSystemException()
}
```

Single handling also implies that recursive call stacks are avoided. For example, this configuration would result in the created `IllegalStateException` propogating to the client.

```kotlin
install(StatusPages) {
    exception<IllegalStateException> { cause ->
        throw IllegalStateException("")
    }
}
```


### Status 

The `status` configuration provides a custom actions for status responses from within the application. Below is a basic configuration that provides information about the http status code within the response text.

```kotlin
install(StatusPages) {
    status(HttpStatusCode.NotFound) {
        call.respond(TextContent("${it.value} ${it.description}", ContentType.Text.Plain.withCharset(Charsets.UTF_8), it))
    }
}
```

### StatusFile 

While the `status` configuration provides customizable actions on the response object, the more common solution is to provide an error HTML page that visitors will see on an error or authorization failure. The `statusFile` configuration provides that type of functionality.

```kotlin
install(StatusPages) {
    statusFile(HttpStatusCode.NotFound, HttpStatusCode.Unauthorized, filePattern = "error#.html")
}
```

This will resolve two resources from the classpath.

1. On a 404, it will return error404.html.
2. On a 401, it will return error401.html.

The `statusFile` configuration replaces any `#` character with the value of the status code within the list of configured statuses.
