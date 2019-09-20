# Working with Diesel

---

# You should have

- Diesel CLI 1.4.0
- Rust 1.31 or later
- PostgreSQL

---

# Exercises

`github.com/sgrif/diesel_workshop`

---

# What even is Diesel?

---

# Diesel CLI Manages Your Schema

^ Run Diesel setup, note the generated migration, schema.rs, that the database
has been created

---

# `schema.rs` represents your database

---

[.code-highlight: all]
[.code-highlight: 4]

```rust



table! {
  posts {
    id -> Integer,
    title -> Text,
    body -> Nullable<Text>,
  }
}
```

---

# `posts::table`

---

```rust, [.highlight: 5]



table! {
  posts {
    id -> Integer,
    title -> Text,
    body -> Nullable<Text>,
  }
}
```

---

## `posts::table.select(posts::id)`

---

# `RunQueryDsl`

## How we execute queries

---

```rust







sql_query("INSERT INTO ...").execute(&conn)
```

---
[.code-highlight: none]
[.code-highlight: 5]
[.code-highlight: 6]
[.code-highlight: 7]
[.code-highlight: 8]

```rust





posts::table.load(&conn) // load multiple records
posts::table.get_results(&conn) // alias for load
posts::table.first(&conn) // add LIMIT 1 and get one record
posts::table.get_result(&conn) // get one record (no LIMIT)
```

---

[.build-lists: true]

# `Queryable` deserializes results

- `Integer` -> `i32`
- `Text` -> `String`
- `Nullable<T>` -> `Option<U>`
- `(Integer, Text)` -> `(i32, String)`

---

# Exercise time!

---

## `git checkout 01`

### Make the tests in `01_queryable` pass

---

[.autoscale: true]
[.build-lists: false]

# Cheatsheet

- `posts::table.select(posts::id)`
  - select a single column
- `some_query.load(&conn)`
  - load the results of a SQL query
- `#[derive(Queryable)]`
  - automatically derive `Queryable` for a struct
- `println!("{}", diesel::debug_query::<diesel::pg::Pg, _>(&query));`
  - print the generated SQL for a Diesel query

---

# Insertion

---

[.code-highlight: all]
[.code-highlight: 6]
[.code-highlight: 7]

```rust






diesel::insert_into(table)
  .values(data)
```

---

# Things you can pass to `.values`

- `posts::title.eq("...")`
- `(title.eq("..."), body.eq("..."))`
- `#[derive(Insertable)]`
- A `Vec` or slice of any of these

---
[.code-highlight: all]
[.code-highlight: 5]
[.code-highlight: 2]
[.code-highlight: 7]
[.code-highlight: 8]

```rust


use crate::schema::posts;

#[derive(Insertable)]
#[table_name = "posts"]
struct NewPost {
  title: String,
  body: Option<String>,
}
```

---

## When should I use a struct?

---

# Default to the tuple form

```rust
(
  title.eq("..."),
  body.eq("..."),
)
```

vs

```
NewPost {
  title: "...",
  body: "...",
}
```

---

## Use `#[derive(Insertable)]` when your data is already in a struct

---

# Running the query

- `.execute(&conn)` // Run the query without returning data
- `.get_result(&conn)` // Insert a single row and get the data
- `.get_results(&conn)` // Insert many row and get the data

---

# Exercise time!

---

# `git checkout 03`

## Make the tests in `02_insert` pass

---
[.autoscale: true]
[.build-lists: false]

# Cheatsheet

- `insert_into(table).values(data)`
- `(title.eq("..."), body.eq("..."))`
- `.execute`
  - Run the query without loading anything
- `.get_results`
  - Return the rows that were inserted
- `#[derive(Insertable)]`
  - Don't forget to add `#[table_name = "..."]`!

---

# Updates

---

[.code-highlight: all]
[.code-highlight: 5]
[.code-highlight: 6]
[.code-highlight: 7]
[.code-highlight: 5]

```rust





diesel::update(target)
  .set(changes)
  .execute(&conn)
```

---

# Things you can pass to `update`

- `posts::table`
  - Updates the whole table
- `posts::table.find(1)`
  - Update the record with ID 1
- `&Post`

---
[.code-highlight: all]
[.code-highlight: 5]
[.code-highlight: 6]
[.code-highlight: 3]
[.code-highlight: 8]

```rust



use crate::schema::posts;

#[derive(Identifiable)]
#[table_name = "posts"]
struct Post {
  id: i32,
}
```

---

```rust, [.highlight: 6]





diesel::update(target)
  .set(changes)
  .execute(&conn)
```

---
[.code-highlight: all]
[.code-highlight: 5]
[.code-highlight: 2]
[.code-highlight: 7]
[.code-highlight: 8]
[.code-highlight: 9]

```rust


use crate::schema::posts;

#[derive(AsChangeset)]
#[table_name = "posts"]
struct Post {
  id: i32,
  title: String,
  body: Option<String>,
}
```

---

# Exercise time!

---

# `git checkout 05`

## Make the tests in `03_update` pass

---
[.autoscale: true]
[.build-lists: false]

# Cheatsheet

- `update(target).set(changes)`
- `(title.eq("..."), body.eq("..."))`
- `.execute`
  - Run the query without loading anything
- `.get_results`
  - Return the rows that were updated
- `#[derive(Identifiable, AsChangeset)]`
  - Don't forget to add `#[table_name = "..."]`!

---

# Why do we care so much about returning inserted/updated rows?

---

# Why do I need so many structs?

---

# Intermediate querying

---

# `QueryDsl` manipulates select statements

^ Open the docs, talk about how to find things

---

# `expression_methods` adds methods for various operators

---
[.autoscale: true]
[.build-lists: true]

# What is an operator named in Diesel?

- Does Rust have the same operator? (e.g. `>`)
  - Use the name that Rust uses (e.g. `.gt`)
- Is the operator a word? (e.g. `LIKE`)
  - Use that word (e.g. `.like`)
- Does the operator have a simple description? (e.g. `@>` for arrays)
  - Use that description (e.g. `.contains`)
- `<<%`
  - ¯\\\_(ツ)\_/¯

^ Returns true if its second argument has a continuous extent of an ordered trigram set that matches word boundaries, and its similarity to the trigram set of the first argument is greater than the current strict word similarity threshold set by the `pg_trgm.strict_word_similarity_threshold` parameter.

---

# `diesel::dsl` contains various other top level functions

- `sum`
- `not`
- `now`
- etc

---

# Exercise time!

---

## `git checkout 07`

### Make the tests in `04_intermediate_querying` pass

---

[.autoscale: true]
[.build-lists: false]

# Cheatsheet

- `query.filter(...)`
  - Add to the `WHERE` clause
- `query.order_by(...)
  - Add an `ORDER BY` clause
- `column.like("%some string%")`
  - Returns whether `column` contains "some string"

---

# Associations

---

# Diesel's associations may be different than you're used to

---

# Unambiguous whether data is loaded

---

# Unidirectional

---

# Granular

---

# We're going to work through this exercise together
