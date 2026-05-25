REST and HTTP are related, but they are **not the same thing**.

Here’s the simplest way to think about it:

* **HTTP** = a communication protocol
* **REST** = an architectural style that often uses HTTP

---

## HTTP (HyperText Transfer Protocol)

HTTP is the set of rules that lets clients and servers communicate on the web.

Examples:

* Your browser requesting a webpage
* A mobile app calling an API
* Frontend talking to backend

HTTP defines things like:

* Methods (`GET`, `POST`, `PUT`, `DELETE`)
* Status codes (`200`, `404`, `500`)
* Headers
* Request/response format

Example HTTP request:

```http
GET /users/123 HTTP/1.1
Host: api.example.com
```

Example response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Alice"
}
```

---

## REST (Representational State Transfer)

REST is a design style for building APIs.

A REST API typically:

* Uses HTTP
* Organizes data into **resources**
* Uses URLs to identify resources
* Uses HTTP methods semantically

Example REST API:

```text
GET    /users        -> list users
GET    /users/123    -> get one user
POST   /users        -> create user
PUT    /users/123    -> update user
DELETE /users/123    -> delete user
```

REST has principles such as:

* Stateless communication
* Client-server separation
* Uniform interface
* Resource-based URLs

---

## Key Difference

| HTTP                        | REST                                  |
| --------------------------- | ------------------------------------- |
| Protocol                    | Architecture/style                    |
| Defines communication rules | Defines how APIs should be structured |
| Can be used for anything    | Usually used for web APIs             |
| Exists independently        | Commonly built on top of HTTP         |

---

## Important Note

Not every HTTP API is RESTful.

Example:

```text
POST /getUser
POST /deleteUser
```

This uses HTTP, but it’s not very RESTful because it treats actions like RPC calls instead of resources.

A more RESTful design:

```text
GET    /users/123
DELETE /users/123
```

---

## In One Sentence

> HTTP is the language of web communication, while REST is a set of design conventions for using that language to build APIs.
