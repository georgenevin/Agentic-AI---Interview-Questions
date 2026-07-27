# Azure / .NET / C# Interview Prep Notes

---

# PART 1: Azure

## Azure Service Bus & Messaging

### Topics & Subscriptions — Error Handling

**Q: A message is published to a Service Bus topic and consumed by multiple subscribed endpoints. If one endpoint fails to process it, how do you handle or inform that subscriber?**

**The core mechanism: each subscription has its own independent Dead-Letter Queue (DLQ)**

With a Service Bus **topic**, one published message is delivered independently to every **subscription** — each subscription behaves like its own queue. If subscriber A fails to process the message but subscriber B succeeds, that's tracked completely separately; one subscription's failure doesn't affect another's.

**What happens when processing fails:**

1. When your endpoint's message handler throws an exception (or the message lock simply expires because processing took too long), Service Bus doesn't delete the message — it **abandons** it and returns it to that subscription's queue for another delivery attempt.
2. This is tracked via **`DeliveryCount`**, checked against **`MaxDeliveryCount`** (default: 10, configurable per subscription).
3. Once `DeliveryCount` exceeds `MaxDeliveryCount`, Service Bus automatically moves the message to that subscription's **dead-letter queue (DLQ)** instead of retrying forever.
4. Each subscription's DLQ is a built-in sub-queue — addressed as `<topic-name>/Subscriptions/<subscription-name>/$deadletterqueue`. It's created automatically and can't be deleted or managed independently of the parent subscription.

**How to actually get notified when this happens**

You generally don't rely on the failing endpoint to "announce" the error inline — you build a **separate watcher** on the DLQ:

- **Attach a consumer to the DLQ itself** — it's just another queue, so an Azure Function trigger, Logic App, or background service can read from it, log the `DeadLetterReason`, and then send a notification (Teams/email) or resubmit the message once the underlying issue is fixed.
- **Set up Azure Monitor alerts** on the DLQ message count metric, so the team is notified as soon as messages start dead-lettering, rather than discovering it days later.
- Checking the DLQ regularly in production is standard practice — it's one of the most reliable early warning signals that something's going wrong upstream or downstream.

**Distinguishing "retry-worthy" vs. "give up now" errors**

Don't let every failure blindly burn through the retry count:

- Some messages are **inherently unprocessable** — malformed JSON, missing required fields, a reference to data that no longer exists. Retrying won't help. Validate up front and dead-letter immediately with a descriptive reason instead of wasting delivery attempts.
- **Transient errors** (a downstream API timeout, a momentary DB hiccup) should be allowed to go through the normal retry/`MaxDeliveryCount` mechanism.
- A well-designed DLQ handler **categorizes** failures — invalid messages get discarded, transient failures get resubmitted, unknown failures get flagged for a human — and tracks resubmission counts to avoid infinite retry loops.

**One-line interview answer:** *"Service Bus retries a failing message automatically up to `MaxDeliveryCount`, then moves it to that subscription's own dead-letter queue. To 'inform' the rest of the system, you attach a separate consumer or Azure Monitor alert to the DLQ — you don't handle it inline inside the failing endpoint."*

---

### How do you know a message was NOT delivered?

**Q: How can you tell if a message is not delivered in Azure Service Bus?**

Two different cases, detected two different ways:

**1. Never reached the broker (send failure)** — `SendMessageAsync()` throws an exception immediately if the send itself fails (network issue, auth failure, quota exceeded, entity full). This one is easy: your own send-side code already knows, as long as you don't swallow the exception.

**2. Reached the broker, but no consumer ever got it** — this is the silent one:
- **No one's reading it** — check the **Active Message Count** metric. If it's high or growing, consumers aren't keeping up (too slow, disconnected, or not running).
- **It expired (TTL)** — every message has a Time-To-Live; once it passes, the message is **silently discarded by default**. It only shows up in the dead-letter queue if you've explicitly enabled **dead-lettering on message expiration** on that queue/subscription — otherwise it just vanishes with no trace at all.

**How to actually detect it in practice:**
- Monitor `ActiveMessageCount` and `DeadLetterMessageCount` via Azure Monitor metrics/alerts.
- Enable dead-lettering on message expiration, so expired-and-undelivered messages leave a record instead of disappearing.
- Service Bus gives no built-in "was this exact message consumed" API — track a `MessageId`/`CorrelationId` through your own logs at both send and receive time if you need true end-to-end confirmation.

**One-liner:** *"Send failures throw right away. Consumption failures don't — you detect them via Active Message Count, the DLQ, or a silently expired TTL, which won't hit the DLQ unless you've turned that on."*

---

### When can you say an item was NOT delivered?

**Q: At what specific point can you say an item was not delivered in Azure Service Bus?**

You can call it "not delivered" once any of these conditions is actually met — not just after a single failed attempt:

