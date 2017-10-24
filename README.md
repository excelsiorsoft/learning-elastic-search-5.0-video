# Learning-elastic-search-5.0-video

My notes while following the following video: https://www.safaribooksonline.com/library/view/learning-elasticsearch-50/9781783984589/


- List all existing indexes: 
curl -XGET http://localhost:9200/_cat/indices?v

- Querying particular index:
curl -XGET http://localhost:9200/schools?pretty

- Create an index:
PUT /books
{
	"settings":{
		"number_of_shards":1,
		"number_of_replicas":0
	}
}

- Now read what's been created: 
GET /books

##### \# of shards is immutable
##### \# of replicas can be changed later

- Modifying index's attributes:
PUT /books/_settings
{
	"number_of_replicas":1
}


- Check cluster's health:
curl -XGET http://localhost:9200/_cluster/health?pretty

##### In Kibana (authors is a type of document): 

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

- No ID assignment:

curl -XPOST http://localhost:9200/books/author/- d
{
	name: Michael J.
}


- With pre-assigned ID:

curl -XPOST http://localhost:9200/books/author/1/_create- d
{
	name: Michael J.
}

- Delete by ID:

curl -XDELETE http://localhost:9200/books/author/1/_create- d
{
	name: Michael J.
}

##### Single-node cluster ends up with an 'yellow' status (warning):

GET /books/_cluster/health

##### Adding document in Kibana dev tools:

- an ID is auto-assigned here: 

POST /books/author/
{
	name: Michael
}

- creating with a specific ID:

POST /books/author/1/_create
{
	name: Michael
}

- Add an additional field:

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

- Delete a document: 

DELETE /books/author/1

- Confirm deletion: 

GET /books/author/1

##### Adding multiple documents at a time (Bulk API):

curl -POST http://localhost:9200/_bulk -d
{index: {_index: books, _type: author, _id: 1}}
{name: John, isbn: 25874}
{index: {_index: books, _type: author, _id: 2}}
{name: Michael, isbn: 9632471}

POST /books/_bulk
{index: {_index: books, _type: author, _id: 1}}
{name: John, isbn: 25874}
{index: {_index: books, _type: author, _id: 2}}
{name: Michael, isbn: 9632471}

curl -XPOST http://localhost:9200/books/_bulk --data-binary @books.json
curl -XGET http://localhost:9200/books/book/18?pretty

###### string = text (indexed) + keyword (not indexed)
##### scaled_float parameter with scale factor of 100: 10.45 = 1045

- Searching

curl -XGET http://localhost:9200/INDEX_NAME/_search?q=name:John
GET /INDEX_NAME/_search?q=name:John

GET /books/_count

GET /books/book/_count

GET /books/book/_count
{
	query: {
		match: {
			author: "Robert Gerlach"
		}
	}
}









