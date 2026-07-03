# Chapter 8 — API Design: REST, SOAP, WebSocket & Data Formats

> JD **must-have**: "API Design and Development", plus WebSocket, REST/SOAP, JSON/YAML/TOML. Expect a design exercise: "design an API for X."

## 8.1 HTTP in 60 seconds (the foundation)

```
GET /api/v1/machines/42 HTTP/1.1        ← method + path + version
Host: api.example.com
Authorization: Bearer <token>            ← headers
Accept: application/json

HTTP/1.1 200 OK                          ← status line
Content-Type: application/json

{"id": 42, "rpm": 15000}                 ← body
```

**Status codes by class (know one example each):**
| Class | Meaning | Must-know codes |
|---|---|---|
| 2xx | success | 200 OK, 201 Created, 204 No Content |
| 3xx | redirection | 301 Moved, 304 Not Modified |
| 4xx | client error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests |
| 5xx | server error | 500 Internal Error, 502 Bad Gateway, 503 Unavailable |

**Trap:** 401 = "who are you?" (not authenticated); 403 = "I know you, you're not allowed" (not authorized).

## 8.2 REST — principles and resource design

REST = **resources** (nouns) identified by URLs, manipulated with standard HTTP **methods**, **stateless** (each request self-contained).

| Method | Meaning | Idempotent? | Safe? |
|---|---|---|---|
| GET | read | ✅ | ✅ |
| POST | create / action | ❌ | ❌ |
| PUT | replace entirely | ✅ | ❌ |
| PATCH | partial update | ❌ (usually) | ❌ |
| DELETE | remove | ✅ | ❌ |

**Idempotent** = repeating the request gives the same result (crucial for retries!).

### Designing a resource API (example: spinning machines)

```
GET    /api/v1/machines              # list (with filters & pagination)
POST   /api/v1/machines              # create → 201 + Location header
GET    /api/v1/machines/42           # read one
PUT    /api/v1/machines/42           # replace
PATCH  /api/v1/machines/42           # partial update
DELETE /api/v1/machines/42           # delete → 204
GET    /api/v1/machines/42/sensors   # nested resource
POST   /api/v1/machines/42/restart   # action that isn't CRUD — verb as sub-resource
```

Rules interviewers listen for:
- **Nouns, plural, no verbs** in paths (`/machines`, not `/getMachine`).
- **Version the API** (`/api/v1/`) — breaking changes go to v2.
- **Pagination** for lists: `?page=2&per_page=50` or cursor-based (`?after=abc123` — stable under inserts, scales better).
- **Filtering/sorting** via query params: `?status=running&sort=-created_at`.
- Consistent **error body**: `{"error": {"code": "MACHINE_NOT_FOUND", "message": "..."}}`.
- **Auth**: `Authorization: Bearer <JWT>` header; never credentials in URLs. HTTPS always.
- **Rate limiting**: 429 + `Retry-After` header.
- **Idempotency keys** for POST retries (client sends `Idempotency-Key` header; server dedupes) — senior-level point.

### A REST endpoint in Rust (axum) — shows JD skills together

```rust
use axum::{routing::get, extract::Path, Json, Router};
use serde::Serialize;

#[derive(Serialize)]
struct Machine { id: u32, rpm: f64 }

async fn get_machine(Path(id): Path<u32>) -> Json<Machine> {
    Json(Machine { id, rpm: 15000.0 })
}

#[tokio::main]
async fn main() {
    let app = Router::new().route("/api/v1/machines/{id}", get(get_machine));
    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

## 8.3 SOAP — what to know (legacy but in the JD)

- XML-based protocol with a strict **WSDL contract**; transport-independent (usually HTTP POST); built-in standards for security/transactions (WS-*).
- Verbose envelope structure:

```xml
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header/>
  <soap:Body>
    <GetMachineStatus><MachineId>42</MachineId></GetMachineStatus>
  </soap:Body>
</soap:Envelope>
```

| | REST | SOAP |
|---|---|---|
| Format | JSON (usually) | XML only |
| Contract | OpenAPI (optional) | WSDL (mandatory) |
| Weight | light | heavy |
| Where you meet it | modern services | enterprise/industrial legacy, ERP, banking |

**Interview line:** "I'd choose REST for new services; I can consume SOAP where legacy industrial or ERP systems require it — generated clients from the WSDL keep it manageable."

## 8.4 WebSocket — real-time, bidirectional (JD keyword)

HTTP is request/response; WebSocket upgrades an HTTP connection into a **persistent, full-duplex** channel — ideal for live machine telemetry, dashboards, chat.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: GET /ws  (Upgrade: websocket, Sec-WebSocket-Key)
    S->>C: 101 Switching Protocols
    Note over C,S: Same TCP connection, now message frames both ways
    S-->>C: {"rpm": 15012}   (server pushes without being asked!)
    C-->>S: {"cmd": "setSpeed", "value": 15500}
    Note over C,S: ping/pong keepalives; either side can close
```

