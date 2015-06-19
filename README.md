# AGCO JSON API Profiles ( JSONAPI 1.0 )

## Profile

AGCO JSON API [profiles](http://jsonapi.org/extending/), standardises [Filtering](./public/filtering-profile.md), 
[Search](./public/search-profile.md) and [Change Events](./public/change-events-profile.md)..

### Usage

Add the profile link relations in the 'meta' section of the response to allow the API consumer to learn about the additional semantics
```
GET http://api.example.com/dealers
```
``` javascript
{
  "meta": {
    "profile": [
        "https://github.com/agco-adm/json-api-search-profile/blob/master/public/filtering-profile.md",
        "https://github.com/agco-adm/json-api-search-profile/blob/master/public/aggregation-profile.md",
        "https://github.com/agco-adm/json-api-search-profile/blob/master/public/change-events-profile.md"
    ]
  },

  "dealers": [{
    // ...
  }]
}
```

## Stack

- Nodejs :
[harvester](https://github.com/agco/harvesterjs)
[elastic-harvester](https://github.com/agco/elastic-harvesterjs)








