# Byte Wedge
This is a demo project to run Byte Wedge in dev mode. Byte Wedge is an OLAP supports append only schemaless JSON documents.
It rebuilds the index for every query. No large json file, it will panic if invalid json is supplied.

## 1. prepare a JSON file
--------------------------

* save a json file, name it test.json
```sh
❯❯❯ curl -o test.json https://raw.githubusercontent.com/prust/wikipedia-movie-data/master/movies.json
```

* start server in the same directory. server is MacOS binary
```sh
❯❯❯ ls
server
test.json
❯❯❯ ./server
2019/12/08 08:56:28 connect to http://localhost:8080/ for GraphQL playground
```

## 2. Query
--------------------

* Download and install [GraphQL IDE](https://github.com/prisma/graphql-playground)

setup url endpoint on GraphQL IDE
```url
http://localhost:8080
```

* note: where clause expression is [C style CEL](https://github.com/google/cel-spec), path is dot notation json path

* exact token match
```graphql
query {
  json(
    labels: "env=dev"
    range: { begin: 0, end: 0 }
    select: [{ path: "title"}, {path: "year", sort: asc} {path:"cast"}]
    where: "genres.contains('^Drama$')"
    limit: 10
  )
}
```
equivalent SQL
```sql
select title, year, cast from movie where generes = 'Drama' limit 10
```

* full text search
```graphql
query {
  json(
    labels: "abc=adf"
    range: { begin: 0, end: 0 }
    select: [
      { path: "title", func: count, sort: desc }
      { path: "cast" }
      { path: "genres", group: true }
    ]
    where: "cast.contains('Brown') && year >= 2000"
    limit: 10
  )
}
```
equivalent SQL
```sql
select count(title), cast, genres from movie where cast like '%Brown%' and year >= 2000 group by genres limit 10
```