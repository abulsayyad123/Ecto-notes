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
