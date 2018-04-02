---
title: Static Content
caption: Serving Static Content   
section: Features
permalink: /features/static-content.html
feature:
    artifact: io.ktor:ktor-server-core:$ktor_version
    class: io.ktor.routing.Routing
---

Ktor has built-in support for serving static content. This can come in useful when wanting to serve style sheets, scripts, images, etc. 

{% include feature.html %}

### Specifying Files and Folders

Using the `static` function we can tell Ktor that we want certain URIs to be treated as static contents and also define where the contents resides. All content is relative to the current working directory. 
See [Defining a custom root folder](#defining-a-custom-root-folder) to set a different root. 
      
```kotlin
    routing {
        static("static") {
            files("css") 
        }
    }
```

The example above tells `ktor` that any request to the URI `/static` is to be treated as static content. The `files("css")` defines the folder under which these files
 are located - anything that is in the folder `css` will be served. In essence this means that a request such as
 

**/static/styles.css** will serve the file **css/styles.css**. 

In addition to folders, we can also include specific files using `file`, which can optionally take a second parameter that maps to the actual physical filename, if different.

 
```kotlin
    routing {
        static("static") {
            files("css")
            files("js")
            file("image.png")
            file("random.txt", "image.png")
            default("index.html")
        }
    }
```

We can also have default files that can be served using `default`. For instance, on calling

**/static** with not filename,  **index.html** will be served.

### Defining a custom root folder

To specify a different root folder, other than the working directory, we set the value of `staticRootFolder` which expect a type `File`.

```kotlin
    static("custom") {
        staticRootFolder = File("/system/folder/docs")
        files("public")
    }
```

### Defining subroutes

We can also define sub-routes, i.e. `/static/themes` for instance

```kotlin
    static("static") {
        files("css")
        static("themes") {
            files("data")
        }
    }
```

### Serving embedded resources

If you embed your static content as resources into your application, you can serve them right from there using `resource` and `resources` 
functions:

```kotlin
    static("static") {
        resources("css")
        resource("favicon.ico")
    }
```

There is also `defaultResource` similar to `default` for serving default page for a folder, 
and `staticBasePackage` similar to `staticRootFolder` for specifying base resource package for static content. 

### Handling Errors

If the requested content is not found, a `FileNotFoundException` is thrown. It should be handled in `StatusPages` with the `exception` handler 
to produce a corresponding `404 Not Found`, otherwise it propagates to the engine and causes 500 Internal Server Error. 

### Customising Content Types

When files are served, the content type is determined from the file extension, using `ContentType.defaultForFile(file)`. The data corresponding
to each file type is obtained from the `mimelist.csv` resource file which is located in `ktor-server-core`. 

### Under the covers

The function `static` is defined as

```kotlin
    fun Route.static(remotePath: String, configure: Route.() -> Unit) = route(remotePath, configure)
````

which essentially is just another route definition. 


