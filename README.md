# Ecto-notes

### Ecto Modules: ###

* REPO: *
Repo is the heart of Ecto and acts as a kind of proxy for your database. All communication to and from the database goes through Repo.

* Query: *
The Query module contains Ecto’s powerful but elegant API for writing queries.

* Schema *
A schema is a kind of map, from database tables to your code. The Schema module contains tools to help you create these maps with ease. The best part is Ecto schemas are very flexible—you’re not locked into a simple one-to-one relationship between your tables and your structs.

* Changeset: *
Many database layers have one or two kinds of change. Ecto understands that one size does not fit all, so it provides the changeset: a data structure that captures all aspects of making a change to your data. The Changeset module provides functions for creating and manipulating changesets, allowing you to structure your changes in a way that is safe, flexible, and easy to test.

* Multi: *
You often need to coordinate several database changes simultaneously, where they must all succeed or fail together. The transaction function works great for simple cases, but the Multi module can handle even very complex cases while still keeping your code clean and testable.

* Migration: *
Change happens. As your app grows and evolves, so too must the underlying database. Changing the structure of a database can be tricky, particularly when multiple developers are involved, but Migration helps you coordinate these changes so that everyone stays in sync.

### Repo Module: ###
The Repo module is the heart of Ecto. Repo contains a lot of functions you’d expect: get, insert, update, delete, and the like. You never call these functions directly. Instead, you create your own Repo module that lives in your app’s codebase,then integrate Ecto.
Repo’s functions with Elixir’s use macro. The otp_app option is required. It tells Ecto where to find the configuration values it needs to connect to your database.

 In file lib/music_db/repo.ex:

 ```elixir
defmodule MusicDB.Repo do
  use Ecto.Repo,
  otp_app: :music_db,  #this is otp_app option
  adapter: Ecto.Adapters.Postgres
end
```
So otp_app will look into  config/dev.exs file and there it looks for `music_db` option

```
config :music_db, MusicDB.Repo,
  database: "music_db",
  username: "postgres",
  password: "postgres",
  hostname: "localhost"
```

This is where the otp_app option comes into play. It tells Ecto to look here tobfind the values it needs to communicate with the database. Different database adapters may require different settings.In our example, we used separate values for these settings but it’s possible
to combine all of these into a single url parameter. The format for the URL
should be ecto://USERNAME:PASSWORD@HOSTNAME/DATABASE_NAME. For our configuration, we could use this:
config :music_db, MusicDB.Repo,
url: "ecto://postgres:postgres@localhost/music_db"


## Playing with Ecto: ##

### INSERT OPERATION ###
To *insert* record in database:
```
alias MusicDB.Repo
Repo.insert_all("artists", [[name: "John Coltrane"]])
```
For inserting multiple records:

```
Repo.insert_all("artists",
[[name: "Max Roach", inserted_at: DateTime.utc_now()],
[name: "Art Blakey", inserted_at: DateTime.utc_now()]])
#=> {2, nil}
```

In these examples, we specified the values using keyword lists, but you can also use maps. This snippet will do the exact same thing as the previous one:

```
Repo.insert_all("artists",
[%{name: "Max Roach", inserted_at: DateTime.utc_now()},
%{name: "Art Blakey", inserted_at: DateTime.utc_now()}])
#=> {2, nil}
```
### UPDATE OPERATION ###
To *update* records, we can use the update_all function:

Repo.update_all("artists", set: [updated_at: DateTime.utc_now()])
#=> {1, nil}

`set` tells you which column of the table you want to change.`update_all` provides some other options for making changes:

* inc: This increments the given field by the given value; we can decrement by supplying a negative number
* push: This works on columns containing an array, and pushes the given value onto the end of the array
* pull: This also works on array columns—it removes the given value from the array

Refer this for other options: https://hexdocs.pm/ecto/Ecto.Query.html#update/3-operators

### DELETE OPERATION ###
For Deleting bunch of records:
`Repo.delete_all("tracks")`
#=> {1, nil}

### Getting values back: ###
The standard return value for the *_all functions is {some_number, nil}. The first field is the number of rows effected and second is return value. Right now , we are not returning anything that it is nil.

