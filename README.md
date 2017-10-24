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

-Modifying index's attributes:
PUT /books/_settings
{
	"number_of_replicas":1
}
