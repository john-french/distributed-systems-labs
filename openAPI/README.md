# GMIT Distributed Systems

## Lab: Designing a REST API using OpenAPI

### Overview
This is the first of two labs where we'll take a *Design First* approach to building a REST API. In this lab we'll design our API by specifying it's features in the [OpenAPI format](https://swagger.io/specification/). As we discussed in lectures, OpenAPI is the de facto standard for describing REST APIs, and is essentially an interface definition language for this architectural style.


## Lab Procedure
### Create Swaggerhub account
We could download and install various Swagger tools for designing APIs with OpenAPI, but it's easier and faster to use a cloud platform that provides these tools freely online. Head to [SwaggerHub](https://app.swaggerhub.com/signup?channel=directWithinApp) and select **Create Account with Github**. This will redirect you to github.com to enter your credentials. Log in to your github account. You'll be redirected back to Swaggerhub and will be logged in with your Github username.

### Creating a new API
We're going to design part of API for a record label. The record label has a database of artists with the following information:
- Artist name
- Artist genre
- Number of albums published under the label
- Artist ID

The API will allow clients to
- list all label artists
- get information on specific artists
- add new artists

To start we need to create a (more or less) blank document in which we'll write our API definition.
In the navigation tab on the left, select `Create New -> Create New API`. In the popup that follows, select/enter the following
- `OpenAPI Version: 3.0` (we're going to use version 3.0 of the OpenAPI specification)
- `Template: None` (we want a more or less empty document to start)
- `Name: ArtistAPI` (the name under which our API will be stored in Swaggerhub)
- `Version: 1`
- `Title: Simple Artist API`
- `Auto Mocking: Off`

You can leave the rest blank. Hit `Create API`. You'll now have a template to start writing your OpenAPI definition. The meta-data section will be pre-populated with the information you entered in the dialog box.

The things we'll need to define in the API will be:
- paths
- operations
- responses
- reusable data objects

#### Writing YAML
We'll be writing the OpenAPI definition in the YAML language. YAML consists of `key: value` pairs separated by semi-colons. Hierarchy is indicated by indentation, e.g.:
```
top-level: stuff
    lower-level: more stuff
```

### Defining Paths
First off we'll define the paths in our API. Paths are the API endpoints that clients will be able to call. The API will have two endpoints for `artist` resources:
- `/artists`
    - `GET`: get list of all artists
    - `POST`: create new artist
- `/artists/{artistId}`
    - `GET`: get specific artist based on `artistId`

Let's start with `/artists`. Under `/paths:` and indented by one level, add the `/artists` path:
```
paths:
  /artists:
```
Now we have a **path**, we need to define what **operations** are supported on this path. We'll start with `GET`. Under `/artists` (again indented by one level) add `get:`, and a description:
```
paths:
  /artists:
    get:
      description: Returns a list of artists
```
What responses can we expect from a `GET` request to this endpoint? A successful response will be a `200` HTTP status code. Add the following indented under `get:`
```
responses:
  '200':
    description: Successfully returned a list of artists
```
A successful response will also contain a list of artists, in some format. `artist` is a data structure we'll need to use more than once. If define it in a `components:` section at the end of our OpenAPI definition, then we can refer to it whenever we need to:
```
components:
  schemas:
    Artist:
      type: object
      required:
        - artist_id
      properties:
        artist_name:
          type: string
        artist_genre:
            type: string
        albums_recorded:
            type: integer
        username:
            type: string
```
The YAML snippet above defines an `Artist` object, along with its properties and their types.

Now we can use this `Artist` in defining the `content` of the successful response to the `GET /artists` API call:
```
responses:
  '200':
    description: Successfully returned a list of artists
    content:
      application/json:
        schema:
          type: array
          items:
            $ref: '#/components/schemas/Artist'
```
This indicates that the response will be an array of `Artist` objects in JSON format. We've now successfully defined:
- an API endpoint
- a response from that endpoint
- a data structure that we can reuse elsewhere in our API definition

### More Paths
Based on the example above, and the [OpenAPI 3.0 Specification](https://swagger.io/specification/), write definitions for the following endpoints and operations:
- `POST` to `/artists` (creates a new artist)
  - will contain a `requestBody` made up of an `Artist` in JSON format.
- `GET` to `/artists/{artistId}`
  - uses a required `parameter` called `artistId`, which is a path parameter.


_Adapted from the [Swaggerhub OpenAPI 3.0 Tutorial](https://app.swaggerhub.com/help/tutorials/openapi-3-tutorial)_
