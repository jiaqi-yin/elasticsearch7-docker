## Import a single document

```
curl --request PUT 'localhost:9200/movies' \
--header 'Content-Type: application/json' \
--data-raw '{
	"mappings": {
		"properties": {
			"year": {
				"type": "date"
			}
		}
	}
}'

```

```
curl --request GET 'localhost:9200/movies/_mapping' \
--header 'Content-Type: application/json'
```

```
curl --request POST 'localhost:9200/movies/_doc/109487' \
--header 'Content-Type: application/json' \
--data-raw '{
	"genre": ["IMAX", "Sci-Fi"],
	"title": "Interstellar",
	"year": 2014
}'
```

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json'
```

## Import many document - the bulk API

```
curl --request PUT 'localhost:9200/_bulk' \
--header 'Content-Type: application/json' \
--data-binary @movies.json
```

## Update documents

### Partial update

```
curl --request POST 'localhost:9200/movies/_doc/109487/_update' \
--header 'Content-Type: application/json' \
--data-raw '{
	"doc": {
		"title": "Interstellar"
	}
}'
```

### Whole update

```
curl --request PUT 'localhost:9200/movies/_doc/109487' \
--header 'Content-Type: application/json' \
--data-raw '{
	"genre": ["IMAX", "Sci-Fi"],
	"title": "Interstellar",
	"year": 2014
}'
```

## Delete documents

```
curl --request DELETE 'localhost:9200/movies/_doc/109487' \
--header 'Content-Type: application/json'
```

## Analyzers and Tokenizers

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"match": {
			"title": "star trek"
		}
	}
}'
```

```
curl --request DELETE 'localhost:9200/movies' \
--header 'Content-Type: application/json'
```

```
curl --request PUT 'localhost:9200/movies' \
--header 'Content-Type: application/json' \
--data-raw '{
	"mappings": {
		"properties": {
		    "id": {"type": "integer"},
			"year": {"type": "date"},
			"genre": {"type": "keyword"},
			"title": {"type": "text", "analyzer": "english"}
		}
	}
}'
```

```
curl --request PUT 'localhost:9200/_bulk' \
--header 'Content-Type: application/json' \
--data-binary @movies.json
```

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"match_phrase": {
			"genre": "Sci-Fi"
		}
	}
}'
```

## Data modeling and parent/child relationships

```
curl --request PUT 'localhost:9200/series' \
--header 'Content-Type: application/json' \
--data-raw '{
	"mappings": {
		"properties": {
			"film_to_franchise": {
				"type": "join",
				"relations": {"franchise": "film"}
			}
		}
	}
}'
```

```
curl --request PUT 'localhost:9200/_bulk' \
--header 'Content-Type: application/json' \
--data-binary @series.json
```

```
curl --request GET 'localhost:9200/series/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"has_parent": {
			"parent_type": "franchise",
			"query": {
				"match": {
					"title": "Star Wars"
				}
			}
		}
	}
}'
```

```
curl --request GET 'localhost:9200/series/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"has_child": {
			"type": "film",
			"query": {
				"match": {
					"title": "The Force Awakens"
				}
			}
		}
	}
}'
```

## Query lite - URI SEARCH

```
curl --request GET 'localhost:9200/movies/_search?q=title:star' \
--header 'Content-Type: application/json'
```

```
curl --request GET 'localhost:9200/movies/_search?q=+year>2010+title:trek' \
--header 'Content-Type: application/json'
```

## Json search

### Request body search

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"match": {
			"title": "star"
		}
	}
}'
```

### Boolean query with a filter

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"bool": {
			"must": {"term": {"title": "trek"}},
			"filter": {"range": {"year": {"gte": 2010}}}
		}
	}
}'
```

### Some types of filters

term: filter by exact values

`{"term": {"year": 2014}}`

terms: match if any exact values in a list match

`{"terms": {"genre": ["Sci-Fi", "Adventure"]}}`

range: find numbers or dates in a given range (gt, gte, lt, lte)

`{"range": {"year": {"gte": 2010}}}`

exists: find documents where a field exists

`{"exists": {"field": "tags"}}`

missing: find documents where a field is missing

`{"missing": {"field": "tags"}}`

bool: combine filters with Boolean logic (must, must-not, should)

### Some types of queries

match_all: returns all documents and is the default. Normally used with a filter.

`{"match_all": {}}`

match: searches analyzed results, such as full text search.

`{"match": {"title": "star"}}`

multi_match: run the same query on multiple fields.

`{"multi_match": {"query": "star", "fields": ["title", "synopsis"]}}`

bool: works like a bool filter, but results are scored by relevance.

## Phrase matching

must find all terms, in the right order

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"match_phrase": {
			"title": "star wars"
		}
	}
}'
```

### Slop

order matters, but you're ok with some words being in between the terms

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"match_phrase": {
			"title": {"query": "star beyond", "slop": 1}
		}
	}
}'
```

### Proximity queries

just use a really high slop

```
curl --request GET 'localhost:9200/movies/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
	"query": {
		"match_phrase": {
			"title": {"query": "star beyond", "slop": 100}
		}
	}
}'
```
