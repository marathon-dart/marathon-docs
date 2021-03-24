# Marathon Sessions (Specification)

This is an internal design document for Marathon's sessions module.

## What are sessions?

Sessions are key to providing a reactive user experience on the server. Essentially, the idea is that the server stores
some data set correspondent to a user id. Responses can then leverage the data associated with that user id
(the "session" data) to provide a customized user experience.

Essentially, without some sort of session management, we couldn't customize our services on a per-user basis! For
example, imagine trying to build a banking service without the server knowing who the user is. It doesn't work, does it?

Session IDs are typically stored using cookies, which are a type of storage mechanism in browsers. This cookie is
transmitted along with every request; the presence (or lack thereof) of a valid session ID in a request can indicate a
user's status with the system.

## Session Security

There are a few vectors of attack on a typical session-based system. The simplest is just a brute force attack: by
trying enough different user IDs, an attacker could gain access to user data. This is why session IDs must always be
UUIDs, or another such construct with a low chance of collision.

A more advanced attacker might think to record a user's user ID and then send it to the server, thereby gaining access
to all the user's data and privileges. While it's practically impossible to ENSURE that a client won't leak this data,
we should take steps to mitigate the likelihood. This includes:

- HTTPS only communication (plain HTTP is insecure and could leak the cookie with the ID)
- Setting an expiry on the session cookie (The session ID is automatically deleted ~30 min after it's received by the
  client, unless it's re-received).

## API Design

If you are not familiar with [the shelf library](https://pub.dev/packages/shelf), I highly recommend you read the
documentation there before reading this API design. This design will gloss over concepts and classes introduced there,
mostly in the interest of brevity and simple code snippets.

#### Basics

Marathon sessions should be convenient middleware on a shelf server. This means something like:

```dart

var app = Pipeline().addMiddleware(sessionManager()).addHandler(_performSessionBasedActions);
```

This sessionManager function should, internally, construct a SessionManager object, and attach it to the Request. shelf
Requests have a property `Map<String, dynamic> context`, which the middleware can write to. The manager can thus be
attached to the context in the middleware, and then read and used from a handler. For example:

```dart
Response _performSessionBasedActions(Request req) {
  var sessionManger = req.context['sessions'] as SessionManager;
  // perform some actions with sessionManager
  return Response.ok('yay!');
}
```

#### SessionManager

I've made references to a `SessionManager` class without explaining its purpose. Essentially, the SessionManager is
responsible for providing the user an interface to read/edit/refresh sessions. 

SessionManager should have the following parameters (optional parameters are italicized with their default values beside
them):

- ***boolean cookieHttpOnly = true***: exposes session id cookie's httpOnly attribute
- ***Duration cookieMaxAge = Duration(minutes: 30)*** : exposes session id cookie's maxAge attribute, with a sane,
  secure default.
- ***boolean cookieSecure = false*** : exposes session id cookie's secure attribute, which limits the cookie to being
  only transmittable over HTTPS. Though this is HIGHLY recommended for production servers, it can cause issues in dev
  environments, so the default is false.
- ***String Function() genID = _uuidGenerator*** : the function to call when generating a new session ID. This should,
  by default, use a function that generates a UUID.
- ***String cookieName = 'marathon.sid'*** : the name of the session ID cookie. The session ID will be written into the
  Response header under this cookieName and read from the Request header under this cookieName.
- **SessionStore store** : the store abstraction containing all the user sessions. Explained in depth in the next
  section.
  
Method, Property lists TBD.

#### SessionStore

In the SessionManager docs, I made some references to a SessionStore class. This "class" is really an abstract class
meant to be implemented as an interface. It should essentially provide CRUD methods, with User IDs as the keys. I
won't go into too much detail on the abstract class; it should be noted that the onus is on the user of this library
to implement the SessionStore interface (and thereby provide a way of storing user sessions). 

We should provide a Map-backed implementation (perhaps "LocalSessionStore"), which is purely for development and 
testing purposes. Of course, this is not a real solution: storing user-sessions in memory is absurd. However, providing
such an implementation allows users of this library to test and run their servers without actually using a full 
SessionStore.

When we eventually provide ORM services, it should definitely implement this interface, so that the ORMs can be
used as SessionStores without any struggles.
