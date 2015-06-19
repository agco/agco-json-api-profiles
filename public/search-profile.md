# AGCO JSON-API Search Profile

The search profile extends JSON-API with search engine features.

## Search verb

The resource suffix '/search', routes requests to the search engine rather than the primary source of record.

```
/dealers/search
```

The search data has the same shape as the base resource, supports the same json-api features
(inclusions, sparse fields...) and the [Filtering](./filtering-profile.md) profile

The main difference is that search engines offer a huge performance increase when performing queries across multiple resources

Modern search engines such as Elasticsearch and SOLR also have advanced analytics / aggregation capabilities,
the section below describes how these features map can be mapped into the JSON-API spec.

## Aggregations

Below is a quick example which showcases an aggregation

```
# Get count of dealers per zip code 
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&zip_agg.property=zip
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
It contains a single or comma separated list of values which label the aggregations. 

Each label has an attribute which indicates the 'type' of aggregation. Depending on the value provided (e.g. terms, stats, top_hits) other attributes can be set.

The aggregations output is inserted into the 'meta' portion of the response, the keys ( e.g. zip_agg ) correlate with the labels set as part of the request URL.

## Aggregation Categories
The various aggregation types can be divided into 2 categories

### Metrics
These aggregations return value(s) derived from the documents returned by the your query.

```
# Compute statistics (min/max/avg/sum/count) on the number_of_employees 
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

### Buckets

Bucket aggregations define criteria for ‘buckets’ (think of them as ‘groups’) and documents 'fall' into relevant buckets. A bucket therefore contains a document set

The example ( Get count of dealers per zip code ) at the start of the [Aggregations section](#aggregations) uses a terms aggregation, which is a common type of bucket aggregation. 

#### Nesting
```
/dealers/search?aggregations=zip_agg&...&zip_agg.aggregations=emp_stats_agg...
```
Each label referring to a a bucket aggregation may specify an additional .aggregations attribute, this may contain a single or comma separated list of additional nested aggregation labels. 

##### Metrics - Buckets
Metrics aggregations can be nested within Bucket aggregations, this makes them execute in the context of that bucket. 

```
# Compute statistics (min/max/avg/sum/count) on the number_of_employees per zip code
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&&zip_agg.property=zip&zip_agg.aggregations=emp_stats_agg&emp_stats_agg.type=stats&emp_stats_agg.property=dealer_misc.number_of_employees
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

```
# Get 5 most recent founded dealerships per zip code
/dealers/search?aggregations=zip_agg&zip_agg.type=terms&&zip_agg.property=zip&zip_agg.aggregations=mostrecent_agg&mostrecent_agg.type=top_hits&mostrecent_agg.sort=-dealer_misc.founded_date&&mostrecent_agg.size=5
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
##### Buckets - Buckets
Bucket aggregations can also be nested within other Bucket aggregations.

```
# Get count of dealers per brand / product_type / zip
/dealers/search?aggregations=brand_agg&brand_agg.type=terms&brand_agg.property=current_contracts.brand.code&brand_agg.aggregations=product_type_agg&product_type_agg.type=terms&product_type_agg.property=current_contracts.product_type.code&product_type_agg.aggregations=zip_agg&zip_agg.type=terms&zip_agg.property=zip
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

## Features Interop

The aggregation features may be combined with simple or linked resource Filtering
```
/dealers/search?filter.current_contracts.brand.code=MF&aggregations=...
```
[inclusion](http://jsonapi.org/format/#fetching-includes) and  [sparse fieldsets](http://jsonapi.org/format/#fetching-sparse-fieldsets) can be applied as well on top of the top_hits aggregation.

Here is an elaboration of a previous example  ( Get 5 most recent founded dealerships per zip code ) :
```
...&mostrecent.include=current_contracts,current_contracts.brand&mostrecent.fields=id,code,name
```

## Aggregation types

The Aggregation profile is heavily inspired by Elasticsearch Aggregations and borrows a lot of the concepts.

A good place to start for some more reference on the various aggregation types and parameters which can be used is
the [Elasticsearch Aggregations documentation](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current-aggregations.html)

- Metrics :   
  min, max, sum, avg, stats, extended_stats, percentiles, percentile_ranks, top_hits, cardinality, geo_bounds

- Bucketing :
  terms, significant_terms, range, date_range, filter, filters, missing, histogram, date_histogram, geo_distance


###  Elasticsearch vs Aggregation Profile

Some of the paremeters are renamed to remain consistent with the rest of the JSONAPI spec.

Most Elastisearch aggregations require a 'field' parameter (e.g. terms, range,...), this is expected as a 'property' attribute on the aggregation label.
```
?...zip_agg.property=zip
```
The Elasticsearch top_hits aggregation accepts an 'include' parameter, this is expected as a 'fields' attribute on the aggregation label.
```
?...mostrecent_agg.fields=id,code,name
```
The Elasticsearch top_hits aggregation also accepts a 'size' parameter, this is expected as a 'limit' attribute on the aggregation label.
```
?...mostrecent_agg.limit=1
```

## Elasticsearch lock-in

Even though the Aggregation profile GET syntax maps very close to the Elasicsearch syntax, there is no direct coupling as such

It should be possible to remap the syntax to another bit of middleware if we would opt to replace Elasticsearch at some point (e.g. SOLR facets / pivots ).


