# Resolution Patterns

There are several ways to perform resolution in Absinthe. We'll cover the main ones here (more advanced techniques such as using Dataloader will be covered in future workshops).

## Implicit Resolution

You may have noticed that in our schema example earlier we provided a resolver for the root `movie` field but not for the `title` field on the `movie` object. And yet it still worked!

This is because Absinthe is injecting a default resolver called `MapGet`. It's actually doing something like:

<!-- @filename  lib/moviedb_web/schema.ex -->

```elixir
  @desc "A Movie"
  object :movie do
    @desc "The Movie's title at release time"
    field :title, :string do
      resolve fn movie, _, _ ->
        {:ok Map.get(movie, :title)}
      end
    end
  end
```

This works because our movie resolver returned a Map (but it could equally have returned a struct like `%Movie{}`).

We don't have to specify a resolver directly in this way though and can rely on implicit resolution for these cases.

## Resolver Arguments

The function used by resolve can be either a 2 or 3 arity function. That is, it can take 2 or 3 arguments.

### 2 Artity Resolver

A 2 arity resolver passes any arguments on the field and the resolution struct to the resolver (we'll ignore the resolution struct for now).

The resolver on the `movie` field used this kind of resolver.

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
query do
  @desc "Look up a movie by its ID"
  field :movie, :movie do
    @desc "The unique ID of the movie"
    arg :id, non_null(:id)

    # Resolver function takes 2 arguments 
    resolve fn %{id: id} = _args, _resolution ->
      {:ok, Map.get(@movies, id)}
    end
  end
end
```



### 3 Arity Resolver

A 3 arity resolver includes the parent object as the first argument shifting the query args and the resolution struct to arguments 2 and 3.

The `MapGet` implicit resolver uses this (although we ignore args and resolution struct as we only care about the parent):

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
field :title, :string do
  # Resolver function takes 3 arguments
  resolve fn movie, _, _ ->
    {:ok Map.get(movie, :title)}
  end
end
```



## Named Functions

So far we've only looked at inline anonymous functions to do resolution. This works well for simple resolvers but sometimes resolution might be a bit more complex. In these cases we can use named functions:

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
def resolve_movie(%{id: id}, _res) do
  {:ok, Map.get(@movies, id)}
end

field :movie, :movie do
  @desc "The unique ID of the movie"
  arg :id, non_null(:id)

  resolve &resolve_movie/2
end
```

Or we could move the resolver to a separate module. For larger schemas this is definitely the preferred approach.

<!-- @filename lib/moviedb_web/schema.ex -->

```elixir
alias Schema.QueryResolver

# ...

field :movie, :movie do
  @desc "The unique ID of the movie"
  arg :id, non_null(:id)

  resolve &QueryResolver.movie/2
end
```



<!-- @filename lib/moviedb_web/schema/query_resolver.ex -->

```elixir
defmodule Schema.QueryResolver do
  @movies %{
    #...
  }
  
  def movie(%{id: id}, _) do
    {:ok, Map.get(@movies, id)}
  end
end
```



### A Note on Naming

It's generally good practice to give your resolver module the same name as the parent type and the functions in the module after each field that will be resolved.

For example, the root query will have a resolver called `QueryResolver` and the `movie` object will have a resolver called `MovieResolver`.

## Returning Errors

One core principle of GraphQL is that the **shape of the response should match the shape of the request**. So if I query a field that doesn't exist or is not available then I should still be given a return value for it - even if it's `null`.

Take a distributor field on a Movie:

```graphql
{
    movie(id: 1) {
        title
        distributor {
            name
        }
    }
}
```

The movie may not actually have a distributor yet but the response should match the shape of the request. It would look like:

```json
{
    "data" : {
    	"movie" : {
    	    "title" : "A new movie",
        	"distributor" : null
    	}
    }
}
```

However, it is helpful to tell the caller the reason the distrubutor field was not returned. The GraphQL specification uses the errors field.

In order to specify an error in Absinthe, our resolver needs to return an error tuple. Let's use the movie resolver as an example:

```elixir
defmodule Schema.QueryResolver do
  @movies %{
    #...
  }
  
  def movie(%{id: id}, _) do
    case Map.fetch(@movies, id) do
      :error ->
        {:error, "Not Found"}
       
      {:ok, movie} ->
        {:ok, movie}
    end
  end
end
```

Now when we query for a movie that doesn't exist we still get the movie key (but set to null) and we now get an error telling us what happened:

```json
{
    "data" : {
    	"movie" : null
    },
    "errors" : [
        {
            "message" : "Not Found",
            //...some other info too
        }
    ]
}
```

