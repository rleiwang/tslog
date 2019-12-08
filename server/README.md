# Byte Wedge
This is a demo project to run Byte Wedge in dev mode. Byte Wedge is an OLAP supports append only schemaless JSON documents.
This dev server rebuilds the index for every query. No large json file, it will panic if invalid json is supplied. The demo is to show case the basic query functions of Byte Wedge

## 1. prepare a JSON file
--------------------------

* save a json file, name it test.json. The content of test.json must be valid json. This demo uses movies.json.
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
select count(title) as tc, cast, genres from movie
where cast like '%Brown%' and year >= 2000
group by genres
order by tc desc
limit 10
```

## 3. Supported Full Text Search
--------------------

* Prefix Search
```
where: "cast.contains('^Brown') && year >= 2000"
```

* Suffix Search
```
where: "cast.contains('Brown$') && year >= 2000"
```

* Token full text Search
```
where: "cast.contains('rown') && year >= 2000"
```

* Exact token match
```
where: "cast.contains('^Brown$') && year >= 2000"
```

* Wildcard 
```
where: "cast.contains('Br*n') && year >= 2000"
```

## 4. Query Type

* group by, order by, aggrgate functions (avg, min, max, sum, mean, quantile())
```
select: [ { path: "title", func: count, sort: desc } { path: "cast" } { path: "genres", group: true }]
```

## 5. GraphQL schema

```graphql
scalar Map
scalar Int64

enum SortOrder {
  asc
  desc
}

enum Function {
  avg
  sum
  min
  max
  count
}

enum NodeType {
  text
  int
  float
  bool
}

input Node {
  """
  node path. It supports from root only. 
  """
  path: String!

  """
  Data type of the Node
  """
  type: NodeType = text

  """
  function avg, sum, min, max, count
  """
  func: Function

  """
  quantile calculation. 0.5 is median, 0.99 is 99th percentile
  """
  quantile: Float

  """
  use circllhist, please refer to
  https://www.usenix.org/sites/default/files/conference/protected-files/srecon19emea_slides_hartmann.pdf
  https://promcon.io/2019-munich/slides/prometheus-histograms-past-present-and-future.pdf
  """
  approximate: Boolean = true

  """
  grouping column
  """
  group: Boolean = false

  """
  default is unordered
  """
  sort: SortOrder
}

"""
unix time in milliseconds
"""
input Range {
  begin: Int64!
  end: Int64!
}

type Query {
  """
  time series json document
  """
  json (
    """
    Comma separated data source labels. The labels are supplied to data collector through configuration.
    It supports AND only. env=demo,app=auth, means data source labeled as env=demo and app=auth
    """
    labels: String!,

    """
    Time range of this query. The time must be unix time in milliseconds
    """
    range: Range!,

    """
    Select
    """
    select:[Node!]!,

    where: String,
    limit: Int
  ): [Map!]!
}
```