1. **`MaxDeliveryCount` is exceeded** — the message was attempted (locked, handler failed, or lock expired) more times than the configured limit (default 10) → moved to the DLQ. This is the clearest, most explicit signal.
2. **TTL expires before any consumer receives it** — the message sat past its `expires-at-utc` time without being picked up. Silently discarded by default; only becomes a visible "not delivered" event if dead-lettering-on-expiration is enabled.
3. **The receiving app explicitly dead-letters it** — code calls `DeadLetterAsync()` because the message is invalid/unprocessable (bad JSON, missing fields) — no point retrying.
4. **A subscription filter evaluation throws an error** — if dead-lettering on filter-evaluation-exceptions is enabled, a broken filter rule means the message never reaches that subscription's active queue at all.
5. **The send itself fails** — technically "never delivered to the broker" rather than "not delivered to a consumer," but still counts; `SendMessageAsync()` throws an exception at send time.

**Not yet safe to call it "not delivered":** while `DeliveryCount` is still below `MaxDeliveryCount` — a message that's failed once or twice is just mid-retry, not undelivered.

**One-liner:** *"An item is 'not delivered' once it either exhausts `MaxDeliveryCount` and lands in the DLQ, expires via TTL before being consumed, gets explicitly dead-lettered by the consumer, or fails outright at send time — not simply after one failed attempt."*

---

## Azure Cloud Services (App Service, Key Vault, Blob Storage, Bot Framework, Storage Queue)

*(Service Bus is covered in more depth in the "Azure Service Bus & Messaging" section above — a brief recap is included here for completeness alongside the other storage/messaging services.)*

### Microsoft Bot Framework — Direct Line

**Q: How does Direct Line / Bot Framework work end-to-end?**

**Direct Line** is a channel in the Bot Framework that automatically handles reconnection, token expiry, and the conversation lifecycle for you.

**Flow:**
1. The client sends a request to the **Direct Line token endpoint** and receives a token.
2. Using that token, the client calls the **conversation endpoint**, which returns a `conversationId`, a `streamUrl`, and a timestamp.
3. The client connects to the `streamUrl`, establishing a **WebSocket connection**.
4. The client sends an **Activity** — every interaction between user and bot (message, typing indicator, adaptive card, etc.) is represented as an Activity object. One send-and-receive round of an Activity is called a **turn**.
5. The Activity is forwarded to an **Adapter** class, which handles connectivity with the channel. It processes the incoming request into a C# `Activity` object, wraps it in a **`TurnContext`**, and that context flows through the entire turn.
6. The `TurnContext` passes through the **middleware pipeline**, then reaches the bot's `OnTurnAsync` method. `TurnContext` carries information about the activity — sender, recipient, channel data, etc.
7. After processing, the bot responds, typically as another Activity object.

Direct Line ensures that text, Adaptive Cards, and suggested actions are all converted into the Activity object format the SDK understands.

**Conversation state:**
- State is keyed using a combination of `channelId` and `conversation.id` (roughly `{channelId}/conversations/{conversationId}`).
- A **state accessor** is used to get/set properties on the state object, and `SaveChangesAsync()` persists those changes.

**Middleware:**
Gives you the opportunity to run logic before and after each turn of the conversation — e.g., logging, translation, or state management that should wrap every interaction.

---

### Azure App Service

**Q: What is Azure App Service?**

A PaaS (Platform-as-a-Service) offering that lets you run web apps, mobile backends, and RESTful APIs without managing the underlying infrastructure (VMs, OS patching, etc.).

- A **Standard App Service Plan** is deployed to a specific **region**.
- **Availability Zones** — Azure can spread your app instances across different zones, meaning different physically separate datacenters within the same region, for higher resilience against a single datacenter failure.

---

### Azure Key Vault

**Q: What does Azure Key Vault store?**

- **Secrets** — storing sensitive strings (connection strings, passwords, API keys).
- **Keys** — cryptographic keys used for encryption/decryption or signing.
- **Certificates** — X.509 certificates, which Key Vault can also manage the full lifecycle of (including auto-renewal).

---

### Azure Blob Storage

**Q: What is Azure Blob Storage, and what are its access tiers?**

Stores unstructured data (files, images, backups, logs, etc.).

- **Hot** — optimized for frequently accessed data; highest storage cost, lowest access cost.
- **Cool** — for infrequently accessed data, stored for at least 30 days; lower storage cost, higher access cost than Hot.
- **Cold** — for rarely accessed data, stored for at least 90 days; cheaper than Cool, still available for immediate read.
- **Archive** — offline tier, cheapest storage cost by far, but data isn't immediately readable — it must be **rehydrated** first.

**Rehydration** specifically refers to moving a blob out of the **Archive** tier back to an online tier (Hot or Cool) so it can be read again — it's not a general "active tier to online tier" move; Hot/Cool/Cold are already all online.

---

### Azure Service Bus (brief recap)

*(See the "Azure Service Bus & Messaging" section above for the full write-up on dead-lettering and delivery detection.)*

- **Enterprise-level message broker.**
- **Queue** — one sender, and exactly **one** receiver processes each message (point-to-point).
- **Topic + Subscription** — one sender, and **multiple** subscribers each receive their own independent copy (pub/sub).
- **Dead-letter queue** — holds messages that couldn't be successfully delivered/processed.
- **Duplicate detection** — an optional feature where, if a message with the same `MessageId` is sent again within the configured duplicate-detection window (e.g., due to a retried send after a network failure), Service Bus discards the duplicate copy so it isn't processed twice.
- Good fit when you need a **pub/sub model, FIFO ordering** (via sessions), or **larger message sizes** than Storage Queues support.

