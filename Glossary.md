# Glossary

* [General](#general)
* [Address](#address)
* [Image](#image)
* [Description](#description)
* [Category](#category)
* [User](#user)
* [Coordinates](#coordinates)
* [Pagination](#pagination)
* [Try to avoid](#try-to-avoid)

The main goal of the glossary is to unify terms used by public resources in order to give a clear understanding of the Allegro REST API
and to make the integration with Allegro smooth. Below terms should be used in resource’s names, models, objects, query parameters, etc.
The glossary is divided into several sections, each of them represents a specific area of Allegro domain or specific technological aspect (e.g. how to represent a user).

The main goal of the glossary is to unify terms of all our public resources in order to give a clear understanding of the Allegro REST API
and to help our clients seamlessly integrate with the Allegro. Below terms should be used in our **resources' names, models, JSON objects, query parameters** etc.
The glossary was divided into several sections, each of them represents some area of Allegro domain or specific technological aspect (e.g. how to represent a user).
You are invited to contribute the glossary, especially when you are responsible for some area in Allegro, just create a pull-request.

## General
* **id** - entity's identifier (i.e. offer, category)
* **name** - used to store name of an entity (i.e. offer, category)
* **dryRun** - should a function run without any modification of data?
* **active** - is the entity active?
* **inactive** - is the entity inactive?
* **version** - version of an entity or a system
* **count** - number of matched entities (e.g. in case of pagination), used mainly for pagination
where it should tell how many entities matched the clients criteria, NOT how many entities were returned on the given page
* **language** - according to language tag (https://tools.ietf.org/html/bcp47, https://en.wikipedia.org/wiki/IETF_language_tag)

## Address
* **address**
* **street**
* **city**
* **province**
* **postCode**
* **countryCode**

## Image
* **image** - keep in mind that we do not use **~~picture~~**
* **url** - image url
* **title** - image title

```json
{
    "image": {
        "url": "",
        "title": ""
    }
}
```

## Description
* **description**
* **summary** - brief text summary
* **text** - full text description

```json
{
    "description": {
        "summary": "",
        "text": ""
    }
}
```

## Category
* **parent** - category parent
* **leaf** - is the category a leaf?
* **tree** - to which category tree it belongs to?

```json
{
    "category": {
        "id": "...",
        "name": "...",
        "leaf": true,
        "tree": {
            "name": "sample-tree"
        }
    }
}
```

## User
* **user** - Allegro user entity
* **password** - user's password
* **firstName**
* **lastName**
* **companyName**
* **sex**
* **phoneNumber**
* **email** - user's email
* **birthDate** - user's birth date
* **confirmation**
* **registration**

## Coordinates
* **coordinates** - map coordinates
* **lat** - latitude
* **lon** - longitude

```json
{
    "coordinates": {
        "lat": 52.33374,
        "lon": 16.808437
    }
}
```

## Pagination
* **offset** - page index (do not use page, pageIndex, etc.)
* **limit** - page size (do not use pageSize, size, length, etc.)

## Try to avoid
* **metadata** - do not use it. Almost all information from this object could be moved to parent entity
* **picture** - use **image** instead
* **page**, **pageIndex**, **pageNo**, **pageNumber** - do not use them; in case of pagination use `offset` instead
