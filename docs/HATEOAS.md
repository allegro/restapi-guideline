# HATEOAS

TL;DR We do not use HATEOAS and do not support it

RESTful design principles specify [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) which roughly states
that interaction with an endpoint should be defined within metadata that comes with
the output representation and not based on out-of-band information.
HATEOAS also says that all the actions available for a given resource, all its state transitions,
plus all the relations between resources must be a complete URL that uniquely defines the resource.

Although the web generally works on HATEOAS type principles (where we go to a website's
front page and follow links based on what we see on the page), **we are not ready for HATEOAS on APIs just yet.**
When browsing a website, decisions on what links will be clicked are made at run time.
**However, with an API, decisions as to what requests will be sent are made when the API integration code is written, not at run time.**
Moreover the entire logic of mobile applications which are the main clients for the API
should be rewritten either by including URIs in domain model objects to be passed to controllers
or by changing the contracts of the controllers to include the URIs as well as objects.