---

### Azure Storage Queue

**Q: What is Azure Storage Queue, and how does message visibility work?**

- Lightweight and cost-effective — a simpler, cheaper alternative to Service Bus for basic queuing needs.
- **Ordering is not guaranteed** (best-effort FIFO at most — unlike Service Bus sessions, which guarantee ordering).
- Message size limit: **64 KB**.
- **Peek-lock visibility model:**
  1. A worker "peeks" (dequeues) a message, which becomes **invisible** to other workers for a configured visibility timeout — it is *not* deleted yet.
  2. If the worker finishes successfully, it explicitly **deletes** the message.
  3. If the worker crashes or never finishes, the **visibility timeout expires**, and the message automatically reappears in the queue for another worker to pick up.

---

# PART 2: .NET / C# / ASP.NET Core

### C# Language Fundamentals

**Q: When to use `string` vs. `StringBuilder`?**

- `string` is **immutable** — every modification creates a brand-new string object in memory. Fine for a small, fixed number of operations, or when the value genuinely won't change.
- `StringBuilder` is **mutable** — it modifies an internal buffer in place instead of allocating a new object each time.
- **Rule of thumb:** use `StringBuilder` when you're doing many concatenations/modifications (e.g., building a string inside a loop). Use plain `string` when there are few or no modifications — it's simpler and avoids the overhead of `StringBuilder` for a one-off case.

**Q: Can you have just a `try` block on its own?**

No — a bare `try` with nothing else is a compile error. A `try` block must be followed by at least one `catch`, a `finally`, or both:
- `try { } catch { }` ✅
- `try { } finally { }` ✅ (valid — runs cleanup code regardless of exceptions, without handling the exception itself)
- `try { }` alone ❌ — not valid C#.

**Q: `IEnumerable` vs. `IEnumerator`?**

- **`IEnumerable`** represents something that *can be iterated*. It exposes a single method, `GetEnumerator()`, which returns an `IEnumerator`.
- **`IEnumerator`** is the actual cursor doing the iteration — it exposes `MoveNext()`, `Current`, and `Reset()`.
- In short: `IEnumerable` says "I can be looped over"; `IEnumerator` is the thing that actually does the looping (this is what a `foreach` loop uses under the hood).

**Q: How does `async`/`await` actually work?**

- Marking a method `async` allows it to contain `await` expressions and lets the compiler transform it into a state machine.
- Execution proceeds **synchronously, on the calling thread**, right up until it hits an `await` on an operation that is actually asynchronous (e.g., waiting on network I/O, disk I/O, a timer).
- At that point, **for genuinely I/O-bound work, .NET does NOT spawn a new thread.** The method returns control (a `Task`) to the caller, freeing the current thread to do other work while the I/O completes in the background (via the OS/hardware, not a dedicated .NET thread).
- When the awaited operation completes, the continuation resumes — by default back on a thread from the thread pool (in ASP.NET Core; in some UI frameworks, back on the original context via `SynchronizationContext`).
- A new thread *is* involved if you explicitly offload CPU-bound work with `Task.Run(...)`, but that's a deliberate choice, not something `await` does automatically.
- **Why this matters in an interview:** `async`/`await` is primarily about **not blocking threads while waiting on I/O**, which is why it scales so well for web servers — it's not "free parallelism" or "automatic multithreading."

**Q: How does garbage collection handle static variables?**

Static variables basically never get garbage collected on their own.

- A normal variable's object gets cleaned up once nothing references it anymore.
- A static variable belongs to the class itself (not to any object) and acts as a permanent **GC root** — the garbage collector always treats it as "still in use," for as long as the app is running.
- So if a static variable holds something (a list, a cache, etc.), that memory stays alive for the entire lifetime of the app — even if you've stopped using it.

It only gets cleaned up when:
- You manually set the static field to `null` (or otherwise remove the reference), or
- The app shuts down (or, rarely, the containing assembly load context is unloaded — an advanced/uncommon scenario like plugin systems).

**One-liner for an interview:** *"Static fields act as permanent GC roots, so anything they reference stays in memory for the app's lifetime unless you clear the reference yourself."*

**Q: What is a memory leak, and how do static variables cause one?**

A memory leak is when an app keeps memory allocated that it no longer actually needs, and never releases it — usage just keeps climbing over time instead of staying stable.

In C#/.NET, which has garbage collection, this happens differently than in unmanaged languages: **the memory is technically still "reachable," so the GC correctly refuses to collect it, even though the app doesn't need it anymore.** The GC isn't broken — something in the code is still holding a reference it shouldn't be.

Static fields are a classic cause, because they're permanent GC roots:

1. A static `List<T>`, `Dictionary<K,V>`, or cache is created.
2. The app keeps adding items to it as it runs (e.g., logging, caching results, storing sessions).
3. Nothing ever removes old items or clears the field.
4. The GC cleans up everything else normally, but this static collection keeps growing, since it still looks "in use."
5. Over time, memory climbs until the app slows down or crashes with an `OutOfMemoryException`.

```csharp
public class RequestLogger
{
    // Bug: this list grows forever, once per request, for the life of the app.
    private static List<string> _requestLog = new List<string>();

    public void LogRequest(string info)
    {
        _requestLog.Add(info); // never removed, never cleared
    }
}
```

