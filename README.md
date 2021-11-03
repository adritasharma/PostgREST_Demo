# PostgREST Demo

## Setting up in Windows

**Install PostgREST**

- Download from https://github.com/PostgREST/postgrest/releases/latest
- Extract postgrest-v8.0.0-windows-x64.zip, we will find `postgrest.exe`
- Keep `postgrest.exe` in the project folder
- Run command `postgrest`, it should print the PostgRest version

  ````
  postgrest
  ````

## Database Setup

- Create Database PostgRestDemo

- **Create Schema**

```
create schema api;
```

- **Create Table**

```
create table api.todos (
  id serial primary key,
  done boolean not null default false,
  task text not null
);

```

- **Insert Data**

```
insert into api.todos (task) values
  ('finish tutorial'), ('pat self on back');
```

- **Create a role to use for anonymous web requests**

```
create role web_anon nologin;

grant usage on schema api to web_anon;
grant select on api.todos to web_anon;

```

- **Create as dedicated role for connecting to the database authenticator and grant it the ability to switch to the web_anon role**

```
create role authenticator noinherit login password 'mysecretpassword';
grant web_anon to authenticator;
```

### Run PostgREST

- Create a file `connection.conf` inside the project folder

```
# The standard connection URI format - "postgres://user:pass@host:5432/dbname"
db-uri = "postgres://authenticator:password@localhost:5432/PostgRestDemo"

# The name of which database schema to expose to REST clients
db-schema = "api"

# The database role to use when no client authentication is provided.
db-anon-role = "web_anon"
```

- Run the server:

```
postgrest connection.conf
```

- We should see

```
Listening on port 3000
Attempting to connect to the database...
Connection successful
```

- Go to http://localhost:3000/todos to get the list of todos