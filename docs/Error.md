# Error

## Provide Trace-Ids for Introspection

Include the `Trace-Id` header in each API response that contains an
UUID value. By logging these values on the client, server and any backing
services, it provides a mechanism to trace, diagnose and debug requests.

## Generate structured errors

A client sets the requested `Accept-Language` header ([RFC code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)) to get a localized userMessage (see [Translations](#translations) for more info):

```
Accept-Language: ru-UA
```

In case of an error response, the client gets:

```
HTTP/1.1 422 Unprocessable Entity
```


```json
{
	"errors": [
	    {
	        "message": "Delivery point data not passed",
	        "code": "MissingDeliveryPointException",
	        "details": "Exception was thrown from https://api.allegro.pl/points/1 in line 2",
	        "path": "Endpoint.getDeliveries.arg1",
	        "userMessage": "Nie wybrano punktu dla odbioru osobistego."
	    }
    ]
}
```

Generate consistent, structured response bodies concerning errors.

* message - internal error message for a developer,
* code - error code in a string representation; it can be an exception name e.g. `MissingDeliveryPointException`,
* details - more specific message for an internal developer; it will be null in case of a production environment,
* path - information about failed parameters validation; can be null,
* userMessage - localized message (based on `Accept-Language` header) to be presented to an application user (much more general than a message);
this field is mandatory.

A developer can use `Trace-Id` header from a response to determine the exact flow for a given error in micro-services.