**How to avoid it:**
- Don't use static fields for anything that grows unbounded over the app's lifetime.
- Use a cache with built-in eviction (`IMemoryCache`, an LRU cache, expiration policies) instead of a raw static collection.
- Use `WeakReference<T>` if you must hold a static reference to something large, so the GC can still reclaim it under memory pressure.
- Periodically clear/trim static collections that only need recent data.

**One-liner:** *"A memory leak in .NET isn't unreleased memory like in C — it's memory that's still technically reachable (usually via a static reference) but that the app no longer needs, so the GC correctly keeps it alive forever."*

---

### OOP & Design Patterns

**Q: Liskov Substitution Principle?**

An object of a subclass must be usable anywhere an object of the parent class is expected, without breaking the application. In practice: overridden methods in derived classes must honor the behavior/contract the base class promises (same or weaker preconditions, same or stronger postconditions).

**In plain language:** if class `Bird` has a method `Fly()`, and you create a subclass `Penguin : Bird`, LSP says anywhere your code expects a `Bird`, you should be able to hand it a `Penguin` instead and nothing should break. The classic violation: `Penguin` can't actually fly, so if `Penguin.Fly()` throws an exception or behaves completely differently from what `Bird.Fly()` promised, you've broken LSP — code written to work with any `Bird` will crash or misbehave when it happens to get a `Penguin`.

**One-line version for an interview:** *"A subclass should be swappable in for its parent class without the caller needing to know or care — it shouldn't surprise anyone."*

**Q: Singleton pattern — implementation?**

A creational pattern that ensures a class has exactly one instance, with a global access point to it.

```csharp
// Eager initialization — simple, thread-safe by default (CLR guarantees
// static field initializers run once), but the instance is created
// even if it's never used.
public sealed class Singleton
{
    private static readonly Singleton _instance = new Singleton();
    private Singleton() { }
    public static Singleton Instance => _instance;
}

// Lazy initialization — only created when first accessed.
// Use Lazy<T> for built-in thread safety instead of hand-rolled null checks.
public sealed class LazySingleton
{
    private static readonly Lazy<LazySingleton> _lazy =
        new Lazy<LazySingleton>(() => new LazySingleton());
    private LazySingleton() { }
    public static LazySingleton Instance => _lazy.Value;
}
```

**Q: Factory pattern?**

A creational pattern where a dedicated factory class/method is responsible for creating objects of related subclasses, so calling code doesn't need to know the concrete type it's instantiating — it just asks the factory for "a product" and gets back the right implementation.

---

### Dependency Injection

**Q: What is Dependency Injection?**

To avoid tight coupling, a class receives (is "injected with") the objects/services it depends on from the outside — typically via its constructor — rather than creating them itself.

**Q: DI service lifetimes?**

- **Singleton** — one instance is created (either at startup or on first request) and shared by every request for the lifetime of the application. Example: a logging service, a configuration cache.
- **Scoped** — one instance per HTTP request; shared within that request but not across requests. Example: **`DbContext`** (Entity Framework Core registers `DbContext` as **Scoped** by default. This matters because a Scoped `DbContext` tracks changes consistently for the whole request without leaking state between unrelated requests).
- **Transient** — a brand-new instance is created every single time the service is requested/injected. Best for lightweight, stateless services with no shared state — e.g., a simple email-formatting helper.

---

### ASP.NET Core Fundamentals

**Q: Key features of .NET Core / ASP.NET Core?**

- Open source, cross-platform (Linux, Windows, macOS).
- Integrates easily with modern frontend frameworks (React, Angular, etc.).
- Can be hosted via Kestrel, IIS, or behind Nginx/Apache as a reverse proxy.
- Built-in Dependency Injection container (no third-party DI library required).

**Q: Project structure?**

*(Note: `Startup.cs` was standard through .NET 5; from **.NET 6 onward**, the default templates use the "minimal hosting model," where `Startup.cs` is merged into `Program.cs`. Both patterns still work — older/enterprise codebases often still use `Startup.cs`.)*

- **`Program.cs`** — the application's entry point.
- **`Startup.cs`** (pre-.NET 6, or still optional later) — configures services and the HTTP request pipeline.
- **`appsettings.json`** — configuration values, e.g., database connection strings.
- **`wwwroot/`** — static assets: JS, CSS, images.

**Q: What does `Program.cs` do?**

On startup, `Main()` builds the host (which configures and creates the Kestrel — and optionally IIS integration — web server), then runs the app, which hands off request handling to the configured pipeline (via `Startup.cs` in the older model, or configured directly in `Program.cs` in the minimal hosting model).

**Q: What does `Startup.cs` do?**

- **`ConfigureServices(IServiceCollection services)`** — register services into the DI container.
- **`Configure(IApplicationBuilder app)`** — build the HTTP request pipeline by adding middleware components, in order.

**Q: What is middleware?**

A component that runs as part of the request-processing pipeline for every request in an ASP.NET Core app — each middleware can act on the request, pass it to the next middleware, and/or act on the response on the way back out.

**Q: What's the typical middleware pipeline order?**

