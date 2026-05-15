# CDC2026 Elasticsearch를 활용한 Private AI system 구축하기
Resource share for the Elasticsearch HandsOn session in CDC2026

### Speaker
Insuk Cho (https://insukcho.github.io/)

### Handson Scope
<img width="1317" height="757" alt="Screenshot 2026-05-13 at 9 56 45 PM" src="https://github.com/user-attachments/assets/e98de79b-1f10-42a0-8c6e-825eb097bf0c" />


### Elasticsearch 설치하기

- Option 1. 클라우드 서비스 트라이얼 계정 생성 및 서버리스 프로젝트 생성 (추천)
  - https://cloud.elastic.co/
- Option 2. Download and install
  - Elasticsearch: https://www.elastic.co/downloads/elasticsearch
  - Kibana: https://www.elastic.co/downloads/kibana

### 실습 데이터 업로드 (Kibana > Data Visualizer)
https://github.com/insukcho/CDC2026/blob/main/parks.csv

### checking the raw index mapping info

```
GET parks_raw/_mapping
```

### Create a new index with adding `simentic_text` field type and `copy_to` the field

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
          "type": "semantic_text"
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
GET _tasks/<TASK-ID>
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
  "size": 5,
  "query": {
    "match": {
      "description": "Which park is the best for camping and swimming with kids?"
    }
  },
  "fields": [
    "park_code",
    "name",
    "description"
  ],
  "_source": false
}
```

### Simantic search

```
GET parks_new/_search
{
  "size": 5,
  "query": {
    "semantic": {
      "field": "description_semantic",
      "query": "Which park is the best for camping and swimming with kids?"
    }
  },
  "fields": [
    "park_code",
    "name",
    "description"
  ],
  "_source": false
}
```

### Hybrid search

```
GET parks_new/_search
{
  "size": 5,
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
  },
  "fields": ["park_code", "name", "description"],
  "_source": false
}
```

### Hybrid search with ESQL

```
POST /_query?format=txt
{
  "query": """
    FROM parks_new METADATA _score
    | WHERE description: "Which park is the best for camping and swimming with kids?" OR match(description_semantic, "Which park is the best for camping and swimming with kids?", { "boost": 0.75 })
    | SORT _score DESC
    | KEEP park_code, name, description
    | LIMIT 5
  """
}
```

### References

- https://www.elastic.co/docs/solutions/search/hybrid-semantic-text


