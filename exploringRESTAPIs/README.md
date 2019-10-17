# GMIT Distributed Systems

## Lab: Exploring a REST API
_Inspired by Chapter 2 of [Designing APIs with Swagger and OpenAPI](https://www.manning.com/books/designing-apis-with-swagger-and-openapi) by Josh Ponelat, under development, expected publication Spring 2020._

### The SWAPI
For this lab we'll be exploring the [Star Wars API](https://swapi.co), or SWAPI. To quote the author:
>The Star Wars API is the world's first quantified and programmatically-formatted set of Star Wars data.
After hours of watching films and trawling through content online, we present to you all the People, Films, Species, Starships, Vehicles and Planets from Star Wars.
We've formatted this data in JSON and exposed it to you in a RESTish implementation that allows you to programmatically collect and measure the data.

The SWAPI is a good playground for initial exploration of the REST architectural style because:
- it's simple
- it's openly accessible without authentication
- it exhibits good REST design practices

Exploring this API will help you to:
- learn how to use an API based on its documentation
- get experience in sending HTTP requests using an industry standard tool
- see an example of good REST API design

### Postman
To explore this API we'll need some way to send HTTP requests. We could do this on the command line using `curl` or, if we liked pain, `telnet`. `curl` can be really handy for quick & dirty command line requests, but a more user friendly alternative is Postman. Postman is a graphical tool for sending HTTP requests with a bunch of features which make our lives easier. If you're using a lab machine, download this [portable version](https://portapps.io/download/postman-portable-win64-7.7.3-20-setup.exe) and install it to your OneDrive. If you're on your own machine, [get your version here](https://www.getpostman.com/downloads/).

When you start up Postman it'll prompt you to create an account. You **don't need** to create an account.

### Exploring the SWAPI
#### The Base URL
[Here's the documentation](https://swapi.co/documentation) for the SWAPI. The first thing to note is the **Base URL**: `https://swapi.co/api/`. This URL forms the base of all the API requests we'll make. It identifies:
- the protocol `https`
- the hostname of the server `swapi.co`
- the path to where the api is hosted on the server `/api`

The documentation tells us that a call to this URL provides information on all the resources in the API. Using Postman, send a HTTP `GET` request to https://swapi.co/api/. You should see the following response:
```
{
    "people": "https://swapi.co/api/people/",
    "planets": "https://swapi.co/api/planets/",
    "films": "https://swapi.co/api/films/",
    "species": "https://swapi.co/api/species/",
    "vehicles": "https://swapi.co/api/vehicles/",
    "starships": "https://swapi.co/api/starships/"
}
```
This JSON response tells us that we can information on various types of entities in the Star Wars universe (`people`, `planets` etc.), and gives us the URLs we can use to access these resources. Note the lower case plural names (`people` as opposed to `Person`). This naming style is standard RESTful convention.

#### Films
Since we know that this is a RESTful API, we can assume that the `films` endpoint refers to film resources,  and that a film resource is a specific Star Wars film. We can also expect that a HTTP `GET` to the `films/` endpoint will return all the available film resources.

- Using Postman, send a HTTP `GET` to https://swapi.co/api/films/ and look at the response. You should see a listing of all the film resources in the system in JSON format.

Rather than getting all the film resources, it may be more useful to find a specific film. Let's say we want to get data on The Empire Strikes Back. The SWAPI documentation tells us that we can search film resources by title using the `search=` **query parameter**. A query parameter is something that is appended to a URL as an additional parameter, and is separated from the URL using `?`. Query parameters are always key=value pairs.

- Using Postman, send a HTTP `GET` to https://swapi.co/api/films?search=Empire. This should return just one result (note `"count": 1,` in the response, for The Empire Strikes Back. The film resource here will be contained in a `"results: []"` array in the response body.

It may be useful for our client application to know where to find this resource again without having to search for it. Helpfully, the SWAPI includes a resource's URL as part of its representation.

- What's the URL of the films resource for The Empire Strikes Back?
    <details><summary>Answer</summary>
    https://swapi.co/api/films/2/
    (The number 2 here is the unique ID for this film.)
    </details>

- Send a HTTP `GET` request to the URL for The Empire Strikes Back. This time you should just get the resource itself. Note how this differs from the search response we got in the previous request.

#### HATEOAS
Note how each API response contains URLs to other related resources. This is an excellent example of the HATEOAS principle: Hypertext As The Engine Of Application State. HATEOAS is a constraint on REST that says that a client of a REST application need only know a single fixed URL to access it. Any and all resources should be discoverable dynamically from that URL through hyperlinks included in the representations of returned resources.

#### Other Resources
Send HTTP `GET` requests to the endpoints below to see what kind of data we get back:
- `/people`
- `/starships`
- `/vehicles`
- `/species`
- `/planets`

Using the `?search=FooBar` We can search for other resources on different fields:
- `people`: name
- `starships`: name, model
- `vehicles`: name, model
- `species`: name
- `planets`: name

### Questions
Using just API requests and the hypermedia links returned in responses find answers to the following questions.
- What's the orbital period of Darth Vader's homeworld?
    <details>
    <summary>Answer</summary>

    ```
    "orbital_period": "304"
    ```

    </details>

- What colour hair does the pilot of the A-Wing starship have?
    <details>
    <summary>Answer</summary>
        brown
    </details>

- What species are the residents of Endor
    <details><summary>Answer</summary>
    - Ewok
    </details>


### Conclusion
In this lab we explored a simple but powerful and well-designed REST API and saw:
- how to get a list of all resources of a particular type
- how to access a particular resource
- how to search for a particular resource
- how HATEOAS can help us to navigate our way around the API
