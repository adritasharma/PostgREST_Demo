# PostgREST Demo

### Create Schema

```create schema api;```

### Create Table

```
create table api.todos (
  id serial primary key,
  done boolean not null default false,
  task text not null,
  due timestamptz
);


```
