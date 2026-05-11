# CDC2026-Elasticsearch-HandsOn
Resource share for the Elasticsearch HandsOn session in CDC2026

### checking the raw index mapping info

```
GET parks_raw/_mapping
```

### Create a new index with adding `simentic_search` field

```
PUT parks_new
{
    "mappings": {
      "properties": {
        "country_id": {
          "type": "keyword"
        },
        "description": {
          "type": "text",
          "copy_to": [
            "description_semantic"
          ]
        },
        "description_semantic": {
          "type": "semantic_text",
          "inference_id": ".elser-2-elasticsearch",
          "model_settings": {
            "service": "elasticsearch",
            "task_type": "sparse_embedding"
          }
        },
        "designation": {
          "type": "text"
        },
        "directions_info": {
          "type": "text"
        },
        "directions_url": {
          "type": "keyword"
        },
        "full_name": {
          "type": "text"
        },
        "latitude": {
          "type": "double"
        },
        "location": {
          "type": "geo_point"
        },
        "longitude": {
          "type": "double"
        },
        "name": {
          "type": "text"
        },
        "park_code": {
          "type": "keyword"
        },
        "park_id": {
          "type": "keyword"
        },
        "states": {
          "type": "keyword"
        },
        "url": {
          "type": "keyword"
        },
        "weather_info": {
          "type": "text"
        }
      }
    }    
}
```

### checking the new index mapping info

```
GET parks_new/_mapping
```

### reindexing 

```
POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "parks_raw",
    "size": 10
  },
  "dest": {
    "index": "parks_new"
  }
}
```

### Check reindexing task

```
GET _tasks/MtbdXlP5Qc2I56Yp4XURqA:23109
```

### checking the new index data count

```
GET parks_new/_count
```

### Query text analyze simulation

```
GET parks_new/_analyze
{
  "field": "description",
  "text": "Which park is the best for camping and swimming with kids?"
}
```

### Standard search

```
GET parks_new/_search
{
  "query": {
    "match": {
      "description": "Which park is the best for camping and swimming with kids?"
    }
  }
}
```

### Simantic search

```
GET parks_new/_search
{
  "query": {
    "semantic": {
      "field": "description_semantic",
      "query": "Which park is the best for camping and swimming with kids?"
    }
  }
}
```

### Hybrid search (https://www.elastic.co/docs/solutions/search/hybrid-semantic-text)

```
GET parks_new/_search
{
  "retriever": {
    "rrf": {
      "retrievers": [
        {
          "standard": {
            "query": {
              "match": {
                "description": "Which park is the best for camping and swimming with kids?"
              }
            }
          }
        },
        {
          "standard": {
            "query": {
              "semantic": {
                "field": "description_semantic",
                "query": "Which park is the best for camping and swimming with kids?"
              }
            }
          }
        }
      ]
    }
  }
}
```


