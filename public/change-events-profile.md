# AGCO JSON-API Change Events Profile


```
/canAlerts
```
```javascript
{
    "trackingData": [
        {
            "id": "54cb8f423db5e80300afbe33",
            "raw": 69,
            "rawUnit": "%",
            "imp": 69,
            "met": 69,
            "metUnit": "%",
            "links": {
                "trackingPoint": "53b18b5205904f3e687be511",
                "canVariable": "54b80db395bc95bb604b351b"
            }
        },
        {
            "id": "54cb8f423db5e80300afbe32",
            "raw": 2986,
            "rawUnit": "m/s",
            "imp": 6.65878,
            "impUnit": "f/s",
            "met": 10.7496,
            "metUnit": "m/s",
            "links": {
                "trackingPoint": "53b18b5205904f3e687be510",
                "canVariable": "54b80db395bc95bb604b347d"
            }
        }//,... more trackingData
    ]
}

```

## Poll changes

/canAlerts/changes

```javascript
{
    "trackingData_changes": [
        {
            "event": "trackingData_insert",
            "id": "6113607438701690883",
            "data": {
                "id": "54cb8f423db5e80300afbe33",
                "raw": 69,
                "rawUnit": "%",
                "imp": 69,
                "met": 69,
                "metUnit": "%",
                "links": {
                    "trackingPoint": "53b18b5205904f3e687be511",
                    "canVariable": "54b80db395bc95bb604b351b"
                }
            }
        },
        {
            "event": "trackingData_insert",
            "id": "6113607438701690884",
            "data": {
                "id": "54cb8f423db5e80300afbe32",
                "raw": 2986,
                "rawUnit": "m/s",
                "imp": 6.65878,
                "impUnit": "f/s",
                "met": 10.7496,
                "metUnit": "m/s",
                "links": {
                    "trackingPoint": "53b18b5205904f3e687be510",
                    "canVariable": "54b80db395bc95bb604b347d"
                }
            }
        }//,... more trackingData change events
    ]
}
```

### Paging

```
/canAlerts/changes?seq=ge=6113607438701690884&limit=20
```

```javascript
{
    "trackingData_changes": [
        {
            "event": "trackingData_insert",
            "id": "6113607438701690884",
            "data": {
                "id": "54cb8f423db5e80300afbe32",
                "raw": 2986,
                "rawUnit": "m/s",
                "imp": 6.65878,
                "impUnit": "f/s",
                "met": 10.7496,
                "metUnit": "m/s",
                "links": {
                    "trackingPoint": "53b18b5205904f3e687be510",
                    "canVariable": "54b80db395bc95bb604b347d"
                }
            }
        }//,... more trackingData change events
    ]
}
```



## Stream changes


[server sent events](http://www.w3.org/TR/2009/WD-eventsource-20091029/)
standardises event streaming across plain HTTP/1.1, which makes it more proxy friendly than websockets
and in addition supports automatic reconnection and transmission of arbitrary events


```
/canAlerts/changes/stream
```
```
"event": "trackingData_insert",
"id": "6113607438701690883",
"data": {"id": "54cb8f423db5e80300afbe33","raw": 69,"rawUnit": "%","imp": 69,"met": 69,"metUnit": "%","links": {"trackingPoint": "53b18b5205904f3e687be511","canVariable": "54b80db395bc95bb604b351b"}}

"event": "trackingData_insert",
"id": "6113607438701690884",
"data": {"id": "54cb8f423db5e80300afbe32","raw": 2986,"rawUnit": "m/s","imp": 6.65878,"impUnit": "f/s","met": 10.7496,"metUnit": "m/s","links": {"trackingPoint": "53b18b5205904f3e687be510","canVariable": "54b80db395bc95bb604b347d"}}
```

The [Last Event Id mechanism](http://www.w3.org/TR/2009/WD-eventsource-20091029/#last-event-id) should be supported
in order to enable resilience on connection breaks and initiating a stream from an arbitrary point in time


### Consume SSE

ootb server libraries are available on various platforms : e.g. [Node](https://github.com/aslakhellesoy/eventsource-node),
[Java](https://github.com/aslakhellesoy/eventsource-java)

EventSource is a standard HTML5 element, with [polyfills available](http://html5please.com/#eventSource) for dated browsers



## Filtering

Simple Filtering may be applied to change events, both Poll and Stream methods should be supported

```
/canAlerts/changes/stream?filter.equipment=<uuid>
```



