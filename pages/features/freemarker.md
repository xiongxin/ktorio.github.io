---
title: Freemarker
caption: Using Freemarker Templates
section: Features
permalink: /features/freemarker.html
keywords: html
feature:
    artifact: io.ktor:ktor-freemarker:$ktor_version
    class: io.ktor.freemarker.FreeMarker
---

Ktor includes support for [FreeMarker](http://freemarker.org/) templates through the FreeMarker
feature.  Initialize the FreeMarker feature with a
[TemplateLoader](http://freemarker.org/docs/pgui_config_templateloading.html):

```kotlin
    install(FreeMarker) {
        templateLoader = ClassTemplateLoader(TheApp::class.java.classLoader, "templates")
    }
```

This TemplateLoader sets up FreeMarker to look for the template files on the classpath in the
"templates" package, relative to the current class path.  A basic template looks like this:

{% include feature.html %}

```html
<html>
<h2>Hello ${user.name}!</h2>

Your email address is ${user.email}
</html>
```

With that template in `resources/templates` it is accessible elsewhere in the the application
using the `call.respond()` method:

```kotlin
    get("/{...}") {
        val user = User("user name", "user@example.com")
        call.respond(FreeMarkerContent("index.ftl", mapOf("user" to user), "e"))
    }
```