The returning option lets us specify any values we’d like returned to us after the operation completes. This option takes a list of the field names we’re interested in, and Ecto returns the values as a map. Note that this option works in Postgres, but not in MySQL.

We can use returning to have Ecto show us the IDs after inserting the records:
```
Repo.insert_all("artists", [%{name: "Max Roach"},
%{name: "Art Blakey"}], returning: [:id, :name])
#=> {2, [%{id: 12, name: "Max Roach"}, %{id: 13, name: "Art Blakey"}]}
```

***This option works with any of the *all functions.***


### Executing queries: ###
The Ecto.Adapters.SQL module has a function called query that will take good old-fashioned SQL:
```
Ecto.Adapters.SQL.query(Repo, "select * from artists where id=1")
```
*VV Important:*

Ecto also makes this function available from Repo—this shortcut doesn’t appear in the documentation for Repo but it’s simpler to call:
`Repo.query("select * from artists where id=1")`

SQL in string form can get pretty clumsy and even unsafe, particularly as
you start adding dynamic values. However, this approach can be useful when
debugging, or if you want to run a quick SQL statement within an IEx session.


### Customizing your Repo ###

There may be times when you find yourself calling some particular Repo functions over and over with the same set of options, or maybe you’d like to add some behavior that Repo doesn’t currently have. Fortunately, the Repo module you created is a plain old Elixir module just like any other, so it’s possible to add customized behavior just by adding more functions (Here `Repo` module we are talking about is lib/music_db/repo.ex).
Suppose we want to do lots of counting in our app on different models. Getting the number of records in a table is fairly easy with Repo’s aggregate.

`Repo.aggregate("albums", :count, :id)`
`aggregate` function gives us access to a number of aggregate functions supplied by the underlying database: count, avg, min, max, sum, and so on

This function is simple enough, but if we know we’ll be doing this often and want to be truly lazy, we can add a custom count function to our Repo module to save some typing. In the sample project, open lib/music_db/repo.ex and add this function:
```elixir
def count(table) do
  aggregate(table, :count, :id)
end
```

Another useful customization is adding an implementation of the init callback. This runs when Ecto first initializes and allows you to add or override configuration parameters.


V.V.V.Important:

When loading a database connection URL from an environment variable. The init callback is where you’d want to do that:
```
def init(_, opts) do
  {:ok, Keyword.put(opts, :url, System.get_env("DATABASE_URL"))}
end
```


# Ecto Query: #

`To make these command run import *Ecto.query* and to execte run *Repo.all(q)*`

If we want to take dynamic value from user to make the query we could directly interpolate it in the query. But, it will throw error.
```
artist_name = "Bill Evans"
q = from "artists", where: [name: artist_name], select: [:id, :name]
```
** (Ecto.Query.CompileError) variable `artist_name` is not a valid query
expression. Variables need to be explicitly interpolated in queries with ^

Explanation for error:  Ecto’s query syntax is implemented using macros, so the rules are a little different. We need to alert the macro that we’re using an expression that needs evaluating by adding ^ .

Correct Syntax:
```
artist_name = "Bill Evans"

```

***  But if you’ve got a more complex expression, you need to wrap it in parentheses, like this: ^("Bill" <> " Evans") ***


### Query Bindings ###
We can make a simple `where` query as below
` q = from "artists", where: [name: "Bill Evans"], select: [:id, :name] `
here `where` checks if equality of `Bill Evans` with `name` variable. But we cannot use `===` operator as below.
` q = from "artists", where: name == "Bill Evans", select: [:id, :name]`
Because Ecto can’t easily figure out what name is supposed to be. A variable? A function or macro defined elsewhere? We need a way to tell Ecto that name is a column in our artists table. We can do this with query bindings.

You create a query binding by using `in` along with the usual `from`. It works a lot like table aliases in SQL and effectively gives you a variable for referring to your table throughout your query. Our problematic query can be rewritten like this:
`q = from a in "artists", where: a.name == "Bill Evans", select: [:id, :name]`

### Query Expressions ###
Ecto provides a long list of functions that you can use with where and other query keywords. These are documented in detail in the Ecto.Query.API module, but here are a few examples to give you an idea of what’s possible:

