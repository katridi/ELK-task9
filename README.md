# To download docker image from [here](https://opensearch.org/docs/latest/opensearch/install/docker/#run-the-image)
docker-compose up -d


# To ensure everithing is done correctly

curl -XGET https://localhost:9200 -u 'admin:admin' --insecure

curl -XGET https://localhost:9200/_cat/nodes?v -u 'admin:admin' --insecure

curl -XGET https://localhost:9200/_cat/plugins?v -u 'admin:admin' --insecure

# To open OpenSearch Dashboards

http://localhost:5601

# Add data

# Answer questions below

## Data analytics task (Ecommerce data)

- Find top-20 most common first name of the customers from Birmingham

``` console
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        {"term": {
          "geoip.city_name": "Birmingham"
        }}
      ]
    }
  }, 
  "aggs": {
    "user_agg": {
      "terms": {
        "field": "customer_first_name.keyword",
        "size": 20
        }
      }
    }
}
```

- What is the busiest day (most orders) for women buying products cheaper than 75$

``` console
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "query": { 
    "bool": { 
      "filter": [ 
        {"term": { "customer_gender": "FEMALE"}},
        {"range": {"products.price": {"lte" : 75 }}}
      ]
    }
  },
  "aggs": {
    "most_busy_dow": {
      "terms": {
        "field": "day_of_week",
        "size": 1,
        "order": { "_count": "desc" }
        }
      }
    }
}
```


- How many products were bought in the last 3 days from Great Britain?

``` console
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "query": { 
    "bool": { 
      "filter": [ 
        {"term": {"geoip.country_iso_code": "GB"}},
        {"range": {"order_date": {"gte": "now-3d/d","lt": "now/d"}}}
      ]
    }
  },
  "aggs": {"products_sum": { "sum": { "field": "products.quantity" } }
}
}
```


## Data analytics task (Flights data)

- Total distance in miles travelled by all flights

``` console
GET opensearch_dashboards_sample_data_flights/_search
{
  "size": 0,
  "aggs": {"total_distance": { "sum": { "field": "DistanceMiles" } }}
}
```

- Geo distance aggregation from Zurich Airport (or any other if you don't have it) with several 1000km ranges

``` bash
# 47.464699, 8.54917 - is Zurich's long/lat coordinates
GET opensearch_dashboards_sample_data_flights/_search
{
  "aggs": {
    "rings": {
      "geo_distance": {
        "field": "OriginLocation",
        "origin": [47.464699, 8.54917],
        "unit": "km",
        "distance_type": "plane",
        "ranges": [
          { "to": 1000},
          { "from": 1000, "to": 2000 },
          { "from": 2000, "to": 3000 }
        ]
      }
    },
    "rings_dest": {
      "geo_distance": {
        "field": "DestLocation",
        "origin": [47.464699, 8.54917],
        "unit": "km",
        "distance_type": "plane",
        "ranges": [
          { "to": 1000},
          { "from": 1000, "to": 2000 },
          { "from": 2000, "to": 3000 }
        ]
      }
    }
  }
}
```


- Top-10 most delayed destination airports

``` console
GET opensearch_dashboards_sample_data_flights/_search
{
  "size": 0,
  "aggs" : {
    "dest": {
      "terms": {
        "field": "DestAirportID",
        "size": 10,
        "order": { "max_delay_sum": "desc" }
      },
      "aggs": {"max_delay_sum" : { "sum" : { "field" : "FlightDelayMin" } }
    }
  }}
}
```

## Data analytics task (Logs data)

- Top 5 tags in logs in which contains request to deliver css files

``` console
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "query": { 
    "bool": { 
      "filter": [ 
        {"term": { "extension": "css"}}
      ]
    }
  },
  "aggs": {
    "top_tags": {
      "terms": {
        "field": "tags.keyword",
        "size": 5,
        "order": { "_count": "desc" }
      },
      "aggs": {}
    }}
}
```

- What is sum of all RAM for Windows machines that have requests from 6am to 12pm

``` console
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "query": { 
    "bool": { 
      "filter": [ 
        {"prefix": {"machine.os": {"value": "win"}}},
        {
              "script": {
                "script": {
                  "source": "doc['@timestamp'].value.hourOfDay >= params.min && doc['@timestamp'].value.hourOfDay <= params.max",
                  "params": {
                    "min": 6,
                    "max": 24
                  }
                }
              }
            }
      ]
  }},
  "aggs": {
    "total_ram": { "sum": {
      "field": "machine.ram",
      "script": { "source": "_value/(1024)/(1024)" }
    }
    }
  }
}
```

- Find total number of logs with IP in range from 176.0.0.0 to 179.255.255.254

``` console
GET opensearch_dashboards_sample_data_logs/_count
{
  "query": {
    "range": {
      "clientip": {
        "gte": "176.0.0.0",
        "lt": "179.255.255.254"
      }
    }
  }
}
```
