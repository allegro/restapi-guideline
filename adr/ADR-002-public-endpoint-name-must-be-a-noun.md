# ADR-002: Public endpoint name must be a noun

## Status

ACCEPTED

## Context

To facilitate a seamless API integration process, it is crucial to maintain consistency and adhere to current standards.

The majority of API providers opt to comply with
the [second level of the Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html#level2),
which provides well-defined guidelines on how to design and interact with APIs and has become widely accepted as a
standard in the industry.

At this level, the primary objective is to structure APIs as a set of endpoints, where each endpoint represents a
specific resource. The resources are named using nouns, and the HTTP methods utilized in the API align with the
corresponding operations performed on those resources, functioning as verbs. This approach significantly enhances the
clarity and consistency of API design, enabling developers to understand and effectively utilize the API.

Our API already meets the requirements of this level, and we utilize the `command pattern` to handle custom cases that
don't fit within the standard HTTP methods. However, during our assessment, we identified several issues with the
current pattern:

1. **Non-compliance with [RFC-9110](https://www.rfc-editor.org/rfc/rfc9110):** The current pattern utilizes the HTTP `PUT` method, but according to RFC
   specifications, `PUT` should represent the desired state of the target resource, which is not the case for the
   command pattern.
2. **Dedication to asynchronous processing:** While the current pattern is designed for asynchronous processing, there
   are scenarios where we require a synchronous approach and expect the command response immediately.
3. **Client-side UUID generation requirement:** The current pattern enforces the generation of UUIDs on the client side.
   Although this was originally intended to ensure idempotency on the server side, it is not always necessary.
4. **Lack of support for safe operations:** There are situations where we only want to perform custom reads without
   modifying any state on the server side. Unfortunately, the current command pattern does not provide this capability.

Considering these issues, we have made the decision to revisit and improve the existing command pattern.

## Decision

**Each endpoint name defined in public API must be a noun, for custom operations query or command pattern must be
applied**

To handle custom operations that cannot be expressed solely through HTTP methods, we have introduced three patterns that
are compliant with the second level of the Richardson Maturity Model. These patterns are as follows:

- **Query Pattern:** This pattern allows the execution of queries on the API to retrieve specific data or perform
  calculations.

- **Synchronous Command Pattern:** With this pattern, custom operations can be executed synchronously, meaning that the
  API response is received immediately after the operation is completed.

- **Asynchronous Command Pattern:** This pattern allows for the execution of custom operations in an asynchronous
  manner. In
  this approach, the API initiates the operation and provides an identifier that can be used to check the execution
  result. The actual completion of the operation may happen at a later time.

By incorporating these patterns, our API remains compliant with the second level of the Richardson Maturity Model while
accommodating custom operations that go beyond the standard HTTP methods.

### Query Pattern

#### Purpose

- Advanced queries which cannot be expressed by noun and `GET` method.
- Queries that have complicated query attributes

#### Assumptions

1. Implemented by single Endpoint
2. Used with HTTP `POST` method
3. Ends with suffix `-queries`
4. Request and Response body has no fixed structure
5. Behaves like safe method (it doesn't alter the state of the server)

#### Example

Lets imagine that we want to implement endpoint which returns cheapest offer which fulfills given requirements.
Additionally we have more than 10
input parameters which makes it problematic to implement it by HTTP `GET` method and query parameters. Query pattern
could be implemented as bellow:

**Request:**
We are providing all criteria which need to be fulfilled as a request body attributes
`POST /find-cheapest-offer-queries`

````json
{
  "category": "",
  "state": "new",
  "stock": {
    "available": 7,
    "unit": "UNIT"
  },
  "marketplaces": [
    "allegro-pl",
    "allegro-cz"
  ],
  "delivery": {
    "carrier": "UPS",
    "handlingTime": "PT24H"
  },
  "payments": {
    "invoice": "VAT"
  }
}
````

**Response:**

As a response we are receiving id of cheapest offer fulfilling given criteria.

```json
{
  "offerId": "1234"
}

```

### Synchronous command pattern

#### Purpose

- Advanced operations which cannot be expressed by connection of noun and HTTP request method

#### Assumptions

1. Implemented by single endpoint
2. Used with HTTP `POST` method
3. Ends with suffix `-commands`
4. Request has no fixed structure
5. (Optionally) Request has `id` attribute to assure idempotency
6. Response has no fixed structure

#### Example

Let's consider case where seller would like increase price in multiple offers with single API call. Command to handle
such case could be implemented as bellow:

**Request**
We have command which allows to modify offer prices.

`POST /modify-offer-prices-commands`

Request body contains attributes which can specify price modification such
as `changeType`, `modificationValue`, `modificationUnit`, `includedOffers`.

```json
{
  "changeType": "INCREASE",
  "modificationValue": 15,
  "modificationUnit": "PERCENTAGE",
  "includedOffers": [
    "12345",
    "45678",
    "56789"
  ]
}
```

**Response**

In response body we are receiving list of performed modifications.

```json
{
  "modifications": [
    {
      "offerId": "12345",
      "previousPrice": 10,
      "newPrice": 11.5
    },
    {
      "offerId": "45678",
      "previousPrice": 4,
      "newPrice": 4.6
    },
    {
      "offerId": "56789",
      "previousPrice": 30,
      "newPrice": 34.5
    }
  ]
}
```

### Asynchronous command pattern

#### Purpose

- Advanced operations which cannot be expressed by connection of noun and HTTP request method
- Operations that are time consuming

#### Assumptions

1. Implemented by the two endpoints:
    - For trigger operation `POST /xxx-commands`
    - For status check `GET /xxx-commands/{id}`
2. Ends with suffix `-commands`
3. Request has no fixed structure
4. (Optionally) Request has `id` attribute to assure idempotency
5. Response returns operation `id`
6. Response from status check has no fixed structure

#### Example

Let's refer to example from Synchronous command pattern. When operation is time consuming asynchronous command patter
could be a better option, in such case command could be implemented as bellow:

**Request for command trigger**

Same as before, we have command which allows to modify offer prices.

`POST /modify-offer-prices-commands`

```json
{
  "modifications": [
    {
      "offerId": "12345",
      "previousPrice": 10,
      "newPrice": 11.5
    },
    {
      "offerId": "45678",
      "previousPrice": 4,
      "newPrice": 4.6
    },
    {
      "offerId": "56789",
      "previousPrice": 30,
      "newPrice": 34.5
    }
  ]
}
```

**Response for command trigger**

Instead of command result we are receiving `commandId` which we can use to check command result

```json
{
  "commandId": "1b172506-1bec-11ee-be56-0242ac120002"
}
```

**Request for command result check**

To check command result we are using second endpoint where we are passing command id.

`GET /modify-offer-prices-commands/1b172506-1bec-11ee-be56-0242ac120002`

**Response for command result check**

Response additionally contains `commandStatus` field which indicates whether command was finished, or is still in
progress.

```json
{
  "commandStatus": "DONE",
  "modifications": [
    {
      "offerId": "12345",
      "previousPrice": 10,
      "newPrice": 11.5
    },
    {
      "offerId": "45678",
      "previousPrice": 4,
      "newPrice": 4.6
    },
    {
      "offerId": "56789",
      "previousPrice": 30,
      "newPrice": 34.5
    }
  ]
}
```

## Consequences

By making this decision, we ensure compliance with the second level of the Richardson Maturity Model. Moreover,
developers will have standardized patterns for handling custom cases that are in line with RFC-9110. This standardized
approach is expected to have a positive impact on API consistency and improve the integrators' understanding of the API.