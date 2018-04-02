---
title: Auto Head Response
caption: Enable Automatic HEAD Responses
section: Features
permalink: /features/autoheadresponse.html
feature:
    artifact: io.ktor:ktor-server-core:$ktor_version
    class: io.ktor.features.AutoHeadResponse
---

Ktor can automatically provide responses to `HEAD` requests for existing routes that have the `GET` verb defined. 

{% include feature.html %}

### Usage

To enable automatic `HEAD` responses, install the `AutoHeadResponse` feature


```kotlin
fun Application.main() {
  ...
  install(HeadRequestSupport) 
  ...
}
```

### Configuration options

None.

### Under the covers

This feature automatically responds to `HEAD` requests by routing as if it were `GET` response and discarding 
the body. Since any `FinalContent` produced by the system has lazy content semantics, it does not incur in any performance
costs for processing a `GET` request with a body. 
