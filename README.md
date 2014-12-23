# JSON API Search Profile

## Profile

This repository contains a JSON API [extension spec](http://jsonapi.org/extending/),
which standardises search features such as [Linked Resource Filtering and Aggregations](./public/profile.md).

Add a profile link relation in the 'meta' section of the response to allow the API consumer to learn about the additional semantics
```
GET http://api.example.com/dealers/search
```
``` javascript
{
  "meta": {
    "profile": "https://github.com/agco-adm/json-api-search-profile/blob/master/public/profile.md"
  },

  "dealers": [{
    // ...
  }]
}
```

## Stack

- Nodejs : [elastic-fortune](https://github.com/agco-adm/elastic-fortune)








