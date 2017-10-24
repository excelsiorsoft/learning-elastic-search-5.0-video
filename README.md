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

##### In Kibana (authors is a type): 

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

##### Single-node cluster ends up with an 'yellow' status (warning)
