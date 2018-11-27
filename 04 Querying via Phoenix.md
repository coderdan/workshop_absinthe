# Querying via Phoenix



To use Absinthe in Phoenix we need to use `Plug` and specifically the `absinthe_plug` package.

Add the package to your deps:

<!-- @filename mix.exs -->

```elixir
defp deps do
  {:absinthe_plug, "~> 1.4"},
  # ...
end
```

Then get the dependencies:

<!-- @command -->

```sh
> mix deps.get
```

Now we can add our schema to the phoenix router:

<!-- @filename lib/moviedb_web/router.ex -->

```elixir
forward "/api", Absinthe.Plug,
    schema: MoviedbWeb.Schema,
    json_codec: Jason
```



You can check that the `/api` route has been added by running:

<!-- @command -->

```sh
> mix phx.routes
page_path  GET  /     MoviedbWeb.PageController :index
           *    /api  Absinthe.Plug [schema: MoviedbWeb.Schema]
```



Now let's try making a GraphQL query using `curl`!

First start your server:

<!-- @command -->

```sh
iex -S mix phx.server
```

And then execute the query:

<!-- @command -->

```sh
curl -v -H "Content-Type: application/graphql" --data "{movie(id: 1) {title}}" http://localhost:4000/api
```



## GraphiQL

By far the best way to run queries against a GraphQL server in development is GraphiQL. You can add it to your router as follows (assuming you have installed `absinthe_plug`).

<!-- @filename lib/moviedb_web/router.ex -->

```elixir
  if Mix.env == :dev do
    forward "/graphiql", Absinthe.Plug.GraphiQL,
      schema: MoviedbWeb.Schema,
      json_codec: Jason,
      interface: :playground
  end
```

I've used the `:playground` interface here but you can omit the `:interface` option to use the default.

![Screenshot 2018-11-27 11.02.35](/Users/daniel/Dropbox/Screenshots/Screenshot 2018-11-27 11.02.35.png)