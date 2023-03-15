# Documentation

Each resource method (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`) has to provide brief documentation for itself. There should be a description
of what given resource is responsible for and how a client interprets it. This documentation should contain information on expected input
parameters (`GET`) or body (`POST`, `PUT`, `PATCH`), required and optional parameters, the response a client should
expect and possible error codes. Documentation should be compatible with [swagger spec 2.0](https://github.com/swagger-api/swagger-spec/blob/master/versions/2.0.md)
and should be accessible under a well-known endpoint (e.g. service-host/swagger.json).