1. Exception/error handling (`UseExceptionHandler` / `UseDeveloperExceptionPage`)
2. HSTS (`UseHsts`) — production only
3. HTTPS redirection (`UseHttpsRedirection`)
4. Static files (`UseStaticFiles`)
5. Routing (`UseRouting`)
6. CORS (`UseCors`) — if needed
7. Authentication (`UseAuthentication`)
8. Authorization (`UseAuthorization`)
9. Custom/session middleware
10. Endpoints (`UseEndpoints` / `MapControllers`, etc.)

**Q: In-process vs. out-of-process hosting?**

- **In-process** — the app runs directly inside the IIS worker process (`w3wp.exe`). Faster, since there's no extra network hop.
- **Out-of-process** — Kestrel runs the app (as `dotnet.exe`) behind a reverse proxy (IIS, Nginx, or Apache), which forwards requests to Kestrel. Needed when you want a platform other than Windows/IIS, or extra flexibility.

**Q: What is Kestrel?**

A lightweight, cross-platform web server built into ASP.NET Core, used to host ASP.NET Core applications (either directly, or behind a reverse proxy).

**Q: What is `RequestDelegate`, and what do `Use`/`Run`/`Map` do?**

`RequestDelegate` is the delegate type that handles an HTTP request; middleware is built using it via three extension methods:
- **`Use`** — runs this middleware, then calls the next one in the pipeline.
- **`Run`** — a terminal middleware; it doesn't call anything after it, so it's placed at the end of the chain.
- **`Map`** — branches the pipeline: runs the given middleware only if the request path matches the specified path.

**Q: Where should you store secrets?**

- **Not** directly in `appsettings.json` if it's committed to source control.
- **.NET User Secrets** (Secret Manager tool) — for local development; stores secrets outside the project folder, not in source control.
- **Azure Key Vault** — for production secrets, retrieved securely at runtime.
- **Environment variables** — common for containerized/cloud deployments.
- **Command-line arguments** — less common, mainly for local/dev overrides.

**Q: What is attribute routing?**

The ability to define and customize a controller action's URL route directly using `[Route]`/`[HttpGet]`-style attributes, rather than relying solely on convention-based routing.

**Q: What are "metapackages"?**

A metapackage (like `Microsoft.AspNetCore.App`) bundles together the commonly-needed ASP.NET Core packages — hosting, routing, IIS integration, Kestrel, configuration (environment variables, JSON files), logging/console logging — so you don't have to reference each one individually.

**Q: What does `UseStaticFiles` do?**

Enables serving static files (JS, CSS, images, etc.) directly to the client from `wwwroot`.

**Q: What is `global.json`?**

Pins the **.NET SDK version** a project should build with — useful for making sure everyone on a team (and CI) uses a consistent SDK version, not just general "solution-level settings."

**Q: What is `launchSettings.json`?**

Defines launch profiles and environment variables (like `ASPNETCORE_ENVIRONMENT`) used when running the app locally, e.g. from Visual Studio or `dotnet run`.

**Q: What is `bundleconfig.json`?**

Used for bundling and minifying static assets (JS/CSS) in **older** ASP.NET / ASP.NET Core projects. In modern ASP.NET Core projects, this has largely been superseded by frontend build tools (Webpack, Vite, esbuild, etc.) — worth mentioning this is legacy rather than current best practice.

**Q: What is `project.json`?**

`project.json` was used only in early, pre-release versions of .NET Core (around RC1/RC2, before the 1.0 release). It was **replaced by the standard MSBuild `.csproj` format** starting with .NET Core 1.0 RTM (2017) and is no longer used in any current .NET project. Not worth bringing up as current knowledge in an interview — mention `.csproj` instead if asked about project-level configuration.

**Q: How do you write custom middleware?**

```csharp
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // ...logic before the next middleware...
        await _next(context);
        // ...logic after the next middleware runs (on the way back out)...
    }
}
```

**Q: Session vs. TempData?**

- **Session** — server-side storage that persists data across multiple requests for the duration of a user's session (until it times out or the browser closes). Good for data needed across many pages/requests.
- **TempData** — designed to persist data for exactly one subsequent request — typically used in the Post-Redirect-Get pattern to pass data from one controller action to the next after a redirect. Backed by session or cookies under the hood.

**Q: How is model validation done?**

Primarily via **data annotations** (`[Required]`, `[StringLength]`, `[Range]`, etc.) on model properties, checked via `ModelState.IsValid`. FluentValidation is a popular alternative/complement for more complex validation rules.

**Q: How does the developer exception page get shown/hidden?**

Controlled by the `ASPNETCORE_ENVIRONMENT` variable (commonly set in `launchSettings.json` for local dev). When it's `Development`, `env.IsDevelopment()` returns true, and you'd typically call `app.UseDeveloperExceptionPage()` to show detailed error info — which you do **not** want enabled in production.

**Q: What is model binding?**

The process of automatically mapping data from an incoming HTTP request (route values, query string, form data, JSON body) to parameters/properties on a controller action method.

**Q: What is CORS?**

**Cross-Origin Resource Sharing** — a browser-enforced security mechanism that restricts a web page/script from making requests to a different origin (domain/scheme/port) than the one that served the page, unless the target server explicitly allows it via CORS headers.

