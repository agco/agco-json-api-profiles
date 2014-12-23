#JSON API Search Profile
** Linked Resource Filters and Multi-Level Aggregations **

The data model below represents a (simplified) view of a Dealer API domain and will be used throughout the document to further explain the concepts in this profile

![Dealer data model](https://raw.githubusercontent.com/agco-adm/json-api-search-profile/master/public/search-example-dealer-api.png)

## Linked Resource Filters
The JSON API spec standardises filtering on the [primary resource](http://jsonapi.org/format/#fetching-filtering). 
``` 
/dealers?zip=10007 
```
This profile standardizes applying filters on attributes of [linked resources](http://jsonapi.org/format/#document-structure-resource-relationships)

### Examples

- Filter a linked resource   
  ```
  # Fetch all dealers located in the state of NY  
  /dealers?state_province.code=US-NY
  ```

- Combine with a primary resource filter   
  ```
  # Fetch all dealers with a zip code of 10005 and an 'MFP' contract code
  /dealers?zip=10005&current_contracts.code=MFP
  ```
  
- Filter a linked resource 2 levels deep : current_contracts -> brand  
  ``` 
  # Fetch all dealers which have a contractual agreement to sell 'MF'   equipment  
  /dealers?zip=10005&current_contracts.brand.code=MF
  ```
  
- Filter multiple linked resources in one go  
  ```
  # Fetch all dealers located in the state of NY and have a contract agreement to sell 'MF' equipment  
  http://example.com/dealers?state_province.code=US-NY&current_contracts.brand.code=MF
  ```
  

## Multi-Level Aggregations
This section standardises aggregating primary or linked resources data. 

### Quick Example 
```
/dealers?aggregations=zip_agg&zip_agg.type=terms&zip_agg.field=zip
```

```javascript
//...
"meta": {
    "aggregations": {
      "zip_agg": [
        {
          "key": "10005",
          "count": 573
        },
        {
          "key": "10010",
          "count": 299
        }
        //... more results
      ]
    }
  }
//... 
```


The aggregation features may be combined with primary or [linked resource filters](#linked-resource-filters), and is also fully interoperable with JSON API  [inclusion](http://jsonapi.org/format/#fetching-includes) and  [sparse fieldsets](http://jsonapi.org/format/#fetching-sparse-fieldsets) features. 
                    

