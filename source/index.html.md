---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='mailto:support@tolemi.com'>Contact us</a>
  - <a href='https://tolemi.com'>Learn more about Tolemi</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Introduction

Welcome to Tolemi's API! You can use our API to look-up your properties metadata. Some common use cases are:

- validate parcel addresses against your assessment data
- hydrate your operational systems with parcel metadata

We have language bindings in Shell. You can view code examples in the dark area to the right.

# Authorization

<!-- > To authorize, use this code: -->

<!-- ```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
``` -->

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here" \
  -H "Authorization: Bearer your-access-token" \
  -H "city-alias: tolemi-provided-city-alias"
```

> Make sure to update `your-access-token` and `tolemi-provided-city-alias`

API requests are authorized using a combination of an access token and city alias.

We expect the access token and city alias to be included in requests to the server in headers that look like this:

`Authorization: Bearer your-access-token` <br/>
`city-alias: tolemi-provided-city-alias`

<aside class="notice">
You must replace <code>your-access-token</code> with your access token from the login endpoint.
</aside>
## Log in / generate access token

```shell
curl "https://cg.tolemi.com/login" \
  --header "city-alias: tolemi-provided-city-alias"
  --data username=tolemi-provided-username \
  --data 'password=tolemi-provided-password'
```

> The above command returns json like this

```json
{
  "id": "1",
  "accessToken": "/AyDFpYWY3Qbcj8zQ0wfi2ZT8vazvsfz2tcoCvta30Wk=",
  "email": "tolemi-hq@tolemi.com",
  "firstName": "Tolemi",
  "lastName": "External API User",
  "title": null,
  "department": null,
  "name": "Tolemi External API User",
  "phoneNumber": null,
  "username": "tolemi-provided-username"
}
```

> Use the provided access token in all following API requests

This endpoint provides an <code>access token</code> for provided valid credentials - username and password.

### HTTP Request

`POST https://cg.tolemi.com/login`

### Form URL encoded parameters

| Parameter | Description                 |
| --------- | --------------------------- |
| username  | Username provided by Tolemi |
| password  | Password provided by Tolemi |

# Search

All search requests are formed using [GraphQL](https://graphql.org/) query language. This allows the developer to configure the request to retrieve only the required fields.

## Search assets (parcels) and corresponding owners

> GraphQL query:

```graphql
query SearchAssets($query: String!) {
  assets(query: $query) {
    id
    parcelId
    commonName
    latitude
    longitude
    addresses {
      id
      fullAddress
      city
      state
      zipCode
      primary
    }
    assetIdentities(status: ACTIVE, type: "OWNER") {
      id
      identity {
        id
        name
        address
        cityName
        stateName
        zip
      }
    }
  }
}
```

> curl request using the above graphql query

```shell
curl --request POST \
  --url http://cg.tolemi.com/q \
  --header 'Authorization: BEARER your-access-token' \
  --header 'Content-Type: application/json' \
  --header 'city-alias: tolemi-provided-city-alias' \
  --data '{"query":"query SearchAssets($query: String!) {\n  assets(query: $query) {\n    id\n\t\tparcelId\n\t\tcommonName\n\t\tlatitude\n\t\tlongitude\n\t\n\t\taddresses {\n\t\t\tid\n\t\t\tfullAddress\n\t\t\tcity\n\t\t\tstate\n\t\t\tzipCode\n\t\t\tprimary\n\t\t}\n\t\tassetIdentities (status: ACTIVE, type: \"OWNER\") {\n\t\t\tid\n\t\t\tidentity {\n\t\t\t\tid\n\t\t\t\tname\n\t\t\t\taddress\n\t\t\t\tcityName\n\t\t\t\tstateName\n\t\t\t\tzip\n\t\t\t}\n\t\t}\n  }\n}\n","variables":{"query":"2403300530003501"}}'
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "assets": [
      {
        "id": "23984008",
        "parcelId": "2403300530003501",
        "commonName": "",
        "latitude": 31.294264999999996,
        "longitude": -92.47244,
        "addresses": [
          {
            "id": "35e277bd-2bb6-42b1-807c-650c8044b972",
            "fullAddress": "3111 ELLIOTT ST",
            "city": "ALEXANDRIA",
            "state": "LA",
            "zipCode": "71301",
            "primary": false
          }
        ],
        "assetIdentities": [
          {
            "id": "12345",
            "identity": {
              "id": "54321",
              "name": "DOE LLC",
              "address": "4704 McCrory Place",
              "cityName": "ALEXANDRIA",
              "stateName": "LA",
              "zip": "71303"
            }
          }
        ]
      }
    ]
  }
}
```

This endpoint retrieves all parcels and corresponding requested data given a query string.

### HTTP Request

`POST http://cg.tolemi.com/q`

### Request body (json)

| Parameter | Default     | Description                         |
| --------- | ----------- | ----------------------------------- |
| query     | -           | A search GraphQL query              |
| variables | {query: ""} | Variables used in the GraphQL query |

The <code>query</code> parameter can be either a full, partial address, or a parcel id.

<!-- <aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside> -->

<!-- ## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require("kittn");

let api = kittn.authorize("meowmeowmeow");
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

| Parameter | Description                      |
| --------- | -------------------------------- |
| ID        | The ID of the kitten to retrieve |

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require("kittn");

let api = kittn.authorize("meowmeowmeow");
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted": ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

| Parameter | Description                    |
| --------- | ------------------------------ |
| ID        | The ID of the kitten to delete | -->
