# 🔌 Spring WebSocket — Concept Reference Guide

> Covers the three pillars of Spring WebSocket support: the raw **WebSocket API**, the **SockJS Fallback** layer, and the **STOMP** messaging sub-protocol with all its nested topics.
> Focus: concepts, use cases, annotations, and when things matter in practice.

---

## Table of Contents

- [1. WebSocket API](#1-websocket-api)
- [2. SockJS Fallback](#2-sockjs-fallback)
- [3. STOMP](#3-stomp)
    - [3.1 Overview](#31-overview)
    - [3.2 Benefits](#32-benefits)
    - [3.3 Enable STOMP](#33-enable-stomp)
    - [3.4 WebSocket Transport](#34-websocket-transport)
    - [3.5 Flow of Messages](#35-flow-of-messages)
    - [3.6 Annotated Controllers](#36-annotated-controllers)
    - [3.7 Sending Messages](#37-sending-messages)
    - [3.8 Simple Broker](#38-simple-broker)
    - [3.9 External Broker](#39-external-broker)
    - [3.10 Connecting to a Broker](#310-connecting-to-a-broker)
    - [3.11 Authentication](#311-authentication)
    - [3.12 Token Authentication](#312-token-authentication)
    - [3.13 Authorization](#313-authorization)
    - [3.14 User Destinations](#314-user-destinations)
    - [3.15 Events](#315-events)
    - [3.16 Interception](#316-interception)
    - [3.17 WebSocket Scope](#317-websocket-scope)
    - [3.18 Performance](#318-performance)

---

## 1. WebSocket API

The low-level Spring API for working directly with WebSocket connections — handlers, sessions, handshake, and server configuration.

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **WebSocket Protocol (RFC 6455)** | A standardized, full-duplex TCP communication channel initiated via an HTTP `Upgrade` request. Once the handshake completes (server responds `101 Switching Protocols`), the connection stays open for bidirectional messaging over a single TCP connection. | Any real-time feature: live dashboards, collaborative editing, multiplayer games, stock tickers, chat. | N/A — protocol-level concept | Unlike HTTP (request → response), WebSocket keeps one long-lived connection open. Works on ports 80 and 443, so firewalls generally don't block it. |
| **WebSocketHandler** | The core interface you implement to handle WebSocket messages on the server side. Spring provides two convenience base classes: `TextWebSocketHandler` (for text frames) and `BinaryWebSocketHandler` (for binary frames). You override lifecycle methods to react to connection open, messages, errors, and close. | Building custom WebSocket logic without STOMP — raw message exchange, proxies, lightweight protocols. | `WebSocketHandler`, `TextWebSocketHandler`, `BinaryWebSocketHandler` — introduced in **Spring 4.0 (2013)** | If you need STOMP, you typically never touch `WebSocketHandler` directly. Use it only when you need full control of the raw wire format. |
| **WebSocketSession** | Represents an active WebSocket connection. Injected into your handler methods. Used to send messages back to the client, query connection metadata (URI, headers, remote address), and close the connection programmatically. | Sending targeted messages to a specific connected client, inspecting connection attributes, managing session lifetime. | `WebSocketSession` interface — available in every handler callback | The underlying JSR-356 `WebSocketSession` is not thread-safe for concurrent sends. Wrap it with `ConcurrentWebSocketSessionDecorator` if multiple threads may write simultaneously. |
| **@EnableWebSocket** | Configuration annotation that activates Spring's WebSocket support on a `@Configuration` class that implements `WebSocketConfigurer`. Without this, no WebSocket infrastructure is registered. | Bootstrapping WebSocket support in any Spring MVC or standalone Spring application. | `@EnableWebSocket` — **Spring 4.0** | Pair with `WebSocketConfigurer.registerWebSocketHandlers()` to map handlers to URL paths. |
| **WebSocketConfigurer** | Interface your `@Configuration` class implements to register `WebSocketHandler`s at specific URL paths, add handshake interceptors, and configure allowed origins — all in a type-safe Java config style. | Defining which handler handles which URL, applying interceptors, restricting CORS origins. | `WebSocketConfigurer`, `WebSocketHandlerRegistry` — **Spring 4.0** | Equivalent XML support exists via the `<websocket:handlers>` namespace. |
| **WebSocket Handshake** | The one-time HTTP negotiation that upgrades the connection from HTTP to WebSocket. Spring models this through `HandshakeInterceptor` (intercept before/after) and `DefaultHandshakeHandler` (customize the upgrade logic — validate origin, negotiate sub-protocol, etc.). | Injecting authentication info or HTTP session attributes into the WebSocket session before it opens; blocking handshakes from unauthorized origins. | `HandshakeInterceptor`, `DefaultHandshakeHandler`, `HttpSessionHandshakeInterceptor` — **Spring 4.0** | `HttpSessionHandshakeInterceptor` is a convenient built-in that copies HTTP session attributes into the WebSocket session's attribute map — useful for passing user identity. |
| **Allowed Origins** | Spring enforces same-origin policy for WebSocket connections by default (since Spring 4.1.5). You can configure a specific list of allowed origins or use `*` to allow all. When SockJS is enabled, the behavior also controls iframe transport and `X-Frame-Options` headers. | Securing WebSocket endpoints from cross-origin misuse in browser clients; enabling cross-domain connections in trusted scenarios. | `registry.addHandler(...).setAllowedOrigins(...)` — **Spring 4.1.5** | Same-origin is the default and most secure. Opening to `*` enables all transports but removes origin protection entirely. Production systems should list explicit domains. |
| **Configuring the Server** | Tuning the underlying WebSocket server container — max message buffer sizes, idle timeouts, max sessions. Done through `ServletServerContainerFactoryBean` for Jakarta WebSocket servers, or a configurer callback for Jetty. | Large payloads (raise buffer size), long-idle connections (tune timeout), high-concurrency deployments. | `ServletServerContainerFactoryBean` — container-level config, not a Spring annotation | Default buffer sizes are often too small for real-world payloads. For STOMP usage, also configure STOMP transport properties separately. |
| **Deployment & JSR-356** | The Jakarta WebSocket API (JSR-356) introduces two deployment mechanisms that can conflict with Spring MVC's `DispatcherServlet`. Spring works around this with a `RequestUpgradeStrategy` so both WebSocket and regular HTTP requests can flow through one front controller. | Any production deployment on Tomcat, Jetty, Undertow, or other Jakarta EE containers. | `RequestUpgradeStrategy`, `WebSocketHttpRequestHandler` | JSR-356 SCI scanning at startup can slow boot times significantly. Use `<absolute-ordering/>` in `web.xml` to control which web fragments get scanned. |

---

## 2. SockJS Fallback

A browser-compatibility layer that emulates WebSocket behavior using alternative HTTP transports when WebSocket is blocked by proxies, firewalls, or old browsers.

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Why SockJS Exists** | WebSocket connections are sometimes blocked by restrictive HTTP proxies that don't understand the `Upgrade` header, or by corporate firewalls that close long-lived idle connections. SockJS provides a transparent fallback so the application code doesn't change — the transport layer degrades gracefully. | Public-facing apps where you can't control the network environment; enterprise apps behind corporate proxies; supporting legacy browsers. | Conceptual — no annotation needed | SockJS is a client+server protocol. You need the SockJS JavaScript client on the browser side and the Spring SockJS server support on the backend. |
| **SockJS Transports** | SockJS tries transports in order of preference: WebSocket first, then HTTP Streaming (server sends a continuous chunked response), then HTTP Long Polling (client polls and server holds the response until data is available). The client and server negotiate the best available option. | Gradual degradation: WebSocket for modern environments, streaming for older proxies, polling as last resort. | Transport negotiation is automatic — no annotation | Each transport has different latency, overhead, and proxy compatibility. WebSocket is always preferred. Long polling works everywhere but adds overhead. |
| **Enabling SockJS in Spring** | SockJS support is enabled by chaining `.withSockJS()` on a handler registration. Spring then automatically exposes the SockJS protocol endpoints (info endpoint, transport endpoints) under the registered path. | Adding fallback support to any existing WebSocket handler without changing your handler code. | `.withSockJS()` on `WebSocketHandlerRegistry` — **Spring 4.0** | The SockJS info endpoint (`/info`) is always a plain HTTP GET that SockJS clients use to detect server capabilities before choosing a transport. |
| **SockJS Client (Server-Side)** | Spring also provides a `SockJsClient` — a Java SockJS client that lets your server-side code connect to a SockJS server endpoint, useful for server-to-server communication or testing. Uses the same fallback transports as the browser client. | Integration testing of WebSocket/SockJS endpoints; server-to-server real-time communication. | `SockJsClient`, `WebSocketTransport`, `RestTemplateXhrTransport` — **Spring 4.1** | The server-side SockJS client is distinct from the JavaScript browser client. It supports both WebSocket and XHR (HTTP) transport strategies. |
| **Heartbeats** | SockJS has built-in heartbeat messages — the server sends periodic heartbeat frames to keep the connection alive and prevent proxies from timing out idle connections. Configurable on the server side. | Long-lived connections that may appear idle (e.g., a user on a page waiting for notifications). | Configured via `SockJsServiceRegistration.setHeartbeatTime()` | Default heartbeat interval is 25 seconds. If your proxy has a 30-second idle timeout, the heartbeat keeps the connection alive. Set it lower than the proxy's timeout. |
| **Session Cookies & CORS** | SockJS depends on cookies for session continuity across HTTP fallback requests, which creates cross-origin complexity. Spring handles the `CORS` headers automatically for SockJS endpoints, but the interaction with `withCredentials` on the client side requires careful origin configuration. | Browser apps using cookie-based authentication over SockJS fallback transports. | Configured via `setAllowedOrigins()` when SockJS is enabled | When SockJS is enabled, iframe-based transports are disabled if you restrict origins to a list (not `*`). This means IE6–IE9 are unsupported in that mode. |

---

## 3. STOMP

STOMP (Simple Text-Oriented Messaging Protocol) is a sub-protocol layered on top of WebSocket. It defines a message format, commands (SEND, SUBSCRIBE, UNSUBSCRIBE, etc.), and a destination system — turning raw WebSocket into a structured messaging channel with pub/sub semantics.

---

### 3.1 Overview

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **STOMP Protocol** | A simple, frame-based messaging protocol inspired by HTTP. A STOMP frame has a command (e.g., `SEND`, `SUBSCRIBE`, `MESSAGE`), headers (key-value pairs), and an optional body. Unlike raw WebSocket, STOMP defines *what* messages mean, enabling routing and pub/sub on the server. | Any application needing structured messaging over WebSocket — chat apps, collaborative tools, notification systems, live feeds. | Protocol-level concept — no Spring annotation | STOMP was originally designed for scripting languages to connect to message brokers like ActiveMQ. Spring adopted it as the high-level WebSocket messaging protocol of choice. |
| **STOMP Destinations** | A destination string in a STOMP message tells the broker or server where to route it. Spring uses destination prefixes to distinguish between application-bound messages (e.g., `/app/...`) and broker-bound messages (e.g., `/topic/...`, `/queue/...`). | Routing messages to controller methods vs. broadcasting to subscribers; targeting specific topics or individual user queues. | Configured via `configureMessageBroker()` in `WebSocketMessageBrokerConfigurer` | `/topic` destinations broadcast to all subscribers (pub/sub). `/queue` destinations are typically for point-to-point delivery (one recipient). Naming is conventional — the broker enforces the semantics. |

---

### 3.2 Benefits

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Why STOMP over Raw WebSocket** | Raw WebSocket gives you a byte pipe with no routing, no pub/sub, no message format, and no acknowledgment. STOMP adds all of these on top, and Spring's support integrates STOMP with Spring Security, Spring MVC-style controllers, and external brokers — giving you a structured application layer. | Production messaging features: user-specific messages, topic subscriptions, message filtering, security integration. | N/A — design rationale | Using raw `WebSocketHandler` is fine for simple custom protocols. For anything resembling pub/sub or routing, STOMP saves enormous amounts of boilerplate. |
| **Interoperability with Brokers** | Because STOMP is a standard protocol, you can replace Spring's in-memory broker with a full-featured external message broker (RabbitMQ, ActiveMQ) without changing your application code — just re-point the relay configuration. | Scaling beyond a single server; persistent message queues; durable subscriptions; message replay. | `StompBrokerRelayMessageHandler` | This is one of STOMP's biggest advantages — you get broker features (persistence, clustering, TTL) for free by swapping the broker. |

---

### 3.3 Enable STOMP

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **@EnableWebSocketMessageBroker** | The master switch for STOMP-over-WebSocket in Spring. Applied to a `@Configuration` class that implements `WebSocketMessageBrokerConfigurer`. It registers all the necessary infrastructure — message broker, channel interceptors, annotation-driven controller support — in one step. | Any Spring application that wants STOMP messaging over WebSocket. | `@EnableWebSocketMessageBroker` — **Spring 4.0** | Once you use STOMP, you don't need `@EnableWebSocket` separately. `@EnableWebSocketMessageBroker` includes the WebSocket infrastructure. |
| **WebSocketMessageBrokerConfigurer** | The configuration interface you implement to customize the STOMP setup: defining application destination prefixes, configuring the message broker type (in-memory or relay), registering STOMP endpoints with optional SockJS fallback. | Full control over STOMP endpoint URLs, destination prefixes, and broker type. | `WebSocketMessageBrokerConfigurer.registerStompEndpoints()`, `.configureMessageBroker()` | The two key methods are `registerStompEndpoints()` (maps the WebSocket/SockJS URL) and `configureMessageBroker()` (defines prefixes and broker config). |
| **STOMP Endpoint Registration** | Registering a STOMP endpoint tells Spring which URL WebSocket clients connect to in order to initiate a STOMP session. You can chain `.withSockJS()` for fallback support. | Defining the WebSocket connection URL that the JavaScript STOMP client connects to. | `registry.addEndpoint("/ws")` in `registerStompEndpoints()` | One application can have multiple STOMP endpoints — useful for separating public vs. authenticated connection points. |

---

### 3.4 WebSocket Transport

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **STOMP Transport Configuration** | Fine-tuning the WebSocket transport layer specifically for STOMP: message size limits, send buffer limits, send timeouts per session. Separate from the container-level WebSocket configuration, these settings apply to STOMP message channels. | Preventing slow or unresponsive clients from consuming server resources; setting limits on large STOMP message payloads. | `configureWebSocketTransport(WebSocketTransportRegistration)` in `WebSocketMessageBrokerConfigurer` — **Spring 4.0.3** | `setSendTimeLimit()` and `setSendBufferSizeLimit()` prevent a slow client from blocking the server's send queue indefinitely. Important for production stability. |

---

### 3.5 Flow of Messages

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Message Channel Architecture** | Spring STOMP uses an internal channel pipeline. Incoming messages from clients flow through the `clientInboundChannel`, are dispatched to annotated controllers or the broker, and outgoing messages flow through the `clientOutboundChannel` back to clients. The `brokerChannel` handles messages going from the application to the broker. | Understanding how messages move through the system; knowing where to apply interceptors; debugging message routing issues. | `clientInboundChannel`, `clientOutboundChannel`, `brokerChannel` — all `MessageChannel` implementations | All three channels are thread-pool backed. Their thread pool sizes are independently configurable — important for high-throughput applications. |
| **Message Routing** | Spring routes a STOMP SEND message based on its destination prefix. If the destination starts with the app prefix (e.g., `/app`), it goes to a `@MessageMapping` controller method. If it starts with the broker prefix (e.g., `/topic` or `/queue`), it goes directly to the broker without hitting any controller. | Separating business logic (controller methods) from pure relay messaging (direct broker traffic). | Destination prefix config in `configureMessageBroker()` | This routing split is fundamental. Many bugs come from misconfiguring prefixes — e.g., forgetting the `/app` prefix when sending to a controller method. |

---

### 3.6 Annotated Controllers

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **@Controller (STOMP context)** | STOMP controller classes are annotated with the regular Spring `@Controller`. Spring's STOMP infrastructure scans for `@MessageMapping` and `@SubscribeMapping` methods inside these classes — similar to how `@RequestMapping` works for HTTP. | Centralizing WebSocket message handling logic in a Spring-managed bean; leveraging DI, validation, and other Spring features inside STOMP handlers. | `@Controller` — works for STOMP with no change since **Spring 4.0** | No special annotation needed. Any `@Controller` bean with `@MessageMapping` methods is picked up automatically. |
| **@MessageMapping** | Maps a STOMP `SEND` message with a specific destination to a handler method. Equivalent to `@RequestMapping` but for WebSocket messages. The method can accept the message payload, session info, headers, and `Principal`. | Handling client-sent actions: submitting a chat message, broadcasting a move in a game, triggering a server operation via WebSocket. | `@MessageMapping("/chat")` — **Spring 4.0** | The destination in `@MessageMapping` is relative to the app prefix. If the prefix is `/app`, `@MessageMapping("/chat")` handles messages sent to `/app/chat`. |
| **@SubscribeMapping** | Maps a STOMP `SUBSCRIBE` frame to a handler method. The method's return value is sent directly back to the subscribing client as a response — a request-reply style over WebSocket, without going through the broker. | Fetching initial data when a client subscribes to a topic — e.g., loading the current state of a board when joining a game. | `@SubscribeMapping("/initial-data")` — **Spring 4.0** | The key difference from `@MessageMapping`: the reply goes directly to the subscriber, not broadcast through the broker. Useful for one-time subscription responses. |
| **@MessageExceptionHandler** | Handles exceptions thrown by `@MessageMapping` methods in the same controller. Similar to `@ExceptionHandler` for REST. Can return a response message sent back to the client. | Sending structured error messages back to the WebSocket client when a handler throws an exception; preventing silent failures. | `@MessageExceptionHandler` — **Spring 4.0** | Without this, exceptions in `@MessageMapping` methods are swallowed or logged but not returned to the client. Pair with `@SendToUser` to send error responses directly to the sender. |
| **@SendTo** | Specifies that the return value of a `@MessageMapping` method should be broadcast to all subscribers of a given destination (via the broker). If omitted, Spring infers the destination from the mapping. | Broadcasting a processed result to all clients subscribed to a topic after handling a client message — e.g., broadcasting an updated scoreboard after a player's move. | `@SendTo("/topic/scores")` — **Spring 4.0** | The return value is serialized (typically to JSON via Jackson) and sent as a STOMP MESSAGE frame to the specified destination. |
| **@SendToUser** | Specifies that the return value should be sent only to the user who triggered the handler method — delivered to that user's personal queue (e.g., `/user/queue/reply`). Spring resolves the user via the authenticated `Principal`. | Sending a private response back to a specific user: personal notifications, error messages, private data. | `@SendToUser("/queue/reply")` — **Spring 4.0** | Requires the user to be authenticated (Spring Security + WebSocket). Spring prepends `/user/{username}` to the destination automatically. |
| **@Payload** | Extracts and deserializes the STOMP message body into the annotated method parameter. Spring uses the message's content-type header and registered converters (typically Jackson) to deserialize. | Binding the incoming WebSocket message body to a Java object in your handler method. | `@Payload` — **Spring 4.0** | Validation via `@Validated` / `@Valid` works on `@Payload` parameters, enabling bean validation of incoming WebSocket message payloads. |
| **@Header / @Headers** | Extracts a specific STOMP header (`@Header("nativeHeader")`) or all headers (`@Headers` into a `Map`) from the incoming message. Useful for reading custom metadata sent by the client. | Reading custom STOMP headers such as auth tokens, locale, client version, or routing hints from incoming messages. | `@Header`, `@Headers` — **Spring 4.0** | `@Header` reads from Spring's internal `MessageHeaders`. For STOMP-native headers (set by the client), use `SimpMessageHeaderAccessor.getNativeHeader()`. |
| **@DestinationVariable** | Binds a template variable from the destination pattern (e.g., `/chat/{roomId}`) to a method parameter — equivalent to `@PathVariable` for WebSocket destinations. | Dynamic destinations: per-room chats, per-game sessions, per-user channels where the room or session ID is embedded in the destination string. | `@DestinationVariable("roomId")` — **Spring 4.2** | Works with `@MessageMapping("/chat/{roomId}")`. The variable is extracted from the destination string at routing time. |

---

### 3.7 Sending Messages

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **SimpMessagingTemplate** | The primary Spring class for programmatically sending STOMP messages from anywhere in your application — not just from controller return values. Inject it and call `convertAndSend()` or `convertAndSendToUser()`. | Sending server-initiated messages: push notifications, scheduled broadcasts, database change events, background job results. | `SimpMessagingTemplate` — **Spring 4.0** | This is how you push messages from a `@Service` or `@Scheduled` method without waiting for a client to send something first. The most commonly used Spring WebSocket class in practice. |
| **SimpMessageSendingOperations** | The interface that `SimpMessagingTemplate` implements. Useful for dependency injection by interface, making it easier to mock in tests. | Testable, interface-driven injection of the messaging template in services and components. | `SimpMessageSendingOperations` — **Spring 4.0** | Functionally identical to `SimpMessagingTemplate`. Prefer the interface for clean architecture and easier unit testing. |

---

### 3.8 Simple Broker

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **In-Memory Simple Broker** | Spring's built-in, lightweight message broker that runs inside the JVM. Handles STOMP SUBSCRIBE, UNSUBSCRIBE, and MESSAGE frames in memory. No external broker needed. Enabled via `enableSimpleBroker()` in the config. | Development, prototyping, small-scale deployments where a single server is sufficient and message durability is not required. | `configureMessageBroker().enableSimpleBroker("/topic", "/queue")` — **Spring 4.0** | The simple broker has no message persistence, no clustering, and no replay. If the server restarts, all subscriptions and queued messages are lost. Use external broker for production. |
| **Broker Heartbeats (Simple Broker)** | The simple broker supports configurable heartbeat intervals — the server and client negotiate STOMP-level heartbeats to detect dead connections at the STOMP layer (separate from SockJS heartbeats). | Detecting dead WebSocket connections before the TCP stack times out; cleaning up subscriptions for disconnected clients. | `enableSimpleBroker(...).setHeartbeatValue(new long[]{10000, 10000})` — **Spring 4.2** | Two values: `[server-send-interval, server-receive-interval]` in milliseconds. If a client stops sending heartbeats for longer than the receive interval, the server can consider it dead. |

---

### 3.9 External Broker

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Broker Relay** | Instead of the in-memory broker, Spring can relay STOMP messages to a full external message broker (RabbitMQ, ActiveMQ, ActiveMQ Artemis) using the STOMP protocol. Spring acts as a bridge between WebSocket clients and the external broker. | Production-scale applications; multi-server deployments; use cases requiring durable queues, message persistence, or fan-out across clustered servers. | `configureMessageBroker().enableStompBrokerRelay("/topic", "/queue")` — **Spring 4.0** | With the broker relay, Spring does not process subscriptions itself — they go directly to the external broker. The broker handles delivery guarantees, persistence, and clustering. |
| **StompBrokerRelayMessageHandler** | The internal Spring component that manages the TCP connection(s) to the external STOMP broker. Maintains a system connection (for server-initiated sends) and per-client relay connections (one per WebSocket session). | Understanding what happens under the hood when using an external broker; diagnosing connection issues. | `StompBrokerRelayMessageHandler` — **Spring 4.0**, uses Reactor Netty (since Spring 5) for the TCP connection | Spring 5 switched the TCP relay from the older `Reactor 1.x` TCP client to `Reactor Netty`. If you're on Spring 5+ with an external broker, `Reactor Netty` must be on the classpath. |

---

### 3.10 Connecting to a Broker

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **System & Client Relay Connections** | The broker relay maintains two types of connections to the external broker: a single **system connection** used by the application to send server-initiated messages, and individual **client relay connections** — one per connected WebSocket session — that proxy the client's STOMP session to the broker. | Scaling message delivery through an external broker; ensuring server-sent messages (via `SimpMessagingTemplate`) go through the broker for proper fan-out. | Configured via `setRelayHost()`, `setRelayPort()`, `setSystemLogin()`, `setSystemPasscode()` on the broker relay config | System connection login/passcode can differ from client credentials. The system connection is always active even when no clients are connected, enabling server-initiated messages at any time. |
| **Virtual Host** | Some message brokers (particularly RabbitMQ) use virtual hosts to separate environments or tenants. Spring's broker relay supports setting a virtual host in the CONNECT frame sent to the broker. | Multi-tenant apps using a shared RabbitMQ instance with separate vhosts per tenant or environment. | `setVirtualHost("myVHost")` on broker relay config | If not set, the broker's default virtual host is used. For RabbitMQ, this is typically `/`. |

---

### 3.11 Authentication

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **WebSocket Authentication** | Spring Security integrates with Spring WebSocket to authenticate users during the WebSocket handshake using the existing HTTP session's security context. The authenticated `Principal` is then available in all `@MessageMapping` methods and stored in the STOMP session. | Any application with user authentication — the WebSocket session inherits the user's authenticated identity from the HTTP session. | Spring Security's `AbstractSecurityWebSocketMessageBrokerConfigurer` — **Spring Security 4.0** | Authentication happens at the HTTP handshake level, not at the STOMP CONNECT frame level. This means your existing Spring Security HTTP configuration (sessions, tokens) naturally extends to WebSocket. |
| **STOMP CONNECT Authentication** | In some configurations, you may want to authenticate users via the STOMP CONNECT frame (passing login/passcode headers) rather than relying on the HTTP session. Spring provides a `ChannelInterceptor`-based approach to validate these credentials at the STOMP protocol level. | Applications using stateless authentication (e.g., mobile clients that don't have HTTP sessions) where credentials are passed per-connection in the STOMP CONNECT frame. | Custom `ChannelInterceptor` on `clientInboundChannel` | CONNECT-frame authentication is more complex to implement than HTTP-session-based auth. Prefer HTTP-session-based authentication when possible. |

---

### 3.12 Token Authentication

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **JWT / Token-Based WebSocket Auth** | WebSocket clients that use JWT or other tokens (not HTTP session cookies) need a way to pass the token during connection. Since WebSocket doesn't support custom HTTP headers after the handshake, the typical approach is to pass the token as a query parameter in the WebSocket URL, then validate it in a `HandshakeInterceptor` before the connection is established. | SPAs using JWT for REST APIs that also need WebSocket; mobile clients; any stateless authentication scenario. | `HandshakeInterceptor` + Spring Security's `UsernamePasswordAuthenticationToken` or similar | Passing tokens in query strings has security implications (URL logging). An alternative is passing the token in the STOMP CONNECT frame headers and validating via a channel interceptor. |

---

### 3.13 Authorization

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Message-Level Authorization** | Spring Security can authorize individual STOMP messages based on the destination, the message type (SEND vs SUBSCRIBE), and the user's roles. This is configured via `AbstractSecurityWebSocketMessageBrokerConfigurer.configureInbound()`. | Restricting which users can send to certain topics; preventing unauthorized subscriptions to private queues; role-based access to WebSocket destinations. | `AbstractSecurityWebSocketMessageBrokerConfigurer`, `.configureInbound(MessageSecurityMetadataSourceRegistry)` — **Spring Security 4.0** | STOMP destinations are separate from HTTP URLs — you need WebSocket-specific security rules in addition to (or instead of) `HttpSecurity`. For example, `.simpDestMatchers("/app/admin/**").hasRole("ADMIN")`. |

---

### 3.14 User Destinations

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **User-Specific Destinations** | Spring provides the convention `/user/{username}/...` as a way to address messages to a specific authenticated user. Clients subscribe to `/user/queue/reply` and the server uses `convertAndSendToUser("username", "/queue/reply", payload)`. Spring's `UserDestinationMessageHandler` resolves the username to the actual session-specific destination. | Private notifications, direct messages, personal alerts — anything that should go to one specific user, not a broadcast. | `SimpMessagingTemplate.convertAndSendToUser()`, `@SendToUser`, `UserDestinationMessageHandler` — **Spring 4.0** | A single user may have multiple active sessions (e.g., open on multiple browser tabs). Spring resolves the user destination to all active sessions for that user by default. |
| **UserDestinationResolver** | The component responsible for translating `/user/{username}/queue/reply` into the broker-level session-specific destination. Uses `SimpUserRegistry` to look up which sessions belong to a given user. | Understanding or customizing how user-specific message routing works under the hood; integrating with custom user session tracking. | `UserDestinationResolver`, `SimpUserRegistry` — **Spring 4.0** | If you use an external broker and want user destinations to work across a cluster, you need a shared `SimpUserRegistry` (e.g., backed by a distributed data store) so all nodes know which server holds each user's sessions. |

---

### 3.15 Events

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Application Context WebSocket Events** | Spring publishes standard `ApplicationEvent`s when WebSocket sessions open, close, and when STOMP CONNECT/DISCONNECT/SUBSCRIBE frames are received. You can listen to these with `@EventListener`. | Tracking active connections, updating user presence/online status, cleaning up session data when a user disconnects, logging connection metrics. | `SessionConnectedEvent`, `SessionDisconnectEvent`, `SessionSubscribeEvent`, `SessionUnsubscribeEvent`, `BrokerAvailabilityEvent` — **Spring 4.0** | `BrokerAvailabilityEvent` is especially important when using an external broker — it fires when the relay loses and regains connection to the broker, so you can alert the application or pause sends. |

---

### 3.16 Interception

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **ChannelInterceptor** | Allows you to intercept messages on any of the three internal channels (`clientInboundChannel`, `clientOutboundChannel`, `brokerChannel`) before they are processed or sent. Intercept at send time, before/after handling, or after completion. | Authentication/authorization at the message level; logging all WebSocket messages; rate limiting; modifying message headers before delivery. | `ChannelInterceptor`, `ExecutorChannelInterceptor` — `configureClientInboundChannel().addInterceptors(...)` — **Spring 4.0** | `ExecutorChannelInterceptor` extends `ChannelInterceptor` with `beforeHandle` and `afterMessageHandled` callbacks that run in the executor thread — useful for thread-local setup (e.g., logging MDC context). |

---

### 3.17 WebSocket Scope

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **WebSocket-Scoped Beans** | Spring introduces a `websocket` bean scope — a bean is created once per WebSocket session and lives for the duration of that session. Similar to `request` and `session` scope in HTTP, but tied to a WebSocket/STOMP session lifecycle. | Storing per-session state in a Spring-managed bean: user preferences for the session, accumulated game state, shopping cart tied to a live session. | `@Scope("websocket")` or `@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)` — **Spring 4.0** | Requires `@EnableWebSocketMessageBroker`. WebSocket-scoped beans must be injected via scoped proxy (use `proxyMode = TARGET_CLASS`) when injected into singleton beans. |

---

### 3.18 Performance

| Concept | Explanation | Use Cases | Annotations / Key Classes | Notes |
|---|---|---|---|---|
| **Thread Pool Tuning** | The three message channels (`clientInboundChannel`, `clientOutboundChannel`, `brokerChannel`) each have their own thread pool executor. Tuning core/max thread counts directly affects throughput and latency under load. | High-concurrency WebSocket applications; applications with slow message handlers that cause queue buildup. | `configureClientInboundChannel(ChannelRegistration)`, `configureClientOutboundChannel(ChannelRegistration)` — **Spring 4.0** | Default thread counts are based on available CPU cores — suitable for development but often too low for production. Profile under realistic load before sizing. |
| **Message Converter Configuration** | Spring uses `MessageConverter` instances (by default, Jackson's `MappingJackson2MessageConverter`) to serialize/deserialize message payloads. You can configure additional converters or customize the Jackson `ObjectMapper` via `configureMessageConverters()`. | Custom serialization requirements; supporting multiple content types (JSON and binary); adding custom Jackson modules. | `WebSocketMessageBrokerConfigurer.configureMessageConverters(List<MessageConverter>)` — **Spring 4.0** | Returning `false` from `configureMessageConverters()` replaces the defaults entirely. Returning `true` (or not overriding) adds your converters alongside the defaults. |
| **Monitoring (Metrics)** | Spring exposes internal STOMP infrastructure metrics — connected user count, subscription count, message rates, slow client counts, unresolved user destinations — accessible via `SimpUserRegistry` and internal counters. Can be exported to Micrometer/Actuator. | Production observability: detecting message queue buildup, counting active WebSocket sessions, identifying slow clients, alerting on broker relay disconnects. | `SimpUserRegistry`, Spring Boot Actuator WebSocket metrics — **Spring 4.2 / Boot 2.x** | Expose `/actuator/metrics/spring.websocket.*` in Spring Boot. For custom dashboards, inject `SimpUserRegistry` and query session/subscription counts programmatically. |

---

## Quick Annotation Cheat Sheet

| Annotation / Class | Layer | Purpose | Since |
|---|---|---|---|
| `@EnableWebSocket` | WebSocket API | Activates basic WebSocket support | Spring 4.0 |
| `@EnableWebSocketMessageBroker` | STOMP | Activates STOMP messaging infrastructure | Spring 4.0 |
| `@MessageMapping` | STOMP Controller | Maps STOMP SEND to a method | Spring 4.0 |
| `@SubscribeMapping` | STOMP Controller | Maps STOMP SUBSCRIBE to a method (reply-to-sender) | Spring 4.0 |
| `@SendTo` | STOMP Controller | Broadcasts method return value to a destination | Spring 4.0 |
| `@SendToUser` | STOMP Controller | Sends method return value to a specific user | Spring 4.0 |
| `@MessageExceptionHandler` | STOMP Controller | Handles exceptions from `@MessageMapping` methods | Spring 4.0 |
| `@Payload` | STOMP Controller | Binds message body to method parameter | Spring 4.0 |
| `@Header` / `@Headers` | STOMP Controller | Binds STOMP header(s) to method parameter | Spring 4.0 |
| `@DestinationVariable` | STOMP Controller | Binds path variable from destination pattern | Spring 4.2 |
| `SimpMessagingTemplate` | STOMP Messaging | Programmatic send from anywhere in the app | Spring 4.0 |
| `@Scope("websocket")` | Bean Lifecycle | Per-WebSocket-session scoped bean | Spring 4.0 |

---

*Source: [Spring Framework WebSocket Reference Docs](https://docs.spring.io/spring-framework/reference/web/websocket.html) — Spring Framework 7.x*
