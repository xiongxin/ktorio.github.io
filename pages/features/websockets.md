---
title: WebSockets
caption: WebSockets
section: Features
permalink: /features/websockets.html
feature:
    artifact: io.ktor:ktor-websockets:$ktor_version
    class: io.ktor.websocket.WebSockets
---

This feature adds support for WebSockets to Ktor.
WebSockets are a mechanism to keep a bidirectional real-time ordered connection between
the server and the client.
Each message of this channel is called Frame: a frame can be a text or binary message,
or a close or ping/pong message. Frames can be marked as incomplete or final.

{% include feature.html %}

**Table of contents:**

* TOC
{:toc}

## Installing
{: #installing}

In order to use the WebSockets functionality you first have to install it: 

```kotlin
install(WebSockets)
```

You can adjust a few parameters when installing if required:

```kotlin
install(WebSockets) {
    pingPeriod = Duration.ofSeconds(60) // Disabled (null) by default
    timeout = Duration.ofSeconds(15)
    maxFrameSize = Long.MAX_VALUE // Disabled (max value). The connection will be closed if surpassed this length. 
    masking = false
}
```

## Usage
{: #usage}

Once installed, you can define `webSocket` routes for the [routing](/features/routing.html) feature:

Instead of short-lived normal route handlers, webSocket handlers are meant to be long-lived.
And all the relevant WebSocket methods are suspend, so the function will be suspended in
a non-blocking way while receiving or sending messages.

`webSocket` methods receive a callback with a [WebSocketSession](#WebSocketSession)
instance as the receiver. That interface defines an `incoming` (ReceiveChannel) property and an `outgoing` (SendChannel)
property, as well as a `close` method. Check the full [WebSocketSession](#WebSocketSession) for more information.

### Usage as an suspend actor
{: #actor}

```kotlin
routing {
    webSocket("/") { // websocketSession
        while (true) {
            val frame = incoming.receive()
            when (frame) {
                is Frame.Text -> {
                    val text = frame.readText()
                    outgoing.send(Frame.Text("YOU SAID: $text"))
                    if (text.equals("bye", ignoreCase = true)) {
                        close(CloseReason(CloseReason.Codes.NORMAL, "Client said BYE"))
                    }
                }
            }
        }
    }
}
```

An exception will be thrown while receiving a Frame if the client closes the connection
explicitly or the TCP socket is closed. So even with a `while (true)` loop, this shouldn't be
a leak.
{: .note}

### Usage as a Channel
{: #channel}

Since the `incoming` property is a ReceiveChannel, you can use it with its stream-like interface:

```kotlin
routing {
    webSocket("/") { // websocketSession
        incoming.mapNotNull { it as? Frame.Text }.consumeEach { frame ->
            val text = frame.readText()
            outgoing.send(Frame.Text("YOU SAID $text"))
            if (text.equals("bye", ignoreCase = true)) {
                close(CloseReason(CloseReason.Codes.NORMAL, "Client said BYE"))
            }
        }
    }
}
``` 

## Interface
{: #interface}

### The WebSocketSession interface
{: #WebSocketSession}

You receive a WebSocketSession as the receiver (this), giving you direct access
to these members inside your webSocket handler.

```kotlin
interface WebSocketSession {
    // Basic interface
    val incoming: ReceiveChannel<Frame> // Incoming frames channel
    val outgoing: SendChannel<Frame> // Outgoing frames channel
    fun close(reason: CloseReason)

    // Convenience method equivalent to `outgoing.send(frame)`
    suspend fun send(frame: Frame) // Enqueue frame, may suspend if outgoing queue is full. May throw an exception if outgoing channel is already closed so it is impossible to transfer any message.

    // The call and the context
    val call: ApplicationCall
    val application: Application

    // Modifiable properties for this request. Their initial value comes from the feature configuration.
    var pingInterval: Duration?
    var timeout: Duration
    var masking: Boolean // Enable or disable masking output messages by a random xor mask.
    var maxFrameSize: Long // Specifies frame size limit. Connection will be closed if violated
    
    // Advanced
    val closeReason: Deferred<CloseReason?>
    suspend fun flush() // Flush all outstanding messages and suspend until all earlier sent messages will be written. Could be called at any time even after close. May return immediately if connection is already terminated.
    fun terminate() // Initiate connection termination immediately. Termination may complete asynchronously.
}
```
{: .compact }

If you need information about the connection. For example the client ip, you have access
to the call property. So you can do things like `call.request.origin.host` inside
your websocket block.
{: .note}

### The Frame interface
{: #Frame}

A frame is each packet sent and received at the WebSocket protocol level.
There are two message types: TEXT and BINARY. And three control packets: CLOSE, PING and PONG.
Each packet has a payload `buffer`. And for Text or Close messages, you can
call the `readText` or `readReason` to interpret that buffer.

```kotlin
enum class FrameType { TEXT, BINARY, CLOSE, PING, PONG }
```

```kotlin
sealed class Frame {
    val fin: Boolean // Is this frame a final frame?
    val frameType: FrameType // The Type of the frame
    val buffer: ByteBuffer // Payload
    val disposableHandle: DisposableHandle

    class Binary : Frame
    class Text : Frame {
        fun readText(): String
    }
    class Close : Frame {
        fun readReason(): CloseReason?
    }
    class Ping : Frame
    class Pong : Frame
}
```

## Testing
{: #testing}

You can test WebSocket conversations by using the `handleWebSocketConversation`
method inside a `withTestApplication` block.

```kotlin
@Test
fun testConversation() {
    withTestApplication {
        application.install(WebSockets)

        val received = arrayListOf<String>()
        application.routing {
            webSocket("/echo") {
                try {
                    while (true) {
                        val text = (incoming.receive() as Frame.Text).readText()
                        received += text
                        outgoing.send(Frame.Text(text))
                    }
                } catch (e: ClosedReceiveChannelException) {
                    // Do nothing!
                } catch (e: Throwable) {
                    e.printStackTrace()
                }
            }
        }

        handleWebSocketConversation("/echo") { incoming, outgoing ->
            val textMessages = listOf("HELLO", "WORLD")
            for (msg in textMessages) {
                outgoing.send(Frame.Text(msg))
                assertEquals(msg, (incoming.receive() as Frame.Text).readText())
            }
            assertEquals(textMessages, received)
        }
    }
}
```
{: .compact}
