---
title: Partial Content
caption: Streaming Movies and Other Content   
section: Features
permalink: /features/partial-content.html
feature:
    artifact: io.ktor:ktor-server-core:$ktor_version
    class: io.ktor.features.PartialContent
---

This feature adds support for handling Partial Content requests:
requests with the `Range` header. It intercepts the generated
response adding the `Accept-Ranges` and the `Content-Range` header and slicing
the served content when required.

Partial Content is well-suited for streaming content or resume partial downloads with
download managers, or in unreliable networks.

It is specially useful along the [Static Content Feature](/features/static-content.html).

{% include feature.html %}

## Usage

To install the PartialContent feature with the default configuration:

```kotlin
import io.ktor.features.*

fun Application.main() {
    // ...
    install(PartialContent)
    // ...
}
```

## Configuration

```kotlin
install(PartialContent) {
    // Maximum number of ranges that will be accepted from HTTP request.
    // If HTTP request specifies more ranges, they will all be merged into a single range.
    maxRangeCount = 10
}
```

## Detailed description

This feature only works with *HEAD* and *GET* requests.
And it will return a `405 Method Not Allowed` if the client tries to use the `Range`
header with other methods.

It disables compression when serving ranges.

It is only enabled for responses that define the `Content-Length`. And it:

* Removes the `Content-Length` header
* Adds the `Accept-Ranges` header 
* Adds the `Content-Range` header with the requested Ranges
* Serves just the requested slice of the content

It should work with any served content of the type `OutgoingContent.ReadChannelContent`
as long as its length is defined, like for example the `LocalFileContent`.
{: .performance.note}

This HTTP mechanism for Partial Content is described in the [RFC-7233](https://tools.ietf.org/html/rfc7233#section-4.1).
{: .note}