| | REST (polling) | WebSocket |
|---|---|---|
| Direction | client asks | both push |
| Latency | poll interval | instant |
| Overhead | headers per request | tiny frames after handshake |
| State | stateless | per-connection state (scaling consideration!) |

Rust: `tokio-tungstenite` or axum's built-in `WebSocketUpgrade` extractor. Scaling note to mention: many persistent connections → async runtime (Ch 4/7), and sticky sessions or a pub/sub backplane (Redis/Kafka) when you have multiple server instances.

Also name-drop: **SSE** (server→client only, simpler than WS) and **gRPC** (HTTP/2 + protobuf, typed contracts, streaming) as alternatives.

## 8.5 Data formats: JSON vs YAML vs TOML (all three in the JD)

Same config in each:

```json
{ "server": { "host": "0.0.0.0", "port": 8080, "tags": ["prod", "eu"] } }
```

```yaml
server:
  host: 0.0.0.0
  port: 8080
  tags: [prod, eu]     # indentation-sensitive; comments allowed
```

```toml
[server]
host = "0.0.0.0"
port = 8080
tags = ["prod", "eu"]  # flat and unambiguous; Cargo.toml uses this
```

| | JSON | YAML | TOML |
|---|---|---|---|
| Comments | ❌ | ✅ | ✅ |
| Typical use | APIs, data exchange | K8s/CI configs | Rust/app configs |
| Gotchas | no trailing commas | indentation errors, `no` → false (the "Norway problem") | less nesting-friendly |

In Rust, **serde** handles all three with the same derive:

```rust
#[derive(serde::Deserialize)]
struct Config { server: Server }
#[derive(serde::Deserialize)]
struct Server { host: String, port: u16 }

let cfg: Config = toml::from_str(&text)?;        // or serde_json / serde_yaml
```

---

## 🎯 Chapter 8 Interview Q&A

**Q1. What makes an API RESTful?**
Resources as nouns/URLs, standard HTTP methods with correct semantics, statelessness, proper status codes, and representations (JSON) decoupled from server internals.

**Q2. PUT vs PATCH vs POST?**
PUT replaces the whole resource (idempotent), PATCH modifies part of it, POST creates or triggers non-idempotent actions.

**Q3. Why does idempotency matter?**
Networks fail and clients retry. Idempotent operations can be retried safely; for POST, an Idempotency-Key header lets the server deduplicate.

**Q4. How do you version an API and evolve it safely?**
URL versioning (`/v1/`) or header-based. Additive changes (new optional fields) are non-breaking; removals/renames require a new version and deprecation window.

**Q5. When would you choose WebSocket over REST?**
When the server must push (live telemetry, notifications) or chatty low-latency bidirectional traffic makes per-request HTTP overhead wasteful. Otherwise REST is simpler to scale and cache.

**Q6. How do you secure a REST API?**
HTTPS everywhere; authenticate with tokens (JWT/OAuth2) in the Authorization header; authorize per-resource; validate all input; rate-limit; don't leak internals in error messages.

**Q7. What is JWT and its main caveat?**
A signed token carrying claims, verified statelessly by the server. Caveat: can't be revoked before expiry — keep lifetimes short and pair with refresh tokens.

**Q8. Offset vs cursor pagination?**
Offset (`page=100`) is simple but skews when rows are inserted and gets slow at deep offsets. Cursor (`after=<id>`) is stable and O(1) per page — preferred for large/changing datasets.

**Q9. REST vs gRPC?**
REST: human-readable, cacheable, universal. gRPC: binary protobuf over HTTP/2, strict typed contracts, streaming, much faster for service-to-service — but browsers need a proxy.

**Q10. A WebSocket service must scale to 3 servers — what breaks?**
Connection state is per-server: a message published on server A must reach a client connected to server B. Fix: shared pub/sub backplane (Redis, Kafka) plus sticky load-balancer sessions for the connections themselves.

**Q11. How would you design "get machine telemetry history" — REST or WS?**
History = finite dataset → REST GET with time-range params and pagination. Live stream = WebSocket/SSE subscription. Combining both is the standard telemetry pattern.

**Q12. Content negotiation?**
Client states preferred formats via `Accept` (e.g., `application/json`); server responds with `Content-Type` accordingly, or 406 if unsupportable.