**Q: In-memory caching vs. distributed caching?**

- **In-memory caching** — cached data lives in the memory of the same server/process hosting the app. Fast, but not shared across multiple server instances.
- **Distributed caching** — data is stored on a separate service (e.g., Redis), so it's shared and consistent across multiple app instances/servers — necessary once you scale horizontally.

---

### Auth & Web APIs

**Q: Authentication vs. Authorization?**

- **Authentication** — verifying *who you are* (e.g., checking a username/password).
- **Authorization** — determining *what an authenticated user is allowed to access*.

**Q: Types of authentication?**

- **Cookie-based authentication** — server issues a cookie after login; browser sends it back on subsequent requests.
- **Token-based authentication (e.g., JWT)** — server issues a signed token; client sends it in the `Authorization` header on each request.
- **Windows authentication** — uses the OS-level Windows credentials (common in intranet/enterprise apps).
- **OAuth 2.0 / OpenID Connect** — the modern standard for delegated auth and SSO (e.g., "Sign in with Google/Microsoft"); worth mentioning instead of the now-defunct Passport auth.

**Q: What is JWT authentication?**

- **JWT (JSON Web Token)** is a compact, signed token format.
- Client sends username/password; server validates the credentials and, if valid, returns a **signed JWT token** to the client (**not** the signing secret/key itself — that must stay private on the server).
- The client stores this token and sends it on subsequent requests in the `Authorization: Bearer <token>` header.
- The server verifies the token's signature (using its own secret/key) and, if valid, processes the request.

**Q: REST vs. "RESTful"?**

**REST** (Representational State Transfer) is an architectural style defined by a set of constraints. **"RESTful"** is just the adjective used to describe an API that actually follows REST's constraints — there's no separate technical definition; every "RESTful API" *is* a REST API.

REST's core constraints:
- **Client-server separation** — client and server evolve independently.
- **Statelessness** — the server stores no client session state between requests; each request from the client must contain all the information needed to process it.
- **Uniform interface** — resources are identified via URLs, and interacted with through a consistent set of operations (HTTP verbs: GET, POST, PUT, DELETE, etc.).
- **Cacheability** — responses should indicate whether they can be cached, to improve performance.
- **Layered system** — the client shouldn't need to know whether it's talking directly to the server or through intermediaries (load balancers, gateways, etc.).

---

*General note: several answers above depend on the .NET version in use (e.g., `Startup.cs` vs. minimal hosting model, `bundleconfig.json` vs. modern bundlers). If asked in an interview, it's worth naming which era/version you're describing rather than presenting one as universally current.*

---

### OOP Basics

**Q: What is a class?** A blueprint used to create objects.

**Q: What is an object?** An instance of a class, containing its own data (state) and behavior (methods).

**Q: What is a constructor?** A special method used to initialize an object; it's called automatically when the object is created.

**Q: What are access specifiers?** Keywords that define the visibility/accessibility of a class and its members.

**Q: What are C#'s access modifiers, specifically?**

- **`public`** — accessible from anywhere.
- **`internal`** — accessible within the same assembly.
- **`protected`** — accessible within the declaring class and its derived classes.
- **`private`** — accessible only within the declaring class.
- **`protected internal`** — accessible from the same assembly OR from derived classes (even in other assemblies).

**Q: What is an interface?** A contract that an implementing class must fulfill — it contains method/property signatures but no implementation. Since a C# class can implement multiple interfaces (unlike inheriting multiple base classes), interfaces are how C# achieves a form of "multiple inheritance."

**Q: What is abstraction?** Hiding implementation details and exposing only the essential features/behavior a consumer actually needs.

**Q: Types of constructors?**

- **Default constructor** — added automatically by the compiler if you don't define any constructor yourself; takes no parameters.
- **Parameterized constructor** — takes arguments used to initialize the object's fields; this is also the mechanism behind **constructor-based dependency injection** (the DI container passes in the object's dependencies as constructor parameters).
- **Static constructor** — runs once, automatically, before the class is used for the first time (either on first instantiation or first access of a static member); used to initialize static members.

---

### SOLID Principles

**Q: Single Responsibility Principle?** A class should have exactly one responsibility/reason to change — this keeps classes easier to understand, modify, and maintain.

**Q: Open/Closed Principle?** Classes should be open for extension but closed for modification — you should be able to add new behavior without changing existing, tested code (typically via inheritance, interfaces, or composition).

**Q: Dependency Inversion Principle?** High-level modules shouldn't depend on low-level modules directly — both should depend on abstractions (interfaces). Dependency Injection is the common technique used to implement this principle in practice.

---

### Generics

**Q: What are generics, and what do they actually give you?**

Generics let you define a class, method, or property with a **placeholder type** — you don't have to commit to a specific data type until the generic is actually used (e.g., `List<T>` becomes `List<int>` or `List<string>`).

**What generics actually give you (corrected):**
- **Compile-time type safety** — the compiler enforces the type at usage, catching mismatches before runtime instead of throwing a runtime `InvalidCastException`.
- **No boxing/unboxing overhead for value types** — a `List<int>` stores actual `int`s directly; the old `ArrayList` stored everything as `object`, meaning every `int` had to be boxed onto the heap and unboxed on retrieval — real, measurable performance overhead.
- **Better performance overall**, precisely *because* it avoids that boxing and avoids runtime type-checking/casting.

