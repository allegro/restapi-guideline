# Representation

* [Property name format](#property-name-format)
* [Provide resource (UU)IDs](#provide-resource-uuids)
* [Null values](#null-values)
* [Empty collections](#empty-collections)
* [Use UTC times formatted in ISO8601](#use-utc-times-formatted-in-iso8601)
* [Time without date](#time-without-date)
* [Country, language and translations](#country,-language-and-translations)
	* [Value representation](#value-representation)
	* [Translations](#translations)
* [Price and currency](#price-and-currency)
* [High precision numbers](#high-precision-numbers)
* [Enum values](#enum-values)
* [Nesting foreign resources relations](#nesting-foreign-resources-relations)
* [Provide full resources where available](#provide-full-resources-where-available)
* [Accept JSON in request bodies](#accept-json-in-request-bodies)
* [Keep JSON response minified](#keep-json-response-minified)
* [Filtering](#filtering)
	* [Simple filters in query string](#simple-filters-in-query-string)
	* [Filter by multiple values in one filed](#filter-by-multiple-values-in-one-filed)
	* [Advanced filtering (similar to search) concerning many parameters and nested structures](#advanced-filtering-similar-to-search-concerning-many-parameters-and-nested-structures)
* [Sorting](#sorting)
* [Wrap collection in object](#wrap-collection-in-object)
* [Consistent paging scheme](#consistent-paging-scheme)
* [Keep response gziped](#keep-response-gziped)

## Property name format

Property names should be meaningful.
Use camel case in parameters and properties (e.g. `firstName`) instead of underscoring (e.g. `first_name`):

```javascript
"firstName": "John",
"lastName": "Smith"
```

Array types should have plural property names. All other property names should be singular.

```javascript
{
  // Singular
  "user": "123456",
  // An array of siblings, plural
  "offers": [{},{}],
  // "totalItem" doesn't sound right
  "totalCount": 10,
  // But maybe "offersCount" or just "count" is better
  "offersCount": 10,
}
```

## Provide resource (UU)IDs

Assign an id attribute by default to each resource. Use UUIDs unless you
have a very good reason not to. Do not use IDs that will not be globally
unique across instances of the service or other resources in the
service, in particular auto-incrementing IDs.

Render UUIDs as a lowercase string in `8-4-4-4-12` format, e.g.:

```javascript
"id": "01234567-89ab-cdef-0123-456789abcdef"
```

## Null values

Blank fields are generally included as `null` instead of being blank strings or omitted. If a property state or value is unknown, consider setting the property to `null`.

Example of an unknown value or state:
```javascript
{
  // ...
  "buyerAddresses":[
        {
           "type":1,
           "company":null,
           "name":"Jaś Fasola",
           "street":"Zakręt 56",
           "postCode":"00-999",
           "city":"Lądek Zdrój"
        }
     ],
  // ...
}
```

## Empty collections

If you want to return empty collection, return `[]` instead of `null`. Some client frameworks cannot iterate over `null` and need extra null checking.  Iteration over `[]` is always possible. e.g.:

```javascript
{
	"status": "OK",
	"errors": []
}
```


## Use UTC times formatted in ISO8601

Accept and return times in UTC only. Render times as a string ISO8601 format (yyyy-MM-dd'T'HH:mm:ss.SSSZ),
e.g.:

```javascript
{
  // ...
  "createdAt": "2012-01-01T12:00:00.000Z",
  "updatedAt": "2012-01-01T13:00:00.000Z",
  // ...
}
```

## Time without date

Render time value as a string in format (HH:mm:ss.SSS) without time zone information, e.g.:

```
"from": "08:00:00.000",
"to": "16:00:00.000"
```

There is no need to add information about a time zone – a client should present value received from an API without any modification.
If you need to present time in a specific time zone, use UTC for full date and time.

## Country, language and translations

### Value representation

If you need to send some property that indicates a country then use the [ISO 3166 Alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1#Current_codes)
codes for that - 2 letters, always upper-cased.

Sample JSON with such a property:
```
{
    "address" : {
        // ...
        "city" : "Poznań"
        "countryCode": "PL"
    }
}
```

If you want to refer to a language saved in the property, use the [RFC 1766](http://tools.ietf.org/html/rfc1766)
language tag. It combines the previously mentioned [ISO 3166](http://en.wikipedia.org/wiki/ISO_3166-1) country codes and
[ISO 639](https://en.wikipedia.org/wiki/ISO_639) language names.

Sample JSON with such a property:
```
{
    "user" : {
        // ...
        "prefferedLanguage": "ru-UA" // Russian used in eastern Ukraine
    }
}
```

Java sample for [Locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html) supporting these formats:

```java
Locale locale = Locale.forLanguageTag("ru-UA");
String countryCode = locale.getCountry();
// countryCode == "UA"
```

The above actually parses according to the [RFC 5646](http://tools.ietf.org/html/rfc5646) standard which is a backward
compatible update to the previously mentioned [RFC 1766](http://tools.ietf.org/html/rfc1766).

### Translations

In the section above we presented how the values of country and language references should be
represented in JSON, query params, etc. Translating user messages for the client is a different topic.
The HTTP standard defines an appropriate header for that: `Accept-Language` [RFC 2616, 14.4](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4).
If the client desires to receive localized user messages in the response, it should set the `Accept-Language` header in its request.

This is required to handle at least two types of "Accept-Language" :

```
Accept-Language: en-* (* is any valid type eg. "GB", "US")

```

and

```
Accept-Language: pl-PL
```

By sending the information presented above, you will inform the backing service that all user messages (including [error](#error) messages) should be translated to English or Polish, respectively. If "Accept-Language" header is omitted then default value is en-US.

## Price and currency

All prices (included in response and request bodies) are requested and returned as a structure with `amount` and `currency`
fields as proposed by [paypal](https://developer.paypal.com/docs/api/#common-objects-for-payouts), e.g.

```javascript
{
    "buyNow": {
        "amount": "11.25",
        "currency": "PLN"
    }
}
```

* `currency` - 3 letter currency code as defined in [ISO 4217](http://en.wikipedia.org/wiki/ISO_4217#Active_codes)
* `amount` - string representation which follows the rules for high precission numbers described in the next chapter

If you want to submit price `amount` and `currency` as a response to the GET request, you should use:

```bash
curl https://api.allegro.pl/contests?totalAmount.amount=11.25&totalAmount.currency=PLN  \
    -H "Accept: application/vnd.allegro.public.v1+json"
```

## High precision numbers

Many libraries and languages don't deserialize your JSON numeric fields with high precision in mind and are hard to
customize (JavaScripts **eval** method for eq.). Because of this limitation you should serialize numeric fields that
require much care when handling to a string which follows these rules:

  * if the decimal separator is used then it must be a **dot (.)**
  * at least one digit must be present before a decimal separator
  * no other separators can be used
  * the exponent field cannot be used
  * non numerical values like "NaN" inside the string are not allowed
  * don't use more digits after the decimal separator then you actually need

Some `amount` field examples which follow the above rules:

* "0"
* "0.01"
* "1"
* "1.1"
* "11.25"
* "1234567.25"
* "9999999.999"

## Enum values

Enum values are represented as uppercase strings.

As API grows, enum values may be added, removed or changed. Using strings as enum values ensures that
downstream clients can gracefully handle changes to enum values.

Java code:

```java
public enum Color {
  WHITE,
  BLACK,
  RED,
  YELLOW,
  BLUE
}
```

JSON Object:

```javascript
{
  "color": "WHITE"
}
```

## Nesting foreign resources relations

Nest foreign resources references, even if the only information is `id` of the object referred to, with a nested object, e.g.:

```javascript
{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "offer-name",
  "seller": {
    "id": "5d8201b0...",
    "name": "user-name"
  },
  // ...
}
```

Instead of a flat structure e.g.:

```javascript
{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "offer-name",
  "sellerId": "5d8201b0...",
  "sellerName": "user-name"
  // ...
}
```

This approach makes it possible to inline more information about the
related resource without having to change the structure of the response
or introduce more top-level response fields, e.g.:

```javascript
{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "offer-name",
  "seller": {
    "id": "5d8201b0...",
    "name": "user-name",
    "email": "user@allegrogroup.com"
  },
  // ...
}
```

## Provide full resources where available

Provide the full resource representation (i.e. the object with all properties) whenever possible in the response.

## Accept JSON in request bodies

Accept JSON as Content-Type data in `PUT`/`PATCH`/`POST` request bodies, either
instead of or in addition to form-encoded data. This creates symmetry
with JSON response bodies, e.g.:

```bash
curl -X POST https://api.allegro.pl/users \
    -H "Content-Type: application/vnd.allegro.public.v1+json" \
    -d '{"name": "user-name"}'
```

```javascript
{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "user-name",
  // ...
}
```

## Keep JSON response minified

Extra whitespace adds needless response size to requests, and many
clients (e.g. web browsers) will automatically "improve" JSON
output. It is best to keep JSON responses minified e.g.:

```json
{"id": "01234567-89ab-cdef-0123-456789abcdef","name":"offer-name","seller":{"id":"5d8201b0...",
"name":"user-name","email":"user@allegrogroup.com"}
}
```

Instead of e.g.:

```json
{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "offer-name",
  "seller": {
    "id": "5d8201b0...",
    "name": "user-name",
    "email": "user@allegrogroup.com"
  }
}
```

## Filtering
### Simple filters in query string

Use a query parameter for each field that supports filtering (most of them probably do not provide such support; you should document what fields are filterable).
For example, when submitting a request for a list of general delivery points to the `/general-deliveries` endpoint, you may want to limit them to the given name and city.
To do so, use a request such as:

```bash
curl -X GET https://api.allegro.pl/general-deliveries?name=UP+Poznań+41&address.city=Poznań -H "Accept: application/vnd.allegro.public.v1+json"
```

Here, `name` and `address.city` are fields that support filtering.

Simple rules for query string filters:

* refer directly to collection entities (`name`, not `points.name`)
* use "." (dot) to indicate a (unique) field name in nested structure (`address.city`, not just `city`)

Sample response:

```json
{
  "points": [
    {
      "id": "de305d54-75b4-431b-adb2-eb6b9e546014",
      "name": "UP Poznań 41",
      "address": {
        "street": "Ulica Starołęcka 42",
        "code": "61-360",
        "city": "Poznań"
      }
    }
  ]
}
```

To support ranges in filtering use virtual fields with suffixes :

- **gt** - greater than
- **lt** - less than
- **gte** - greater or equal
- **lte** - less or equal

Example request: 

```bash
curl -X GET https://api.allegro.pl/general-deliveries?rate.gt=200 -H "Accept: application/vnd.allegro.public.v1+json"
```

### Filter by multiple values in one filed

If you want to filter many values in one field, you should repeat the field name many times in URL adding different values, e.g.
to filter shipping payments in `PRE` and `POST` values use:

```bash
curl -X GET https://api.allegro.pl/general-deliveries?shipments.payments=PRE&shipments.payments=POST -H "Accept: application/vnd.allegro.public.v1+json"
```

This type of convention, i.e. `shipments.payments=PRE&shipments.payments=POST` is supported by many frameworks out of the box.



### Advanced filtering (similar to search) concerning many parameters and nested structures
For advanced filtering:

* create search id from your data – sending `POST` body to "new resource" (e.g. `/product-searches`) will return search id (in `Location` header) for a query
* get collection for search id – use search id returned after the operation mentioned above to get expected collection of entities by calling base resource (e.g. `/products?search.id=`)
* get search data for search id – use search id returned after the operation mentioned above to get search data by calling "new resource" (e.g. `/product-searches`)

Rules:

* "new resource" should be presented in `/*-searches` convention where the base resource name is singular, e.g. in case of `/products` it should be named `/product-searches`
* entity in a request `POST` body in `/*-searches` (e.g. `/product-searches`)
* handle query string parameter `search.id` in base resource (e.g. `/products`) to handle filters
* entities created by a "new resource" should be versioned as the base resource – it should be possible to get a
suitable version of `/product-searches` resource using any `search.id` for each new version of e.g. `/products` resource
* the new resource version should not break backward compatibility (if possible)

Sample entities in `/products` collection

```json
{
    "products": [
        {
            "id": "7d3e4d5a-817c-45f8-930b-e319dbcedc5c",
            "name": "Doładowanie Heyah 5 zł",
            "parameterGroups": [
                {
                    "id": "4be770c3-6805-4967-8f13-0457dc8ed446",
                    "name": "Bazowe informacje",
                    "parameters": [
                        {
                            "id": "ce87a49d-55c3-476c-b7f8-349dfaf89510",
                            "name": "Rodzaj usługi",
                            "values": [
                                "Doładowanie"
                            ]
                        },
                        {
                            "id": "63baaaf2-534a-4c20-b1ed-e44952d7e556",
                            "name": "Nazwa",
                            "values": [
                                "Doładowanie Heyah 5 zł"
                            ]
                        },
                        {
                            "id": "ab98c5ec-f9c6-440f-9af0-a19caaba7eca",
                            "name": "Operator",
                            "values": [
                                "Heyah"
                            ]
                        },
                        {
                            "id": "7fdd3aa4-3cab-4d4f-93c4-9c2ff05bec86",
                            "name": "Wartość doładowania",
                            "values": [
                                "5 zł"
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
```


Let’s assume you are looking for `products` with:

* `parameterGroups.id` with value `4be770c3-6805-4967-8f13-0457dc8ed446`
* dynamic parameters with many values, e.g. a dynamic parameter of `id` `ce87a49d-55c3-476c-b7f8-349dfaf89510`
with `value` `Doładowanie` and parameters of `id` `ab98c5ec-f9c6-440f-9af0-a19caaba7eca` with `value` `Heyah`

you can submit a request for `/product-searches`:

```bash
curl -X POST https://api.allegro.pl/product-searches -H "Accept: application/vnd.allegro.public.v1+json" -d
{
    "limit": 100,
    "offset": 150,
    "parameterGroups": [
        {
            "id": "4be770c3-6805-4967-8f13-0457dc8ed446",
            "parameters": [
                {
                    "id": "ce87a49d-55c3-476c-b7f8-349dfaf89510",
                    "values": [
                        "Doładowanie"
                    ]
                },
                {
                    "id": "ab98c5ec-f9c6-440f-9af0-a19caaba7eca",
                    "values": [
                        "Heyah"
                    ]
                }
            ]
        }
    ]
}
```

In the sample above server should write down the version of search data used to create it from the `Accept` header – `application/vnd.allegro.public.v1+json`.
It will improve the process of mapping from one search data version to another in the future.

In return, you get an id for the created "search" (in `Location` header, status `HTTP 201 Created`), which holds your search results. To view them, provide:

```bash
curl -X GET https://api.allegro.pl/product-searches/4be770c3-4967-6805-8f13-0457dc8ed446 -H "Accept: application/vnd.allegro.public.v1+json"
```

Important note:  There is no way to override filters for a given id - you must create a new search resource.


Use `search.id` parameter to get filtered collection of entities with a sort parameter such as:

```bash
curl -X GET https://api.allegro.pl/products?search.id=4be770c3-4967-6805-8f13-0457dc8ed446&sort=-parameterGroups.name -H "Accept: application/vnd.allegro.public.v1+json"
```

## Sorting

Sorting: A generic parameter `sort` can be used to describe sorting rules.
Adjust complex sorting requirements by letting the sort parameter take in a list of comma separated fields, each with a possible unary negative to imply descending sort order.
Let's look at some examples:

* `GET /offers?sort=-buyNow` – Retrieves a list of offers put in a descending order by buyNow price
* `GET /offers?sort=-buyNow,createdAt` – Retrieves a list of offers put in a descending order by buyNow price. Within a specific buyNow price, older offers are presented as first.

## Wrap collection in object

Always return root element as an object. This way you can add extra `metadata` fields to the response without compatibility breakdown.

For example, in the case of the `offers` collection, you can add a metadata field such as `count` containing a number of matching offers:

```json
{
    "offers": [
        {
            "id": "01234567-89ab-cdef-0123-456789abcdef",
            "name": "PINK FLOYD: THE ENDLESS RIVER"
        },
        {
            "id": "11234567-89ab-cdef-0123-456789abcdef",
            "name": "Ufomammut - Eve"
        }
    ],
    "count": 4674
}
```

When a collection is in the root, you cannot add extra field there.

```json
[
    {
        "id": "01234567-89ab-cdef-0123-456789abcdef",
        "name": "PINK FLOYD: THE ENDLESS RIVER"
    },
    {
        "id": "11234567-89ab-cdef-0123-456789abcdef",
        "name": "Ufomammut - Eve"
    }
]
```

Metadata should only contain direct properties of the response set, not properties of the members of the response set.

## Consistent paging scheme

Use `offset` instead of `page` and `limit` parameters. It is much more flexible for the client, e.g.

```json
GET /offers?offset=0&limit=100
GET /offers?offset=100&limit=500
```

## Keep response gziped

All responses should be gziped based on the client `Accept-Encoding: gzip` header and return information about used encode method in the `Content-Encoding: gzip` header.
