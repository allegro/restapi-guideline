# ADR-001: Public API is not versioned

## Status

ACCEPTED

## Context

Versioning of complex API which additionally has a dynamic development proces is hard and brings a lot of challenges.
Based on our experience we observed following problems with API versioning:

- Growth of API complexity
- Different versions of endpoints are confusing to clients
- Clients assume that there will be no changes without new version introduction even if changes are no breaking
- Growth of maintenance cost
- It's harder to document versioned API

## Decision

**We don't introduce new API versions, all changes also those that are breaking would be introduced under already
existing
version.**

Instead of API versioning we have the following procedures to handle new changes depends on change kind:

1. Changes which could be safely introduced without breaking client code
2. Changes requiring deprecation period
3. Changes requiring non-breaking adoption on client side
4. Changes requiring breaking adoption on client side

### Changes which could be safely introduced without breaking client code

Changes in this group could be deployed safely without breaking client code.
Incoming change can be announced before release, however there is no risk to break client code even if client ignore this
notification. 

**To make such changes not breaking, clients needs to be ready for receiving new unknown fields in response.**

### Changes requiring deprecation period

Changes in this group should be handled in two steps:

1. Marking element as deprecated
2. Remove element after agreed period

Depends on case deprecation period could take few months up to few years.

### Changes requiring non-breaking adoption on client side

In those kind of changes we can notify client about incoming deployment in advance. Time between notification and
deployment depends on scope of changes, but usually takes few months. During this time clients should be able to prepare
their code to incoming change without breaking current flow. It there is a need we can introduce beta version to give
clients possibility to try new changes before official release.

### Changes requiring breaking adoption on client side

There is group of changes where it is hard to client to prepare adoption for incoming changes without breaking current
flow.
In other words, when client deploy adoption for incoming change it will be not compatible with current API.

For such case following migration process should be applied:

1. Introduction of beta version which includes changes that would be deployed to stable version in future
2. Agree with clients period when they switch to beta version and date when changes would be released to stable version
3. After release, ask clients to switch back to stable version

### Change classification

- Changes which could be safely introduced without breaking client code:
    - New Endpoint introduction
    - Adding new media type of request / response
    - Adding new field in response
    - Adding new optional
        - request header
        - query param
        - request body field
    - Tighten constraint in response
    - Releasing constraint in request
- Changes requiring non-breaking adoption on client side:
    - Tightening input constraint
    - Releasing output constraint
- Changes requiring deprecation period:
    - Endpoint removal
    - Response body field removal
    - Removal:
        - request header
        - query param
        - request body field
- Changes requiring breaking adoption on client side:
    - Changing of request body structure
    - Changing of response body structure
    - Adding new mandatory:
        - header
        - query param
        - request body field

### Special cases

There are few special changes which could be split to smaller ones which will simplify change introduction.

#### Endpoint  renaming

Change introduction could be done in two steps

1. New endpoint introduction
2. Old endpoint removal.

#### Response body field renaming

1. New field introduction
2. Removal old field.

#### Request body field renaming

1. Introduction of new optional field.
2. Making new optional field mandatory (Tightening input constraint)
3. Making old field optional (Releasing input constraint)
4. Removal old field

#### Adding new mandatory query param, request body field, header

1. Adding new optional element
2. Make optional element required (Tightening input constraint)

## Consequences

Thanks to this decision we can reduce complexity of API without resignation from breaking changes and
independent deploy-ability for our tech teams. We are aware that this approach requires closer communication with
clients and higher engagement from their side. Some changes could be less comfortable for clients, although we
prepared process for most common changes to make this process as painless as possible.