# AGCO JSON-API Change Events Profile

This json api profile describes how Change Events can be accessed for a given resource

Let's assume the following data is inserted at the /equipment resource :

```javascript
{
    "data": [
        {
            "type": "equipment",
            "id": "54ef5abec52a0203000bef23",
            "attributes": {
                "description": "EMS-wb226a 2015-02-26T11:41:17-06:00",
                "serialNumber": "wb226a-00001",
                "vin": "15099-5493",
                "warranty": "0058959619",
                "year": "2015",
            },
            "relationships": {
                "product": {
                    "data": { "type": "products", "id": "54ef5abd127e4c030093aacc" }
                },
                "tracker": {
                    "data": { "type": "trackers", "id": "54ef5abe2fe70f0300727643" }
                }
            }
        },
        {
            "type": "equipment",
            "id": "54ed5a1dfa8319030031ea87",
            "attributes": {
                "description": "Topcon End-To-End Test (5494) 2015-02-24T23:14:06-06:00",
                "serialNumber": "1223-09000226",
                "vin": "15099-5494",
                "warranty": "0086274666",
                "year": "2015",
            },
            "relationships": {
                "product": {
                    "data": { "type": "products", "id": "54ed5a1d0a7f2c0300da74af" }
                },
                "tracker": {
                    "data": { "type": "trackers", "id": "54ed5a1dfa8319030031ea88" }
                }
            }
        }
    ]
}
```

## Request changes

Change events can be retrieved with a simple REST call using the /changes suffix

```
/equipment/changes
```
```javascript
{
    "data": [
        {
            "type": "equipment_insert",
            "id": "6113607438701690883",
            "attributes": {
                "timestamp": "2013-03-31T14:32:21Z",
                "new": {
                    "type": "equipment",
                    "id": "54ef5abec52a0203000bef23",
                     "attributes": {
                        "description": "EMS-wb226a 2015-02-26T11:41:17-06:00",
                        "serialNumber": "wb226a-00001",
                        "vin": "15099-5493",
                        "warranty": "0058959619",
                        "year": "2015",
                    },
                    "relationships": {
                        "product": {
                            "data": { "type": "products", "id": "54ef5abd127e4c030093aacc" }
                        },
                        "tracker": {
                            "data": { "type": "trackers", "id": "54ef5abe2fe70f0300727643" }
                        }
                    }       
                }
            }
        },
        {
            "type": "equipment_insert",
            "id": "6113607438701690884",
            "attributes": {
                "timestamp": "2013-03-31T14:32:22Z",
                "new": {
                    "type": "equipment",
                    "id": "54ed5a1dfa8319030031ea87",
                     "attributes": {
                        "description": "Topcon End-To-End Test (5494) 2015-02-24T23:14:06-06:00",
                        "serialNumber": "1223-09000226",
                        "vin": "15099-5494",
                        "warranty": "0086274666",
                        "year": "2015",
                    },
                    "relationships": {
                        "product": {
                            "data": { "type": "products", "id": "54ed5a1d0a7f2c0300da74af" }
                        },
                        "tracker": {
                            "data": { "type": "trackers", "id": "54ed5a1dfa8319030031ea88" }
                        }
                    }
                }
            }
        }
    ]
}
```

### Paging

Paging may be used when requesting changes

```
/equipment/changes?sort=-timestamp&limit=1
```

## Stream changes

In order to reduce network overhead and associated latency, the change events can be streamed across as well

This part of the spec builds on [server sent events](http://www.w3.org/TR/2009/WD-eventsource-20091029/),
which standardises event streaming on top of plain HTTP/1.1

The fact that it's leveraging pure HTTP makes it more proxy friendly than websockets,
in addition to that it also supports automatic reconnection and transmission of arbitrary events


As demonstrated below, it's sufficient to append a /stream suffix to get back a stream of data

```
/equipment/changes/stream
```
```
"event": "equipment_insert",
"id": "6113607438701690883",
"data": {"type": "equipment_insert","id": "6113607438701690883","attributes": {"timestamp": "2013-03-31T14:32:21Z","new": {"type": "equipment","id": "54ef5abec52a0203000bef23","attributes": {"description": "EMS-wb226a 2015-02-26T11:41:17-06:00","serialNumber": "wb226a-00001","vin": "15099-5493","warranty": "0058959619","year": "2015"},"relationships": {"product": {"data": {"type": "products","id": "54ef5abd127e4c030093aacc"}},"tracker": {"data": {"type": "trackers","id": "54ef5abe2fe70f0300727643"}}}}}}

"event": "equipment_insert",
"id": "6113607438701690884",
"data": {"type": "equipment_insert","id": "6113607438701690884","attributes": {"timestamp": "2013-03-31T14:32:22Z","new": {"type": "equipment","id": "54ed5a1dfa8319030031ea87","attributes": {"description": "Topcon End-To-End Test (5494) 2015-02-24T23:14:06-06:00","serialNumber": "wb226a-00001","vin": "15099-5494","warranty": "0086274666","year": "2015"},"relationships": {"product": {"data": {"type": "products","id": "54ed5a1d0a7f2c0300da74af"}},"tracker": {"data": {"type": "trackers","id": "54ed5a1dfa8319030031ea88"}}}}}}
```

The [Last Event Id mechanism](http://www.w3.org/TR/2009/WD-eventsource-20091029/#last-event-id) should be supported
This enables resilience on connection breaks and initiating a stream from an arbitrary point in time


### Consume SSE

Out-of-the-box server libraries are available on various platforms : e.g. [Node](https://github.com/aslakhellesoy/eventsource-node),
[Java](https://github.com/aslakhellesoy/eventsource-java), ...

On the browser side EventSource is a standard HTML5 element, with [polyfills](http://html5please.com/#eventSource) available for dated browsers



## Filtering

Simple Filtering may be applied to change events, either when requesting or streaming change events

```
/canAlerts/changes/stream?filter.equipment=...
```



