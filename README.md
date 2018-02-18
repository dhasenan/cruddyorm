# CruddyORM

CruddyORM is a simple object relational mapping library for postgresql and vibe.d.

## Why use it?

Because you're tired of writing manual converters from SQL result set rows to your structs.

It's best for:

* simple projects
* using postgres
* using vibe.d
* using SQL directly


## How do I use it?

```D
import cruddyorm;
import datefmt;

struct Person
{
    UUID id;
    string name;
    SysTime birthday;
}

void main()
{
    cruddyorm.connectionString = "dbname=people username=minnie password=mouse";
    auto p = Person(
        UUID.randomUUID,
        "Minnie Mouse",
        datefmt.parse("1928-11-18", ISO8601FORMAT));

    // Creates and closes its own connection
    insert(p);

    // Performs several queries with the same connection
    inConnection!((conn)
    {
        conn.queryConn!Person("SELECT * FROM persons WHERE name = ?", p.name)
            .each!writeln;
        conn.insertConn(someRandomPerson());
    });
}
```


## Cruddiness

* It only supports one database.
* It assumes that you only want to persist fields, not properties.
* It assumes that the id field is a UUID named `id`.
* It uses std.experimental.logger instead of vibe's logging. Vibe doesn't offer a topic-based
  logging configuration, so that wouldn't let the ORM be stricter about its logging than
  application code.
* It doesn't do connection pooling.
* It supports a narrow collection of types natively.
* It assumes your tables are named according to an overly simple convention: type "Review" is backed
  by table "reviews", type "Party" is backed by table "partys", type "Child" is backed by table
  "childs", etc.

A lot of these are problems that will be fixed in the fullness of time.


## Why not hibernated?

[Hibernated](http://code.dlang.org/packages/hibernated) is a good option. You should prefer
hibernated if it works for you. It doesn't work with vibe.d, though, and it doesn't work well with
std.uuid.


## Why not entity?

[Entity](http://code.dlang.org/packages/entity) is another good option. It promotes the use of a
query builder, which is a good thing for ensuring your queries have correct types. It also insulates
you from writing DB-specific SQL.

In the real world, you're going to write DB-specific SQL, and a SQL generator will occasionally
produce bad results. I didn't see a way to write raw SQL queries.
