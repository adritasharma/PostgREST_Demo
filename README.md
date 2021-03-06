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
create table api.employees (
  id serial primary key,
  first_name text not null,
  last_name text not null,
  salary int not null,
  department text not null	
);

```

- **Insert Data**

```
insert into api.employees values
  (1,'Siri','Hunt',10000,'HR'),
  (2,'Mark','Stone', 20000,'Analyst')
```

- **Create a role to use for anonymous web requests**

```
create role web_anon nologin;

grant usage on schema api to web_anon;
grant select on api.employees to web_anon;
grant insert on api.employees to web_anon;
grant update on api.employees to web_anon;
grant delete on api.employees to web_anon;

```

- **Create as dedicated role for connecting to the database authenticator and grant it the ability to switch to the web_anon role**

```
create role authenticator noinherit login password 'mysecretpassword';
grant web_anon to authenticator;
```

## Run PostgREST

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

- Go to http://localhost:3000/employees to get the list of employees

```
GET  http://localhost:3000/employees

[
    {
    "id": 1,
    "first_name": "Siri",
    "last_name": "Hunt",
    "salary": 10000,
    "department": "HR"
    },
    {
    "id": 2,
    "first_name": "Mark",
    "last_name": "Stone",
    "salary": 20000,
    "department": "Analyst"
    }
]
```


## Filtering 

**Common Operators**

<table border="1" class="docutils">
<colgroup>
<col width="10%">
<col width="20%">
<col width="69%">
</colgroup>
<thead valign="bottom">
<tr><th class="head">Abbreviation</th>
<th class="head">In PostgreSQL</th>
<th class="head">Meaning</th>
</tr>
</thead>
<tbody valign="top">
<tr class="row-even"><td>eq</td>
<td><code><span class="pre">=</span></code></td>
<td>equals</td>
</tr>
<tr><td>gt</td>
<td><code><span class="pre">&gt;</span></code></td>
<td>greater than</td>
</tr>
<tr class="row-even"><td>gte</td>
<td><code><span class="pre">&gt;=</span></code></td>
<td>greater than or equal</td>
</tr>
<tr><td>lt</td>
<td><code><span class="pre">&lt;</span></code></td>
<td>less than</td>
</tr>
<tr class="row-even"><td>lte</td>
<td><code><span class="pre">&lt;=</span></code></td>
<td>less than or equal</td>
</tr>
<tr><td>neq</td>
<td><code><span class="pre">&lt;&gt;</span></code> or <code><span class="pre">!=</span></code></td>
<td>not equal</td>
</tr>
<tr class="row-even"><td>like</td>
<td><code><span class="pre">LIKE</span></code></td>
<td>LIKE operator (use * in place of %)</td>
</tr>

<tr class="row-even"><td>in</td>
<td><code><span class="pre">IN</span></code></td>
<td>one of a list of values, e.g. <code><span class="pre">?a=in.(1,2,3)</span></code>
??? also supports commas in quoted strings like
<code><span class="pre">?a=in.("hi,there","yes,you")</span></code></td>
</tr>
<tr><td>is</td>
<td><code><span class="pre">IS</span></code></td>
<td>checking for exact equality (null,true,false)</td>
</tr>
<tr class="row-even"><td>fts</td>
<td><code><span class="pre">@@</span></code></td>
<td><a class="reference internal" href="#fts"><span class="std std-ref">Full-Text Search</span></a> using to_tsquery</td>
</tr>
<tr class="row-even"><td>cs</td>
<td><code><span class="pre">@&gt;</span></code></td>
<td>contains e.g. <code><span class="pre">?tags=cs.{example,</span> <span class="pre">new}</span></code></td>
</tr>
<tr><td>cd</td>
<td><code><span class="pre">&lt;@</span></code></td>
<td>contained in e.g. <code><span class="pre">?values=cd.{1,2,3}</span></code></td>
</tr>
<tr class="row-even"><td>not</td>
<td><code><span class="pre">NOT</span></code></td>
<td>negates another operator, see below</td>
</tr>
</tbody>
</table>


### Vertical Filtering (Columns)

**Select** 

```
GET  http://localhost:3000/employees?select=first_name,department

[
    {
        "first_name": "Siri",
        "department": "HR"
    },
    {
        "first_name": "Mark",
        "department": "Analyst"
    }
]
```


**Renaming Columns** 

```
GET  http://localhost:3000/employees?select=Name:first_name

[
    {
        "Name": "Siri"
    },
    {
        "Name": "Mark"
    }
]
```

**Casting Columns** 

```
GET  http://localhost:3000/employees?select=first_name,salary::text

[
    {
        "first_name": "Siri",
        "salary": "10000"
    },
    {
        "first_name": "Mark",
        "salary": "20000"
    }
]
```


### Horizontal Filtering (Rows)

**Equals**

```
GET  http://localhost:3000/employees?id=eq.1

