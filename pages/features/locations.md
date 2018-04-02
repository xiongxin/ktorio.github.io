---
title: Locations
caption: Type-safe Routing   
section: Features
permalink: /features/locations.html
feature:
    artifact: io.ktor:ktor-locations:$ktor_version
    class: io.ktor.locations.Locations
---

{::options toc_levels="1..2" /}

Ktor provides a mechanism to create routes in a typed way, for both:
constructing urls and reading the parameters.

Locations are an experimental feature.
{: .note.experimental}

**Table of contents:**

* TOC
{:toc}

{% include feature.html %}

## Installing the feature
{: #installing }

The Locations feature doesn't require any special configuration:

```kotlin
install(Locations)
```

## Defining route classes
{: #route-classes }

For each typed route you want to handle, you need to create a class (usually a data class)
containing the parameters that you want to handle.

The parameters must be of any type supported by the [Data Conversion](/features/data-conversion.html) feature.
By default you can use `Int`, `Long`, `Float`, `Double`, `Boolean`, `String`, enums and `Iterable` as parameters.

### URL parameters
{: #parameters-url }

That class must be annotated with `@Location` specifying
a path to match with placeholders between curly brackets `{` and `}`. For example: `{propertyName}`.
The names between the curly braces must match the properties of the class.

```kotlin
@Location("/list/{name}/page/{page}")
data class Listing(val name: String, val page: Int)
```

* Will match: `/list/movies/page/10`
* Will construct: `Listing(name = "movies", page = 10)`

### GET parameters
{: #parameters-get }

If you provide additional class properties that are not part of the path of the `@Location`,
those parameters will be obtained from the GET's query string or POST parameters:

```kotlin
@Location("/list/{name}")
data class Listing(val name: String, val page: Int, val count: Int)
```

* Will match: `/list/movies?page=10&count=20`
* Will construct: `Listing(name = "movies", page = 10, count = 20)`

## Defining route handlers
{: #route-handlers }

Once you have [defined the classes](#route-classes) annotated with `@Location`,
this feature artifact exposes new typed methods for defining route handlers:
`get`, `options`, `header`, `post`, `put`, `delete` and `patch`.

```kotlin
routing {
    get<Listing> { listing ->
        call.respondText("Listing ${listing.name}, page ${listing.page}")
    }
}
```

Some of these generic methods with one type parameter, defined in the `io.ktor.locations`, have the same name as other methods defined in the `io.ktor.routing` package. If you import the routing package before the locations one, the IDE might suggest you to generalize those methods instead of importing the right package. You can manually add `import io.ktor.locations.*` if that happens to you.
Remember this API is experimental. This issue is already [reported at github](https://github.com/ktorio/ktor/issues/368).
{: .note}


## Building URLs
{: #building-urls }

You can construct URLs to your routes by calling `application.locations.href` with
an instance of a class annotated with `@Location`:

```kotlin
val path = application.locations.href(Listing(name = "movies", page = 10, count = 20))
```

So for this class, `path` would be `"/list/movies?page=10&count=20""`.

```kotlin
@Location("/list/{name}") data class Listing(val name: String, val page: Int, val count: Int)
```

If you construct the URLS like this, and you decide to change the format of the URL,
you will just have to update the `@Location` path, which is really convenient.

## Subroutes with parameters
{: #subroutes }

The Location feature also exposes a `location` method to define typed subroutes:

```kotlin
routing {
    location<Type> { // /type/{name}
        get<TypeEdit> { typeEdit -> // /edit
            // ...
        }
        get<TypeList> { typeList -> // /list/{page}
            // ...
        }
    }
}
```
 
To obtain parameters defined in the superior locations, you just have to include
those property names in your classes for the internal routes. For example:

```kotlin
@Location("/type/{name}") data class Type(val name: String)
// In these classes we have to include the `name` property matching the parent.
@Location("/edit") data class TypeEdit(val name: String)
@Location("/list/{page}") data class TypeEdit(val name: String, val page: Int)
```

You can also express the same without sublocations, like this:

```kotlin
@Location("/type/{name}/edit") data class TypeEdit(val name: String)
@Location("/type/{name}/list/{page}") data class TypeEdit(val name: String, val page: Int)

routing {
    get<TypeEdit> { typeEdit -> // /type/{name}/edit
        // ...
    }
    get<TypeList> { typeList -> // /type/{name}/list/{page}
        // ...
    }
}
```
