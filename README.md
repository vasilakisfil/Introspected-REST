# Introspected REST
(Sliding away from Roy's `REST` model)

There has been a great confusion of what a `REST` API is.
Most people think that `REST` API is just a CRUD over HTTP.
Or a CRUD with some links.
Or a nicely formatted, a sophisticated CRUD.

In this _manifesto_, we will give a specific definition of what `REST` is, according to Roy,
and see **the majority of APIs and API specs** ([JSONAPI](http://jsonapi.org/format), [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08) etc) **fail to follow this model**.
Then, we will propose a **new model** that brings into the table the same things,
yet it's much simpler to implement while at the same time being backwards compatible with any current (sane) API.

As part of this _manifesto_ we will also present, MicroTypes, small reusable modules that compose a Media Type and facilitates
the evolvability and extensability of our new model.

## 1. Definitions
First some definitions, that we will use through the text:

* `REST`, `RESTful`: The model that Roy defined in his [thesis](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) (along with his blog post [REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)).
* `RESTly`: APIs that follows all parts `REST` model, except HATEOAS in which they support mostly links (specs like [JSONAPI](http://jsonapi.org/format), [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08) etc)
* `RESTless`: APIs that have a plain JSON API without any links (follows `REST` model other than HATEOAS)
* `Introspected REST`: APIs that follow the definition of the model we provide in this _manifesto_

We will use the term APIs and networked APIs interchangeably.

## 2. Introduction
`REST` defined by Roy was a magnificent piece of work, much ahead of its time
which took us 10+ years to understand what its capabilities are.
However, now, almost 20 years later `REST` model shows its age. It's unflexibile,
difficult to implement, difficult to test, with performance and implementation issues.
But most importantly, **any implementation of `REST` model is _very_ complex**.

Now, one could say that, most APIs are not build with mind to last for decades and maybe
that's the reason that this model hasn't seen much adoption.

The former is true but even if the latter is also true could it mean that this model is
not suitable for short-term APIs?

We firmly believe that `REST` is much better than any API that does not follow `REST` principles
(like `RESTly` APIs), even for short-term APIs.
Networked services have very peculiar characteristics which, until now, only `REST` has fully addressed them
(see [related Work](#related-work) for an explanation why GraphQL is not an equivelant alternative).
Being able to evolve your API without breaking the clients is critical.

Imagine the following scenario: you have built an Online Social Network and an iOS app that talks to the API on your backend.
Now imagine that, after a company meeting, your CEO needs you to make tiny yet important change in the signup page: require the user
to fill in her age, a field in the signup form you didn't have before.
Essentially, this means, in API terms, add an extra field in the accepted object and require it from the client to be filled in
by the user before sending it over.

If your API is `RESTly` and not `REST`, this means that you need to fix the code in the iOS side, test it and send a new iOS app to Apple store.
It takes roughly 1 week for Apple to review your app and if your app won't be rejected for some reason, your
tiny change will take action at least a week later after requested.
If your API _was_ `REST` that would mean a change on the server's response denoting which fields are required to submit the form.
You would have the change deployed 10 minutes later.

Roy notes in his thesis:

>  A system intending to be as long-lived as the Web must be prepared for change
>
> --- Roy Fielding
>

In the world of startups and money-driven companies, the following will sound more appealing:

>  **If you want to move fast, you should build a change-first API.**
>
>
An API that can change the state of the client without needing the latter to change.

Given that, **how can we have a simpler model than `REST`, yet derive the same, if not more, functionality of
`REST`?**

As we will show, `Introspected REST` is an API architectural style that solves that.
An architectural style is not an implementation and it's not a spec either.
As Roy notes:

>  An architectural style is a coordinated set of architectural constraints that restricts
>  the roles/features of architectural elements and the allowed relationships among those
>  elements within any architecture that conforms to that style.
>
> --- Roy Fielding

`Introspected REST` is based on Roy's initial model but removes the need for runtime HATEOAS.
Instead, the client derives the state on demand, using introspection.

Eventually this brings the same advantages as Roy's model while being it's much simpler,
much more flexible and backwarde compatible with any Restful API.

But first let's discuss about Networked Services.

## 3. Networked Services and APIs
Nowadays JSON has become so popular that developers almost forget that there is whole bunch of
protocols below it.
Developers also forget that JSON is just a specification in the message level, like XML.
It's not the only one and definitely it's not the best we could use.
Nevertheless it's simple and simplicity is a virtue.

While OSI model is the conceptual model that we use to describe computer networks,
with TCP/IP following 5 out of 7 OSI's abstraction layers, in our case, we will make a more API-specific description.
When we want to request a resource from a networked hypermedia-based API, we _roughly_
have the following levels:

### 3.1. Application level
In the application level, the client starts content negotiation (or content selection), usually asking
for only one Media Type.
A Media Type provides information about the structure of the content and the message format used in the data it describes,
as described by [RFC 2046](https://tools.ietf.org/html/rfc2046).

In the HTTP the content negotiation is achieved by a client using the Accept header which denotes the Media Types that it
can understand, in a preference order.
Then the server responds with a Media Type proposed by the client in Content-Type header.

`application/json` is a Media Type that denotes that the data format of the requested
representation is in JSON data format.
Specifically the type of this Media Type is `application` while the subtype is `json`.
**JSON itself is not a Media Type but a message format**.

Media Types can be a bit more complex as well: `application/vnd.api+json`, the media type of [JSONAPI](http://jsonapi.org/format) spec, (roughly) means that
* the main type is `application`
* the subtype is `vnd.api` which _roughly_ denotes the Media Type name
* the underlying structure follows JSON semantics

In theory, [JSONAPI](http://jsonapi.org/format) spec spemantics could also be applied using XML as the data format (like in the case of [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)),
or even YAML, however in practice we tend to forget that and we treat all Media Types as single and not composite.

However, it should also be noted that the **Media Types and the content negotiation in general, are
not restricted to HTTP only**.
Although HTTP is one of the most popular application network protocols today, the same logics could be applied
in other (mostly text-based) protocols like SIP, CoAP, QUIC etc.

To sum up, the application level semantics are defined by the Media Type requested and should not be tightly coupled to the semantics of the
message level (like JSON) or the underlying protocol level (like HTTP).

### 3.2. Message level
In the message level we find the format that is used for the actual representation.
Nowadays we have almost mixed the message level with JSON but in practice other
formats could successfully be used: XML, YAML, TOML to name a few.

Usually the message format that is used is described by the Media Type's suffix.

### 3.3. Protocol level
In the protcol level, the requests are usually sent using the HTTP.
After all, nowadays most of the development happens around the Web and
HTTP is the only protocol that browsers officially support.

Nonetheless, there are other similar stateless protocols as well.
QUIC is a HTTP alternative protocol that is targeted for low latency and uses UDP
underneath.
CoAP is targeted in the IoT and also uses UDP underneath (full TCP/IP stack is quite heavy for constrainted devices).
SIP is also a text-based protocol with the same semantics as HTTP and is used in VoIP.

### 3.4. Network level
Finally, in the network level, the browser (or any other non-browser client) sends the networked request
in one of the TCP, UDP, etc.

The actual protocol depends on the protocol used in the protocol level.


## 4. Roy's `REST` model
Roy's `REST` model is an arhictectural style which is not tight to any spec, protocol or format of the
aforementioned levels.

> a RESTful API is just a website for users with a limited vocabulary (machine to machine interaction)
>
> --- Roy Fielding


When Roy talks about `REST`, he mentions 5 crucial properties of a `REST` model:

### 4.1. Access methods have the same semantics for all resources
> induces visible, scalable, available through layered system, cacheable, and shared caches

Failure to provide consistency on access would imply that you don't provide a generic interface but instead
you have resource-specific or even object-specific interfaces.

Actually a common interface is one of the most crucial parts of REST: without a common uniform interface
it would be impossible to derive REST:

> The central feature that distinguishes the REST architectural style from other
> network-based styles is its emphasis on a uniform interface between components.
> By applying the software engineering principle of generality to the component interface,
> the overall system architecture is simplified and the visibility of interactions is improved.
> Implementations are decoupled from the services they provide, which encourages independent
> evolvability. The trade-off, though, is that a uniform interface degrades efficiency,
> since information is transferred in a standardized form rather than one which is specific
> to an application's needs. The REST interface is designed to be efficient for
> large-grain hypermedia data transfer, optimizing for the common case of the Web,
> but resulting in an interface that is not optimal for other forms of architectural interaction.
>
> In order to obtain a uniform interface, multiple architectural constraints are needed to guide the behavior of components.
> REST is defined by four interface constraints: identification of resources; manipulation of resources through
> representations; self-descriptive messages; and, hypermedia as the engine of application state.
>
> --- Roy Fielding
>

Subsequently, the next 4 constraints, the core of REST, is a result of the effort to obtain a _uniform interface_ between different components.

### 4.2. All important resources are identifed by one resource identifer mechanism
> induces simple, visible, reusable, stateless communication

Roy explains that very well in his thesis:
> A resource is a conceptual mapping to a set of entities, not the entity that corresponds to the mapping at any particular point in time.
>
> More precisely, a resource R is a temporally varying membership function M<sub>R</sub>(t),
> which for time t maps to a set of entities, or values, which are equivalent.
> The values in the set may be resource representations and/or resource identifiers. [...]
> Some resources are static in the sense that, when examined at any time after their creation,
> they always correspond to the same value set.
> Others have a high degree of variance in their value over time.
> The only thing that is required to be static for a resource is the semantics of the mapping,
> since the semantics is what distinguishes one resource from another.
>
> --- Roy Fielding
>


### 4.3. Resources are manipulated through the exchange of representations
> induces simple, visible, reusable, cacheable, and evolvable (information hiding)

The respresentation that you expose from your public API could be totally different from
your implementation internally (usually in your db).
It could also be the same.
Nevertheless the client expects and is expected to manipulate any resource using the representation
you expose.

### 4.4. Representations are exchanged via self-descriptive messages
> induces visible, scalable, available through layered system, cacheable, and shared caches
> induces evolvable via extensible communication

> Interaction is stateless between requests, standard methods and Media Types are used to
> indicate semantics and exchange information, and responses explicitly indicate cacheability.
>
> --- Roy Fielding
>

This would mean that the data of the response should follow the Media Type that the client
requested and unrestands.
Given that the client negotiated for that Media Type, **it should be able to parse and understand any part of the response**.

If your Media Type is very weak (like `application/json`) and you need functionality that the Media Type does not describe
then you need to define another Media Type which will describe the new semantics and wait until client(s) incorporate the new Media Type changes.

Breaking your Media Type's semantics, or just extending them with new functionality, will have exatly the same result for the client:
not self-descriptive messages that will require out-of-band information, like documentation.

### 4.5. Hypertext as the engine of application state (HATEOAS)
> induces simple, visible, reusable, and cacheable through data-oriented integration
> induces evolvable (loose coupling) via late binding of application transitions

This is one of the most misunderstood parts of Roy's REST model. The idea here is that,
once the client and server have reached a concensus on the Media Type after the negotiation,
the server's response should strictly provide all the available options for the client
to manipulate the resource and navigate to other resources.

As Roy notes:

> A REST API should be entered with no prior knowledge beyond the initial URI (bookmark)
> and set of standardized Media Types that are appropriate for the intended audience
> (i.e., expected to be understood by any client that might use the API).
>
> From that point on, all application state transitions must be driven by client
> selection of server-provided choices that are present in the received representations
> or implied by the user’s manipulation of those representations.
>
> The transitions may be determined (or limited by) the client’s knowledge of media
> types and resource communication mechanisms, both of which may be improved
> on-the-fly (e.g., code-on-demand).
>
> --- Roy Fielding
>

However, **one of the requirements for HATEOAS to work is that the Media Type itself _must_ allow
to its vocubulary hypermedia.**
For instance, with `application/json` Media Type this wouldn't work as JSON itself
(`application/json` Media Type is nothing more than a JSON) does not provide any of those mechanisms.

Instead, the server and client must agree on a format that provide such mechanisms.

In practice however, we put `application/json` in our Content-Type header denoting
that the response type follows that Media Type and then inside the response we add
semantics regarding hypermedia. Then we hand off out-of-band information to the client,
like documentation, and demand to check them before identifying parsing and using the hypermedia
semantics of our API.

## 5. API Clients and Applications

### 5.1. Client and Application responibilities
Client and Application responsibilities some times are mixed together.

A client is responsible for understanding, interacting with the API and manipulating any API's resources, based on the Media Type's semantics
and runtime HATEOAS. The client is responsible for providing in the application the list of resources that are available in the API,
their fields, their capabilities, available actions and any hypermedia available.

The application responsibility on the other hand should not include API specific details.
Instead, using the client, it should fetch whatever is needed by the application domain, within the API's capabilities.

Think about the traditional home telephone devices.
The phone wire and its signals is the API.
The device used to enode/decode the wire signals is the API client.
On top of a device we can run our application.
The PSTN, ISDN, (A)DSL etc are all different Media Types for the same API (wire signals).
For each one, we need a client (device/modem) that will understand (encode/decode) the wire signals of that Media Type.
Using that client we can built any type of application, in the feasible space of the Media Type.
The application does not deal with the API's semantics, but instead it uses the Client to perform its tasks.

### 5.2. The Human interaction principle
There are 2 types of human involvement when building an API client:
* **1-fold**: Programming the client only once to understand the Media Type correctly and let the
client work for any API that follows that Media Type even when APIs evolve, given that it adhere in the Media Type specs.
The only thing that the client requires is the initial URI of the API.
* **multi-fold**: Programming the client once to understand the Media Type.
Then modify the client to parse and understand the API correctly using some offline contract (like available resources, fields, pagination etc) and then
every time the API evolves (like adding a resource or a field), reprogram the client accordingly. The extend of human involvement
during that phase is variable depending on the weakness of the Media Type.

Strictly speaking, an API that follows the `REST` model should be evolvable without the need
of human interaction in the client side, given that the client understands the Media Type.
As a result, **versioning should not take place in the URL but in the Media Type itself**.

>Versioning an interface is just a polite way to kill deployed applications
>
> Roy Fielding
>

## 6. REST applied in a modern API
When engineering a REST API, there are 2 approaches:
* design a specialized, usually UI-driven, API: the resources and their browsability is tightly coupled with the specific application that was built for
* design a generic, usually data-driven, API: the resources are more generic and the API's capabilities allow a plethora of transformations.

Specialized APIs could be more efficient, or have crucial advantegeous characteristics for the domain that were built for
since they are optimized only for that specific case.
However, they pose difficulties when they need to be reused by any other application which does not share the same UI.
As a result, such APIs are very special and a bit rare.

On the other hand, the data-driven APIs, are more generic and facilitate any application to request the data optimized
(in the framework of the API's capabilities) for its use case.
Being able to specify your application's needs when requesting data from an API is crucial,
especially if your business depends on the adoptability of your API.

For the following subsections, we will mostly focus in the generic data APIs,
however most of the things mentioned here can also be applied in a specialized or UI-driven API.


### 6.1. Requirements from a modern REST API
REST model is built for machine-to-machine communication, of any type.
However, as this form of communication is getting more and more common,
clients are requesting and having more and more requirements from a server response.
It's not enough to just request and get the resource but you should be able to specify
to the server what transformations you need.

Nowadays we have been using networked APIs so much that now we essentially have to
provide an ORM to the client over the HTTP (or any other protocol).

We provide here a list of features (we call them capabilities) that we think should be built in a modern networked API,
in 2017.

While most of these capabilities are related to Data APIs, some of them apply to UI APIs as well.

#### 6.1.1. Sparse fields (collection/resource)
The client should be able to ask and get specific attributes (i.e. a subset) of the resource representation.
Also related, we should note that a representation of a resource could have completely different set of
attributes for different clients, usually depending on the client's permissions or user role that it represents.

#### 6.1.2. Associations on demand (collection/resource)
The client should be able to ask related associations to the main initial resource, in the same request.

What deffirintiates an association from an attribute is that the former has
a dedicated identification. What is more, if the API exposes the association as a dedicated resource,
the id can be used as identification.

#### 6.1.3. Sorting & pagination (collection only)
The client should be able to sort based on one or more attributes and paginate the collection
based on the page, page size and possibly an offset.

#### 6.1.4. Filtering collections (collection only)
The client should be able to run any sort of collection filtering, as long as it does not pose
any security thread or slows down the API performance.

#### 6.1.5. Aggregation queries (collection only)
The client should be able to run any sort of aggregation queries, as long as it does not pose
any security thread or slows down the API performance.

#### 6.1.6. Data types !
The client should know the data types of the attributes of the requested representation of a resource.
Message formats provide some data types but they are pretty basic.
For instance, JSON defines `String`, `Boolean`, `Number`, `Array`, and `null`.
Anything more than that we need to define it in the documentations.

**We feel that these 5 data types that JSON provides are just a joke for modern APIs** and that we should
have a much larger list of options to select from.
Additionally, we should be able to provide custom types in an easy way, for instance, a field is `String` but
has maximum length of 255 characters, it follows a specific regex etc.

#### 6.1.7 The list does not end here
Although we feel that _today_ these capabilities should exist in any modern API, **this list is not exlusive**.
In fact, there could be capabilities in the future that might not seem necessary today, but help the client can get the necessary data in the structure needed.
For example, joining together one or more resources, other db-inspired operations applied on resources,
internationalization and localization of the data and other capabilities that we haven't thought yet.
In any case, **these capabilities must be transparent to the client without any documentation or human involvement**.


### 6.2. Media Types vs HATEOAS
Now the reader could be wondering: where is the appropriate place to describe those capabilities,
in our API's Media Type or using HATEOAS ?
What goes where?

#### 6.2.1. Defining a new Media Type is not easy and should be avoided
Creating a new Media Type for our API is genrally considered bad practice.
Create a new Media Type only if you are sure that none of the already published
Media Types can fit in your API design.

Also, extending an existing Media Type or adding a complementing Media Type to an
existing one (like `application/vnd.api+json+my_custom_data_types`) wouldn't work.
Not only the existing Media Type specification does not provide any extensibility principles,
but also, the main reason is that **the client _must_ understand the Media Type before hand**.
As a result, if we would like to use some _new_ custom types in our (already deployed) API, we would have to publish
the Media Type before hand and let **humans** implement code to fully parse API responses that
follow this Media Type or API responses that their media type also include this new media type.


#### 6.2.2. HATEOAS can get pretty heavy
Imagine if you have to describe in a resource, all the available actions along with the available API
capabilities _in that specific resource_.
Your API response would just explode in terms of size while making your API super complex.

#### 6.2.3. Balancing between Media Types and HATEOAS
The idea is that Media Types descibe the generic capabilities while HATEOAS
describe the resource-specific capabilities.

However we should note that **Media Types are not parsed by the client** (there was never such intention anyway)
which means that the client must be programmed by a human before hand in order to support that Media Type.

As a result, the Media Type can't be very restrictive because that would mean it would restrict the API designer's freedom
to design the API the way she wants.

For instance, for pagination, most RESTy APIs use a `page` and a `per_page` parameter in the URL.
If the Media Type describes how to do pagination using, say, a URL template on the resource path (like `/{resource}?page={page}&per_page={per_page}&offset={offset}`)
this would mean that all APIs following this Media Type should have the pagination following that URL template.
The level of restriction becomes more obvious when describing more complex capabilities.

On the other hand, if everyone follows that Media Type then it's easier to program our clients.
Specifically, especially when having a restrictive Media Type, if we create a client that parses responses using that Media Type
then it's easy to "configure" it for another API which also follows that Media Type.

HATEOAS should describe on a per-resource basis if the pagination is supported, what is the maximum `per_page` etc.
Essentially, HATEOAS should provide any details missing from the Media Type for the client to work.

#### 6.2.4. An alternative architecture
We feel that the current Media Type specification and use is dated.
If Software Engineering has learned us something, is that composition can enforce Single Responsibilty Principle if used correctly.
Inspired by that, later, we will suggest a new concept,  MicroTypes, small reusable modules that combined together can form a Media Type.
As a result, clients should be able to even negotiate parts of the Media Type and not the Media Type as a whole.

Also, instead of mixing up data with HATEOAS in the API responses, we will introduce introspectiveness of our resources.


## 7. API Specs Today
Now that we defined what REST is, according to Roy, and what capabilities modern APIs should provide,
let's see the specs for REST(y) APIs available as today, April 2017, and what they provide.

### 7.1. Our use case
In our use case we will follow the aforementioned points of the `REST` model.

Our use case is a minature of yet another Social App.
Specifically, in our API domain, we have a `User` resource which has other, associated resources, like `Micropost`, `Like`, etc

For our message format, we will use JSON as it's the most popular but it could be anything like XML, YAML etc.

* `User`
  * `id`, a String, never empty or NULL, primary ID of the resource
  * `email`, a String, never empty or NULL, with maximum length up to 255 characters, email format
  * `name`, a String, with maximum length up to 150 characters
  * `birth_date`, a String, representing a Date according to `iso8601`, in `2017-04-01` format.
  * `created_at`, a String, never empty or NULL, representing a DateTime according to `is8601`, in UTC
  * `microposts_count` an Integer

So given those `REST` model properties we _could_ have the following routes:
* `Users` resource (`/users`):
  * List users (`GET /users`): Gets a collection of `User` resources
  * Create a new user (`/users`): Creates a new `User` with the specified attributes.

* `User` resource (`/users/{id}`):
  * Get a user (`GET /users/{id}`): Gets the attributes of the specified `User`
  * Update a user `PATCH /users/{id}`: Updates a `User` with the specified attributes
  * Delete a user `DELETE /users/{id}`: Updates a `User` with the specified attributes

_`Users` and `User` are 2 distinct resources which are often, mistankingly, missthought as a single, one, resource_

As we mentioned, `User` resource has also some associations (or relations/relationships if you prefer).

#### 7.1.1. User resource
In plain JSON the User resource would look like:
```json
{
  "user": {
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }
}
```

#### 7.1.2. Users resource (a collection of User resources)
A collection of `User` resources, the `Users` resource, would look like:

```json
{
  "users": [{
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }, {
    "id":"9124",
    "email": "robert.clarsson@gmail.com",
    "name": "Robert Clarsson",
    "birth_date": "1940-11-10",
    "created-at": "2016-10-06T16:01:24Z",
    "microposts-count": 17,
  }]
}
```


Now that we defined the scope of our little API, let's see how this would be implemented
in the specs for REST(y) APIs currently available. We feel that most current APIs
have a lot of similarities with the following specs, namely the structure and the HATEOAS part (regarding
linking), and as a result by comparing those specs with our model would be sufficient.

We will evaluate the specs for the following:
* whether they follow Roy's `REST` model
* whether their messages are **not** self descriptive, meaning that other than supporting the API's Media Type in our client
we also need to read and understand the documentation to develop our client
* whether they require multi-fold human interaction while the API evolves

### 7.2. [JSONAPI](http://jsonapi.org)
JSONAPI was originally created by [Yehuda Katz](http://yehudakatz.com/), as part of Ember's ember-data library.
Since then a lot of people have contributed and has rised as one of the most supported
API specs as of 2017 in terms of tools and libraries.

#### 7.2.1. User resource
```json
{
  "data":{
    "id":"1",
    "type":"users",
    "attributes":{
      "email":"vasilakisfil@gmail.com",
      "name":"Filippos Vasilakis",
      "birth_date":"1988-12-12",
      "created-at":"2014-01-06T20:46:55Z",
      "microposts-count":50
    },
    "relationships":{
      "microposts":{
        "links":{
          "related":"/api/v1/microposts?user_id=1"
        }
      },
      "likes":{
        "links":{
          "related":"/api/v1/likes?user_id=1"
        }
      }
    }
  }
}
```

#### 7.2.2. Users resource (a collection of User resources)

```json
{
  "data":[
    {
      "id":"1",
      "type":"users",
      "attributes":{
        "email":"vasilakisfil@gmail.com",
        "name":"Filippos Vasilakis",
        "birth_date":"1988-12-12",
        "created-at":"2014-01-06T20:46:55Z",
        "microposts-count":50
      },
      "relationships":{
        "microposts":{
          "links":{
            "related":"/api/v1/microposts?user_id=1"
          }
        },
        "likes":{
          "links":{
            "related":"/api/v1/likes?user_id=1"
          }
        }
      }
    },
    {
      "id":"9124",
      "type":"users",
      "attributes":{
        "email":"robert.clarsson@gmail.com",
        "name":"Robert Clarsson",
        "birth_date":"1940-11-10",
        "created-at":"2016-10-06T16:01:24Z",
        "microposts-count":17
      },
      "relationships":{
        "microposts":{
          "links":{
            "related":"/api/v1/microposts?user_id=9124"
          }
        },
        "likes":{
          "links":{
            "related":"/api/v1/likes?user_id=9124"
          }
        }
      }
    }
  ],
  "links":{
    "self":"/api/v1/users?page=1&per_page=10",
    "next":"/api/v1/users?page=2&per_page=10",
    "last":"/api/v1/users?page=3&per_page=10"
  }
}
```

#### 7.2.3. Reflections

While the spec makes a great effort describing the structure of the document, we see some
notable issues. Namely:
 * Limited links (no URI templates, treats the client as totally stupid)
 * No actions
 * No info on available attributes
 * No info on data types
 * No attributes description

To sum up, it doesn't entirely follow `REST` model while it requires both
documentation and multi-fold human interaction.

### 7.3. [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)
HAL was created by Mike Kelly in 2012.
The key feature of HAL when it was released was the browsability/explorability of any API that adopted.
Another feature is the idea of curies, links inside the resource that lead to the documentation.
However, this feature is rather controversial since the information these links provide are targeted for humans and not machines.

The resources of our use case that are presented here use JSON as a message format, but HAL is not tighed to that.

#### 7.3.1. User resource
```json
{
    "_links": {
        "self": {
            "href": "/api/v1/users/{id}"
        },
        "microposts": {
            "href": "/api/v1/microposts/user_id={id}",
            "templated": true
        },
        "likes": {
            "href": "/api/v1/likes/user_id={id}",
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

#### 7.3.2. Users resource (a collection of User resources)
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
                "href":"/api/v1/users/{id}",
                "templated": true
              },
              "microposts":{
                "href":"/api/v1/microposts?user_id={id}",
                "templated": true
              },
              "likes": {
                "href": "/api/v1/likes/user_id={id}",
                "templated": true
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
                "href":"/api/v1/users/{id}",
                "templated": true
              },
              "microposts":{
                "href":"/api/v1/microposts?user_id={id}",
                "templated": true
              },
              "likes": {
                "href": "/api/v1/likes/user_id={id}",
                "templated": true
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

#### 7.3.3. Reflections

While the spec does have templated links, we see some notable issues. Namely:
 * No actions (they are supported by an unofficial extension)
 * No info on available attributes
 * No info on data types
 * No attributes description, requires documentation (however it does provide a link to documentation)

To sum up, it doesn't entirely follow REST while it requires documentation and multi-fold human interaction (curries facilitate that).

### 7.4. [Siren](https://github.com/kevinswiber/siren)
Siren was created by Kevin Swiber in 2012 and revolves around _entities_, a URI-addressable resource that has properties and actions associated with it.

The resources of our use case that are presented here use JSON as a message format, but Siren is not tighed to that.

#### 7.4.1. User resource
```json
{
  "class": [ "user" ],
  "properties": {
    "name": "Filippos Vasilakis",
    "email": "vasilakisfil@gmail.com",
    "createdAt": "2014-01-06T20:46:55Z",
    "micropostsCount": 50,
  },
  "actions": [
    {
      "name": "get-user",
      "title": "Get User",
      "method": "GET",
      "href": "https://example.com/api/v1/users/1",
      "type": "application/json",
    },
    {
      "name": "update-user",
      "title": "Update User",
      "method": "PUT",
      "href": "https://example.com/api/v1/users/1",
      "type": "application/json",
      "fields": [
        { "name": "name", "type": "text" },
      ]
    },
    {
      "name": "delete-user",
      "title": "Get User",
      "method": "GET",
      "href": "https://example.com/api/v1/users/1",
      "type": "application/json",
    }
  ],
  "links":[
    { "rel":["self"], "href":"https://example.com//api/v1/users/1" },
    { "rel":["microposts"], "href":"/api/v1/microposts?user_id=1" }
    { "rel":["likes"], "href":"/api/v1/likes?user_id=1" }
  ]
}
```

#### 7.4.2. Users resource (a collection of User resources)
```json
{
  "class":["users"],
  "properties":null,
  "entities":[
    {
      "class":["user"],
      "rel":["https://example.com/v1/users/1"],
      "href":"https://example.com/v1/users/1",
      "properties":{
        "name": "Filippos Vasilakis",
        "email": "vasilakisfil@gmail.com",
        "createdAt": "2014-01-06T20:46:55Z",
        "micropostsCount": 50,
      },
      "links":[
        { "rel":["self"], "href":"https://example.com//api/v1/users/1" },
        { "rel":["microposts"], "href":"/api/v1/microposts?user_id=1" }
      ]
    },
    {
      "class":["user"],
      "rel":["https://example.com/v1/users/9124"],
      "href":"https://example.com/v1/users/9124",
      "properties":{
        "email": "robert.clarsson@gmail.com",
        "name": "Robert Clarsson",
        "birth_date": "1940-11-10",
        "created-at": "2016-10-06T16:01:24Z",
        "microposts-count": 17,
      },
      "links":[
        { "rel":["self"], "href":"https://example.com/api/v1/users/9124" },
        { "rel":["microposts"], "href":"https://example.com/api/v1/microposts?user_id=9124" }
        { "rel":["likes"], "href":"https://example.com/api/v1/likes?user_id=9124" }
      ]
    }
  ],
  "actions":[
    {
      "name":"create-user",
      "title":"Create User",
      "method":"POST",
      "href":"https://example.com/v1/users/",
      "type":"application/json",
      "fields": [
        { "name": "name", "type": "text" },
        { "name": "email", "type": "text" },
        { "name": "birth_date", "type": "date" },
      ]
    }
  ],
  "links":[
    {"rel":["self"], "href":"https://example.com.api/v1/users"},
    {"rel":["next"], "href":"https://example.com.api/v1/users?page=2"}
  ]
}
```

#### 7.4.3. Reflections
The spec takes a huge leap towards REST principles by supporting, links, actions with fields and data types, there are
still some issues that require human-involvement:
 * No custom types for the attributes of actions
 * No info on available and required attributes
 * No info on data types on response objects
 * Limited description for fields and resources

To sum up, it doesn't entirely follow REST while it requires documentation and multi-fold human interaction.

## 8. Ideal `REST` API
**How many years these specs could sustain in terms of evolvability ? Are they built with a lifespan of 2-3 years or are they
built with a life span of 50 years?**

### 8.1. Capabilities of an Ideal `REST` API
In an ideal REST API, the client should be able to have all the necessary information for both
the request and response.

* About each resource returned from the API to the client:
  * default attributes and available attributes of the resource, based on the user's permissions
    * default attributes is a subset of the available attributes
  * data types for each attribute in the resource or any embedded association
  * Sorting/pagination, filtering and aggregation queries availability
  * data type of each attribute
  * Ideally some description targeted for humans
  * default embedded associations and available associations to embed
    * recusrively apply the same information for each association available for embedding
* About each resource sent to the API from the client
  * available actions on the resource
  * attributes, per action, the client can modify, based on the user's permissions
  * required attributes of a resource (attributes a resource _must_ before sending it over)
  * data types of the attributes (could be different from the resource found in the response)
  * associations that are required or can be embedded to the initial request
    * recusrively apply the same information for each association available for embedding

Although this list is not exhaustive, an architecture style is timeless anyway,
we feel that the aforementioned capabilities ought to appear in an idealized modern REST API.

We should also note that the reason we don't mention anything about the headers that are required, or, the status codes
is because we feel that these belong to the Protocol level and not in the Application level.
Any changes on this level imply that the API breaks the protocol.

However, we are pragmatic and we understand that an API designer could want to _add_ (not change)
a status code or a header in a given request/response and as a result, ideally, this should also be possible to be described.

#### 8.1.1. Today's REST is far from ideal
Now to the reader, it should be obvious that even if we manage to offload some of the aforementioned information
to the Media Type, we would still have a _very_ complex, massive, response from the server for the HATEOAS.

In our experience, such responses are very hard to implement correctly, test, be performant and even debug. After all,
a human will sit down and write the initial code and debugging the code by the eye is important.
Simplicity is crucial.

Another thing: some clients might not be interested in hypermedia and evolvability at all. Only the data.
However such APIs force the clients to deal with it.

Ideally we would like to give the option to the client to decide the extend of the hypermedia that it
will support and follow, without taking on defaults. Some clients might want to follow 100% the HATEOAS part
of the API (and as a result be evolvable) some other clients might want the 50%, some clients might be interested
only in data.

By outputing a whole bunch of hypermedia-related information to the clients that, after all, might never use
them is a bad practice.


#### 8.1.2. Making an API REST-compliant by downplaying its capabilities
One could argue that we require all APIs to support features that shouldn't, like resource manipulation.
For instance, we could have a weather API with `application/vnd.weather+yaml` Media Type
that is only supposed to provide a single attribute with its value, as Integer:

```yaml
temperature: 25
```

This API _should_ be REST-compliant by not providing any API capabilities, hypermedia or actions.
The imaginery Media Type `application/vnd.weather+yaml` is supposed to provide all the necessary information
because otherwise the client would fail to udnerstand things like

* what are the attributes of the response
* what is the data type of the temperature value (float, double, integer, bignum etc)

We feel that although this is true, most APIs are not as simple as that.
Moreover such APIs can't actually be evolved without releasing a new Media Type and breaking the existing API clients.
**There is no way of introducing change, which essentially breaks REST's principles.**

However we are pragmatic: we understand that such APIs will exist and engineers want to spend as less time as possible to build such APIs.
Introspected architecture solves that by serving hypermedia information on side and in an incremental way without breaking
the simplicity.

#### 8.1.3. A JSON API back in time
A JSON-based API built around 2006 would return just data. No hypermedia, no HATEOAS, only data.

In our use case, User resource would look like this:
```json
{
  "user": {
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }
}
```

As simple as that.

Compared with a HATEOAS-ed response it's simple as hell, obvious, easy to debug and understand by a human (and a client).

**Is it possible to build an API that is simple as that, be Hypermedia driven and give the client the option to decide
the level and type of HATEOAS it will follow?**

### 8.2. Deriving the need for a new model
#### 8.2.1. REST is complex
As we descrined earlier, mixing data with hypermedia leads to increased complexity, for both the server and the client developer.
Just compare the size of our [resource's data](#711-user-resource) and the size of our resource when represented by
[Siren](#741-user-resource), a hypermedia-ed API that doesn't even being REST-compatible by missing numerous information as described
in its [reflections](#743-reflections).
Imagine how bloated the response would look like, if we would added all the capabilities described in [section 6.1](#61-requirements-from-a-modern-rest-api).

Moreover, the hypermedia must be tailored for the user role the client acts on behalf of.
For instance, a user with very basic access role might only have access to retrieving resources and not manipulating them.
Or such role could only have access to specific capabilities of the API.
As a result, the hypermedia provided on the response object should reflect that by not providing hypermedia that will lead to unauthorized access.
In fact, such design is quite difficult to implement and test from the server side.

#### 8.2.2. REST enforces possibly useless information
In REST, even if the hypermedia are rendered by taking into account the user's role, eventually we might send more data that the client wants.
**Exactly because we don't know in advance what the client might need, we must send all the possible hypermedia options to the client, just in case**.
The client however could only be interested in the data, or specific hypermedia types, like only links, but instead gets a fully bloated response by the server.

#### 8.2.3. REST sacrifices performance for evolvability
Complex or long-lived APIs tend to have many hypermedia data (links, actions, custom metadata) related to the resource itself, its associations and related resources.
As a result, even if the actual data could be very small the resulted response object gets much larger in size slowing down the server rendering
and the client receiving and parsing.
The performance issues become more apparent on lossy networks like mobile clients, a trend that has increased over the past decade,
or on constrained devices and environments, like IoT.

#### 8.2.4. REST does not support caching of hypermedia
In practice, the hypermedia part of a resource rarely changes, compared to the resource's data.
In REST, by design, the client can't cache the hypermedia part of the resource, even for relatively small amount of time, because
hypermedia is part of the resource, thus caching the hypermedia can't be separate from caching the response itself.

#### 8.2.5. REST doesn't make it easy to evolve hypermedia
Another issue of REST is that due to the fact that everything is mixed in together, evolving hypermedia separately from the data
can't happen.
We understand that this is actually another feature of REST design and not an issue, treating a response object as a whole and not breaking into
different parts like hypermedia and data, however, in practice, this poses difficulties for easier evolvement and maintenance of the API.

#### 8.2.6. REST's power is limited by HTTP and related protocols (SIP, CoAP etc)
Although REST is not dependent on any protocol or spec, the truth is that it has dominated in HTTP.
As we [described earlier](#31-application-level), in protocols like HTTP, content negotiation between client and server is achieved using Media Types,
which is the the only mechanism to define the semantics of a response.
Given that composite Media Types never had real compositability, and the fact that they cannot be parsed by clients, there is a trade off between what should go to the Media Type and what to the actual response through
hypermedia, as described in section [6.2.3](#623-balancing-between-media-types-and-hateoas).
This limits the design flexibility and evolvability.
As a result Media Types become big monoliths that are unflexible and limit the evolvability of the API.

##### 8.2.6.1. No backwards compatible with any RESTly or RESTless API
In a perfect world, APIs are built to be alive for many decades and clients are exploiting every little feature of the API and its Media Type.
However, we are in a pragmatic world where nothing is perfect and clients are built by humans who take decisions based on their time and money.

Although we firmly believe that a REST api is better than any RESTly or RESTless API, we understand that there could be cases where API designers
_have_ to initially skip hypermedia part.

The problem is that when REST is applied to HTTP, it doesn't allow you to easily integrate hypermedia at a later point.
The reason is that, in a RESTless API, adding hypermedia at a later stage would mean that we would need a new Media Type because
otherwise it would break the current semantics.

We would like to see a model that embraces both architectural API styles:
* APIs that are built to last decades and thus, support full hypermedia from the very first day of their release
* APIs that don't have hypermedia (the reason is none of our business), yet it is required to add hypermedia, later, in an incremental way without
breaking existing clients or limiting API's flexibility

##### 8.2.7.2 REST does not embrace composition
Although REST does not rejects the idea of composability of different API capabilities using different specs in the same response, or composite Media Types,
it doesn't embrace it either.
The symptom of non-composability is clearly visible in protocols like HTTP where Media Types
act as big monoliths trying to describe everything in one place.

As we will see later, the MicroTypes is a solution to the outdated Media Type concept that allows us to mix-in different concepts for different
kind of metadata of a resource, yet have all of them on demand and separated by the actual data.

## 9. Introspected REST
>  Simple things should be simple and complex things should be possible.
>
> --- Alan Kay
>
>

In the following section we will describe our new architectural style based on a model for Networked APIs that goes beyond `REST`.
The model itself steps on initial Roy's `REST` model but with the difference that instead of providing resource hypermedia at
runtime, it provides them on the side, only if requested.
Hence, by keeping the _uniform interface_ the derived 3 out of 4 REST constraints that Roy defined still exist in this model:
_identification of resources_; _manipulation of resources through representations_ and _self-descriptive messages_.
However instead of having the constraint of _hypermedia as the engine of application state_ (HATEOAS), we have
_introspection as the engine of application state_ (IATEOAS).

Composition of different specs is a vital part of our model and for that we will use a new concept,
MicroTypes, small reusable modules that a final Media Type can be composed of.
Hence, before moving on, we will give concise definitions over hypermedia and metadata and break it down to different kinds of classes.
These terms can overlap in the REST model, however we feel that each one has its own place in Introspected REST that embraces composability.


### 9.1. Data, metadata and hypermedia
#### 9.1.1. Data
Data is the main variables of a resource, at a given state, at a given time.
Data is very volatile compared to other parts of a response.

#### 9.1.2. Hypermedia
Originally the hypermedia term was mostly used for linked data, in the sense of hyperlinks.
In `REST`,eventually it also includes information for interaction and resource manipilation.
Hypermedia can be dynamic or static but regardless **they are not considered part of the response data, because they define
ways to interact with the data**.

Hypermedia is a very broad term and needs to be broken down in different parts.
Although there isn't any clear definition or concencus in the literature and the community, we will try to provide definitions and descriptions for
all the different types of Hypermedia, according to our model's perception.

##### 9.1.2.1. Links
The most basic class of hypermedia, basically URIs or IRIs that can be used to provide linking between releated resources to the primary resource.
The properties of a link, like placement inside the response, strictly follow the semantics of the Media Type agreed.

##### 9.1.2.2. Actions
Actions are links along with information for manipulating a resource.
Although CRUD are the most popular actions of a resource, the beauty with REST, and consequently Introspected REST, is that actions can go beyond plain CRUD.
In fact, you can define any type of action or meta-action of your internal resource, through the representation that you expose.
As a result, actions of a resource could be quite complex or simplistic depending on the needs and decisions of the API designer.
In any case, actions should also describe any relevant information for the client to perform it, unless the Media Type itself describe those details.

##### 9.1.2.3. Forms
Another way of describing the manipulation options of a resource is the notion of forms.
The difference between actions and forms is that the latter are mostly targeted for the UI.
For instance, they could provide all the necessary information to be semantically equivelent to an HTML form,
or could provide UI-related information like positioning, suggested colors, etc.


#### 9.1.3. Metadata
If hypermedia is links and actions, then what is metadata ?

Metadata are information about the resource that is not related with the data, including hypermedia.
In essence, metadata is a superset of hypermedia and it's crucial for the client
to understand API's responses and access the API's capabilities and manipulate the resources.

Metadata could be **API-specific, resource-specific, action-specific or even object-specific**.
There could also be different kinds of metadata like runtime (pagination information), structural (data types of an object),
hypermedia (links, actions, forms), informational (targeted for humans).
Last but not least, metadata are not restricted to a response but a request could also have metadata.

Usually metadata are not volatile, except runtime metadata that depend on the request and the resource at the given time and state respectively.



### 9.2. MicroTypes: modules composing a Media Type
> Imagine how poor the Web would have been if we had limited HTML to what was
> needed by an FTP client. That's what most JSON APIs are today.
>
> --- Roy Fielding
>

Currently media types act as a big monolith that clients need to understand beforehand through human involvement.
We believe that Media Types should be broken in smaller
media types, microtypes, each describing very carefully a specific functionality of a modern API.

Clients and Server should still do the regular negotiation flow even for those sub-media-types.

The reasoning is that, in our experience, we have seen that different API specs define the same functionalities in different ways.
Common foobar should allow us to interchange each of them lorem ipsum.

We will need a MicroType for describing each of following:

* querying language over url (filters, aggregations, pagination and sorting)
* requesting specific attributes/associations of a resource and its associations
* linking other resources or associations
  * figure out when links should be placed on the introspection and when in Links header defined by RFC5988
* semantic data of a response
* actions supported by the resource
  * required fields, available fields
* resource data types

+ specfy uploads
+ specify http2 server push
+ etc

#### 9.2.1. MicroTypes in HTTP
The Content-Type header is limited up to 128 characters so we might need another header for that.
Content-Type could describe the overall Media Type while Foo header could describe sub-media-types used to produce that Media Type.

### 9.3. Introspection as the engine of application state (IATEOAS)
In the following section we will describe the architecture style of the Introspected REST.
The main principles of Introspected REST build upon Roy's initial REST model but deviates in the way HATEOAS is derived
Specifically the state of the client is Introspected, possibly cached.
The main idea is that we separate any metadata from the actual data and deliver the metadata on the side, on demand.


#### Definition of Introspection
The idea of introspection is to be able to examine properties of a system at runtime.
In the case of Introspected REST, introspection defines a process for a client to be able to introspect
the API's, resource's, action's or even object's metadata at runtime.

We believe that REST fits perfectly for the implementation of that process.
However we would like to point out some key properties of the introspection.

##### Composition over monolithic architecture
Introspected REST encourages the use of different MicroTypes to form a Media Type instead of trying to define everything in the same spec.

This would allow us to break the metadata in different

##### Plain data separated from metadata
The process for requesting metadata of an API should be different from requesting data, denoting the sense of introspection.
As a result, introspection responses should not include any data but only metadata while when requesting data,
responses should not include any metadata, apart from runtime metadata.

##### Querying MicroTypes
The process should allow to query different MicroTypes.

##### Caching
The process should make it possible to allow the client to cache the metadata using the protocol's tools.
For instance, in HTTP, the server should denote the max age of the metadata.

Moreover, the process should make it possible to cache all metadata for each MicroType at once.
This would speed up the client.

##### Method of transport
The server can describe the meta-data of a resource in the response body of the `OPTIONS` request.
The reason we choose `OPTIONS` here is because this method has been historically used
for getting informtation on methods supported on a specific resource.

HAve some properties:
* ability to be cached
* separated from data
* support queries

#### Request Response inconsistency


#### Automating the documentation generation
documentation generation could have extra stuff, by assigining a param in the url.


## 10. An Introspected REST API prototype in the world of HTTP and JSON
In the following we will describe the architecture of the Introspected REST APIs through
a proposed implementation.
The reader though should not confuse the proposed implementation details with the actual
architecture style.

This is by no means a Media Type, but just an example of the potential of Introspected REST.
The actual MicroTypes and Media Types should be created by the community.

We will use JSON and JSON Schemas.
But the reader could apply the same ideas using any message format.

### Separating meta-data from the actual data
#### Classes of meta-data

#### Plain Data
The main purpose of introspected REST _manifesto_ is to **separate actual data from resource meta-data, like hypermedia**.

When the client requests a resource (using `GET` method), it should get only the data:

```json
{
  "user": {
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }
}
```

The actual format of the data could vary regarding the root element or possible the place of the primary id, but essentially
the data does not contain any hypermedia or meta-data.

We should note that resource's meta-data that depend on the data ( which depend on the actual request for that time)
like pagination information, should be still in the same response.
The way those meta-data are represented is left to the API designer, usually though they are put under a
`meta` attribute regardless if it's a resource or a collection of resources.

#### Structural meta-data
In order to describe our data, we will use JSON Schemas.
It's a vocubulary that enabled to describe and as a result validate JSON data.

For our use case the JSON schema would be the following:

##### User resource

```json
{
	"$schema": "http://json-schema.org/draft-04/schema#",
	"id": "http://example.com/example.json",
	"properties": {
		"user": {
			"properties": {
				"id": {
					"maxLength": 255,
					"type": "string"
				},
				"email": {
					"maxLength": 255,
					"type": "string",
					"format": "email"
				},
				"name": {
					"maxLength": 255,
					"type": "string"
				},
				"birth_date": {
					"type": "string",
          "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}$"
				},
				"created_at": {
					"maxLength": 255,
					"type": "string",
					"formate": "date-time"
				},
				"microposts_count": {
					"type": "integer"
				}
			},
			"required": [
				"id",
				"email",
				"name",
				"birth_date",
				"created_at",
				"microposts_count",
			],
			"type": "object"
		}
	},
	"required": [
		"user"
	],
	"type": "object"
}
```

##### Users resource

```json
{
  "$schema":"http://json-schema.org/draft-04/schema#",
  "id":"http://example.com/example.json",
  "properties":{
    "users":{
      "items":{
        "additionalProperties":false,
        "properties":{
          "id":{
            "type":"string"
          },
          "email":{
            "type":"string"
          },
          "name":{
            "type":"string"
          }
          "birth_date":{
            "type":"string"
          },
          "created_at":{
            "type":"string"
          },
          "microposts_count":{
            "type":"integer"
          },
        },
        "required":[
          "name",
          "created_at",
          "email",
          "microposts_count",
          "birth_date",
          "id"
        ],
        "type":"object"
      },
      "type":"array",
      "uniqueItems":true
    }
  },
  "required":[
    "users"
  ],
  "type":"object"
}
```
Essentially the JSON Schema describes the data types and the structure of the data document.

### Request Response inconsistency



### Hypermedia
For the Hypermedia part we will use JSON Hyper Schemas



### Method of transport
The server can describe the meta-data of a resource in the response body of the `OPTIONS` request.
The reason we choose `OPTIONS` here is because this method has been historically used
for getting informtation on methods supported on a specific resource.

### Automating the documentation generation
documentation generation could have extra stuff, by assigining a param in the url.



## Related Work
### GraphQL
no urls, basically a query language, no MicroTypes

### Linked Data and Semantic Web

#### JSON-LD and HYDRA

### RESTful API Description Languages
Over tha past years, there has been a trend on creating API documentation through specialized tools, like OpenAPI specification (ex. Swagger).

As we have already noted, in a REST API documentation, in the sense of offline contracts,
shouldn't even exist and thus such approach is fundamentaly wrong.
By giving so much weight on the documentation but at the same time treating it as something different, separated from the code
leads to inconsistencies beteween the actual API and the API description.
Those tools have been improved so much lately that now allow you to write the documentation and let them generate
the basis of your code, depending on your language/framework, which could fix the inconsistencies issues.
However, unfortunatey such a approach leads to an RPC design instead of a hypermedia-based system.


We believe that API designers are limited by marrying these tools.
The tools themselves have limitations,
but also, having tools that aim to provide all-in-one to the API designer is against our philosophy: tools that do one thing and do it well.


### API directories
Another trend for APIs is to register them  in an online service, called API dictionary and possible push there the API documentation as well.
We feel that this is not a very helpful structure. APIs should be discoverable by themselves without using centralized services.
The API's root url should provide everyhing that is needed, or using already published protocols
like WebFinger, which builds upon [Well-Known Uniform Resource Identifiers RFC](http://www.rfc-editor.org/rfc/rfc5785.txt) and can give API information
for client bootstraping.

### Overloading Link rel
There is a tendency to overload Link rel for links unrelated to application format etc.
We feel that this is a bad practice and definitely not the right location to add the Microtypes.
Link rel should be used for very few specific things.
For instance Media type and links. Not overloading. Dereference only.

### Linksets
Linksets is yet another work forwarded by Erik Wilde. The idea is...

We think that linksets is anohter small piece towards an introspected REST APIs.

## Future Work
It is obvious that after this Manifesto freezes people will start researching more on the introspected-based.

We would like to give some guidelines towards that direction.


## Conclusion
We are not giving a solution here. We are giving food for thought.

We are not giving a solution here. We are giving food for thought.

We see that people fail to understand the full extend of Roy's initial `REST` model and what is happening is that
some elements of that REST are applied, some other not and eventually we have a model that has the downsides of both worlds.

In this _manifest_ we showed that we need to separate actual data from resource metadata and hypermedia. Documentation
should be generated.

No you don't need GraphQL

We want to embrace even the simplest APIs and allow them to provide the elements of REST that need, yet being easy to impelement
and backwards compatible.
The key thing here is backwards compatibility, because it allows you to incrementally add REST HATEOAS incrementally.

In this Manifesto we will try to kill Roy's model.
It gave us great insights but let's be pragmatic: it will never work out.

We need to **be brave enough and move on**: Roy's HATEOAS-based REST model can be declared as deprecated.

Introspected REST is an alternative backwards compatible API. No breaking changes are needed.

##############################################################
If you want to build the next Introspected-REST spec, you can follow the following reasoning.
Note that this reasoning is message-agnostic, meaning that we use here JSON just because we know it better
but your spec could use anything, yaml, xml etc.

* Start with some SANE defaults and the axion: The simpler your API (and the lesser it deviates from defaults), the simpler the introspection-meta-data should be
* Reach a concencus on a Introspection spec using already defined specs like JSON-Schemas.
* Reach a concencus on a querying language over url (filter + aggregation + pagination)
* Reach a concencus on an URL-API for attribute/association inclusion
* Reach a concencus on linking
* Reach a concencus on denoting linked/semantic data
* Reach a concencus on document structure (root element, meta attributes which should appear in the simple response as well etc)
Each of those could be a separate Media Type

So we keep 80% of the REST constraints and while we understand the benefits of other 20% we switch it with an on-demand alternative that makes the final thing
more flexible and powerful while keeping the final data simple.

There are 3 kinds of criticizers of REST model.
1. The ones who understand what REST is and feel that due to its complexity, they prefer loosing some features and deliver something
simpler, yet easier to implement and test and deliver a RESTful approach
2. The ones who understand what REST brings on the table but given that they control the client as well,
why should they bother with the whole HATEOAS thing?
3. The ones who don't understand REST and just want a plain JSON because it's simple enough


Introspected REST model is flexible enough to cover all those user cases.
It's not a model that is black or white: your API is either Introspected-REST-compliant or it isn't, like REST.



It should be noted this model is not something we conceived in a lab. Some [people]()
have already tried to implement something similar, probably without really knowing
what they were doing.

You see, the shadow of Roy Fielding is above any API developer:
we are afraid to deprecate Roy's REST model and as a result what we are doing is that
we take some elements of Roy's model, apply them, and name our API or spec as RESTSful.
Eventually however, the final result is even worse. It doesn't have Roy's key elements for
a Markov-chain-like client (we still have offline contracts) yet we have added complexity
to our API for little result.


Probably Roy won't like that. He will either:
* declare that Introspected REST is a stupid manifesto that has nothing to do with his REST or
* he will declare the Introspected REST is just yet another REST as he defined it and we never
read his dissertation to actually see that we are defining yet another REST style.

In either case, given that very few has really implemented a Roy-compliant REST API means
that Roy himself failed to explain his model correctly.



Are we sliding a lot from Roy's initial model? No, we modernize it a little bit.


##### Roadmap to json-specific defined Introspected REST specs

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



Good question.
Let's let Roy answer it.

We should describe them somehow though in our API without relying in offline contracts (like documentation)


#### 5.3. An alternative architectural style maybe?
Most of those Media Types specifications would not be needed if the APIs were built
with introspection in mind.

Imagine that we have a Media Type that allows us to describe new media types, called `generic_media_type`.
Then the clients would only need to understand and parse this `generic_media_type` and derive the other
Media Types.
Of course, this scenario is more difficult than it sounds and the goal of this _manifesto_ is not
to provide a generic Media Type.
Nevertheless, API introspection, as we will see, can provide us with information on API's
capabilities along with hypermedia in a much flexible and cleaner way, _without having data and hypermedia (representation metadata) tangled together in 
the representation_.



JSON-LD + Collection+JSON ? https://sookocheff.com/post/api/on-choosing-a-hypermedia-format/


WRITE ABOUT VERSIONING IN THE URL


RFC5988: A means of indicating the relationships between resources on the Web,
   as well as indicating the type of those relationships, has been
   available for some time in HTML [W3C.REC-html401-19991224], and more
   recently in Atom [RFC4287].  These mechanisms, although conceptually
   similar, are separately specified.  However, links between resources
   need not be format specific; it can be useful to have typed links
   that are independent of their serialisation, especially when a
   resource has representations in multiple formats.
Oh wait..we just figured out that having the links in there might not be the ideal.


Vale kai auta tou Roy pou exei sta presentations tou



Are you sure you want an architectural style and not an architecture ? (probably yes)

Is bold text a good idea?
Should we add link to everything?

The ratio of data/hypermedia of a resource

add querying specific elements of an introspection (like links, attributes etc)


who to ping: https://www.mnot.net/ Eric Wilde, Roy, wycats, Stevel Klabnik

Say that about custom capabilities like running functions or having an HTTP2 push. Say also that on metadata.


The types of metadata depend on the Media Type's capabilities.
Thus, the following could be considered as metadata but an Introspected REST API doesn't require all of them.
Actually an Introspected REST doesn't require anything.
It is up to the needs of the API designer to add the Introspected information.
There are 3 types of meta-data a resource could have:
  * For the expected request object:
    * Media Type's metadata, like pagination, querying language etc
    * structural information of the object, data types of the object's attributes and description for each resource and attribute, targeted to humans (SDT&D)
  * For the returned response object:
    * links, for linking other resources
    * actions, for manipulating the resource
    * structural information of the object, data types of the object's attributes and description for each resource and attribute, targeted to humans (SDT&D)
    *


Maybe explain somewhere the hypermedia/metadata definitions. Links vs actions?


In a perfect world, APIs are built to be alive for many decades and clients are exploiting every little feature of the API and its Media Type.
However, in a pragmatic where nothing is perfect in this world, clients are built by humans
We would like to embrace both architectural API styles: APIs that are built to last decades, and APIs that are built without
evolvement in mind.
Introspected REST model is flexible enough to cover all those use cases.

Resource, object, response definitions!!!


Secondly, we want to let clients to be able to retrieve plain data without dealing with metadata.
Last but not least, we would like to embrace the idea of composition when we are composing the final response.

We should note that these reasons apply to both, the API designed and the API client.

The client that is interested solely in the data, is releaved by getting just the data without any metadata mixed in.
In the future, if the client needs to get any metadata, it knows where to find them.
Moreover, this architecture, _should_ make things faster: not only it should allow the API designers to move faster in their development cycle, but
faster in the sense of request/response cycle. Parsing a bunch of metadata (but also rendering those from the server)
slows things down. In the case when these data are not needed then some bandwidth is wasted, the UI lags a bit, user waits slightly more, just in case
a client might need the metadata.

Maintaining a response that mixes up data with metadata is hard and makes things move slow.
Moreover, the API designer might not be interested to add some metadata (like hypermedia actions) from the very beginning of
the API release. Furthermore, this solution allows current APIs to inherit hypermedia, on the side, without making any breaking change,
specifically in the level they need and want.
We are pragmatic with the fact that not all APIs developed with 50 years ahead in mind and some APIs might be needed for relatively very
small amount of time.
We want to embrace these APIs too, with Introspected REST model, by allowing them to integrate any metadata types in the level they need.

When a client requests a resource, given that it already knows how to make the request and how to parse the response,
it should only await plain data.
Thus we need to find a way to provide any secondary data, like meta-data, through another channel, on the side.


The problem is that REST doesn't allow you to integrate it at a later stage.
Say that you already have a simple RESTless API, as a part of a prorotype product in your shining startup, and now you want to make a public
release.
You understand the benefits of having a REST API, so you are looking to add any hypermedia needed for the clients.
It turns out that, you will have to change a lot of stuff in order to integrate. In fact, we probably can't call it integration,
it's a completely new re-design.
If we want to be precise, in a RESTless API, adding hypermedia at a later stage would mean that we would need a new Media Type because
otherwise it would break the current semantics.
Moreover, the response would change every time

#### 8.2.3. REST does not make it easy to integrate different APIs together
Consider a product resource.
The product resource contains information about the product itself, as well as related information and services such as similar products.
However, information about payment options is not provided by the product resource itself, and is provided via a linkset instead.
This means any client requesting payment options will request payment links from a separate server.
This not only decouples product resources from payment management; it also allows the payment options to be more easily customized based on the specifics of the client, meaning that the complete "product and payment options" view is a combination of the product resource (and associated hypermedia controls), and payment links provided by the specialized payment options service.

Twitter!
Building high performant web services is hard, but to justify the deprecation of Roy's REST model is multiple times harder.
Obviously the reason is that he had done pretty good damn job (almost perfect if you take into account when it was published).

caching headers for data and metadata!

explain that when we talk about Media Type, we don't neceserily mean HTTP's media type!
Also say that hypermedia ~= HATOEAS

say about atom+json example!! A totally new Media Type that we need to manually understand..
explain that when we talk about Media Type, we don't neceserily mean HTTP's media type!


### 9.1. Collections, Resources, endpoints, actions and methods
There is a small confusion of different terms used around API literature.
We would like to give a small description and definition of each one.

#### Collection
A collection is a set, an array, of resources.
The resources should be of the same type but rarely a collection could also contain
resources of different types, or polymorphic resources.

#### Resource
A resource is an entity that exposes a set of different actions that can be performed on it.
Usually actions are a (sub)set of CRUD but it can go way beyond that and depends on the API designer decisions.

#### 9.1.1. Methods
In the RPC world, a method is a remote method called by another remote object.
In HTTP and related protocols, when we talk about a method, we mean the actual protocol method used, for instance `REGISTER` in SIP,
or `DELETE` in HTTP.

Details about the method as well as the available methods are described by the protocol itself.

#### 9.1.2. Actions and endpoints
Actions or endpoints mean the same thing: it's the combination of a specific protocol method in a specific url.
For instance, `GET /api/users` is an action, or, an endpoint.

The difference between action or endpoint, is that an endpoint could also be outside

##### Runtime Metadata
Runtime metadata are depending on the response and object which resulted by the incoming request.

Inappropriately, object-specific, dynamic metadata are also considered part of the object's data, like pagination information.

##### Structural Metadata
Structural metadata are related to the structure of the data, either the response object or the request object.

##### Hypermedia Metadata


## Informational metadata
runtime metadata
human-targeted metadata
other metadata like data types?


##### 9.1.3.1 Request's metadata
These metadata could be static for a resource, an endpoint or dynamic and volatile,
determined by the parameters of the request and the state of the resource at that given time.

##### 9.1.3.2 Media Type fill-ins
Metadata are data that describe the data or the hypermedia.
These metadata could be static for a resource, an endpoint or dynamic and volatile,
determined by the parameters of the request and the state of the resource at that given time.


#### Types of metadata
In REST, when requesting a resource you get different kind of information mixed in with the actual data.
Nowadays, API designers also distribute documentation to descrine some information about the API and its resources as well.

If we try to identify all the different kind of data and metadata we would get the following list for a resource:
###### 1. Media Type's capabilities fill-ins
These metadata are related to the Media Type used by the API.
The client provides information for the server to take into account when processing the request.
For instance, pagination information (page, items per page, offset) or the desired query (ideally an equievelent to SQL's SELECT).

###### 2. Resource schema
When requesting a response, the client should be able to know what to expect, for instance:
* the structure schema of the object, like the name of the attributes etc
* the data types of the object's attributes
* semantic meaning of each attribute, or of the resource itself
* possible a description for each resource and attribute, targeted to humans

Those information should be available for both response and request objects.


the name of the attributes,
their data types, any semantic meaning of each attribute (using a linked data spec) and possibly any description of the resource itself or its attributes
###### 3. hypermedia metadata
  * links, for linking other resources
  * actions, for manipulating the resource


As we described earlier, the level of Media Type's metadata depends on how strong or weak a Media Type is. If it's strong then the client will
