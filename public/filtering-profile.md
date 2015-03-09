# AGCO JSON-API Aggregations Profile

In terms of filtering the JSON API spec defines a reserved query parameter : filter
This extension spec defines the implementation of that parameter throughout various scenarios

## Simple

```
/equipment?filter.vin=19UYA31581L000000
```

## Regex

```
/equipment?filter.serialNumber=123*
```
In general URIs as defined by RFC 3986 (see Section 2: Characters) may contain any of the following characters:
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~:/?#[]@!$&'()*+,;=.
Any other character needs to be encoded with the percent-encoding (%hh).

## Range

FIQL syntax should be used for range queries :


```
/trackingData?filter.raw=lt=50
/trackingData?filter.raw=ge=50&filter.raw=lt=100

/trackingPoint?filter.alt=le=50
/trackingPoint?filter.alt=gt=50

```


## Linked Resources

Filter on attributes of the [linked resources](http://jsonapi.org/format/#document-structure-resource-relationships)

- Filter a linked resource

  ```
  /trackingData?filter.canVariable.name=ENGINE_LOAD
  ```

- Combine with a primary resource filter   
  ```
  /trackingData?filter.canVariable.name=ENGINE_LOAD&filter.raw=gt=10
  ```
  
- Filter a linked resource 2 levels deep : trackingPoint -> equipment
  ```
  /trackingData?filter.trackingPoint.equipment.vin=19UYA31581L000000
  ```
  
- Filter multiple linked resources in one go  
  ```
  /trackingData?filter.trackingPoint.equipment.vin=19UYA31581L000000&filter.canVariable.name=ENGINE_LOAD&filter.raw=gt=10
  ```