---

### Entity Framework Core

**Q: What is `DbContext`?** Represents a session with the database — the primary class through which you query and save data via EF Core.

**Q: What is `DbSet`?** Represents a table in the database within a `DbContext` — you query and modify rows through it (e.g., `DbSet<Customer>`).

**Q: `IEnumerable` vs. `IQueryable`?**

- **`IEnumerable`** — the query executes **in memory**. If you filter/query an `IEnumerable`, EF Core has already pulled the full result set into memory first, and any further LINQ operations run client-side.
- **`IQueryable`** — the query is **translated and executed in the database**. Filtering/sorting/paging applied to an `IQueryable` gets pushed down into the SQL sent to the database, so only the needed rows come back — generally more efficient for anything beyond trivial queries.

---

### Design Patterns

**Q: What is a DTO (Data Transfer Object)?** An object used purely to transfer data between layers (e.g., API layer ↔ service layer), typically without behavior — just a flat data shape, often used to avoid exposing your internal domain/entity models directly.

**Q: What is the Repository pattern?** Separates the data access layer from the business logic layer, behind an abstraction (interface) — improves maintainability, testability, and readability by decoupling "how data is fetched" from "what the business logic does with it."

**Q: What is the Unit of Work pattern?**

A pattern that groups multiple repository operations (e.g., several inserts/updates across different repositories) into a **single transaction**, ensuring they all succeed or all fail together, and committing them with one `SaveChanges()` call rather than each repository saving independently. Commonly paired with the Repository pattern — repositories handle *how* to query/persist, Unit of Work coordinates *when* changes actually get committed.

---

### Async / Threading Deep Dives

**Q: What goes wrong if you call an async method without `await`ing it (fire-and-forget)?**

- If the task throws an exception, the calling code has no way of knowing — the exception is essentially swallowed (it ends up on the unobserved task, not surfaced to your code).
- If subsequent code depends on the result of that task, it will run before the task is actually done — logic bugs.
- If the task uses resources tied to the current scope (e.g., a Scoped `DbContext` in a web request), that scope might be disposed before the task finishes, causing a runtime error.

**Q: What is `ConfigureAwait(false)` specifically doing?**

By default, `await` tries to resume execution back on the **original synchronization context** (e.g., the original request context in some frameworks, or UI thread in desktop apps). Calling `.ConfigureAwait(false)` tells the awaiter **not** to capture that context — the continuation can resume on **any available thread pool thread** instead. Commonly used in library code to avoid unnecessary context-switching overhead and potential deadlocks, since library code usually doesn't care which thread it resumes on.

**Q: `async void` methods — what's the problem?**

- **Exception handling is broken** — exceptions thrown inside an `async void` method can't be caught by the caller with a normal `try/catch` around the call; they instead surface on the `SynchronizationContext`, often crashing the process.
- **No way to await it** — it returns nothing (not even a `Task`), so the caller has no way of knowing when it finishes, or whether it finished successfully. (`async void` should generally be reserved for top-level event handlers only, where the signature is forced on you.)

---

### Memory Management

**Q: `Dispose()` vs. `Finalize()`?**

- **`Dispose()`** — called **explicitly by the developer** (directly, or implicitly via a `using` block) to release **unmanaged resources deterministically**, right when you're done with them.
- **`Finalize()`** — called **automatically by the garbage collector**, at some non-deterministic future point, as a safety-net cleanup for unmanaged resources if `Dispose()` was never called. Relying on it alone is discouraged since you don't control *when* it runs.

**Q: What is boxing and unboxing?**

- **Boxing** — converting a value type (e.g., `int`) into a reference type by allocating a new object on the **heap** to wrap it.
- **Unboxing** — the reverse: extracting the value back out of that heap object. The runtime checks that the object's actual type matches the target value type, then copies the value back onto the **stack**. (An invalid cast here throws `InvalidCastException`.)

**Q: Where are static variables stored in .NET?**

- Static fields do **not** live on the stack, and they don't live in the regular object heap the way instance objects do either.
- They're stored in memory associated with the **type itself**, managed by the CLR (in what's often called the **high-frequency/loader heap**, tied to the type's metadata) — allocated once when the type is loaded, for the lifetime of the app domain.
- As covered earlier: a static field acts as a **permanent GC root**, so whatever object it references stays alive for the app's lifetime unless the field is explicitly cleared.
- The `.data`/`.bss` segment concept is specific to compiled native binaries (C/C++, ELF/PE layout) — .NET's managed runtime doesn't expose or use that model for your code's static fields.

**Q: What does the `in` keyword do (as a parameter modifier)?**

`in` passes an argument to a method **by reference**, but the method is **not allowed to modify it** — a read-only reference parameter. It's mainly a performance optimization: for a large `struct`, passing `in` avoids copying the whole struct (like `ref` would), while the `in` keyword's compiler-enforced immutability prevents the callee from accidentally mutating the caller's data.

```csharp
void Process(in LargeStruct data) { /* can read data, cannot assign to data.Field */ }
```

**Q: `ref` vs. `out` parameters?**

