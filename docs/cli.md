# A few of the initial features for the marathon cli

- ### Initialise the main app.
  This would be the starting point for the server. It would contain configuration for the whole webserver like authentication backend, default database, fallback database , url , media routes , static files route etc.
- ### New apps
  Create New sub apps which would be mostly plug and play.
- ### Create a super user
  This would create an admin user which has superuser privileges i.e will be the master user.
  Can have multiple superusers.
- ### Start the server.
  The user can start the server on a specific port using this. It would support hot reloading.
- ### Some commands for the database.
  Some basic commands for the db like load fixtures, flush database, makemigrations , migrate db.
