# Create a new Phoenix app

To start making GraphQL queries to an Elixir GraphQL server, we'll use Absinthe running inside Phoenix.

Note that Absinthe can execute GraphQL queries over HTTP (say in a Phoenix Application) but also allows queries to be made directly via text.

Let's start by creating a new phoenix application:

<!-- @command -->

```sh
> mix phx.new moviedb
```

If you're asked to install dependencies, just say yes.

Now edit the `mix.exs` file created inside the application and add Absinthe as a dependency.

<!-- @filename "mix.exs" -->

```elixir
defp deps do
  [
    {:absinthe, "~> 1.4"},
    # ...other deps
  ]
end
```

Install Absinthe by running `mix deps.get` from inside the `moviedb` directory.

Ensure your database configuration is correct in `config/dev.exs`. In my case, I changed the username from "postgres" to "daniel":

<!-- @filename "config/dev.exs" -->

```elixir
config :moviedb, Moviedb.Repo,
  username: "daniel",
  database: "moviedb_dev",
  hostname: "localhost",
  pool_size: 10
```

Now you can create the database (we won't use it for the moment but if Phoenix doesn't have a database at this stage it will spew error messages).

<!-- @command -->

```sh
> mix ecto.create
```

Finally, start `mix` inside an `iex` console and if all goes well it should compile and start.

<!-- @command -->

```sh
└─ $ iex -S mix
Erlang/OTP 20 [erts-9.3.3] [source] [64-bit] [smp:4:4] [ds:4:4:10] [async-threads:10] [hipe] [kernel-poll:false]

Interactive Elixir (1.6.5) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
```