Response:

[
    {
        "id": 1,
        "first_name": "Siri",
        "last_name": "Hunt",
        "salary": 10000,
        "department": "HR"
    }
]
```


**Less Than**

```
GET  http://localhost:3000/employees?salary=lt.15000

Response:

[
    {
        "id": 1,
        "first_name": "Siri",
        "last_name": "Hunt",
        "salary": 10000,
        "department": "HR"
    }
]
```

**Greater than or equal**

```
GET  http://localhost:3000/employees?salary=gte.10000

Response:

[
    {
        "id": 1,
        "first_name": "Siri",
        "last_name": "Hunt",
        "salary": 10000,
        "department": "HR"
    },
    {
        "id": 2,
        "first_name": "Mark",
        "last_name": "Stone",
        "salary": 20000,
        "department": "Analyst"
    }
]
```

**Multiple Operator**

```
GET  http://localhost:3000/employees?or=(salary.gte.15000,salary.lte.12000)

Response:

[
    {
        "id": 1,
        "first_name": "Siri",
        "last_name": "Hunt",
        "salary": 10000,
        "department": "HR"
    },
    {
        "id": 2,
        "first_name": "Mark",
        "last_name": "Stone",
        "salary": 20000,
        "department": "Analyst"
    }
]
```

## Ordering 

**Default ascending**

```
GET  http://localhost:3000/employees?select=id,first_name&order=first_name

Response:

[
    {
        "id": 2,
        "first_name": "Mark"
    },
    {
        "id": 1,
        "first_name": "Siri"
    }
]
```

**Descending**

```
GET  http://localhost:3000/employees?order=salary.desc

Response:

[
    {
        "id": 2,
        "first_name": "Mark",
        "last_name": "Stone",
        "salary": 20000,
        "department": "Analyst"
    },
    {
        "id": 1,
        "first_name": "Siri",
        "last_name": "Hunt",
        "salary": 10000,
        "department": "HR"
    }
]
```


## Insert 

**Single Insert**

```
POST  http://localhost:3000/employees

BODY:

{
    "id": 3,
    "first_name": "John",
    "last_name": "Doe",
    "salary": 30000,
    "department": "Manager"
}
```

**Bulk Insert**

```
POST  http://localhost:3000/employees

BODY:

[
    {
        "id": 4,
        "first_name": "Jane",
        "last_name": "Doe",
        "salary": 25000,
        "department": "Admin"
    },
    {
        "id": 5,
        "first_name": "Tom",
        "last_name": "Doe",
        "salary": 10000,
        "department": "Staff"
    }
]
```

## Update 

**Single Update**

```
PUT  http://localhost:3000/employees?id=eq.3

BODY:

{
    "id": 3,
    "first_name": "John",
    "last_name": "Doe",
    "salary": 60000,
    "department": "Manager"
}
```

**UPSERT**

We can make an **UPSERT** with `POST` and the `Prefer: resolution=merge-duplicates` header:

```
POST  http://localhost:3000/employees

BODY:

[
    {
        "id": 3,
        "first_name": "John",
        "last_name": "Doe",
        "salary": 50000,
        "department": "Manager"
    },
    {
        "id": 6,
        "first_name": "Deepika",
        "last_name": "Das",
        "salary": 40000,
        "department": "Analyst"
    }
]
```

## Delete 

**Single Delete**

```
DELETE  http://localhost:3000/employees?id=eq.6

BODY:

{
    "id": 6
}
```

# Function

### Function Without Parameters

**Create Function**

```
CREATE FUNCTION api.totalsalary()
RETURNS integer AS $$
 SELECT
 SUM(salary)
 FROM api.Employees
$$ LANGUAGE SQL IMMUTABLE;
```


**Test Function**

```
select api.totalsalary()
```
**Request**

Functions are exposed under the `/rpc` prefix

- Restart the server after creating any Function `postgrest connection.conf`:


```
POST  http://localhost:3000/rpc/totalsalary
```

### Function With Parameters

**Create Function**

```
CREATE FUNCTION api.updatesalary(percentage INT)
RETURNS void AS $$
 UPDATE api.Employees 
 SET salary= salary + (salary * percentage / 100);
$$ LANGUAGE SQL VOLATILE;
```


**Test Function**

```
select api.updatesalary(10)
```
**Request**

```
POST  http://localhost:3000/rpc/updatesalary

BODY:

{
    "percentage": 10
}
```

# Stored Procedures

**Create Procedure**

```
CREATE OR REPLACE PROCEDURE api.updatesalary  
(  
    percentage INT
)  
LANGUAGE plpgsql AS  
$$  
BEGIN         
   UPDATE api.Employees 
   SET salary= salary + (salary * percentage / 100);
END  
$$;
```

**Test Procedure**

```
CALL updatesalary(10)
```