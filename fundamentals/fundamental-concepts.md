# Fundamental concepts

For each concept, we answer

1. What is it ?
2. Why do we need it ?
3. A concrete example.

## HTTP cheat sheet

Headers -> Metadata
Cookies -> browser-stored data
Sessions -> server-side user state
REST -> API design style
JSON -> data format
CORS -> browser cross-origin security
TLS -> encryption/security
DNS -> translates domain name to IP

## 1. HTTP request/response

Protocol your browser and servers use to communicate with each other.

Request:

```
GET /users/123 HTTP/1.1
Host: example.com
Accept: application/json
```

Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
{
"id": 123,
"name": 'Alice'
}
```

GET -> retrieve
POST -> Create / perform an action
PUT -> replace
PATCH -> partially update
DELETE -> delete

## 2. HTTP status codes

Success:

```
200 OK
201 Created
204 No content
```

Client errors:

```
400 Bad request
401 Unauthorised
403 Forbidden
404 Not Found
409 Conflict
429 Too many requests
```

Server errors:

```0
500 Internal server error
502 Bad gateway
503 Service unavailable
```

## 3. Headers

Metadata about the request or response. It includes things like, `Content-Type`, `Accept`, `Authorization`, `Cookie`, `Set-Cookie`, `Cache-Control`, `User-Agent`, `Origin`, `Host`.

## 4. Cookie

A small piece of data the browser stores and sends back to a website.

Some attributes include:
HttpOnly. JS can't directly access the cookie.
Secure. It can only be sent over HTTPS.
SameSite. Controls when cookies are sent in cross-site situations.

Common use: Login -> server creates session -> server sends session cookie -> browser stores cookie -> browser sends cookie with future requests -> server can authenticate

## 5. Sessions

A way for a server to remember a user's state across multiple HTTP requests.

HTTP itself is stateless which means that if you make “GET /profile” then “GET /orders”, the server doesn't inherently know that both requests belong to the same user.

A common solution:

Browser
│
│ session_id=abc
↓
Server
│
↓
Session Store
│
└── abc → user_id=123

So:

Cookie
↓
Session ID
↓
Server-side session
↓
User

## 6. REST

An architectural style commonly used for HTTP APIs. You model things as resources.

For example:

/users
/users/123
/products
/products/456
/orders
/orders/789

Then use HTTP methods:
GET /users/123 → Get user 123.
POST /users → Create a user.
PATCH /users/123 → Update user 123.
DELETE /users/123 → Delete user 123.

Each request is stateless which means that each request must contain the information needed to process it.

Don't think:

Request 1 → server remembers everything
Request 2 → magically depends on Request 1

Instead:

Request 1 → independently understandable
Request 2 → independently understandable

## 7. JSON

JSON is a common format for exchanging structured data.

Example API:
Request: GET /users/123
Response:
{
"id": 123,
"name": "Alice"
}

HTTP → communication protocol
JSON → data format

## 8. HTTPS

HTTP protected by TLS.

Without HTTPS: HTTP -> TCP -> IP

With traditional HTTPS: HTTP -> TLS -> TCP -> IP

TLS provides things like: Encryption, Authentication of the server, and Integrity

## 9. TLS

The security protocol underneath HTTPS.

Conceptually:

Browser Server
│ │
│──── Client Hello ────────>│
│ │
│<── Certificate ───────────│
│ │
│ Verify certificate │
│ │
│ Establish keys │
│ │
│════ Encrypted traffic ════│

TLS provides:
Confidentiality: Other people can't easily read the traffic.
Integrity: Traffic can't be silently modified without detection.
Authentication: The browser can verify the server's identity using certificates.

## 10. DNS

DNS translates: web addresses we type into an ip address.

e.g.,

google.com -> DNS -> 142.250.x.x

Your browser needs to discover where the server is before it can connect to it.

Also know that DNS responses are cached.

A simplified lookup path might be:

Browser
↓
OS cache
↓
DNS resolver
↓
Authoritative DNS
↓
IP address

# 11. CORS (Cross-Origin Resource Sharing)

Imagine your frontend is running at: https://myapp.com
and it tries to call: https://api.other.com/users

That's a cross-origin request. The browser has security rules around this. The API can respond with something like:
Access-Control-Allow-Origin: https://myapp.com

This tells the browser: "Requests from this origin are allowed."

CORS is primarily a browser security mechanism. Your server can technically receive a request from another server regardless of CORS. CORS controls whether the browser allows the frontend JavaScript to access the response.

## 12. Authentication vs Authorization

Authentication: Who are you?
e.g.,
Username/password
OAuth
Session
JWT
Passkey

Authorization: What are you allowed to do?
e.g.,
Alice is authenticated.
Alice is an admin.
Alice can delete users.

e.g.,
You log into GitHub.
Authentication: "This is Alice."
Authorization: "Alice has permission to push to this repository."

## 13. Stateless vs Stateful Services

Stateful. The server remembers information about the client between requests.

Request 1
↓
Server stores state
↓
Request 2
↓
Server relies on stored state

Example:

Server A
↓
session = user123

If your next request goes to Server B:

Request
↓
Server B
↓
"Where's the session?"

You have a scaling problem unless the state is shared.

Stateless. Each request contains everything necessary for the server to process it.

Request 1 → Server A
Request 2 → Server B
Request 3 → Server C

Any server can handle the request.

This makes horizontal scaling easier.

For example:

             Load Balancer
             /     |     \
            ↓      ↓      ↓
        Server A Server B Server C

If your servers don't keep local user session state, requests can be distributed freely.
