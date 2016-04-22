# REST BEST PRACTICES

### Design
- Base URL
- Versioning
- Resource Format
- Return Values
- Content Negotiation
- References (Linking)
- Pagination
- Query Parameters
- Associations
- Errors
- IDs
- Method Overloading
- Resource Expansion
- Partial Responses
- Caching & Etags
- Security
- Multi Tenancy
- Maintenance

## Why REST?
- Scalability
- Generality
- Independence
- Latency (Caching)
- Security
- Encapsulation

## Why JSON?
- Ubiquity
- Simplicity
- Readability
- Scalability
- Flexibility

### HATEOAS
- Hypermedia As The Engine Of Application State

## Rest Design
- Nouns, not Verbs
- Coarse Grained, not Fine Grained
- Architectural style for use-case scalability

Only two types of resources
- Collection Resource
```
/applications
```
- Instance Resource
```
/applications/a1b2c3
```

#### PUT for Create
- when Identifier is known by the client:
- Full Replacement
```
PUT /applications/clientSpecifiedId
```
```js
{
  "name": "Best App Ever",
  "description": "Awesomeness"
}
200 OK
```

#### POST as Create
- On a parent resource
```
POST /applications
{
  "name": "Best App Ever"
}
201 Created
Location: https://api.company.com/applications/a1b2c3
```
- partial update of the object
```
POST /applications/a1b2c3
{
  "name": "Best App Ever. Really"
}

200 OK
```

Use
https://api.company.com/v1
- camelCase property names
``` account.givenName ```
- Date/Timestamp
Use ISO 8601
Use UTC!
``` "createdTimestamp": "2016-04-21T18:02:24.343Z" ```

### Linking
- Distributed Hypermedia is paramount!
- Every accessible Resource has a canonical unique URL
- Linking is fundamental to scalability

Reference Expansions (Entity/Link Expansion)

```
GET /accounts/x7y8z9
200 OK
{
  "meta": { "href": "https://api.company.com/v1/accounts/x7y8z9" },
  "givenName": "Tony",
  "surname": "Stark",
  ...
  "directory": {
    "meta": { "href": "https://api.company.com/v1/accounts/x7y8z9/directories/g4h5i6}"
  }
}
```
Account and its Directory
```
GET /accounts/x7y8z9?expand=directory
200 OK
{
  "href": "https://api.company.com/v1/accounts/x7y8z9",
  "givenName": "Tony",
  "surname": "Stark",
  ...
  "directory": {
    "href": "https://api.company.com/v1/accounts/x7y8z9/directories/g4h5i6"
    "name": "Avengers",
    "creationDate": "2016-04-21T18:02:24.343Z",
    ...
  }
}
```
Partial Representation (entity minimalization)
```
GET /accounts/x7y8z9?fields=givenName,surname,directory(name)
```
Pagination params
```
GET /accounts/x7y8z9/groups?offset=0&limit=25

200 OK
{
  "href": ".../accounts/x7y8z9/groups",
  "offset": 0,
  "limit": 25,
  "first": { "href": ".../accounts/x7y8z9/groups?offset=0" },
  "previous": null,
  "next": { "href": ".../accounts/x7y8z9/groups?offset=25" },
  "last": { "href": "..."},
  "items": [
    {
      "href": "..."
    },
    {
      "href": "..."
    }
  ]

}
```

Many To Many
- A group can have many accounts
- An account can be in many groups
- Each mapping is a resource
```
GET /groupMemberships/23lk3j2j3

200 OK
{
  "meta": { "href": ".../groupMemberships/23lk3j2j3" },
  "account": {
    "meta": { "href": "..." }
  },
  "group": {
    "meta": { "href": "..." }
  }
}
```

Provide Many To Many on an object
```
GET /accounts/x7y8z9
200 OK
{
  "meta": { "href": ".../accounts/x7y8z9" },
  "givenName": "Tony",
  "surname": "Stark",
  ...,
  "groups": {
    "meta": { "href": ".../accounts/x7y8z9/groups" }
  },
  "groupMemberships": {
    "meta": { "href": ".../groupMemberships?accountId=x7y8z9" }
  }
}
```

## Errors
- As descriptive as possible
- As much information as possible
- Developers are your customers
(twilio established this pattern)

```
POST /directories

409 Conflict
{
  "status": 409,
  "code": 40924,
  "property": "name",
  "message": "A Directory named 'Avengers' already exists.",
  "developerMessage": "A directory named 'Avengers' already exists. If you have a stale local cache, please expire it now.",
  "moreInfo": "https://www.company.com/docs/api/errors/40924"
}
```

## IDS
- Should be globally unique
- Avoid sequential numbers(contention)
- Good candidates: UUIDs, 'Url64'


### References
https://stormpath.com/blog/designing-rest-json-apis/
