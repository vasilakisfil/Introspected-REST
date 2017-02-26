# Introspected APIs
(Sliding away from Roy Fielding's REST model)

> Imagine how poor the Web would have been if we had limited HTML to what was
> needed by an FTP client. That's what most JSON APIs are today.

## Introduction
There has been great confusion of what a REST API is.

> a RESTful API is just a website for users with a limited vocabulary (machine to machine interaction)


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

Great! let's see the API specs proposed as today 2017..

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


## My good old API


## Introspected APIs

### Networked APIs ιδιωματισμοι
#### Protocol level
HTTP

#### Message level
JSON

#### Application level


### 


