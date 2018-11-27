# GraphQL Schema

To specify how our GraphQL queries will be handled, we must create a schema.

## Deciding on your Schema

It's important to consider what kinds of queries you might want to make available in the root of the schema.

Let's start with something simple. We want to fetch a movie by its `ID`.

```graphql
{
    movie(id: 1) {
        title
    }
}
```

We can create a schema file to satisy this query:

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
defmodule MoviedbWeb.Schema do
  use Absinthe.Schema

  query do
    field :movie, :movie
  end
end
```

Here we've used the `Absinthe.Schema` module which provides a nice DSL (Domain Specific Language) for writing GraphQL schemas. This code specifies a field on the root query called `movie` which returns a `movie` type.

There are a few more things we must do before this schema can be used for making queries though.

### Object Types

Absinthe defines several simple types called "scalar types". Scalar (pronounced Scale-Are) types are simple types that have no additional attributes of their own. Absinthe defines scalar types such as `Int`, `ID` and `Float` in the [BuiltIns Module](https://hexdocs.pm/absinthe/Absinthe.Type.BuiltIns.html).

But `movie` is not a scalar, it's an Object as it has its own attributes (such as `title`) so we'll need to define it.

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
defmodule MoviedbWeb.Schema do
  use Absinthe.Schema

  object :movie do
    field :title, :string
  end

  query do
    field :movie, :movie
  end
end
```

Here we use the `object/1` macro to define the type and the `field/3` macro to specify a single attribute called `title`.

### Arguments

For our movie query we want to find the movie with a specific ID so we need to specify an argument on the schema. We do this with the `arg/3` macro.

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
query do
  field :movie, :movie do
    arg :id, non_null(:id)
  end
end
```

We can also add a description to the argument (though you probably wouldn't normally do so for ID):

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
query do
  field :movie, :movie do
    arg :id, non_null(:id), description: "The unique ID of the movie"
  end
end
```



### Documentation

On the topic of documentation, it's a good idea to document all of the fields and objects in your schema. In Absinthe you can do this with the `@desc` attribute (we can use `@desc` for our argument too if we prefer):

```elixir
defmodule MoviedbWeb.Schema do
  use Absinthe.Schema

  @desc "A Movie"
  object :movie do
    @desc "The Movie's title at release time"
    field :title, :string
  end

  query do
    @desc "Look up a movie by its ID"
    field :movie, :movie do
      @desc "The unique ID of the movie"
      arg :id, non_null(:id)
    end
  end
end
```

Now we have a complete schema definition and we can run our query!

Let's run it directly in `iex` with `Absinthe.run/3`:

```elixir
iex(1)> """
...(1)> {
...(1)>   movie(id: 1) {
...(1)>     title
...(1)>   }
...(1)> }
...(1)> """ |> Absinthe.run(MoviedbWeb.Schema)
{:ok, %{data: %{"movie" => nil}}}
```

The query worked but no movie was returned (it was simply set to nil). We need to *resolve* the query!

### Query Resolution

Resolution is the process which Absinthe (or any GraphQL server for that matter) takes to fetch data that satisfies a given request. There are several different ways to resolve but we'll start with the simplest - an *inline resolver*.

```elixir
defmodule MoviedbWeb.Schema do
  use Absinthe.Schema

  @movies %{
    "1" => %{title: "Star Wars"},
    "2" => %{title: "Godzilla"},
  }

  @desc "A Movie"
  object :movie do
    @desc "The Movie's title at release time"
    field :title, :string
  end

  query do
    @desc "Look up a movie by its ID"
    field :movie, :movie do
      @desc "The unique ID of the movie"
      arg :id, non_null(:id)

      resolve fn %{id: id} = _args, _resolution ->
        {:ok, Map.get(@movies, id)}
      end
    end
  end
end
```

