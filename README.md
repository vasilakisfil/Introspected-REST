# Introspected REST: An alternative to REST and GraphQL
(Sliding away from Roy's `REST` model)

There has been a great confusion of what a `REST` API is.
Most people think that `REST` API is just a CRUD over HTTP.
Or a CRUD with some links.
Or a nicely formatted, a sophisticated CRUD.

In this _manifesto_, we will give a specific definition of what `REST` is, according to Roy,
and see **the majority of APIs and API specs** ([JSONAPI](https://jsonapi.org/format), [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08) etc) **fail to follow this model**.
Then, we will propose a **new model** that brings into the table the same things,
yet it's much simpler to implement while at the same time being backwards compatible with any current (sane) API.

As part of this _manifesto_ we will also present the concept of MicroTypes, small reusable modules that compose a Media Type and facilitates
the evolvability and extensability of our new model.


* [1. Definitions](#1-definitions)
* [2. Introduction](#2-introduction)
* [3. Networked Services and APIs](#3-networked-services-and-apis)
  + [3.1. Application level](#31-application-level)
  + [3.2. Message level](#32-message-level)
  + [3.3. Protocol level](#33-protocol-level)
  + [3.4. Network level](#34-network-level)
* [4. Roy's `REST` model](#4-roys-rest-model)
  + [4.1. Access methods have the same semantics for all resources](#41-access-methods-have-the-same-semantics-for-all-resources)
  + [4.2. All important resources are identifed by one resource identifer mechanism](#42-all-important-resources-are-identifed-by-one-resource-identifer-mechanism)
  + [4.3. Resources are manipulated through the exchange of representations](#43-resources-are-manipulated-through-the-exchange-of-representations)
  + [4.4. Representations are exchanged via self-descriptive messages](#44-representations-are-exchanged-via-self-descriptive-messages)
  + [4.5. Hypertext as the engine of application state (HATEOAS)](#45-hypertext-as-the-engine-of-application-state-hateoas)
* [5. API Clients and Applications](#5-api-clients-and-applications)
  + [5.1. Client and Application responibilities](#51-client-and-application-responibilities)
  + [5.2. The Human factor principle](#52-the-human-factor-principle)
* [6. REST applied in a modern API](#6-rest-applied-in-a-modern-api)
  + [6.1. Requirements from a modern REST API](#61-requirements-from-a-modern-rest-api)
    - [6.1.1. Sparse fields (collection/resource)](#611-sparse-fields-collectionresource)
    - [6.1.2. Associations on demand (collection/resource)](#612-associations-on-demand-collectionresource)
    - [6.1.3. Sorting & pagination (collection only)](#613-sorting--pagination-collection-only)
    - [6.1.4. Filtering collections (collection only)](#614-filtering-collections-collection-only)
    - [6.1.5. Aggregation queries (collection only)](#615-aggregation-queries-collection-only)
    - [6.1.6. Data types !](#616-data-types-)
    - [6.1.7 Plot twist: this list is endless](#617-plot-twist-this-list-is-endless)
  + [6.2. Media Types vs HATEOAS](#62-media-types-vs-hateoas)
    - [6.2.1. Defining a new Media Type is not easy and should be avoided](#621-defining-a-new-media-type-is-not-easy-and-should-be-avoided)
    - [6.2.2. HATEOAS can get pretty heavy](#622-hateoas-can-get-pretty-heavy)
    - [6.2.3. Balancing between Media Types and HATEOAS](#623-balancing-between-media-types-and-hateoas)
    - [6.2.4. An alternative architecture](#624-an-alternative-architecture)
* [7. API Specs Today](#7-api-specs-today)
  + [7.1. Our use case](#71-our-use-case)
    - [7.1.1. User resource](#711-user-resource)
    - [7.1.2. Users resource (a collection of User resources)](#712-users-resource-a-collection-of-user-resources)
  + [7.2. JSONAPI](#72-jsonapihttpsjsonapiorg)
    - [7.2.1. User resource](#721-user-resource)
    - [7.2.2. Users resource (a collection of User resources)](#722-users-resource-a-collection-of-user-resources)
    - [7.2.3. Reflections](#723-reflections)
  + [7.3. HAL](#73-halhttpstoolsietforghtmldraft-kelly-json-hal-08)
    - [7.3.1. User resource](#731-user-resource)
    - [7.3.2. Users resource (a collection of User resources)](#732-users-resource-a-collection-of-user-resources)
    - [7.3.3. Reflections](#733-reflections)
  + [7.4. Siren](#74-sirenhttpsgithubcomkevinswibersiren)
    - [7.4.1. User resource](#741-user-resource)
    - [7.4.2. Users resource (a collection of User resources)](#742-users-resource-a-collection-of-user-resources)
    - [7.4.3. Reflections](#743-reflections)
* [8. Ideal `REST` API](#8-ideal-rest-api)
  + [8.1. Capabilities of an Ideal `REST` API](#81-capabilities-of-an-ideal-rest-api)
    - [8.1.1. Today's REST is far from ideal](#811-todays-rest-is-far-from-ideal)
    - [8.1.2. Making an API REST-compliant by downplaying its capabilities](#812-making-an-api-rest-compliant-by-downplaying-its-capabilities)
    - [8.1.3. A JSON API back in time](#813-a-json-api-back-in-time)
  + [8.2. Deriving the need for a new model](#82-deriving-the-need-for-a-new-model)
    - [8.2.1. REST is complex](#821-rest-is-complex)
    - [8.2.2. REST enforces possibly useless information](#822-rest-enforces-possibly-useless-information)
    - [8.2.3. REST sacrifices performance for evolvability](#823-rest-sacrifices-performance-for-evolvability)
    - [8.2.4. REST does not support caching of hypermedia](#824-rest-does-not-support-caching-of-hypermedia)
    - [8.2.5. REST doesn't make it easy to evolve hypermedia](#825-rest-doesnt-make-it-easy-to-evolve-hypermedia)
    - [8.2.6. REST's power is limited by HTTP and related protocols (SIP, CoAP etc)](#826-rests-power-is-limited-by-http-and-related-protocols-sip-coap-etc)
      * [8.2.6.1. No backwards compatible with any RESTly or RESTless API](#8261-no-backwards-compatible-with-any-restly-or-restless-api)
      * [8.2.7.2 REST does not embrace composition](#8272-rest-does-not-embrace-composition)
* [9. Introspected REST](#9-introspected-rest)
  + [9.1. Data, metadata and hypermedia](#91-data-metadata-and-hypermedia)
    - [9.1.1. Data](#911-data)
    - [9.1.2. Hypermedia](#912-hypermedia)
      * [9.1.2.1. Links](#9121-links)
      * [9.1.2.2. Actions](#9122-actions)
      * [9.1.2.3. Forms](#9123-forms)
    - [9.1.3. Metadata](#913-metadata)
  + [9.2. **MicroTypes: modules composing a Media Type**](#92-microtypes-modules-composing-a-media-type)
    - [9.2.1. **MicroTypes in HTTP**](#921-microtypes-in-http)
  + [9.3. Introspection as the engine of application state (IATEOAS)](#93-introspection-as-the-engine-of-application-state-iateoas)
    - [9.3.1. Composability over monoliths](#931-composability-over-monoliths)
    - [9.3.2. Plain data separated from metadata](#932-plain-data-separated-from-metadata)
    - [9.3.3. Identifiable metadata of each Microtype](#933-identifiable-metadata-of-each-microtype)
    - [9.3.4. Discovery of API resources and capabilities](#934-discovery-of-api-resources-and-capabilities)
    - [9.3.5. Automatic documentation generation](#935-automatic-documentation-generation)
* [10. An Introspected REST API prototype in the world of HTTP and JSON](#10-an-introspected-rest-api-prototype-in-the-world-of-http-and-json)
  + [10.1. Isolating the actual data from metadata](#101-isolating-the-actual-data-from-metadata)
  + [10.2. Composing different metadata MicroTypes together](#102-composing-different-metadata-microtypes-together)
    - [10.2.1. Structural metadata](#1021-structural-metadata)
      * [10.2.1.1. User resource](#10211-user-resource)
      * [10.2.1.2. Users resource](#10212-users-resource)
      * [10.2.1.3. Request Response inconsistency](#10213-request-response-inconsistency)
    - [10.2.2. Hypermedia metadata](#1022-hypermedia-metadata)
      * [10.2.2.1. User resource](#10221-user-resource)
      * [10.2.2.2. Users resource](#10222-users-resource)
    - [10.2.4. Descriptions metadata](#1024-descriptions-metadata)
    - [10.2.5. The case of a non-compatible spec for introspection: Linked Data metadata using JSON-LD](#1025-the-case-of-a-non-compatible-spec-for-introspection-linked-data-metadata-using-json-ld)
      * [10.2.5.1. Extending spec by creating a Shim MicroType](#10251-extending-spec-by-creating-a-shim-microtype)
        + [10.2.3.1. User resource](#10231-user-resource)
        + [10.2.3.2. Users resource](#10232-users-resource)
      * [10.2.5.2. Considering it as runtime metadata](#10252-considering-it-as-runtime-metadata)
  + [10.3. Method of introspection](#103-method-of-introspection)
    - [10.3.1. API capabilities discovery](#1031-api-capabilities-discovery)
  + [10.4. The Errors MicroType](#104-the-errors-microtype)
  + [10.5. **Signaling and negotiating MicroTypes**](#105-signaling-and-negotiating-microtypes)
  + [10.6. Automating the documentation generation](#106-automating-the-documentation-generation)
* [11. **Related Work**](#11-related-work)
  + [11.1. GraphQL](#111-graphql)
  + [11.2. Linked Data and Semantic Web](#112-linked-data-and-semantic-web)
    - [11.2.1. JSON-LD and HYDRA](#1121-json-ld-and-hydra)
  + [11.3. Web Linking and link relation types](#113-web-linkinghttpstoolsietforghtmlrfc5988-and-link-relation-types)
    - [11.3.1. The Profile Media Type Parameter (expired draft)](#1131-the-profile-media-type-parameter-expired-draft)
    - [11.3.2. The 'profile' Link Relation Type](#1132-the-profile-link-relation-type)
    - [11.3.3. Linksets](#1133-linksets)
  + [JSON home](#json-home)
  + [HTTP Hints](#http-hints)
  + [11.4. RESTful API Description Languages](#114-restful-api-description-languages)
  + [11.5. API directories](#115-api-directories)
* [12. **Conclusion**](#conclusion)

## 1. Definitions
First some definitions, that we will use through the text:

* `REST`, `RESTful`: The model that Roy defined in his [thesis](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) (along with his blog post [REST APIs must be hypertext-driven](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)).
* `RESTly`: APIs that follows all parts `REST` model, except HATEOAS in which they support mostly links (specs like [JSONAPI](https://jsonapi.org/format), [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08) etc)
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
**Being able to evolve your API without breaking the clients is critical.**

Imagine the following scenario: you have built an Online Social Network and an iOS app that talks to the API on your backend.
Now imagine that, after a company meeting, your CEO needs you to make tiny yet important change in the signup page: require the user
to fill in her age, a field in the signup form you didn't have before.
Essentially, this means, in API terms, add an extra field in the accepted object and require it from the client to be filled in
by the user before sending it over.

If your API is `RESTly` and not `REST`, this means that you need to fix the code in the iOS side, test it and send a new iOS app to Apple store.
It takes roughly 1 week for Apple to review your app and if your app doesn't get rejection during the review process for some reason, your
tiny change will take action at least a week later after requested.
If your API _was_ `REST` that would mean a change on the server's response denoting which fields are required to submit the form.
You would have the change deployed 10 minutes later.

Roy notes in his thesis:

>  A system intending to be as long-lived as the Web must be prepared for change
>
> --- Roy Fielding
>

In the world of startups and money-driven companies, the following will sound more appealing:

>  If you want to move fast, you should build a change-first API.
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
Instead, the client derives the state on demand, using introspection, by retrieving the necessary metadata
that are of interest.
Eventually this brings the same advantages as Roy's model while being it's much simpler,
much more flexible and backwards compatible with any RESTful API.

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

Media Types can be a bit more complex as well: `application/vnd.api+json`, the media type of [JSONAPI](https://jsonapi.org/format) spec, (roughly) means that
* the main type is `application`
* the subtype is `vnd.api` which _roughly_ denotes the Media Type name
* the underlying structure follows JSON semantics

In theory, [JSONAPI](https://jsonapi.org/format) spec spemantics could also be applied using XML as the data format (like in the case of [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)),
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
Roy came up with the REST model in order to solve issues that were arising by the unique propertied of networked services
during the infancy of Internet.
When you develop an application that will be deployed in a networked environment and is expected to be accessed by other networked services,
you need to think about its evolvability:
if you need to add, remove or change functionality of that application
**you cannot expect services on the other end that talk with your application to be updated by humans**.
Such problems that arise from the peculiarities of networks, like discovery and evolvability must be solved using
machine-to-machine communication.

`REST` model that solves such issues is an arhictectural style which is not tight to any spec, protocol or format of the
aforementioned levels.

> a RESTful API is just a website for users with a limited vocabulary (machine to machine interaction)
>
> --- Roy Fielding

When Roy talks about `REST`, he mentions 5 crucial properties of `REST` model:

### 4.1. Access methods have the same semantics for all resources
> induces visible, scalable, available through layered system, cacheable, and shared caches

Failure to provide consistency on access would imply that you don't provide a generic interface but instead
you have resource-specific or even object-specific interfaces.

Actually a common interface is one of the most crucial parts of REST: without a common uniform interface
it would be impossible to derive REST.

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
your implementation internally or how the data are stored in your database.
It could also be the same.
Nevertheless the client expects and is expected to manipulate any resource using the representation
you expose.

### 4.4. Representations are exchanged via self-descriptive messages
> induces visible, scalable, available through layered system, cacheable, and shared caches
> induces evolvable via extensible communication


This would mean that the data of the response should follow the Media Type that the client
requested and unrestands.
Given that the client negotiated for that Media Type, **it should be able to parse and understand any part of the response**.

> Interaction is stateless between requests, standard methods and Media Types are used to
> indicate semantics and exchange information, and responses explicitly indicate cacheability.
>
> --- Roy Fielding
>

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
to manipulate the resource and navigate to other resources, using semantics defined by
the Media Type agreed.

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

### 5.2. The Human factor principle
There are 2 types of human involvement when building an API client:
* **1-fold**: Programming the client only once to understand the Media Type correctly and let the
client work for any API that follows that Media Type even when APIs evolve, given that it adhere in the Media Type specs.
The only thing that the client requires is the initial URI of the API.
* **multi-fold**: Programming the client once to understand the Media Type.
Then modify the client to parse and understand the API correctly using some offline contract (i.e. documentation for available resources, fields, pagination etc) and then
every time the API evolves (like adding a resource or a field), reprogram the client accordingly. The extend of human involvement
during that phase depends on the weakness of the Media Type.

An API that follows the `REST` model should be evolvable without the need
of human involvement in the client side, given that the client understands the Media Type.
A side effect of such evolvability and 1-fold clients is that the **versioning should not take place in the URL but in the Media Type itself**.

>Versioning an interface is just a polite way to kill deployed applications
>
>The reason to make a real REST API is to get evolvability … a "v1" is a middle finger to your API customers, indicating RPC/HTTP (not REST)
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
clients are expecting more options (capabilities) from the server for their responses.
It's not enough to just request and get the resource but you should be able to specify
to the server what transformations shoult apply, according to your needs.

Nowadays we have been using networked APIs so much that now we essentially have to
provide an ORM to the client over the HTTP (or any other protocol).

We provide here a list of features (we call them capabilities) that we think should be built in a modern networked API,
in 2017.

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

#### 6.1.7 Plot twist: this list is endless
Although we feel that _today_ these capabilities should exist in any modern API, **this list is not exclusive**.
In fact, there could be capabilities in the future that might not seem necessary today.
For example, joining together one or more resources, other db-inspired operations applied on resources,
internationalization and localization of the data, HTTP/2 Server Push on some requests, Generic Event Delivery Using HTTP Push on other
resources on specific states and other capabilities that we haven't even imagined yet.
In any case, **these capabilities must be transparent and self-descriptive to the client without any documentation or human involvement**, other
than programming the client to support the Media Type(s) and pointing it to the initial API URI.


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
then it's easy to "configure" it for another API which also follows the same Media Type.

Essentially, HATEOAS should **complement** the Media Type by providing the Media Type's hypermedia-ed defined semantics in **runtime**
for the client to work properly.
For instance, HATEOAS could describe on a per-resource basis if the pagination is supported, what is the maximum `per_page` etc.

#### 6.2.4. An alternative architecture
We feel that the current Media Type specification and use is dated.
If Software Engineering has learned us something, is that composition can enforce Single Responsibilty Principle, if used correctly.
Inspired by that, later, we will suggest a new concept,  MicroTypes, small reusable modules that combined together can form a Media Type.
As a result, clients should be able to even negotiate parts of the Media Type and not the Media Type as a whole.

Also, instead of mixing up data with HATEOAS in the API responses, we will introduce introspectiveness of our resources.


## 7. API Specs Today
Now that we defined what REST is, according to Roy, and what capabilities modern APIs should provide,
let's see the specs for REST(y) APIs available as today, April 2017, what they provide and how
closely follow the REST model.

### 7.1. Our use case
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

_`Users` and `User` are 2 distinct resources which are often, mistankingly, missthought as a single, one, resource.
Also, the fact that `Users` is a collection of `User` objects is because it suits our needs but it doesn't have necessarily
to be like that._

As we mentioned, `User` resource has also some associations (or relations/relationships if you prefer),
like `Microposts`.

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
in the specs for REST(y) APIs currently available. We feel that most currently deployed APIs
have a lot of similarities with the following specs, namely the structure and the HATEOAS part (regarding
linking), and as a result by comparing those specs with our model would be sufficient.

We will evaluate the specs for the following:
* whether they follow Roy's `REST` model
* whether their messages are **not** self descriptive, meaning that other than supporting the API's Media Type in our client
we also need to read and understand the documentation to develop our client
* whether they require multi-fold human factor while the API evolves

### 7.2. [JSONAPI](https://jsonapi.org)
JSONAPI was originally created by [Yehuda Katz](https://yehudakatz.com/), as part of Ember's ember-data library.
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
 * Limited links (no URI templates, treats the client as completely stupid)
 * No actions
 * No info on available attributes
 * No info on data types
 * No attributes description

To sum up, it doesn't entirely follow `REST` model while it requires both
documentation and multi-fold human factor.

### 7.3. [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)
HAL was created by Mike Kelly in 2012.
The key feature of HAL when it was released was the browsability/explorability of any API that adopted.
Another feature is the idea of curies, links inside the resource that lead to the documentation.
However, this feature is rather controversial since the information these links provide are targeted for humans and not machines.

The resources of our use case that are presented here use JSON as a message format, but HAL is not tight to that.

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
            "href":"https://example.com/docs/rels/{rel}",
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
 * No attributes description, requires documentation (however it does provide a link to documentation, through curies)

To sum up, it doesn't entirely follow REST while it requires documentation and multi-fold human factor (curies facilitate that).

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

To sum up, it doesn't entirely follow REST while it requires documentation and multi-fold human factor.

## 8. Ideal `REST` API
**How many years these specs could sustain in terms of evolvability ? Are they built with a lifespan of 2-3 years or are they
built with a life span of 50 years?**

### 8.1. Capabilities of an Ideal `REST` API
In an ideal REST API, the client should be able to have all the necessary information for both
the request and response.

* About each resource returned from the API to the client:
  * default attributes and available (superset of default) attributes of the resource, based on the user's permissions
  * data types for each attribute in the resource or any embedded association
  * Sorting/pagination, filtering and aggregation queries availability
  * data type of each attribute
  * default embedded associations and available associations to embed
    * recusrively apply the same information for each association available for embedding
  * any other capability (HTTP/2 Server Push, event delivery etc)
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
to the Media Type, we would still have a _very_ complex, massive, response from the server that mostly includes HATEOAS
and not actual data.

In our experience, such responses are very hard to implement correctly, test, be performant and even debug. After all,
a human will sit down and write the initial code and debugging the code by the eye is important.
Simplicity is crucial.

Moreover, some clients might not be interested in hypermedia and evolvability at all but only the data.
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
Introspected REST, an architecture that we will describe later, solves that by serving hypermedia
information on side and in an incremental way without breaking the simplicity.

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
As we described earlier, mixing data with hypermedia leads to increased complexity, for both the server and the client.
Just compare the size of our [resource's data](#711-user-resource) and the size of our resource when represented by
[Siren](#741-user-resource), a hypermedia-ed API that doesn't even being REST-compatible by missing numerous information as described
in its [reflections](#743-reflections).
Imagine how bloated the response would look like, if we added all the capabilities described in [section 6.1](#61-requirements-from-a-modern-rest-api).

Moreover, the hypermedia must be tailored for the user role the client acts on behalf of.
For instance, a user with very basic access role might only have access to retrieving resources and not manipulating them.
Or such role could only have access to specific capabilities of the API.
As a result, the hypermedia provided on the response object should reflect that by not providing hypermedia that will lead to unauthorized access.
In fact, such design is quite difficult to implement and test from the server side.

#### 8.2.2. REST enforces possibly useless information
In REST, even if the hypermedia are rendered by taking into account the user's role, eventually we might send more data that the client wants.
**Exactly because we don't know in advance what the client might need, we must send all the possible hypermedia information to the client, just in case**.
The client however could only be interested in the data, or specific hypermedia types, like only links, but instead gets a fully bloated response by the server.

#### 8.2.3. REST sacrifices performance for evolvability
Complex or long-lived APIs tend to have many hypermedia data (links, actions, custom metadata) related to the resource itself, its associations and related resources.
As a result, even if the actual data could be very small the resulted response object gets much larger in size slowing down the server rendering
and the client receiving and parsing.
The performance issues become more apparent on lossy networks like mobile clients, a trend that has increased over the past decade,
or on constrained devices and environments, like IoT.

#### 8.2.4. REST does not support caching of hypermedia
In practice, **the hypermedia part of a resource rarely changes**, compared to the resource's data.
In REST, by design, the client can't cache the hypermedia part of the resource, even for relatively small amount of time, because
hypermedia is part of the resource, thus caching the hypermedia can't be separate from caching the response itself.

#### 8.2.5. REST doesn't make it easy to evolve hypermedia
Another issue of REST is that due to the fact that everything is mixed in together, **evolving hypermedia separately from the data
can't happen**.
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

Although we firmly believe that a REST API is better than any RESTly or RESTless API, we understand that there could be cases where API designers
_have to_ initially skip hypermedia part.

The problem is that when REST is applied to HTTP, it doesn't allow you to easily integrate hypermedia at a later point.
The reason is that, in a RESTless API, **adding hypermedia at a later stage would mean that we would need a new Media Type because
otherwise it would break the current semantics**.

We would like to see a model that embraces both architectural API styles:
* APIs that are built to last decades and thus, support full hypermedia from the very first day of their release
* APIs that don't have hypermedia (the reason is none of our business), yet it is required to add hypermedia, later, in an incremental way without
breaking existing clients or limiting API's flexibility

##### 8.2.7.2 REST does not embrace composition
Although REST does not rejects the idea of composability of different API capabilities using different specs in the same response, or composite Media Types,
it doesn't embrace it either.
The symptom of non-composability is clearly visible in protocols like HTTP where Media Types
act as big monoliths trying to describe everything in one place.
[RFC 6906 (The 'profile' Link Relation Type)](https://www.ietf.org/rfc/rfc6906.txt) was created to overcome such issues
but as we will see later this specifications lags behind over true composability and
proper negotiation of the different profile types from the client perspective.

In Introspected REST, the MicroTypes is a conceptual solution to the outdated Media Type concept
and allows us to mix-in different concepts for different kind of metadata of a resource,
yet have all of them on demand and separated by the actual data.


## 9. Introspected REST
>  Simple things should be simple and complex things should be possible.
>
> --- Alan Kay
>
>

In the following section we will describe our new architectural style based on a model for Networked APIs that goes beyond `REST`.
The model itself steps on initial Roy's `REST` model but with the difference that instead of providing resource hypermedia at
runtime, **it provides them on the side, only if requested**.
Hence, by keeping the _uniform interface_ the derived 3 out of 4 REST constraints that Roy defined still exist in this model:
_identification of resources_; _manipulation of resources through representations_ and _self-descriptive messages_.
However instead of having the constraint of _hypermedia as the engine of application state_ (HATEOAS), we have
**_introspection as the engine of application state_ (IATEOAS)**.

Composition of different specs is a vital part of our model and for that we will use a new concept,
MicroTypes, small reusable modules that a final Media Type can be composed of.
Before moving on, we will give concise definitions over hypermedia and metadata and break it down to different kinds of classes,
according to Introspected REST model.
These terms can overlap in the REST, however we feel that each one has its own place in our model that embraces composability.


### 9.1. Data, metadata and hypermedia
#### 9.1.1. Data
Data is the main variables of a resource, at a given state, at a given time.
Data is very volatile compared to other parts of a response.

#### 9.1.2. Hypermedia
Originally the hypermedia term was mostly used for linked data, in the sense of hyperlinks.
In `REST`, eventually, it also includes information for interaction and resource manipilation.
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
Although CRUD are the most popular actions of a resource, the beauty with REST, and consequently with Introspected REST,
is that actions can go beyond plain CRUD.
In fact, you can define any type of action or meta-action of your internal resource, through the representation that you expose.
As a result, actions of a resource could be quite complex or simplistic depending on the needs and decisions of the API designer.
Actions should also describe any relevant information for the client to perform it, unless the Media Type itself describe those details.

##### 9.1.2.3. Forms
Another way of describing the manipulation options of a resource is the notion of forms.
The difference between actions and forms is that the latter are strictly semantically **equivelent to an HTML form**,
for the client to render.


#### 9.1.3. Metadata
If hypermedia is links and actions, then what is metadata ?

Metadata are information about the resource that is not related with the data, including hypermedia.
In essence, metadata is a superset of hypermedia and it's crucial for the client
to understand API's responses and access the API's capabilities and manipulate the resources.

Metadata could be **API-specific, resource-specific, action-specific or even object-specific**.
There could also be different kinds of metadata: runtime (i.e. pagination information), structural (i.e. data types of a resource object),
hypermedia (i.e. links, actions, forms), informational targeted to humans (i.e. general information, descriptions), etc.

Usually metadata is much less volatile than data, if not static, except runtime metadata that depend on the request and the resource at the given time and state respectively.


### 9.2. MicroTypes: modules composing a Media Type
> Imagine how poor the Web would have been if we had limited HTML to what was
> needed by an FTP client. That's what most JSON APIs are today.
>
> --- Roy Fielding
>

We have been talking so much about the concept of MicroTypes but what exatly are ?

Currently, Media Types act as big monoliths that clients need to understand beforehand through human involvement.
We believe that Media Types should be broken in smaller
reusable media types, MicroTypes, each describing very carefully a specific functionality of a modern API.
The reasoning is that, in our experience, we have seen that different APIs and API specs define the same functionalities in similar,
but not identical, ways.

Examples of MicroTyes could be semantics for:
* pagination
* querying over url (applying filters, aggregations, pagination/sorting on a resource),
* resource/assotiation inclusion in the same response
* semantic/linked data
* hypermedia actions (required fields, available fields),
* data types and resource scehmas
* error information
* and many more, like HTTP/2 server push for specific resources/states etc

Each one of these could be defined as separate MicroTypes that specify in isolation how that part of the API works.
At the same time they should be generic enough or follow some specific semantics so that it's possible to be referenced parent
Media Types targetd for Introspected APIs.
The parent Media Type doesn't need to know in advance all the MicroTypes that the API designed intends to use
because that would mean that adding new MicroTypes would require a new parent Media Type which consequently means breaking the clients.
Instead, each MicroType should be attachable to a parent Media Type that defines such behaviour.

#### 9.2.1. Benefits of MicroTypes
The benefits when leveraging such architecture are multi-fold.

##### 9.2.1.1. Granular parameterization of API functionality by clients 
First, by allowing the client and server to do the regular negotiation flow even for those sub-media-types, the communication
between the 2 ends is parametrized to the needs of the client, down to the semantics level.
For instance, a server might provide 3 MicroTypes for error information, each one having different representation or semantics.
By letting the server to pick the appropriate MicroType for the client by analyzing the client's incoming request,
might not be efficient as the client can only send a part of its properties through the request, for various reasons like privacy concerns and performance,
and thus the server has partial knowledge of the client's state and properties.
The server has to make an arbiratry choice for the client, what it thinks it's thinks best, using this partial knowledge.

Instead, by giving the client the option to negotiate parts of the API functionality, we shift the responsibility towards the client
to select the best representation and semantics of various, isolated, API functionalities.
Given that the client can know much more about its needs than the server, it will make the best available choice
for each API functionality, from the server's options, which eventually will lead to the optimized combination of
MicroTypes.
As we will see later, this is called reactive negotiation, a forgotten but still valid negotiation mechanism in HTTP protocol.

##### 9.2.1.2. Reusability 
Secondly, the MicroTypes specs and possibly implementations can be re-used by both the servers and clients.
Instead of defining a whole Media Type, API designers will be able to include various small modules
that extend the API functionality they way it's needed.



#### 9.2.2. MicroType shims
According to [RFC 6831](https://tools.ietf.org/html/rfc6838) each Media Type's primary functionality shoud be that of being media formats.

>   Media types MUST function as actual media formats.  Registration of
>  things that are better thought of as a transfer encoding, as a
>  charset, or as a collection of separate entities of another type, is
>  not allowed.  For example, although applications exist to decode the
>  base64 transfer encoding [RFC2045], base64 cannot be registered as a
>  media type.
>
>  This requirement applies regardless of the registration tree
>  involved.
>
>  [RFC 6831](https://tools.ietf.org/html/rfc6838)
>

In our cocept of MicroTypes, the parent Media Type acts as the base media format.
The details however, are defined by small components that define functionalities of different parts of the API.

We still want to preserve the "functionality" requirement in the concept of MicroTypes, however such functionality
should be in the context of media formats as [RFC 6831](https://tools.ietf.org/html/rfc6838) indicates.
Imagine that we want to use an existing spec as a MicroType.
We cannot create a MicroType out of it with just a reference
to the original spec because it lacks the context of the underlying protocol (like HTTP) and Media Type with which it will be
used.
It also lacks information about the requirements of the parent Media Type and the compatability with other MicroTypes.
As a result, we need to extend the original spec with the necessary semantics, additional, semantics in the context
of Media Types.
Those semantics should be as minimal as possible, with respect to the initial specification and without altering its core semantics
but enough for usage in its new context.
When this method is followed, the new MicroType is called a "shim" of the original spec.


### 9.3. Introspection as the engine of application state (IATEOAS)
The idea of introspection is to be able to examine properties of a system at runtime.
In the case of Introspected REST, **introspection defines a process for a client to be able to introspect
the API's, resource's, action's or even object's metadata at runtime**.
Through those metadata, server provides all the available states, manipulation actions as well as the available transitions.
The implementation of the process is up to the API designer although usually a REST interface even for each MicroType's metadata is a wise choise.
In any case, we would like to point out some key properties that should appear on any introspection process.

#### 9.3.1. Composability over monoliths
The process should **embrace the use of distinct MicroTypes** to form a Media Type instead of using a single Media Type.
Such an architecture will lead to a system whose each MicroType's metadata is independent, self-contained and detached from the metadata
of the rest MicroTypes.

The API designer should **first** investigate and embrace the use of MicroTypes, RFCs and specs that are already defined and published, instead of
creating her own custom, unpublished spec.
The reason for this suggestion is that creating a new spec is difficult and usually such custom specs are used only for domain-specific APIs that
were created for and live as long as this API is used.
Instead, by trying to adapt published, battle-tested, RFC-community-reviewed specs assures the API designer in terms of **compatibility,
adoptability, clarity** and possibly implementation, for both ends of the communication.

#### 9.3.2. Plain data separated from metadata
The process of introspection **should be distinctly different** from requesting data.
To that extend, introspection responses should not include any data but only metadata and data
responses should not include any metadata, except possibly runtime metadata.

#### 9.3.3. Identifiable metadata of each Microtype
Given that metadata are already separated from plain data, by being able to identify and retrieve metadata
of a specific MicroType there are various advantages because each MicroType becomes independent and self-sufficient.

For instance, **caching** will be possible using the underlying protocol's mechanisms, for each metadata type separately.
Another example is the **detached evolvability** of each MicroType's metadata, independently, given that the MicroType's semantics permit that.

#### 9.3.4. Discovery of API resources and capabilities ++
Given that for each resource, the client needs to perform an introspection request, this becomes problematic in terms of **performance**.
An Introspected REST API _should_ provide an **API-wide capabilities discovery** that lists all the metadata from all MicroTypes for all resources,
and their states that the client can access, wherever this is possible.

The location of this detailed list should be in the conceptual _root_ resource/URL of the API.

#### 9.3.5. Automatic documentation generation
Possibly the API will provide a MicroType targeted to humans and not machines that contains informational descriptions and explanations.
It should be noted that **this information must not be needed for a client to parse and understand the API responses**,
and even for humans such information should weight very little compared to the rest metadata.

In the same way, the API should **automate the generation of the documentation using all metadata from all MicroTypes for every resource**.
The way the documentation is requested and its format should be distincly defined by a MicroType or the parent Media Type.

## 10. Introspected REST applied to HTTP
Introspected REST architectural style is not bound to any protocol or spec, just as is REST.
Here we will review the challenges that are rising through its adaptation in HTTP protocol.

But how can the client can negotiate with the server the MicroTypes to be used ? 
Given that reactive-based negotiation has never been used, to our knowledge, we will present a possible
implementations of that mechanism, with the least possible changes to the HTTP protocol.

We need to specify the following:
1. How the client informs the server its preferred Media Types along with the MicroTypes to be used with each Media Type
2. How the server informs the selected Media Type along with the MicroTypes
3. How the server informs the client the order of applicability when 2 MicroTypes define similar semantics, overlap or collide

+ indentification/introspectiveness of MicroTypes

### 10.1 Revisiting content negotiation in HTTP
As we have already seen, content negotiation in HTTP is achieved through `Accept` request header but it's not the
only header which can be used by the server to determine the appropriate representation for the client.
`Accept-Charset`, `Accept-Encoding`, `Accept-Language` request headers can also be used.
In practice, `User-Agent` header is also used by the server for choosing the right content for the client
because it contains some device and agent characteristics, although it's not part of the negotiation standard headers.
Lately even, a new draft stadard is created called [HTTP Client Hints](http://httpwg.org/http-extensions/client-hints.html)
that extends the HTTP with new request headers which indicate device and agent characteristics.

The server uses all those headers as hints in order to determine the most suitable representation of the content
to be served to the client.
This hint-based mechanism is called server-driven or proactive content negotiation and although it is used extensively
by the web now, it does have some drawbacks. As [RFC 7231](https://tools.ietf.org/html/rfc7231) notes:

>   Proactive negotiation has serious disadvantages:
>
>   o  It is impossible for the server to accurately determine what might
>      be "best" for any given user, since that would require complete
>      knowledge of both the capabilities of the user agent and the
>      intended use for the response (e.g., does the user want to view it
>      on screen or print it on paper?);
>
>   o  Having the user agent describe its capabilities in every request
>      can be both very inefficient (given that only a small percentage
>      of responses have multiple representations) and a potential risk
>      to the user's privacy;
>
>   o  It complicates the implementation of an origin server and the
>      algorithms for generating responses to a request; and,
>
>   o  It limits the reusability of responses for shared caching.
>
> --- [RFC 7231](https://tools.ietf.org/html/rfc7231)
>

In fact, from the beginnings of HTTP (since [RFC 2068](https://tools.ietf.org/html/rfc2068#section-12.2), published in 1997),
the protocol allowed another negotiation type: agent-driven or reactive content negotiation negotiation.
As [RFC 7231](https://tools.ietf.org/html/rfc7231) notes, reactive content negotiation the server provides a
list of options to the client to choose from.

>  With reactive negotiation (a.k.a., agent-driven negotiation),
>   selection of the best response representation (regardless of the
>   status code) is performed by the user agent after receiving an
>   initial response from the origin server that contains a list of
>   resources for alternative representations.
>
>   (...)
>
>   A server might choose not to send an initial representation, other
>   than the list of alternatives, and thereby indicate that reactive
>   negotiation by the user agent is preferred.  For example, the
>   alternatives listed in responses with the 300 (Multiple Choices) and
>   406 (Not Acceptable) status codes include information about the
>   available representations so that the user or user agent can react by
>   making a selection.
>
> --- [RFC 7231](https://tools.ietf.org/html/rfc7231)
>

With reactive negotiation, the client is responsible for choosing the most appropriate representation,
according to its needs.

### 10.2. Runtime MicroTypes
Runtime microtypes (like pagination or error description) will be negotiated using regular Accept/Content-Type header

Introspetive Microtypes, which are identifiable, will use Options/rel


#### 10.2.1. Signaling and negotiating MicroTypes
Note that delivering problem+json (a Media Type that was never negotiated) is a problem in REST API as well!
2 issues

https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
The Content-Type header is limited up to 128 characters so we might need another header for that.
Content-Type could describe the overall Media Type while Foo header could describe sub-media-types used to produce that Media Type.
The communaity will choose the headers and implementation.

+ the profile link relation
Surprisingly for each new link relation the 

### 10.3. Introspective MicroTypes

#### 10.3.1. Signaling and negotiating MicroTypes

#### 10.4 Method of introspection
After the client has selected its ideal combination of MicroTypes, the client should start
introspecting the metadata of those MicroTypes. Here, we will present two possible
implementations of introspection in the HTTP protocol.

+ Identification of MicroTypes

##### 10.4.1 The established OPTIONS method
The server can describe the meta-data of a resource in the response body of the `OPTIONS` request.
In fact, **OPTIONS method has historically been used for getting informtation on methods supported on a specific resource**.

Specifically, the [RFC 7231](https://tools.ietf.org/html/rfc7231), which is a part of the HTTP RFC series, mentions that this method should be used to determine the capabilities of the server, for that particular resource so
we feel HTTP OPTIONS is a perfect match for API introspection after reactive negotiation.

> The OPTIONS method requests information about the communication
> options available for the target resource, at either the origin
> server or an intervening intermediary.  This method allows a client
> to determine the options and/or requirements associated with a
> resource, or the capabilities of a server, without implying a
> resource action.
>
> --- [RFC 7231](https://tools.ietf.org/html/rfc7231)
>

As the RFC notes, the OPTIONS request should not imply any specific resource action and
as a result should return all the available capabilities for that resource, for all actions.

The same RFC mentions that there isn't any practical use of sending an OPTIONS request
to the root url.

> An OPTIONS request with an asterisk ("\*") as the request-target
> (Section 5.3 of [RFC7230]) applies to the server in general rather
> than to a specific resource.  Since a server's communication options
> typically depend on the resource, the "\*" request is only useful as a
> "ping" or "no-op" type of method; it does nothing beyond allowing the
> client to test the capabilities of the server.  For example, this can
> be used to test a proxy for HTTP/1.1 conformance (or lack thereof).
>
> --- [RFC 7231](https://tools.ietf.org/html/rfc7231)
>

We also feel that this is also a perfect case for hosting an API's discovery for available resources capabilities.
We could keep the `/*` for "ping" or "no-op" type of method as the RFC notes and have the root
`/` for listing all API's capabilities for all resources, as [IATEOAS notes](#934-discovery-of-api-resources-and-capabilities).

##### 10.4.2. Using new relation tyes using Web Linking's rel parameter

- overloading, say about linksets.


#### 10.5. Limitations and enhancements
We need to do an options request everytime, performance issue.
This can be mitigated if we emulate GraphQL/Apple and have the same Media Type for all metadata.
But it's not part of Introspected REST, by design but can be enhanced by parent Media Type.

## 11. An Introspected REST API prototype in the world of HTTP and JSON
In the following we will describe the architecture of the Introspected REST APIs through
a proposed implementation.
The reader though should not confuse the proposed implementation details with the actual
architecture style.

**This is by no means a complete Media Type**, but just an example of the potential of Introspected REST.
The actual MicroTypes and Media Types will be created by the community.

For our solution, we will use [JSON](https://tools.ietf.org/html/rfc7159),
[JSON Schemas](https://tools.ietf.org/html/draft-wright-json-schema-validation),
[JSON super schemas](https://tools.ietf.org/html/draft-wright-json-schema-hyperschema),
[JSON-LD](https://json-ld.org/spec/latest/json-ld)
and [`problem+json`](https://tools.ietf.org/html/rfc7807)
each representing a different MicroType.
But the reader could apply the same ideas using any message format and spec.

Our use case will be the same as the one in [section 7.1](#71-our-use-case), a minature of yet another Social App.

Given that Introspected REST differs only in HATEOAS part of REST, the identification of the resources _should_ be kept the same, namely:
* `Users` resource (`/users`):
  * List users (`GET /users`): Gets a collection of `User` resources
  * Create a new user (`/users`): Creates a new `User` with the specified attributes.

* `User` resource (`/users/{id}`):
  * Get a user (`GET /users/{id}`): Gets the attributes of the specified `User`
  * Update a user `PATCH /users/{id}`: Updates a `User` with the specified attributes
  * Delete a user `DELETE /users/{id}`: Updates a `User` with the specified attributes


Let's assume that our parent Media Type is `application/vnd.api+json`.

### 11.1. Isolating the actual data from metadata
Our top priority is to offload the final response object from the metadata, like hypermedia.
Instead, we will provide to the user only the data and possibly any runtime metadata.

When the client manipulates a `User` resource, the response should contain only the data:

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

Similarly, a `Users` resource will be a collection of `User` resources:

```json
{
  "users": [{
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count": 50
  }, {
    "id":"9124",
    "email": "robert.clarsson@gmail.com",
    "name": "Robert Clarsson",
    "birth_date": "1940-11-10",
    "created-at": "2016-10-06T16:01:24Z",
    "microposts-count": 17,
  }],
  "meta": {
    "page": 1,
    "per_page": 2,
    "offset": 0
  }
}
```
Given that the response should also contain pagination information,
we add this runtime metadata under a `meta` attribute.

The actual format of the data could vary regarding the root element or possibly the place of the primary id.
Such details will be described by the Media Type.
What is important here is that the **data does not contain any metadata**, apart from runtime metadata.

### 11.2. Composing different metadata MicroTypes together
We will describe our APIs capabilities by mixing together different MicroTypes targeted each one for a specific capability
of our API, following the Single Responsibility Principle.
The client will be able to retrieve the information of each metadata MicroType by introspecting the resource.

#### 11.2.1. Structural metadata
One of the most important things for a client to know is the expected structure of the request/response resource object
along with information on the data types.
For that we will use JSON Schemas, a powerful spec that enables you to describe and validate your JSON data.

Given that this specification has been published as an Internet-Draft and its popularity it is very probable that there _is_
an implementation for that MicroType for the client's environment.
Also, a cool side effect of having the structure definition of the resource as a MicroType available through resource's introspection,
is that the client can use this information to first validate the object before sending it over the wire to the server.

##### 11.2.1.1. User resource

```json
{
  "$schema":"https://json-schema.org/draft-04/schema#",
  "$id":"https://example.com/user.json",
  "properties":{
    "user":{
      "type":"object",
      "properties":{
        "id":{
          "maxLength":64,
          "type":"string"
        },
        "email":{
          "maxLength":255,
          "type":"string",
          "format":"email"
        },
        "name":{
          "maxLength":255,
          "type":["null", "string"]
        },
        "birth_date":{
          "type":"string",
          "pattern":"^[0-9]{4}-[0-9]{2}-[0-9]{2}$"
        },
        "created_at":{
          "maxLength":255,
          "type":"string",
          "formate":"date-time"
        },
        "microposts_count":{
          "type":"integer"
        }
      },
      "required":[
        "id",
        "email",
        "name",
        "birth_date",
        "created_at",
        "microposts_count"
      ]
    }
  },
  "required":[
    "user"
  ],
  "type":"object"
}
```

##### 11.2.1.2. Users resource
Note that the Users resource is just a collection of User object and as a result
it references the User schema.

```json
{
  "$schema":"https://json-schema.org/draft-04/schema#",
  "$id":"https://example.com/users.json",
  "properties":{
    "users":{
      "type": "array",
      "$href": "https://example.com/user.json#/properties/user"
    },
    "meta": {
      "type":"object",
      "page": {
        "type": "integer",
        "default": 0,
        "minimum": 0,
        "$ref": "#/definitions/extra"
      },
      "per_page": {
        "type": "integer",
        "minimum": 1,
        "maximum": 100,
        "default": 50
      },
      "offset": {
        "type": "integer",
        "minimum": 0,
        "default": 0
      },
      "required":[
        "page",
        "per_page",
        "offset"
      ],
    }
  },
  "required":[
    "users",
    "meta"
  ],
  "type":"object"
}
```

##### 11.2.1.3. Request Response inconsistency
Although here we have the same object semantics for request and response object, in theory these could be different.
If that's the case, we should denote each object in the response parented under
distinct JSON attributes (like `accepts`/`produces` or `accepts`/`returns`).

#### 11.2.2. Hypermedia metadata
For the Hypermedia part we will use JSON Hyper Schemas.
Specifically we will use the draft [V4](https://tools.ietf.org/html/draft-luff-json-hyper-schema-00) of JSON Hyper Schemas as the
next drafts ([V5](https://tools.ietf.org/html/draft-wright-json-schema-hyperschema-00), [V6](https://tools.ietf.org/html/draft-wright-json-schema-hyperschema-01)) are targeted to hypermedia APIs that
are HTML-equivelents. For instance, there is no way you can define a `method` attribute, restricting you to `GET` and `POST`
depending whether there is a body to send or not.

Resource schemas defined in the previous section are referenced by the following Hyper Schemas, in order to avoid
duplication of our metadata.

##### 11.2.2.1. User resource
```json
{
  "$schema":"https://json-schema.org/draft-04/schema#",
  "$id":"https://example.com/user-links.json",
  "properties":{
    "$href": "https://example.com/user.json#/properties"
  },
  "links": [
    {
      "rel": "microposts",
      "href": "/microposts?user={userId}&page={page}&per_page={per_page}&offset={offset}",
      "hrefSchema": {
        "allOf": [
          {
            "$ref": "https://example.com/users.json#/properties/meta"
          },
          {
            "$ref": "https://example.com/users.json#/properties/user/id"
          },
        ]
      }
    },
    {
      "rel": "update-user",
      "href": "/users",
      "method": "PATCH",
      "targetSchema": {
        "$ref": "https://example.com/user.json"
      }
    },
    {
      "rel": "delete-user",
      "href": "/users",
      "method": "DELETE",
      "targetSchema": {
        "$ref": "https://example.com/user.json"
      }
    }
  ]
}
```

##### 11.2.2.2. Users resource
```json
{
  "$schema":"https://json-schema.org/draft-04/schema#",
  "$id":"https://example.com/users-links.json",
  "properties":{
    "$href": "https://example.com/users.json#/properties"
  },
  "links": [
    {
      "rel": "self",
      "href": "/users?page={page}&per_page={per_page}&offset={offset}",
      "hrefSchema": {
        "$ref": "https://example.com/users.json#/properties/meta"
      }
    },
    {
      "rel": "create-user",
      "href": "/users",
      "method": "POST",
      "targetSchema": {
        "$ref": "https://example.com/user.json"
      }
    }
  ]
}
```

Notice how we define the pagination, by referencing parts of the user's `meta` object.
We don't treat the clients as stupid but smart enough to understand what they need to do on their part to get what they want.


#### 11.2.4. Descriptions metadata
For human-targeted information, we could use a custom MicroType that describes each attribute of the response object.
Note that **this information must not be required to parse and understand the API but to use the API data on our application domain**.
For instance, understanding that when updating the `email` attribute an email is triggered to inform the user for the change,
is not part of the API client responsibility but it's vital for the application developer to to know what to expect from it.


```json
{
  "user": {
    "id": {
      "title": "The identifier of the resource.",
      "description": [
        "This identifier should not be exposed to the user, to avoid any confusions."
      ]
    },
    "email": {
      "title": "The primary email of the user's account",
      "description": [
        "The email is used for any transactional email.",
        "Also, the same email is used when user authenticates to the system.",
        "Please note that whenever you update the email, user receives an automated email describing the change"
      ]
    },
    "name": {
      "title": "The user's full name (first and last name concataned)",
      "description": [
        "This field could be empty or null.",
        "If so, the application should show the email instead for the user's name."
      ]
    },
    "birth_date": {
      "title": "The date of birth of the user",
      "description": []
    },
    "microposts_count": {
      "title": "The number of published microposts the user has.",
      "description": [
        "Please note that due to caching this number could have a small delay to reflect the actual number",
        "The application should either inform the user about that or make sure it manually updates the microposts counter after publishing/deleting a micropost after publishing/deleting a micropost."
      ]
    }
  }
}
```

This metadata will be used for the documentation generation, as we will se in section [10.6](#106-automating-the-documentation-generation).

#### 11.2.5. The case of a non-compatible spec for introspection: Linked Data metadata using JSON-LD
For denoting the semantic meaning of each attribute of our resources we will employ JSON-LD.
It should be noted that JSON-LD spec was developed with the goal to require as little effort as possible from developers
to transform their existing JSON to JSON-LD but also to not require breaking changes to your
existing API, which makes it backwards compatible with any current deployed API.
This conflicts with our design of introspection because having contexts without the data would break the spec.
As a result we have the following 2 options.

##### 11.2.5.1. Extending spec by creating a Shim MicroType
Our first option is to create a wrapper **shim** MicroType that defines how the spec should work
for the clients to parse and understand the data, with the least possible changes.
A naive shim, that we show here, would output the context information in the introspected process.
Then the client should match this information in combination with the runtime data.

###### 11.2.3.1. User resource
```json
{
  "@context": {
    "@vocab": "https://schema.org/",
    "@type": "Person",
    "birth_date": "birthDate",
    "created_at": "dateCreated",
    "microposts_count": null
  }
}
```

###### 11.2.3.2. Users resource

```json
{
  "@context": {
    "@vocab": "https://schema.org/",
    "birth_date": "birthDate",
    "created_at": "dateCreated",
    "microposts_count": null
  },
  "@graph": [
    {
      "@type": "Person"
    }
  ]
}
```

##### 11.2.5.2. Considering it as runtime metadata
Our second option is to exploit the IATEOAS principles regarding runtime metadata
and append them inside the response by considering them as object-specific runtime metadata.
However, we feel that such dicision should be taken only if nothing else is possible,
given that in Introspected REST data and metadata should be distinctively separated.


### 11.4. The Errors MicroType
When there API is supposed to return an unexpected response to the user, like a 4xx or 5xx error,
the response will have a different structure than the resource that the client requested.

Usually the semantics of an error respond are defined in the API's Media Type but we will use the newly-published [`RFC 7807 Problem Details for HTTP APIs`](https://tools.ietf.org/html/rfc7807),
which defines the `problem+json` Media Type for JSON HTTP APIs.
To give an example how the response will seem when following this RFC,
imagine that when updating a User object, the application developer might wrongly send an invalid `birth_date`.
Then the application should respond with the following structure:


```json
{
  "title": "The birthdate has an invalid format.",
  "details": "The birthdate must be in the format of 1985-04-12T23:20:50.52Z.",
  "status": 422
}
```

If you inspect the spec you will notice that **the spec limits us by omitting specifying a way to associate an error message with a specific resource attribute**.
As a result, we can only specify the falsy attribute in the title or details attribute of the error object, which are human-targeted,
and thus informing only the end user and not the client.
We could add extension members, as the spec suggests, to customize the error object in our needs but the final response object wouldn't be
self-descriptive, unless we customly extended it.


The good thing though is that normally such errors should be caught by running the schema validations from the JSON Schema MicroType on the client side.
The error object could be used for more advanced errors, like the following:

```json
{
  "title": "Transaction failed",
  "details": "The remaining amount of virtual coins in your account is not enough for this purchase",
  "status": 403
}
```

Another thing that we should take care is the fact that this RFC requires returning a different Media Type than the one used by the API.
In theory the API's Media Type explain how the errors work using the same semantics as defined in `problem+json` RFC but the RFC
suggests using `application/problem+json` for Media Type.

> The data model for problem details is a JSON [RFC7159] object; when
> formatted as a JSON document, it uses the "application/problem+json"
> media type.
>
> --- [RFC 7807](https://tools.ietf.org/html/rfc7807)
>
However in order for this to work the client needs to negotiate it and accept this Media Type.
In HTTP that would be achieved using the `Accept` header, which could look like that:
```
application/vnd.api+json, application/problem+json
```

But why declaring this Media Type as a MicroType?
Given that such error information is crucial for the user to understand why her action is not advancing,
we feel that the client should be able to **negotiate** the errors MicroType, that is, the information and structure of the
returned errors object.
Some clients might need the most basic error information, might use only the HTTP status code, other clients might
be interested in as much possible information available in order to show it to the user.

The word `MicroType` is used with it's conceptual meaning, that is, it's a real Media Type but
it can be negotiated between the client/server communication, without affecting the API's Media Type.
For instance, the client might show preference to another problems Media Type before falling back to `problem+json`, as
seen in the following Accept header example:

```
application/vnd.api+json, application/problem-extensive+json, application/problem+json; q=0.8
```


### 11.6. Automating the documentation generation
The documentation of our API should be a dedicated page under out's API url namespace (i.e. `/api`),
by returning a regular web page, targeted to humans and not machines.
The technical details is out of the scope of this implementation example but we
can't stress enough that the generated documentation should only use information from MicroTypes available for the machines,
programmatically wrapped in a human-friendly format.

## 12. Related Work
### 12.1. GraphQL
no urls, basically a query language, no MicroTypes

It's Apple for APIs (cost + compatibility)

The reason we have it in related work is because it's related to REST but not REST.

We are happy that people are trying new things and we support such initiatives.
However it has some issues.

### 12.2. Linked Data and Semantic Web

#### 12.2.1. JSON-LD and HYDRA
Using linked data in our APIs is just great.
HYDRA is in the right direction to introspectable APIs.

Hydra doesn't always require you to serve metadata on the side.
In a custom context you might have to serve them runtime.


### 12.3. [Web Linking](https://tools.ietf.org/html/rfc5988) and link relation types
There is a tendency to overload Link rel for links unrelated to application format etc.
We feel that this is a bad practice and definitely not the right location to add the Microtypes.
Link rel should be used for very few specific things.
For instance Media type and links. Not overloading. Dereference only.

#### 12.3.2. [The 'profile' Link Relation Type](https://tools.ietf.org/html/rfc6906)
+say about the [The Profile Media Type Parameter](https://buzzword.org.uk/2009/draft-inkster-profile-parameter-00.html) (expired draft)

Erik Wilde suggested a profiling mechanism of the underlying Media Type through the [HTTP Link header](https://tools.ietf.org/html/rfc5988).

>  A profile is defined not to alter the
>   semantics of the resource representation itself, but to allow clients
>   to learn about additional semantics (constraints, conventions,
>   extensions) that are associated with the resource representation, in
>   addition to those defined by the media type and possibly other
>   mechanisms.
>
> --- [RFC 6906](https://www.ietf.org/rfc/rfc6906.txt)
>

Essentially, the profile parameter, given that the client understands it, would define **additional** semantics of
the response's representation that are not defined through the Media Type used.
The information for the additional semantics would be found in all responses regardless the client but only
the "smarter" clients would be able to parse, understand and use this information whereas the rest would just ignore it.

This link relation type is similar to our work of MicroTypes but unfortunately fails to advocate towards reusable profiles.

>  While this specification
>   associates profiles with resource representations, creators and users
>   of profiles MAY define and manage them in a way that allows them to
>   be used across media types; thus, they could be associated with a
>   resource, independent of their representations (i.e., using the same
>   profile URI for different media types).  However, such a design is
>   outside of the scope of this specification, and clients SHOULD treat
>   profiles as being associated with a resource representation.
>
> --- [RFC 6906](https://www.ietf.org/rfc/rfc6906.txt)
>

By having  profiles attached to specific Media Types results in much less adoptability and flexibility and fails to signal the
actual practicability of such architecture.
However, if profiles take the conceptual form of independent MicroTypes, then the clients can negotiate for those and eventually choose
the one that fits best.
Although the negotiation part is skipped from the RFC, we feel that such works are towards the right direction that will allow us
to build.

+no negotiation

#### 12.3.3. [Linksets](https://tools.ietf.org/html/draft-wilde-linkset-link-rel-02) (draft)
say that Link rel is so overused that Linksets was needed...
As we discussed previously, HTTP Link header tends to be overloaded because it's our only way to signal Hypermedia detached
by the response representation and message format.
In order to mitigate such issues, Linksets proposal tries to offload HTTP Link links from a resource or url, when having them in there is
costly or even not possible for the server.

>  Resources on the Web often convey typed Web Links [RFC5988] as a part
>   of resource representations, for example, using the <link> element
>   for HTML representations, or the "Link" header field in HTTP response
>   headers for representations of any media type.  In some cases,
>   however, providing links by value is impractical or impossible.  In
>   these cases, an approach to provide links by reference (instead of by
>   value) can solve the problem.  This specification defines the
>   "linkset" relation type that allows to link resources to sets of
>   links, thereby making it possible to represent links by reference,
>   and not by value.
>
> --- [Linkset Internet-Draft](https://tools.ietf.org/html/draft-wilde-linkset-link-rel-02)
>

For instance, imagine that a resource needs to link a set of links that are managed by not the host owning the resource, but
by a 3rd party origin.
Another case scenario is when the Link header is overloaded by holding a large number of links that are needed to be inlcluded along with
the resource and would result in HTTP error ("413 Request Entity Too Large" and "414 Request-URI Too Long").

By providing a dereferancable link that points to the 3rd party link set, there could be advantages in numerous cases,
like different caching policies between the 2 origins and decoupling in general.


We think that linksets is anohter small piece towards a introspectiveness and hence we support such initiatives.
However we feel that overloading the Link relation type as we discuss in the next question is not the right way.

### JSON home

### HTTP Hints

### 12.4. RESTful API Description Languages
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


### 12.5. API directories
Another trend for APIs is to register them  in an online service, called API dictionary and possible push there the API documentation as well.
We feel that this is not a very helpful structure. APIs should be discoverable by themselves without using centralized services.
The API's root url should provide everyhing that is needed, or using already published protocols
like WebFinger, which builds upon [Well-Known Uniform Resource Identifiers RFC](https://www.rfc-editor.org/rfc/rfc5785.txt) and can give API information
for client bootstraping.



## Conclusion
We are not giving a solution here. We are giving food for thought.
The actual solutions will come by the community

To hackenrews: Introspected REST: An alternative to REST and GraphQL
Hi, author here. Some comments:


**REST is all about evolvability by applying a uniform interface in your implementation.**.

If you don't have evolvability in your API architecture don't even try to compare it to REST because these are 2 different things.
A non evolvable API can only be compared to RPC.
It can be great RPC but it's an RPC.
Does it mean it's bad ? Not neceserily, it depends on the context it's used.
But for long-term APIs, evolvability is very nice thing to have. But again, it all depends on the amount of time/money you
want to invest for it in combination with the use case.
However, comparing a non-evolvable JSON API and saying that it's better or equivelent than REST then this means that you don't understand what REST is.

If you **do have** evolvability then my guess is that it will be either REST or GraphQL.
If it's not one of these, please show up and tell us how you did it. Tell us what was good and what was bad of your decisions.
It's important to share new architectural styles and common practices.

Here, we propose an alternative to REST and GraphQL, Introspected REST.
It's a new architectural style that tries to mitigate drawbacks of REST (mostly it's complexity)
by requesting metadata/hypermedia on demand instead of mixing up everything in the same response object.
Introspected REST is not perfect either, it does have it's own drawbacks and limitations.
But we feel that in order to move forward to evolvable, managable, sustainable APIs, we need to levarage
such architectures that use composable, reusable modules and serve the metadata/hypermedia on the side, on demand.

We are open to any feedback here or as a github issue/PR :)
The manifesto is still on draft stage, open for suggestions/feedback/pull requests and
it will be locked in one year from now (already got introspected.rest domain for it :) )

### The future is full of posibilities
**How do we negotiate to the client that a resource is available through HTTP/2 Stream Server Push without documentation ?**
That requirement would be very impractical using a REST interface because everything would have to be in the same response.
The profile RFC does not do a correct negotiation and it's problematic
With Introspected REST, we can solve such problems by providing such information only to the clients that are really interested,
in an easy, composable way.

Our solution is by far **not** complete but we have set the basis for the community to experiment and come up with MicroType specs.

We call the community to start experiment in this model and come up with patterns.
In any case we feel that to reach a sustainable API with the evolvability span of REST model, Introspected REST model is necessary.

Another reason is to have a real REST alternative with arguments and remove the develper roy shadow and fear to deprecate Roy's REST.a

We are sure that without such design/architecture we can't have complex yet self-described APIs.

We want to embrace even the simplest APIs and allow them to provide the elements of REST that need, yet being easy to impelement
and backwards compatible.
The key thing here is backwards compatibility, because it allows you to incrementally add REST HATEOAS incrementally.


RFC5988: A means of indicating the relationships between resources on the Web,
   as well as indicating the type of those relationships, has been
   available for some time in HTML [W3C.REC-html401-19991224], and more
   recently in Atom [RFC4287].  These mechanisms, although conceptually
   similar, are separately specified.  However, links between resources
   need not be format specific; it can be useful to have typed links
   that are independent of their serialisation, especially when a
   resource has representations in multiple formats.
Oh wait..we just figured out that having the links in there might not be the ideal.


Final review:
1. check paragraphs
2. check code terms (RESTxx)
3. check links
4. check bolds

who to ping: https://www.mnot.net/ Eric Wilde, Roy, wycats, Stevel Klabnik, I. Grigorik, mark nottigham, amdusen, MIKE AMUNDSEN

The ratio of data/hypermedia of a resource

add querying specific elements of an introspection (like links, attributes etc)

Say that about custom capabilities like running functions or having an HTTP2 push. Say also that on metadata.

#### 8.2.3. REST does not make it easy to integrate different APIs together
Consider a product resource.
The product resource contains information about the product itself, as well as related information and services such as similar products.
However, information about payment options is not provided by the product resource itself, and is provided via a linkset instead.
This means any client requesting payment options will request payment links from a separate server.
This not only decouples product resources from payment management; it also allows the payment options to be more easily customized based on the specifics of the client, meaning that the complete "product and payment options" view is a combination of the product resource (and associated hypermedia controls), and payment links provided by the specialized payment options service.

say about atom+json example!! A totally new Media Type that we need to manually understand..

>  The best software architecture “knows” what changes often and makes that easy.
>
> --- Paul Clements
>

### What Introspected REST does in bullets
* It simplifies REST
  * Separates Data from Metadata
* Gives more to metadata by providing information (vocabularly) to the clinet
not only about hypermedia but also other metadata (like data types etc)
* It embraces composability of different MicroTypes
  * enhances negotiation
* Fixes broken API negotiation (problem+json)
* Say that ALPS and related stuff can still be applied, in a different way!
* Say that MicroTypes is not for Hypermedia only but is **needed** for other micro-definitions as well!


Say in intro that Media Types have reached their limits. For AI/evolvable apis we need more than that.

Say about imaturity of rfcs/specs related to introspection (example: JSON schema v4 vs v5/v6)

Add a note on unfinished RFCs

https://www.mnot.net/blog/2012/10/29/NO_OPTIONS --- important !!

say that this is hard work as REST is mentioned in HTTP RFCs !!!


For introduction: There is a confusion of what REST is. REST is all about evolvability (copy blog post).
We will start by analyzing and giving concrete definitions of what REST is.
Then we will show REST's drawbacks explaining why REST never flew off, why people have been unconciously avoiding it.
We will propose a new model, alternative to REST. We will also present the concept of MicroTypes.


We should note that according to [RFC 6831](https://tools.ietf.org/html/rfc6838) any Media Type parameters must be very well defined beforehand:

> Media types MAY elect to use one or more media type parameters, or
>   some parameters may be automatically made available to the media type
>   by virtue of being a subtype of a content type that defines a set of
>   parameters applicable to any of its subtypes.  In either case, the
>   names, values, and meanings of any parameters MUST be fully specified
>   when a media type is registered in the standards tree, and SHOULD be
>   specified as completely as possible when media types are registered
>   in the vendor or personal trees.

This goes against our concept of arbiratry number of autonomous MicroTypes that can be included by a parent Media Type parameters.
We will see in the next section what are the possible solutions to overcome this limitation.

