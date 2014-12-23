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
#### Quick Example
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
### Syntax
The URL query parameter 'aggregations' denotes the aggregation.  
It can contain a single or comma seperated list of values, which are essentially labels. 

Each label has an attribute which indicates the 'type' of aggregation. Depending on the value provided (e.g. terms, stats, top_hits) other attributes can be set.

The 'aggregations' output is inserted into the 'meta' portion of the response, the keys ( e.g. zip_agg ) correlate with the labels set as part of the request URL.

### Buckets and Metrics
The various aggregation types can be divided into 2 categories
#### Metrics
These aggregations compute metrics over a set of documents

```
/dealers/search?aggregations=emp_stats_agg&emp_stats_agg.type=stats&emp_stats_agg.field=links.dealer_misc.number_of_employees
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
The 'Quick Example' terms aggregation is a bucketing aggregation.  

When the aggregation is executed the specified 'attribute' value is evaluated whether it matches the bucket criteria. If a match, the document is placed inside the bucket and the aggregation continues. 

#### Nesting - Metrics within Buckets
Metrics aggregations can be nested whithin Bucket aggregations so they execute within the context of that bucket.   
```
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&&zip_agg.attribute=zip&zip_agg.aggregations=emp_stats_agg&emp_stats_agg.type=stats&emp_stats_agg.attribute=links.dealer_misc.number_of_employees
```
```javascript
//...
"meta": {
    "aggregations": {
        "zip_agg": [
            {
                "key": "10005",
                "count": 573,
                "emp_stats_ag": {
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
                "emp_stats_ag": {
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
#### Nesting - Buckets within Buckets
Bucket aggregations can also be nested whithin other Bucket aggregations.

- Group dealers by brand, product_type, zip   
  ```
  /dealers/search?aggregations=brand_agg&brand_agg.type=terms&brand_agg.attribute=current_contracts.brand.code&brand_agg.aggregations=product_type_agg&product_type_agg.type=terms&product_type_agg.attribute=current_contracts.product_type.code&product_type_agg.aggregations=zip_agg&zip_agg.type=terms&zip_agg.attribute=zip
  ```

##### Syntax
Each label may specify an additional .aggregations attribute, this may contain a single or comma seperated list of additional aggregation labels. In turn each label specifies the 'type' of aggregation   

The aggregation features may be combined with primary or [linked resource filters](#linked-resource-filters)


, and is also fully interoperable with JSON API  [inclusion](http://jsonapi.org/format/#fetching-includes) and  [sparse fieldsets](http://jsonapi.org/format/#fetching-sparse-fieldsets) features. 
                    

