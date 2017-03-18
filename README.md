# Introspected REST
(Sliding away from Roy Fielding's `REST` model)

There has been a great confusion of what a `REST` API is.
Most people think that `REST` API is just a CRUD over HTTP.
Or a CRUD with some links.
Or a nicely formatted, a sophisticated CRUD.

In this _manifesto_, we will give a specific definition of what `REST` is, according to Roy,
and see that most current APIs and API specs (JSONAPI, HAL etc) fail to follow this model.
Then, we will propose a new model that brings into the table the same things,
yet it's much simpler to implement while at the same time being backwards compatible with any current (sane) API.

## Definitions
First some definitions, that we will use through the text:

* `REST`, `RESTfull`: The model that Roy defined in his [thesis](thesis) (along with his [blog]() comments).
* `RESTly`: APIs that follows most parts of `REST` model, lacking full HATEOAS though (spec like JSONAPI, HAL etc)
* `RESTless`: APIs that have a plain JSON API without any links (follows `REST` model other than HATEOAS)
* `Introspected REST`: APIs that follow the definition of the model we provide in this _manifesto_


## Introduction
`REST` defined by Roy was a magnificent piece of work, much ahead of its time
which took us 10+ years to understand what its capabilities are.
However, now, almost 20 years later `REST` model shows its age. It's unflexibile,
difficult to implement, difficult to test, with performance and implementation issues.
But most importantly, any implementation of `REST` model is _very_ complex.

Now, one could say that, most APIs are not build with mind to last for decades and maybe
that's the reason that this model hasn't seen much adoption.

The former is true but even if the latter is also true could it mean that this model is
not suitable for short-term APIs?

We firmly believe that `REST` is much better than any API that does not follow `REST` principles
(like `RESTly` APIs), even for short-term APIs.
Networked services have very peculiar characteristics which, until now, only `REST` has fully addressed them.
Being able to evolve your API without breaking the clients is critical.

Imagine the following scenario: you have built an Online Social Network and an iOS app that talks the API on your backend.
Now imagine that, after a company meeting, your CEO needs you to make tiny yet important change in the signup page: require the user
to fill in her age. Essentially, this means, in API terms, make a form field required.

If your API is `RESTly` and not `REST`, this means that you need to fix the code in the iOS side, test it and send a new iOS app to apple store.
It takes roughly 1 week for Apple to review your app and if your app won't be rejected for some reason, your
tiny change will take action at least a week later after requested.
If your API _was_ `REST` that would mean a change on the server's response denoting which fields are required to submit the form.
You would have the change deployed 10 minutes later.

Roy notes in his thesis:

>  A system intending to be as long-lived as the Web must be prepared for change
>
> --- Roy Fielding
>

Let me rephrase that in terms you will sound familiar to you:

>  If you want to move fast, you should build a change-first API.
>
>
An API that can change the state of the client without needing the latter to change.

Given that, how can we have a simpler model than `REST`, yet have the same functionality of
`REST`?

As we will show, `Introspected REST` is an API architectural style that solves that.
An architectural style is not an implementation and it's not a spec either.
As Roy notes:

>  An architectural style is a coordinated set of architectural constraints that restricts
>  the roles/features of architectural elements and the allowed relationships among those
>  elements within any architecture that conforms to that style.
>
> --- Roy Fielding

`Introspected REST` is based on Roy's initial model but removes the need for runtime HATEOAS.
Instead, the client derives the state using instrospection.

Eventually this brings the same advantages as Roy's model while being it's much simpler,
much more flexible and backwarde compatible with any Restfull API.

But first let's discuss about Networked Services.

## Networked Services and APIs
Nowadays JSON has become so popular that people almost forget that there is whole bunch of
protocols below it.
People also forget that JSON is just a specification in the message level, like XML.
It's not the only one and definitely it's not the best we could use.
Nevertheless it's simple and simplicity is a virtue.

When we want to request a resource from a networked hypermedia-based API, we roughly
have the following levels:

### Application level
In the application level, the client starts content negotiation, usually asking
for only one media type.

`application/json` is a media type that denotes that the data format of the requested
representation is in JSON data dormat.
JSON itself is not a media type but a message format.

Media types can be composite as well: `application/vnd.api+json` (roughly) means that the data
format of the requested representation is in JSON data dormat in the semantics of the `vnd.api`,
which is the JSONAPI semantics.
In theory, JSONAPI spec spemantics could also be applied using XML as the data format (like in the case of HAL),
however in practice we tend to forget that and we treat all media types as single and not composite.

In the HTTP this is done using the `Accept` header (and server responds with `Content-Type` header).

However, it should also be noted that the media types and the content negotiation in general, are
not restricted to HTTP only.
Although HTTP is one of the most popular network protocols today, the same logics could be applied
in other (mostly text-based) protocols like SIP, CoAP, QUIC etc.

To sum up, the application level semantics are not coupled tight to the semantics of the
message level (like JSON) or the underlying protocol level (like HTTP).

+representation metadata

### Message level
In the message level we find the format that is used for the actual representation.
We nowadays we have almost mixed the message level with JSON but in practice other
formats could successfully be used: XML, YAML, TOML to name a few.

### Protocol level
In the protcol level, the requests are usually sent using the HTTP.
After all, nowadays most of the development happens around the Web and
HTTP is the only protocol that browsers officially support.

Nonetheless, there are other protocols as well.
QUIC is a HTTP alternative protocol that is targeted for low latency and uses UDP
underneath.
CoAP is targeted in the IoT and also uses UDP underneath (full TCP/IP stack is quite heavy for constrainted devices).
SIP is also a text-based protocol with the same semantics as HTTP and is used in VoIP.

### Network level
Finally (well for the scope of this manifesto, in networks the lowest protocols are the one found in the Physical level
which deal with the wire signals), in the network level, the browser (or any other non-browser client) sends the networked request
in one of the TCP, UDP, etc

The actual protocol depends on the protocol used by the protocol level.


## Roy's `REST` model
Roy's `REST` model is an arhictectural style which is not tight to any spec, protocol or format of the
aforementioned levels.

> a RESTful API is just a website for users with a limited vocabulary (machine to machine interaction)
>
> --- Roy Fielding


When Roy talks about `REST` he mentions 5 crucial properties of a `REST` model:

##### All important resources are identifed by one resource identifer mechanism
> induces simple, visible, reusable, stateless communication

##### Access methods have the same semantics for all resources
> induces visible, scalable, available through layered system, cacheable, and shared caches

##### Resources are manipulated through the exchange of representations
> induces simple, visible, reusable, cacheable, and evolvable (information hiding)

##### Representations are exchanged via self-descriptive messages
> induces visible, scalable, available through layered system, cacheable, and shared caches
> induces evolvable via extensible communication

##### Hypertext as the engine of application state (HATEOAS)
> induces simple, visible, reusable, and cacheable through data-oriented integration
> induces evolvable (loose coupling) via late binding of application transitions

### Requirements from a modern API
In 2017 we have progressed so much on Resty APIs that now we essentially have to
provide an ORM to the client over the HTTP (or any other protocol).
A modern API should _also_ have the following properties, at least:

##### Sparse fields
The client needs to be able to ask and get specific attributes of the resource representation.

Applies to both collection or resource.

##### Granular permissions
> for a collection/resource

The same representation could have a set of attributes or a subset of that set based
on the user role and permissions

##### Associations on demand (collection/resource)

The client should be able to ask related associations to the main initial resource, in the same request.

##### Sorting & pagination

The client should be able to sort based on one or more attributes and paginate the collection
based on the page, page size and possible an offset.

###### Filtering collections
> Collection only
The client should be able to run any sort of collection filtering, as long as it does not pose
any security thread or slows down the API performance.

###### Aggregation queries
> Collection only
The client should be able to run any sort of aggregation queries, as long as it does not pose
any security thread or slows down the API performance.

###### **Data types**
> Collection and resource

The client should know the data types of the attributes of the requested representation of a resource.
Message formats (like JSON) provide some data types but they are pretty basic.
We need to know for instance


Great! let's see the API specs proposed as today, April 2017..

## API Specs Today
### Our use case
In our use case we will follow the aforementioned points of the `REST` model.

Our use case is a minature of Twitter.
Specifically, in our API domain, we have a `User` resource and a `Micropost` resource with the following attributes:
* `User`
  * `id`, a String, never empty or NULL, primary ID of the resource
  * `email`, a String, never empty or NULL, with maximum length up to 255 characters, email format
  * `name`, a String, with maximum length up to 150 characters
  * `created_at`, a String, never empty or NULL, representing a DateTime according to `is8601`, in UTC
  * `microposts_count` an Integer

* `Micropost`
  * `id`, a String, never empty or NULL, primary ID of the resource
  * `userId`, a String, never empty or NULL, representing an ID of `User` resource
  * `content`, a String, never empty or NULL, with maximum length up to 160 characters
  * `created_at`, a String, never empty or NULL, representing a DateTime according to `is8601`, in UTC

So given the `REST` model we have the following routes:
* `Users` resource (`/users`):
 * List users (`GET /users`): Gets a collection of `User` resources
 * Create a new user (`/users`): Creates a new `User` with the specified attributes.

* `User` resource (`/users/{id}`):
 * Get a user (`GET /users/{id}`): Gets the attributes of the specified `User`
 * Update a user `PATCH /users/{id}`: Updates a `User` with the specified attributes
 * Delete a user `DELETE /users/{id}`: Updates a `User` with the specified attributes

_these 2 resources are often mistankingly thought as a single, one, resource_

The `User` resource has also some _associations_ (or _relations_/_relationships_ if you prefer).

In plain JSON the resources would look like:

```json
{
  "id":"9123",
  "name":"Filippos Vasilakis",
  "email":"vasilakisfil@gmail.com",
  "created_at": "2014-01-06T20:46:55Z",
  "microposts_count":49
}
```

```json
{
  "id":"323",
  "user_id": "9123"
  "content":"We are live!",
  "created_at": "2017-01-06T20:46:55Z",
}
```

Now that we defined the scope of our little API, let's see how this would be implemented
in the current API specs:

### JSONAPI

##### User
```json
{
    "data": {
        "id": "9123",
        "type": "users",
        "attributes": {
            "name": "Filippos Vasilakis",
            "email": "vasilakisfil@gmail.com",
            "created-at": "2014-01-06T20:46:55Z",
            "microposts-count": 50,
        },
        "relationships": {
            "microposts": {
                "links": {
                    "related": "/api/v1/microposts?user_id=9123"
                }
            }
    }
}
```

##### Users resource

```json
{
    "data": [
      {
        "id": "9123",
        "type": "users",
        "attributes": {
            "name": "Filippos Vasilakis",
            "email": "vasilakisfil@gmail.com",
            "created-at": "2014-01-06T20:46:55Z",
            "microposts-count": 50,
        },
        "relationships": {
            "microposts": {
                "links": {
                    "related": "/api/v1/microposts?user_id=9123"
                }
            }
      },
      {
        "id": "9124",
        "type": "users",
        "attributes": {
            "name": "Robert Clarsson",
            "email": "robert.clarsson@gmail.com",
            "created-at": "2016-10-06T16:01:24Z",
            "microposts-count": 50,
        },
        "relationships": {
            "microposts": {
                "links": {
                    "related": "/api/v1/microposts?user_id=9124"
                }
            }
      },
    ],
    "links": {
        "self": "/api/v1/users?page=1&per_page=10",
        "next": "/api/v1/users?page=2&per_page=10",
        "last": "/api/v1/users?page=3&per_page=1"
    }
}
```

Problems of this spec:
 * Limited links (no URI templates, treats the client as stupid)
 * No actions
 * No info on available attributes
 * No info on data types
 * No attributes description, requires documentation

### HAL

```json
{
    "_links": {
        "self": {
            "href": "/api/v1/users/{id}"
        },
        "microposts": {
            "href": "/api/v1/microposts/user_id={id}",
            "templated": true
        }
    },
    "id": "1",
    "name": "Filippos Vasilakis",
    "email": "vasilakisfil@gmail.com",
    "createdAt": "2014-01-06T20:46:55Z",
    "micropostsCount": 50,
}
```

```json
{
   "_links":{
      "self":{
         "href":"/api/v1/users"
      },
      "curries":[
         {
            "name":"ea",
            "href":"http://example.com/docs/rels/{rel}",
            "templated":true
         }
      ]
   },
   "_embedded":{
      "users":[
         {
            "_links":{
               "self":{
                  "href":"/api/v1/users/{id}"
               },
               "microposts":{
                  "href":"/api/v1/microposts?user_id={id}"
               }
            },
            "id": 9123,
            "name": "Filippos Vasilakis",
            "email": "vasilakisfil@gmail.com",
            "createdAt": "2014-01-06T20:46:55Z",
            "micropostsCount": 50
         }, {
            "_links":{
               "self":{
                  "href":"/api/v1/users/{id}"
               },
               "microposts":{
                  "href":"/api/v1/microposts?user_id={id}"
               }
            },
            "id": 9123,
            "name": "Robert Clarsson",
            "email": "robert.clarsson@gmail.com",
            "created-at": "2016-10-06T16:01:24Z",
            "microposts-count": 50,
         }
      ]
   }
}
```

Goods:
 * Links

Problems with this spec:
 * No actions
 * No info on available attributes
 * No info on data types
 * No attributes description, requires documentation (however it does provide a link to documentation)

### Siren

### Hydra

!### GraphQL

!#### Limitations


**How many years these specs could sustain ? Are they built with a lifespan of 2-3 years or are they
built with a life span of 50 years?**

## Ideal `REST` API
> A REST API should be entered with no prior knowledge beyond the initial URI (bookmark)
> and set of standardized media types that are appropriate for the intended audience
> (i.e., expected to be understood by any client that might use the API).
>
> From that point on, all application state transitions must be driven by client
> selection of server-provided choices that are present in the received representations
> or implied by the user’s manipulation of those representations.
>
> The transitions may be determined (or limited by) the client’s knowledge of media
> types and resource communication mechanisms, both of which may be improved
> on-the-fly (e.g., code-on-demand).

* Describe what actions should include. Maybe move that before the specs ?


## I miss my good old API


## Introspected APIs
In the following we will describe the architecture of the Introspected APIs through
a proposed implementation.
The reader though should not confuse the proposed implementation details with the actual
architecture style.


### Introduction
There are 3 kinds of criticizers of REST model.
1. The ones who understand what REST is and feel that due to its complexity, they prefer loosing some features and deliver something
simpler, yet easier to implement and test and deliver a RESTfull approach
2. The ones who understand what REST brings on the table but given that they control the client as well,
why should they bother with the whole HATOAS thing?
3. The ones who don't understand REST and just want a plain JSON because it's simple enough


Introspected REST model is flexible enough to cover all those user cases.
It's not a model that is black or white: your API is either Introspected-REST-compliant or it isn't, like REST.

We want to embrace even the simplest APIs and allow them to provide the elements of REST that need, yet being easy to impelemnt
and backwards compatible.
The key thing here is backwards compatibility, because it allows you to incrementally add REST HATOAS incrementally.


### Separate Hypermedia from the actual data
JSON Hyper Schemas + HTTP OPTIONS on the endpoint

### Automate documentation


> Imagine how poor the Web would have been if we had limited HTML to what was
> needed by an FTP client. That's what most JSON APIs are today.
>
> --- Roy Fielding
>



### The case of Linked Data and Semantic Web

## Outro

It should be noted this model is not something we conceived in a lab. Some [people]()
have already tried to implement something similar, probably without really knowing
what they were doing.

You see, the shadow of Roy Fielding is above any API developer:
we are afraid to deprecate Roy's REST model and as a result what we are doing is that
we take some elements of Roy's model, apply them, and name our API or spec as RESTSful.
Eventually however, the final result is even worse. It doesn't have Roy's key elements for
a Markov-chain-like client (we still have offline contracts) yet we have added complexity
to our API for little result.

In this Manifesto we will try to kill Roy's model.
It gave us great insights but let's be pragmatic: it will never work out.

Probably Roy won't like that. He will either:
* declare that Introspected REST is a stupid manifesto that has nothing to do with his REST or
* he will declare the Introspected REST is just yet another REST as he defined it and we never
read his dissertation to actually see that we are defining yet another REST style.

In either case, given that very few has really implemented a Roy-compliant REST API means
that Roy himself failed to explain his model correctly.

We need to be brave enough and move on: Roy's HATOEAS-based REST model can be declared as deprecated.

Introspected REST is an alternative backwards compatible API. No breaking changes are needed.


Are we sliding a lot from Roy's initial model? No, we modernize it a little bit.

##### Roadmap to json-specific defined Introspected REST specs
As I said modern APIs have specific properties.
If you want to build the next Introspected-REST spec, you can follow the following reasoning.
Note that this reasoning is message-agnostic, meaning that we use here JSON just because we know it better
but your spec could use anything, yaml, xml etc.

* Start with some SANE defaults and the axion: The simpler your API (and the lesser it deviates from defaults), the simpler the introspection-meta-data should be
* Reach a concencus on a Introspection spec using already defined specs like JSON-Schemas.
* Reach a concencus on a querying language over url (filter + aggregation)
* Reach a concencus on an URL-API for attribute/association inclusion
* Reach a concencus on linking
* Reach a concencus on denoting linked/semantic data
* Reach a concencus on document structure (root element, meta attributes which should appear in the simple response as well etc)

So we keep 80% of the REST constraints and while we understand the benefits of other 20% we switch it with an on-demand alternative that makes the final thing
more flexible and powerful while keeping the final data simple.

#### Why this document
This document describes an architectural style.
It does not describe a specific spec and that's the reason it does not follow the IETF draft standards.

It's a manigesto if you wish.

_Anyone can contribute in this manifesto. Just open a pull request._


The way the introspection is made is up to the API designer, or better, up to the spec. Here we use HTTP OPTIONS as we feel that it's an approriate way.





Roy has done great work on initial HTTP spec and REST definition.
Unfortunately, very few people have truly understood the unique characteristics of networked
APIs which inspired Roy to define the REST.

The idioms of Networked Services are very peculiar.
When we have a client talking over the wire to a server,
neither the client developer nor the server developer has access to the other machine.

This means that if the client needs a specific resource, it must not have an offline contract
on how to retrieve this resource because that would mean
* changes on the server's side is difficult.
* automated API clients are not possible without human interaction.

Instead, the server would help (drive if you will) the client exactly where is needed to.

That's the main reason why Roy is against the version on the URL: because it means that
you take as de-facto that there will be breaking changes at some point.
Instead, a truly REST API should be able to apply changes on the resources without breaking any client
because the client.


* The simpler the API, the simpler the API description.
