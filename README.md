# JSend

* **What?** - Put simply, JSend is a specification that lays down some rules for how [JSON](http://json.org/) responses from web servers should be formatted. JSend focuses on application-level (as opposed to protocol- or transport-level) messaging which makes it ideal for use in [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer)-style applications and APIs.
* **Why?** - There are lots of web services out there providing JSON data, and each has its own way of formatting responses. Also, developers writing for JavaScript? front-ends continually re-invent the wheel on communicating data from their servers. While there are many common patterns for structuring this data, there is no consistency in things like naming or types of responses. Also, this helps promote happiness and unity between backend developers and frontend designers, as everyone can come to expect a common approach to interacting with one another.
* **Hold on now, aren't there already specs for this kind of thing?** - Well... no. While there are a few handy specifications for dealing with JSON data, most notably [Douglas Crockford](http://www.crockford.com/)'s [JSONRequest](http://www.json.org/JSONRequest.html) proposal, there's nothing to address the problems of general application-level messaging. More on this later.
* **(Why) Should I care?** - If you're a library or framework developer, this gives you a consistent format which your users are more likely to already be familiar with, which means they'll already know how to consume and interact with your code. If you're a web app developer, you won't have to think about how to structure the JSON data in your application, and you'll have existing reference implementations to get you up and running quickly.

## So how does it work? 
A basic JSend-compliant response is as simple as this:
```
{
    "status" : "success",
    "message": "Ok",
    "data": {
        "post" : { "id" : 1, "title" : "A blog post", "body" : "Some useful content" }
     }
}
```
When setting up a JSON API, you'll have all kinds of different types of calls and responses. JSend separates responses into some basic types, and defines required and optional keys for each type:

<table>
<tr><th>Type</td><th>Description</th><th>Required Keys</th><th>Optional Keys</td></tr>
<tr><td>success</td><td>All went well, and (usually) some data was returned.</td><td>status, message, data</td><td></td></tr>
<tr><td>fail</td><td>There was a problem with the data submitted, or some pre-condition of the API call wasn't satisfied</td><td>status, message, data</td><td>errors</td></tr>
<tr><td>error</td><td>An error occurred in processing the request, i.e. an exception was thrown</td><td>status, message, data</td><td></td></tr>
</table>

## Example response types 

### Success ### 
When an API call is successful, this would likely respond an http 2xx status code, the JSend object is used as a simple envelope for the results, using the data key, as in the following:
#### GET /posts.json: ####
```
{
    "status": "success",
    "message": "Ok",
    "data": {
        "posts" : [
            { "id" : 1, "title" : "A blog post", "body" : "Some useful content" },
            { "id" : 2, "title" : "Another blog post", "body" : "More content" },
        ]
     }
}
```
#### GET /posts/2.json: ####
```
{
    "status": "success",
    "message": "Ok",
    "data": { "post" : { "id" : 2, "title" : "Another blog post", "body" : "More content" }}
}
```
#### DELETE /posts/2.json: ####
```
{
    "status": "success",
    "message": "Successfully deleted the post",
    "data": {}
}
```
Required keys:

* status: Should always be set to "success".
* message: Default message when fetching entity should be "Ok".
* data: Acts as the wrapper for any data returned by the API call. If the call returns no data (as in the last example), data should be set to an empty object.

### Fail ### 
When an API call is rejected due to invalid data or call conditions, the JSend object's data typically would be an empty object. And when an API call is responding a validation error, the optional JSend object's errors will exist to describe which payload sent with invalid data, with an http status code 422. For example:

#### POST /posts/2.json (unauthorized response with status code: 401): ####
```
{
    "status": "fail",
    "message": "Your session is already expired.",
    "data": {}
}
```

#### POST /posts.json (validation error with status code: 422): ####
```
{
    "status": "fail",
    "message": "Failed to create a blog post",
    "data": {},
    "errors": { "title": ["A title is required"] }
}
```
Required keys:

* status: Should always be set to "fail".
* message: A reason on why this request is failed.
* data: This would likely filled with an empty object.

Optional keys:
* errors: And object keyed with payload giving an invalid request and valued with array containing reasons on why it gives a failure response.

### Error ### 
When an API call fails due to an error on the server. Typically this would respond a 5xx http status code. For example:
#### GET /posts.json: ####
```
{
    "status" : "error",
    "message" : "Server error",
    "data": {}
}
```
Required keys:
* status: Should always be set to "error".
* message: A reason on why this request respond an error.
* data: This would likely filled with an empty object.

## License ##
The JSend specification (this page) is covered under a [modified BSD License](LICENSE.md)
