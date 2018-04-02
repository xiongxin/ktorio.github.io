---
title: Data Conversion
caption: Data Conversion
section: Features
permalink: /features/data-conversion.html
feature:
    artifact: io.ktor:ktor-server-core:$ktor_version
    class: io.ktor.features.DataConversion
---

`DataConversion` is a feature that allows to serialize and deserialize list of values.

By default it handle primitive types and enums, but can be configured to handle additional types. 

If you are using the [Locations feature](/features/locations.html) and want to support
custom types as part of its parameters, you can add new custom converters with this
service.

**Table of contents:**

* TOC
{:toc}

{% include feature.html %}

## Basic Installation
{: #basic-installation }

Installing the DataConversion is pretty easy, and it should be cover primitive types:

```kotlin
install(DataConversion)
```

## Adding Converters
{: #adding-converters }

The DataConversion configuration, provide a `convert<T>` method to define
type conversions. Inside, you have to provide a decoder and an encoder
with the `decode` and `encode` methods accepting callbacks.

* decode callback: `converter: (values: List<String>, type: Type) -> Any?`
  Accepts `values`, a list of strings) representing repeated values in the URL, for example `a=1&a=2`,
  and accepts the `type` to convert to. It should return the decoded value.
* encode callback: `converter: (value: Any?) -> List<String>` 
  Accepts an arbitrary value, and should return a list of strings representing the value.
  When returning a list of a single element it will be serialized as `key=item1`. For multiple values
  it will be serialized in the query string as: `samekey=item1&samekey=item2`.

For example:

```kotlin
install(DataConversion) {
    convert<Date> { // this: DelegatingConversionService
        val format = SimpleDateFormat.getInstance()
    
        decode { values, _ -> // converter: (values: List<String>, type: Type) -> Any?
            values.singleOrNull()?.let { format.parse(it) }
        }

        encode { value -> // converter: (value: Any?) -> List<String>
            when (value) {
                null -> listOf()
                is Date -> listOf(SimpleDateFormat.getInstance().format(value))
                else -> throw DataConversionException("Cannot convert $value as Date")
            }
        }
    }
}
```

## Accessing the Service
{: #service }

You can access the DataConversion service, from any call easily with:

```kotlin
val dataConversion = call.conversionService
```

## The ConversionService Interface
{: #interface }

```kotlin
interface ConversionService {
    fun fromValues(values: List<String>, type: Type): Any?
    fun toValues(value: Any?): List<String>
}
```
{: #ConversionService }

```kotlin
class DelegatingConversionService(private val klass: KClass<*>) : ConversionService {
    fun decode(converter: (values: List<String>, type: Type) -> Any?)
    fun encode(converter: (value: Any?) -> List<String>)
}
```
{: #DelegatingConversionService }
