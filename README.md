# Introspected APIs
(Sliding away from Roy Fielding's REST model)

There has been a great confusion of what a REST API is.
Most people think that REST API is just a CRUD over HTTP. Or a CRUD with some links.

In this manifesto, we will give a specific definition of what REST is, according to Roy,
and we will propose a new model that brings into the table the same things,
yet it's much simpler to implement while at the same time being backwards compatible with any current (sane) API.

## Definitions
First some definitions, that we will use through the text:

* REST: The model that Roy defined in his [thesis](thesis) (along with his [blog]() comments).
* RESTfull: APIs that follows some parts of Roy's REST model, mostly links
* RESTy: APIs that have a plain JSON API (no links)
* Introspected REST: APIs that follow the definition we provide here


## Introduction
We believe that REST defined by Roy was an extraordinary, a magnificent piece of work, much ahead of its time
which took us 10+ years to understand what its capabilities are.
However, now, almost 20 years later this REST model shows its age. It's unflexibile,
difficult to implement, difficult to test, with performance and implementation issues.
But most importantly, any implementation of REST model is _very_ complex.

Now, one could say that, APIs are not build with mind to last for decades and maybe
that's the reason that this model hasn't seen much adoption.
The former is true but even if the later is true could it mean that this model is
not suitable for short-term APIs?

We firmly believe that REST is much better than any API that does not follow REST principles
(even RESTfull APIs), even for short-term APIs.
Networked services have very peculiar characteristics which, until now, only REST has fully addressed them*.

Given that, how can we have a simpler model than REST, yet have the same functionality of
REST?


As we will show, Introspected REST is an API architectural style that solves that.
It's a similar model which is mostly based on
Roy's REST model, brings the same advantages,
but differentiating on key elements to make it simple, easier to test and easier to implement.


**this section should go below explaining HATEOAS**

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


You see, the shadow fo Roy Fielding is above any API developer:
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

We need to be brave enough and move on.

Introspected REST is an alternative backwards compatible API. No breaking changes are needed.



## Roy's REST model

> a RESTful API is just a website for users with a limited vocabulary (machine to machine interaction)

Roy has done great work on initial HTTP spec and REST definition.
Unfortunately, very few people fully people have truly understood the unique characteristics of networked
APIs which inspired Roy to define the REST.

The idioms of Networked Services are very peculiar.
When we have a client talking over the wire to a server,
neither the client developer nor the server developer has access to the other machine.

This means that if the client needs a specific resource, it must not have an offline contract
on how to retrieve this resource because that would mean changes on the server's side is difficult.
As a result the server would help (drive if you will) the client exactly where is needed to.

That's the main reason why Roy is against the version on the URL: because it means that
you take as de-facto that there will be breaking changes at some point.
Anti8eta, a truly REST API should be able to apply changes on the resources without breaking any client
because the client.


Roy cites the following attributes on a REST API:

* All important resources are identifed by one resource identifer mechanism
  * induces simple, visible, reusable, stateless communication
* Access methods have the same semantics for all resources
  * induces visible, scalable, available through layered system, cacheable, and shared caches
* Resources are manipulated through the exchange of representations
  * induces simple, visible, reusable, cacheable, and evolvable (information hiding)
* Representations are exchanged via self-descriptive messages
  * induces visible, scalable, available through layered system, cacheable, and shared caches
  * induces evolvable via extensible communication
* Hypertext as the engine of application state (HATEOAS)
  * induces simple, visible, reusable, and cacheable through data-oriented integration
  * induces evolvable (loose coupling) via late binding of application transitions


Great! let's see the API specs proposed as today, March 2017..

## API Specs Today

### JSONAPI

### HAL

### SIREN

### Hydra

### GraphQL

#### Limitations

## Ideal REST API
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


## I miss my good old API


## Introspected APIs
In the following we will describe the architecture of the Introspected APIs through
a proposed implementation.
The reader though should not confuse the proposed implementation details with the actual
architecture.

* The simpler the API, the simpler the API description.

### Introduction 
#### Networked APIs ιδιωματισμοι
##### Protocol level
HTTP

##### Message level
JSON

##### Application level
The API spec



### Separate Hypermedia from the actual data (API introspection)
JSON Hyper Schemas + HTTP OPTIONS on the endpoint

### Automate documentation


> Imagine how poor the Web would have been if we had limited HTML to what was
> needed by an FTP client. That's what most JSON APIs are today.



### Linked Data and Semantic Web

It should be noted this model is not something we conceived in a lab. Some [people]()
have already tried to implement something similar, probably without really knowing
what they were doing.
