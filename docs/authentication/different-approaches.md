# This page describes how different frameworks handle authentication

&nbsp;

## Django

#### There is not much that goes into wile handling authentication in Django. Here's a brief overview of it.

So the request coming in travels first to a session handler which sees if the session is valid its expiry date and suspicious activity so it deletes its session and a whole bunch of other things.
Then the request travels down to authentication middleware.

Before we jump into what authentication middleware does let's discuss user objects in Django. So there are 2 types of users by default [Anonymous user](https://docs.djangoproject.com/en/3.1/ref/contrib/auth/#anonymoususer-object) and [normal user](https://docs.djangoproject.com/en/3.1/ref/contrib/auth/#user-model). Normal User has fields like username, hashed password, first name, last name email, and different functions like is_authenticated. As in the case of Anonymous user as the name suggests it has methods like is_authenticated always return false.
So coming back to authentication middleware, once the sessions in the request are validated " sort of "by the session middleware auth tries to see if there is a user that exists and attach that user object anonymous or existing user to the request object to be accessed internally easily. It also caches the users but I don't know-how.
I find everything here quite structured. We take the heavy lifting from the user so that he doesn't have to worry about implementing jwt or sessions and all of that stuff and just focus on building stuff. Before all the session and auth middleware it also does a lot of other validations to prevent attacks by using csrf token, etc which would also be good to implement. I am gonna link some more resources down here if you wanna read more about it.

- [Using the Django authentication system](https://docs.djangoproject.com/en/3.1/topics/auth/default/)
- [How to user Sessions](https://docs.djangoproject.com/en/3.1/topics/http/sessions/)
