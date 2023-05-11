![](images/NFJust-Text-Blue.png)

# API Docs for Ziti Events/Metrics for current API 1.0 **BETA**

The following document is a work in progress instruction set for accessing network events and/or metrics from your edge routers/endpoints. These endpoints will be moving soon as of 6-28-22, so expect breaking changes to these APIs in the coming months. Future changes will make accessing this data much more approachable for the API user, and will just involve a template name, with a list of available parameters.

The following API endpoints are reference in this document:
1. Utilization
2. Management Events
3. Network events
4. Service Dial Metrics

----------------------------------------------------------------------------------------

* Utilization
The volume of utilization is VERY high, so it is best queried using aggregates that roll up the data. 

Query: Get All Utilization split by direction for the Entire network for the last 24 hours. 10 minute buckets. 

POST
https://gateway.production.netfoundry.io/rest/v1/elastic/ncutilization/<NETWORK_GROUP_ID>/_search/

```csharp
{
  "aggs": {
    "usage_sums": {
      "terms": {
        "field": "usage_type.keyword",
        "size": 2,
        "order": {
          "usage_bytes": "desc"
        }
      },
      "aggs": {
        "usage_bytes": {
          "sum": {
            "field": "usage"
          }
        }
      }
    }
  },
  "size": 0,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h,
              "lte": "now,
              "format": "epoch_millis"
            }
          }
        },
        {
          "match_phrase": {
            "network_id": {
              "query": "<NETWORK_ID>"
            }
          }
        },
        {
          "match_phrase": {
            "organizationId": {
              "query": "<NETWORK_GROUP_ID>"
            }
          }
        },
        {
          "bool": {
            "should": [
              {
                "match_phrase": {
                  "usage_type.keyword": "usage.ingress.tx"
                }
              },
              {
                "match_phrase": {
                  "usage_type.keyword": "usage.egress.tx"
                }
              }
            ],
            "minimum_should_match": 1
          }
        }
      ]
    }
  }
}
```


* Management Events
Query: Get all provisioning activity from the Netfoundry API for the last 24 hours. 

POST /rest/v1/elastic/ncentityevent/<NETWORK_GROUP_ID>/_search/

```csharp
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h",
              "lte": "now",
              "format": "epoch_millis"
            }
          }
        },
        {
          "match_phrase": {
            "organizationId": {
              "query": "82d70e3f-deda-469f-be1a-9c40561ede5d"
            }
          }
        },
        {
          "match_phrase": {
            "networkId": {
              "query": "d3b30838-1dcd-4268-9e7a-4821f58b783e"
            }
          }
        }
      ]
    }
  },
  "size": 100,
  "sort": [
    {
      "@timestamp": {
        "order": "desc",
        "unmapped_type": "boolean"
      }
    }
  ]
}
```


* Network Events

POST /rest/v1/elastic/ncevents/<NETWORK_GROUP_ID>/_search/

```csharp
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "organizationId": {
              "query": "<NETWORK_GROUP_ID>"
            }
          }
        },
        {
          "match_phrase": {
            "network_id": {
              "query": "<NETWORK_ID>"
            }
          }
        },
        {
          "range": {
            "timestamp": {
              "gte": "now-24h",
              "lte": "now",
              "format": "epoch_millis"
            }
          }
        }
      ]
    }
  },
  "size": 500,
  "sort": [
    {
      "@timestamp": {
        "order": "desc",
        "unmapped_type": "boolean"
      }
    }
  ]
}
```

* Service Dial Matrics
Query: Get service dial metrics for a given service, bucketed in hourly intervals

POST /rest/v1/elastic/ncserviceevents/<NETWORK_GROUP_ID>/_search/


```csharp
{
  "aggs": {
    "items": {
      "date_histogram": {
        "field": "timestamp",
        "interval": 3600000,  # size of time bucket in ms 
        "time_zone": "UTC",
        "min_doc_count": 1
      },
      "aggs": {
        "event_types": {
          "terms": {
            "field": "event_type.keyword",
            "size": 10
          },
          "aggs": {
            "sum": {
              "sum": {
                "field": "count"
              }
            }
          }
        }
      }
    }
  },
  "size": 0,
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h",
              "lte": "now",
              "format": "epoch_millis"
            }
          }
        },
        {
          "match_phrase": {
            "network_id": {
              "query": "<NETWORK ID>"
            }
          }
        },
        {
          "match_phrase": {
            "organizationId": {
              "query": "<NETWORK GROUP ID>"
            }
          }
        },
        {
          "match_phrase": {
            "nf_service_id": {
              "query": "<SERVICE ID>"
            }
          }
        }
      ]
    }
  }
}
```
