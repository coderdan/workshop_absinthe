# GraphQL

## Why GraphQL?

Let's compare to a REST API. Say we have an API for a movies database.

We might fetch a movie:

```json
GET /movies/1
{
    "title" : "The Godfather",
    "releaseDate" : 1972,
    "directorId" : 100,
    "synopsis" : "..."
}
```

Now let's say we also want to fetch the director and genre list for the movie. We could make 2 additional requests.

First, the director:

```json
GET /director/100
{
    "name" : "Francis Ford Coppola",
    "born" : "April 7, 1939",
    "bio" : "..."
}
```

And the genres:

```json
GET /movies/1/genres
[
    { "name" : "Drama" },
    { "name" : "Crime" }
]
```



**What do we notice about the REST calls?**

> We made **3 separate requests** to retrieve the movie along with the director and genres

We could modify the `/movies` endpoint to include the director and genres:

```json
GET /movies/1
{
    "title" : "The Godfather",
    "releaseDate" : 1972,
    "synopsis" : "...",
    "director" : {
        "name" : "Francis Ford Coppola",
        "born" : "April 7, 1939",
        "bio" : "..."
    },
    "genres" : [
        { "name" : "Crime" },
        { "name" : "Drama" }
    ]
}
```

But where do we stop? What about actors? Reviews?

> Each request returned all data associated with the record whether we wanted it or not

The synopsis of a movie could be from a few sentences up to a few pages.

```json
GET /movies/1
{
    "title" : "The Godfather",
    "releaseDate" : 1972,
    "synopsis" : "In late summer 1945, guests are gathered for the wedding reception of Don Vito Corleone's daughter Connie (Talia Shire) and Carlo Rizzi (Gianni Russo). Vito (Marlon Brando), the head of the Corleone Mafia family, is known to friends and associates as \"Godfather.\" He and Tom Hagen (Robert Duvall), the Corleone family lawyer, are hearing requests for favors because, according to Italian tradition, \"no Sicilian can refuse a request on his daughter's wedding day.\" One of the men who asks the Don for a favor is Amerigo Bonasera, a successful mortician and acquaintance of the Don, whose daughter was brutally beaten by two young men because she refused their advances; the men received minimal punishment from the presiding judge. The Don is disappointed in Bonasera, who'd avoided most contact with the Don due to Corleone's nefarious business dealings. The Don's wife is godmother to Bonasera's shamed daughter, a relationship the Don uses to extract new loyalty from the undertaker. The Don agrees to have his men punish the young men responsible (in a non-lethal manner) in return for future service if necessary. Meanwhile, the Don's youngest son Michael (Al Pacino), a decorated US Marine hero returning from World War II service, arrives at the wedding and tells his girlfriend Kay Adams (Diane Keaton) anecdotes about his family, informing her about his father's criminal life; he reassures her that he is different from his family and doesn't plan to join them in their criminal dealings. The wedding scene serves as critical exposition for the remainder of the film, as Michael introduces the main characters to Kay. Fredo (John Cazale), Michael's next older brother, is a bit dim-witted and quite drunk by the time he finds Michael at the party. Santino, who is nicknamed Sonny (James Caan), the Don's eldest child and next in line to become Don upon his father's retirement, is married but he is a hot-tempered philanderer who sneaks into a bedroom to have sex with one of Connie's bridesmaids, Lucy Mancini (Jeannie Linero). Tom Hagen is not related to the family by blood but is considered one of the Don's sons because he was homeless when he befriended Sonny in the Little Italy neighborhood of Manhattan and the Don took him in and saw to Tom's upbringing and education. Now a talented attorney, Tom is being groomed for the important position of consigliere (counselor) to the Don, despite his non-Sicilian heritage."
}
```

What if we only wanted the title? **We're wasting lots of data and compute time.**

### The GraphQL Approach

GraphQL allows arbitrary queries within a data schema. If we want to query just the movie title we can.

```json
{
    movie(id: "1") {
        title
    }
}

{
    "movie" : {
        "title" : "The Godfather"
    }
}
```



We can include whatever data we want in our query, including any associated data such as director or genres.

```json
{
    movie(id: "1") {
        title
        director {
            name
        }
        genres {
            name
        }
    }
}

{
    "movie" : {
        "title" : "The Godfather",
        "director" : {
            "Francis Ford Coppola"
        },
        "genres" : [
            { "name" : "Crime" },
            { "name" : "Drama" }
        ]
    }
}
```







