# Ecto-notes

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