```
# like statements
q = from a in "artists", where: like(a.name, "Miles%"), select: [:id, :name]
# checking for null
q = from a in "artists", where: is_nil(a.name), select: [:id, :name]
# checking for not null
q = from a in "artists", where: not is_nil(a.name), select: [:id, :name]
# date comparison - this finds artists added more than 1 year ago
q = from a in "artists", where: a.inserted_at < ago(1, "year"),
select: [:id, :name]
```

### Inserting Raw SQL ###
There might be cases where your database exposes some specialized function that Ecto doesn’t support. The fragment function
gives you an escape hatch for writing bits of raw SQL that get inserted verbatim into the query.
Here we use fragment so we can call the Postgres lower function:
```
q = from a in "artists",
  where: fragment("lower(?)", a.name) == "miles davis",
  select: [:id, :name]
```


### To see generated raw sql ###
To see the raw sql generated you can use `Ecto.Adapters.SQL.to_sql`:
```
Ecto.Adapters.SQL.to_sql(:all, Repo, q)
```

If this is something you think you’ll be using a lot, you can extend Ecto’s query API by adding your own macro and importing it into your module:

```
defmacro lower(arg) do
  quote do: fragment("lower(?)", unquote(arg))
end
```
Then the query could be rewritten like this:
```
q = from a in "artists", where: lower(a.name) == "miles davis",select: [:id, :name]
```



### Combining Results with union and union_all###

To combine results of different queries, SQL provides the UNION operator. For this to work, the two queries need to have result sets with the same column names and data type.

```
tracks_query = from t in "tracks", select: t.title
union_query = from a in "albums", select: a.title, union: ^tracks_query
Repo.all(union_query)
```

** INTERSECT and EXCEPT **

INTERSECT:
Ecto supports INTERSECT and EXCEPT, and the answer is yes. We can use intersect: to get a list of album titles that are also track titles:
```
tracks_query = from t in "tracks", select: t.title
intersect_query = from a in "albums", select: a.title, intersect: ^tracks_query
```

EXCEPT:
```
tracks_query = from t in "tracks", select: t.title
except_query = from a in "albums", select: a.title,
  except: ^tracks_query
```

### Ordering and Grouping ###
The order by and group by expressions in SQL are available in Ecto via the order_by and group_by keywords.

** order_by:**
```
q = from a in "artists", select: [a.name], order_by: a.name
Repo.all(q)
```
It is by default `ascending`. If you want to do descending, you can specify that.
```
q = from a in "artists", select: [a.name], order_by: [desc: a.name]
Repo.all(q)
```

Specifying multiple columns for order_by
```
q = from t in "tracks", select: [t.album_id, t.title, t.index], order_by: [t.album_id, t.index]
Repo.all(q)
```

Multiple `order_by` with combinations of `asc` and `desc`
```
q = from t in "tracks", select: [t.album_id, t.title, t.index], order_by: [desc: t.album_id, asc: t.index]
Repo.all(q)
```

*** VV. Important: ***
Different database have different importance for NULL value while using `order_by`. You can specify the ordering using
`:asc_nulls_last, :asc_nulls_first, :desc_nulls_last, or:desc_nulls_first `.

```
q = from t in "tracks", select: [t.album_id, t.title, t.index],
order_by: [desc: t.album_id, asc_nulls_first: t.index]
Repo.all(q)
```

** group_by: **
```
q = from t in "tracks", select: [t.album_id, sum(t.duration)],
group_by: t.album_id
```

Suppose we wanted to refine this further and only return the albums whose total length is longer than one hour (3600 seconds). A where clause won’t help us here.
We’ll take the same query and add having to include only the results that have a total duration longer than 3600 seconds:
```
q = from t in "tracks", select: [t.album_id, sum(t.duration)],
group_by: t.album_id,
having: sum(t.duration) > 3600
Repo.all(q)
```

### Join ###
To create the join, we’ll use the join keyword to specify the table, and the on keyword to specify the column.
```
q = from t in "tracks", join: a in "albums", on: t.album_id == a.id
```

We can add a `where` clause to find the very long tracks:

```
q = from t in "tracks",
  join: a in "albums", on: t.album_id == a.id,
  where: t.duration > 900,
  select: [a.title, t.title]
Repo.all(q)

#=> [["Cookin' At The Plugged Nickel", "No Blues"],
#=> ["Cookin' At The Plugged Nickel", "If I Were A Bell"]]
```