- **`ref`** — the variable must already be **initialized** before being passed in; the method can read and/or modify it.
- **`out`** — the variable can be passed in **uninitialized** (even empty), but the method is **required** to assign it a value before returning.
- Both are used when a method needs to return more than one value.

**Q: `yield return`?** Used to implement the iterator pattern lazily — values are produced one at a time as they're requested, without building and returning a full temporary collection in memory up front.

---

### ASP.NET Core — Filters & Exception Handling

**Q: What is a filter (in ASP.NET Core MVC)?** Sits within the **MVC pipeline** specifically (a layer below general middleware) — it only runs when a specific controller action is being routed to, letting you hook into stages like authorization, action execution, or result execution at that finer granularity.

**Q: What is global exception handling?** A mechanism (e.g., exception-handling middleware) that ensures if an error occurs anywhere in the application, instead of the app crashing, the error is caught, logged, and a controlled response is returned to the client.

---

### Var vs. Dynamic

**Q: `var` vs. `dynamic`?**

- **`var`** — type is resolved at **compile time**, inferred from the initializer, and fixed from then on. Can't be initialized to `null` alone (compiler has nothing to infer the type from). Can't be used for class-level fields, only local variables. Can't be used as a method's return type.
- **`dynamic`** — type resolution is deferred to **runtime**; the compiler does minimal checking, and errors that would normally be compile-time type errors instead show up as runtime exceptions.
- *Lambda caveat:* `var` couldn't be used with a lambda through older C# versions, but from **C# 10 onward**, `var` **can** be used with a lambda expression if the compiler can infer a "natural type" for it (e.g., `var f = (int x) => x + 1;` infers a `Func<int,int>`-like delegate type). Worth mentioning the version if this comes up.

---

### Threading & Object Model Extras

**Q: `IEnumerable` vs. `ICollection` vs. `IList`?**

- **`IEnumerable`** — the most basic; only lets you iterate with `foreach`, nothing else.
- **`ICollection`** — adds `Count`, `Add`, `Remove`, `Contains`.
- **`IList`** — adds index-based access, plus `Insert`/`RemoveAt` by index.
- (Each interface extends the previous one — `IList` extends `ICollection`, which extends `IEnumerable`.)

**Q: What is a scoped service captured inside a singleton — why is it a problem?**

A scoped service is meant to live for one request only. If a **singleton** captures a reference to a scoped service (e.g., injecting it into the singleton's constructor), that scoped instance gets held onto for the singleton's entire lifetime — effectively behaving like a second singleton:

- Every subsequent request that goes through that singleton sees the **same** scoped instance — including whatever state was left over from the **first** request that triggered its creation. This is sometimes called a "captive dependency."
- The scoped service also never gets properly disposed at the end of each request, since the singleton is still holding a live reference to it.

**Q: What does the `using` keyword do?** Ensures a resource implementing `IDisposable` is automatically disposed once the block exits — even if an exception is thrown — equivalent to wrapping the code in a `try/finally` that calls `Dispose()`.

**Q: What does short-circuiting mean in a middleware pipeline?** A middleware can choose not to call the next middleware in the chain (i.e., not call `next()`), stopping the pipeline early and returning a response immediately — e.g., an auth middleware rejecting a request before it ever reaches the rest of the pipeline.

**Q: What is `IHostedService`?** The interface used to implement a long-running background service in .NET (e.g., a service that runs on a timer, processes a queue, etc.), managed by the host alongside the rest of the app's lifetime.

**Q: What is an extension method?** A `static` method that appears to be an instance method on an existing type, enabled by using the `this` keyword on the method's first parameter — lets you "add" methods to a type (including types you don't own, like `string` or `List<T>`) without modifying it or subclassing it.

---

### Cosmos DB

**Q: What is a partition key?** An attribute/column on your documents that determines how data is distributed across multiple physical partitions in Cosmos DB — choosing a good partition key (high cardinality, even access distribution) is critical for performance and scalability.

**Q: What is a distributed lock in Cosmos DB?**

Cosmos DB doesn't have a dedicated built-in "lock" primitive, but a distributed lock can be implemented on top of it — typically using **optimistic concurrency control** via the document's `_etag` field: a process reads a document, attempts an update conditioned on the `_etag` matching what it read, and the write fails if another process already changed it in between (meaning someone else "holds the lock"). This pattern is used to coordinate exclusive access across multiple instances/processes without a separate locking service.

**Q: What is a blob lease?**

An **Azure Blob Storage** feature (not Cosmos DB) that lets a client acquire **exclusive write access** to a blob or container for a renewable duration (15–60 seconds, or infinite until explicitly released). While a lease is held, other clients attempting to write to or delete that blob are rejected — commonly used to implement a distributed lock, since only one process can successfully hold the lease at a time.

---

### Sample: Implementing `IDisposable`

```csharp
public class FileHandler : IDisposable
{
    private StreamReader reader;

    public FileHandler(string path)
    {
        reader = new StreamReader(path);
    }

    public void ReadFile()
    {
        reader.ReadLine();
    }

    public void Dispose()
    {
        if (reader != null)
        {
            reader.Dispose(); // Dispose() is preferred over Close() here — Dispose handles cleanup and is the IDisposable-idiomatic call
            reader = null;
        }
    }
}
```

---