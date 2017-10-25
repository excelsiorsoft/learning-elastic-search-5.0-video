# Learning-elastic-search-5.0-video

My notes while following the following video: https://www.safaribooksonline.com/library/view/learning-elasticsearch-50/9781783984589/


- List all existing indexes: 
curl -XGET http://localhost:9200/_cat/indices?v

- Querying particular index:
curl -XGET http://localhost:9200/schools?pretty

- Create an index:
```
PUT /books
{
	"settings":{
		"number_of_shards":1,
		"number_of_replicas":0
	}
}
```
- Now read what's been created: 
```
GET /books
```


##### \# of shards is immutable
##### \# of replicas can be changed later

- Modifying index's attributes:
```
PUT /books/_settings
{
	"number_of_replicas":1
}
```


- Check cluster's health:
curl -XGET http://localhost:9200/_cluster/health?pretty

##### In Kibana (authors is a type of document): 
```
PUT /books
{
settings:{
	number_of_shards:1
	number_of_replicas:0
},
mappings:{
	authors: {
		properties: {
			name: {
				type: string
			},
			book_count: {
				type: integer
			},
			date: {
				type: date
			}
		}
	}
}

}
```

- No ID assignment:
```
curl -XPOST http://localhost:9200/books/author/- d
{
	name: Michael J.
}
```

- With pre-assigned ID:
```
curl -XPOST http://localhost:9200/books/author/1/_create- d
{
	name: Michael J.
}
```

- Delete by ID:
```
curl -XDELETE http://localhost:9200/books/author/1/_create- d
{
	name: Michael J.
}
```

##### Single-node cluster ends up with an 'yellow' status (warning):

GET /books/_cluster/health

##### Adding document in Kibana dev tools:

- an ID is auto-assigned here: 
```
POST /books/author/
{
	name: Michael
}
```

- Creating with a specific ID:
```
POST /books/author/1/_create
{
	name: Michael
}
```
- Add an additional field:
```
PUT /books/_mapping/author/
{
properties: 
{
	total_books:
	{
		type: integer
	}
}
}
```
- Delete a document: 

DELETE /books/author/1

- Confirm deletion: 

GET /books/author/1

##### Adding multiple documents at a time (Bulk API):
```
curl -POST http://localhost:9200/_bulk -d
{index: {_index: books, _type: author, _id: 1}}
{name: John, isbn: 25874}
{index: {_index: books, _type: author, _id: 2}}
{name: Michael, isbn: 9632471}
```
```
POST /books/_bulk
{index: {_index: books, _type: author, _id: 1}}
{name: John, isbn: 25874}
{index: {_index: books, _type: author, _id: 2}}
{name: Michael, isbn: 9632471}
```
```
curl -XPOST http://localhost:9200/books/_bulk --data-binary @books.json
curl -XGET http://localhost:9200/books/book/18?pretty
```
###### string = text (indexed) + keyword (not indexed)
##### scaled_float parameter with scale factor of 100: 10.45 = 1045

- Searching

curl -XGET http://localhost:9200/INDEX_NAME/_search?q=name:John
GET /INDEX_NAME/_search?q=name:John

GET /books/_count

GET /books/book/_count

- Search in a given type: 
```
GET /books/book/_count
{
	query: {
		match: {
			author: "Robert Gerlach"
		}
	}
}
```
- Search across all types in an index, match does the full-text search: 
```
GET /books/_search
{
	query: {
		match: {
			author: "Dennis Alder"
		}
	}
}
```
- Pagination: 

GET /books/_search?size=5&from=10

- Term search looks for *exact* value (and isbn field is of keyword type)
```
GET /books/_search
{
	query: {
		term: {
			isbn: "978398147521772547"
		}
	}
}
```
- A match search across multiple indices & multiple types:
```
GET /books,magazines/book,magazine/_search
{
	query:{
		match: {
			author: "R. L. Stevenson"
		}
	}
}
```
- PUT performs a full-document update, whereas POST does a partial update:
```
PUT /idx_test/type_test/3 {
  name: EA
  goal: 123
}

GET /idx_test/type_test/3

PUT /idx_test/type_test/3 {
   goal: 345
}

GET /idx_test/type_test/3
```
The name field got removed - full update was performed by a PUT command.

- POST does it differently:
```
PUT /idx_test/type_test/4 {
  name: EA
  goal: 123
}

GET /idx_test/type_test/3

POST /idx_test/type_test/3/_update {
   goal: 345
}

GET /idx_test/type_test/3
```
The name field is retained - partial update was performed by a POST command.

###### Search DSL:

curl -XGET http://localhost:9200/INDEX_NAME/_search?q=name:John  or shortcut  
GET /INDEX_NAME/_search?q=name:John

GET /books/_count  

GET /books/book/_count
```
GET /books/_search
{
	query: {
		match: {
			author: "Dennis Alder"
		}
	}
}

GET /books/_search?size=5&from=10

GET /books/_search
{
	query: {
		term: {
			isbn: "978398147521772547"
		}
	}
}
```
- A match search across multiple indices & multiple types:
```
GET /books,magazines/book,magazine/_search
{
	query:{
		match: {
			author: "R. L. Stevenson"
		}
	}
}
```
```
POST /INDEX_NAME/_search
{
	query:{
		match_all: {
			
		}
	}
}
```

```
POST /INDEX_NAME/_search
{
	query:{
		match: {
			author: "R. L. Stevenson"
		}
	}
}
```
```
POST /INDEX_NAME/_search
{
	query:{
		multi_match: {
			query: superman
			fields:[city, state]
		}
	}
}
```
```
POST /INDEX_NAME/_search
{
	query:{
		type: {
			value: TYPE_NAME
		}
	}
}
```

- Range search:
```
POST /INDEX_NAME/_search
{
	query:{
		range: {
			rating:{
				gte: 5
			}
		}
	}
}
```
```
POST /INDEX_NAME/_search
{
	query:{
		query_string: {
			query:"tacos"
			}
		}
	}
}
```
```
POST /INDEX_NAME/_search
{
	query:{
		query_string: {
			query:"tacos"
			fields: ["tags"]
			}
		}
	}
}
```
- Filtered query:

```
POST /INDEX_NAME/_search
{
	query:{
		filtered:{
			filter: {
				range:{
					rating: {
						gte:4.0
					}
				}
			},
			query:{
				query_string: {
					query:"tacos",
					fields:["tags"]
				}
			}
		}
	}
}
```
```
POST /INDEX_NAME/_search
{
	query: {
		filtered: {
			query: {
				match: {
					"address.state": "ny"
				}
			},
			filter: {
				range:{
					rating: {
						gte: 4.0
					}
				}
			}
		}
	}
}
```

- Query with explain parameter:
```
GET /_search?explain
{
	query: {
		match{
			description: "Learning about Elastic Search"
		}
	}
}
```
- Results of query: 
```
{
	_index: INDEX_NAME
	_type: TYPE_NAME
	_id: 10
	_score: 0.922
	....

}
```