This works, but the result is a little hard to read. We can clean things up by changing our select statement. Instead of expressing the select as a list of columns, we can provide a map. Ecto will then return the result as list of maps, each
using the structure we provide:

```
q = from t in "tracks",
  join: a in "albums", on: t.album_id == a.id,
  where: t.duration > 900,
  select: %{album: a.title, track: t.title}
Repo.all(q)
#=> [%{album: "Cookin' At The Plugged Nickel", track: "No Blues"},
#=> %{album: "Cookin' At The Plugged Nickel", track: "If I Were A Bell"}]
```

** By default, the join macro performs an inner join, but other flavors of joins are
available as well: left_join, right_join, cross_join, and full_join. **

If we want to join on multiple tables. Elixir’s keyword lists allow you to specify the same keyword more than once.More joins is simply a matter of adding more join and on options.

```
q = from t in "tracks",
  join: a in "albums", on: t.album_id == a.id,  # Notice here
  join: ar in "artists", on: a.artist_id == ar.id, # Notice here
  where: t.duration > 900,
  select: %{album: a.title, track: t.title, artist: ar.name}
Repo.all(q)
#=> [%{album: "Cookin' At The Plugged Nickel", artist: "Miles Davis",
#=> track: "If I Were A Bell"},
#=> %{album: "Cookin' At The Plugged Nickel", artist: "Miles Davis",
#=> track: "No Blues"}]
```

### Composing Queries (Composibility) ###

Ecto allows us to break up large queries into smaller pieces that can be reassembled at will. This makes them easier to work with, and allows you to re-use parts of queries in more than one place.

How to write common query (Queryable) ?

*VV.Impo*: We’ve always been using "strings" on the right side of the in expression (for example, from a in "albums").  That "string" has always represented the name of a table in our database. But the in expression is actually looking for any data type that has implemented the Ecto.Queryable protocol.

The "Ecto.Queryable" protocol specifies only one function that needs to be implemented: to_query. So you can think of Queryable as “a thing that can be queried.”


#### Extracting Parts of Queries ####
Problem:

```
#First Query
q = from a in "albums",
  join: ar in "artists", on: a.artist_id == ar.id,
  where: ar.name == "Miles Davis",
  select: [a.title]
Repo.all(q)
#=> [["Cookin' At The Plugged Nickel"], ["Kind Of Blue"]]

#Second Query
q = from a in "albums",
  join: ar in "artists", on: a.artist_id == ar.id,
  join: t in "tracks", on: t.album_id == a.id,
  where: ar.name == "Miles Davis",
  select: [t.title]
Repo.all(q)
```
These both query is almost identical. Except in second query we have extra `join` with different `select` statement.

First, let’s extract out the parts that are identical into a separate Query. Both queries refer to albums by Miles Davis, so we can break that logic out into its own query:

```
albums_by_miles = from a in "albums",
  join: ar in "artists", on: a.artist_id == ar.id,
  where: ar.name == "Miles Davis"
```
This is not complete query, it’s missing the select expression.

To make our first query that just fetches the album titles, we use our "albums_by_miles" query, and add select:
```
album_query = from a in albums_by_miles, select: a.title
#=> #Ecto.Query<from a0 in "albums", join: a1 in "artists",
#=> on: a0.artist_id == a1.id, where: a1.name == "Miles Davis",
#=> select: a0.title>
```
Here instead of passing table name after `in` we passed `other query` (which is Queryable).  Ecto takes that original query, then adds the select we’ve provided here.

What happens to the query bindings when a query is built up like below.
```
albums_by_miles = from a in "albums",
  join: ar in "artists", on: a.artist_id == ar.id,
  where: ar.name == "Miles Davis"
```
We defined the a binding first, at the beginning of the from call, then later defined the ar binding in the join. This will be the order that Ecto will expect if the query is used again.
As it happens, we don’t need the artists binding in the second query, but if we did, that binding would have to appear after the albums binding:
`album_query = from [a,ar] in albums_by_miles, select: a.title`
This wouldn’t work:
`album_query = from [ar,a] in albums_by_miles, select: a.title`
