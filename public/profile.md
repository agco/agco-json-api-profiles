#JSON API Search Profile
** Linked Resource Filters and Aggregations **

The data model below represents a (simplified) view of a Dealer API domain and will be used throughout the document to further explain the concepts in this profile

![Dealer data model](https://raw.githubusercontent.com/agco-adm/json-api-search-profile/master/public/search-example-dealer-api.png)

## Linked Resource Filters
The JSON API spec standardises filtering on the [primary resource](http://jsonapi.org/format/#fetching-filtering). 
``` 
/dealers?zip=10007 
```
This section of the profile standardizes applying filters on attributes of [linked resources](http://jsonapi.org/format/#document-structure-resource-relationships)

### Examples

- Filter a linked resource   
  ```
  # Fetch all dealers located in the state of NY  
  /dealers/search?state_province.code=US-NY
  ```

- Combine with a primary resource filter   
  ```
  # Fetch all dealers with a zip code of 10005 and an 'MFP' contract code
  /dealers/search?zip=10005&current_contracts.code=MFP
  ```
  
- Filter a linked resource 2 levels deep : current_contracts -> brand  
  ``` 
  # Fetch all dealers which have a contractual agreement to sell 'MF'   equipment  
  /dealers/search?zip=10005&current_contracts.brand.code=MF
  ```
  
- Filter multiple linked resources in one go  
  ```
  # Fetch all dealers located in the state of NY and have a contract agreement to sell 'MF' equipment  
  /dealers/search?state_province.code=US-NY&current_contracts.brand.code=MF
  ```
  

## Aggregations
This section standardises aggregating data from primary and/or linked resources. 

- Get count of dealers per zip code 
```
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&zip_agg.attribute=zip
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
The URL query parameter 'aggregations' denotes the aggregation.  
It can contain a single or comma seperated list of values, which are essentially labels. 

Each label has an attribute which indicates the 'type' of aggregation. Depending on the value provided (e.g. terms, stats, top_hits) other attributes can be set.

The 'aggregations' output is inserted into the 'meta' portion of the response, the keys ( e.g. zip_agg ) correlate with the labels set as part of the request URL.

### Aggregation families
The various aggregation types can be divided into 2 families
#### Metrics
These aggregations compute metrics over a set of documents

- Compute statistics (min/max/avg/sum/count) on the number_of_employees 
```
/dealers/search?aggregations=emp_stats_agg&emp_stats_agg.type=stats&emp_stats_agg.field=dealer_misc.number_of_employees
```
```javascript
"meta": {
    "aggregations": {
      "emp_stats_ag": {
        "count": 975,
        "min": 94,
        "max": 563,
        "avg": 400.4420512820513,
        "sum": 390431
      }
    }
  }
//...
```

#### Buckets
The terms aggregation is a bucketing aggregation.  

When the aggregation is executed the specified 'attribute' value is evaluated to whether it matches the bucket criteria. If a match, the document is placed inside the bucket and the aggregation continues. 

##### Nesting
```
/dealers/search?aggregations=zip_agg&...&zip_agg.aggregations=emp_stats_agg...
```
Each label may specify an additional .aggregations attribute, this may contain a single or comma seperated list of additional nested aggregation labels. 

###### Metrics - Buckets
Metrics aggregations can be nested within Bucket aggregations in order to have them executed in the context of that bucket. 

- Compute statistics (min/max/avg/sum/count) on the number_of_employees per zip code
```
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&&zip_agg.attribute=zip&zip_agg.aggregations=emp_stats_agg&emp_stats_agg.type=stats&emp_stats_agg.attribute=dealer_misc.number_of_employees
```
```javascript
//...
"meta": {
    "aggregations": {
        "zip_agg": [
            {
                "key": "10005",
                "count": 573,
                "emp_stats_agg": {
                    "count": 573,
                    "min": 94,
                    "max": 563,
                    "avg": 154,9738219895288,
                    "sum": 88800
                }
            },
            {
                "key": "10010",
                "count": 299,
                "emp_stats_agg": {
                    "count": 299,
                    "min": 60,
                    "max": 221,
                    "avg": 134,55518394648829,
                    "sum": 40232
                }
            }
            //... more results
        ],
    }
  }
//...
```
- Get 5 most recent founded dealerships per zip code
```
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&&zip_agg.attribute=zip&zip_agg.aggregations=mostrecent_agg&mostrecent_agg.type=top_hits&mostrecent_agg.sort=-dealer_misc.founded_date&&mostrecent_agg.size=5
```
```javascript
//...
"meta": {
    "aggregations": {
        "zip_agg": [
            {
                "key": "10005",
                "count": 573,
                "mostrecent_agg": [
                {
                        "id": "546e0bc761b40f0200b48100",
                        "code": "328304"
                        // ...
                        "links":{
                            //...
                        }
                    },
                    {
                        "id": "5466de4061b40f0200b77889",
                        "code": "327400"
                        // ...
                        "links":{
                            //...
                        }
                    }
                    // ...
                ]
            }
            //... more results
        ]
    }
  }
//...
```


###### Buckets - Buckets
Bucket aggregations can also be nested whithin other Bucket aggregations.

- Get count of dealers per brand / product_type / zip
```
/dealers/search?aggregations=brand_agg&brand_agg.type=terms&brand_agg.attribute=current_contracts.brand.code&brand_agg.aggregations=product_type_agg&product_type_agg.type=terms&product_type_agg.attribute=current_contracts.product_type.code&product_type_agg.aggregations=zip_agg&zip_agg.type=terms&zip_agg.attribute=zip
```
```javascript  
// ...
"meta": {
    "aggregations": {
        "brand_agg": [
            {
                "key": "MF",
                "count": 256,
                "product_type_agg": [
                    {
                        "key": "PAS",
                        "count": 307,
                        "zip_agg": [
                            {
                                "key": "14530",
                                "count": 1
                            },
                            {
                                "key": "17268",
                                "count": 1
                            }
                            // ... more results
                        ]
                    },
                    {
                        "key": "HAY",
                        "count": 105,
                        "zip_agg": [
                            {
                                "key": "14530",
                                "count": 1
                            },
                            {
                                "key": "17268",
                                "count": 1
                            }
                            // more results
                        ]
                    }
                ]
            }
        ]
    }
}
```
### Interop 

The aggregation features may be combined with primary or [linked resource filters](#linked-resource-filters).
```
/dealers/search?current_contracts.brand.code=MF&aggregations=...
```
[inclusion](http://jsonapi.org/format/#fetching-includes) and  [sparse fieldsets](http://jsonapi.org/format/#fetching-sparse-fieldsets) can be applied as well on top of the top_hits aggregation.

```
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&&zip_agg.attribute=zip&zip_agg.aggregations=mostrecent_agg&mostrecent_agg.type=top_hits&mostrecent_agg.sort=-dealer_misc.founded_date&&mostrecent_agg.size=5&mostrecent.include=current_contracts,current_contracts.brand&mostrecent.fields=id,code,name
```

### Aggregation types

                    

