# Learning Elastic Search 5.0 Video

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
```
GET /books/book/_search
{
	query: {term: {title: "Elastic Search 5.0"}}
	filter: {term: {"price: 10}}
}


GET /books/book/_search
{
	query: {
		bool: {
			must: [
			{match: {title: "Big Data"}}
			],
			filter:[
				{term: {language: "en"}}
				{range: {pub_date: {gte: 2014}}}
			]
		}
	}
}
```

- Term query (most effective with keywords): 
```
GET /books/author/_search
{
	query: {
			term: {name: "Mary Smith"}
		}
}
```
For full-text search use match queries instead. 

- Boosting the importance of 'description' field over the 'title'
```
GET /books/book/_search
{
	query: {
			multi_match:{
				query: "Elastic Search"
				fields: ["title", "description^2"]
			}
		}
}
```
##### Term Queries and Boosting

- Add an index
```
PUT /learning_es{
	mappings: {
		course:{
			properties: {
				full: {
					type: text
				},
				exact: {
					type: keyword
				}
			}
		}
	}
}
```
- Add a document:
```
PUT /leaning_es/course/1
{
	full: "Hello World!"
	exact: "Hello World!"
}
```


##### Full field IS analyzed by ES, the exact one - is NOT

- Run a term query against an exact field:
```
GET /learning_es/course/_count
{
	query: {
		term: {
			exact: "Hello World!"
		}
	}
}
```
returns 1 result back (count=1), 

whereas running the same #term# query against a full field:

```
GET /learning_es/course/_count
{
	query: {
		term: {
			full: "Hello World!"
		}
	}
}
```
returns no results (count=0).

To see the results, we will use a match query instead:
```
GET /learning_es/course/_count
{
	query: {
		match: {
			full: "Hello World!"
		}
	}
}
```
in which case we get a single return(count=1) as expected.

- Takeaway: use match query on full-text field and term query on exact fields(keyword - tag, dates, etc.)

- Range Query (works with numbers and dates):

```
GET /books/book/_search
{
	query: {
		range: {
			price: {gte: 18, lt: 30}
		}
	}
}
```

```
GET /books/book/_search
{
	query: {
		range: {
			pub_date: {gte: 2015-01-01, lt: 2016-12-31}
		}
	}
}
```

- Exist Query

- We want to know whether a given fied has at least one non-null value:
```
GET /INDEX_NAME/_search
{
	query: {
		exists: {field: "FIELD_NAME"}
	}
}
```
- How many documents with a non-null 'isbn' field value exists in our index?
```
GET /books/book/_search
{
	query: {
		exists: {field: "isbn"}
	}
}
```
- Term search for a particular book:
```
GET /books/book/_search
{
	query: {
		term: {_id: 5}
	}
}
```

- All books (in English) which don't contain 'Big Data' in their title:

```
GET /books/book/_search
{
	query: {
		bool: {
			must_not: [
				{match: {title: "Big Data}}
			],
			filter: [
				{term: {language: "en"}}
			]
		}
	}
}
```

##### Aggregation-based analytics - allowing the use of search to build complex summaries of data

*Metrics aggregation* calculates average value over a given numeric field in a set of documents:

```
GET /books/book/_search
{
	size: 0
	aggs:
	{
		avg_book_count: {
			avg: {field: "book_count"}
			missing: 1
		}
	}
}
```
- Cardinatility aggregation is a single-value metrics that allows aggregation of **distinct** values:

```
GET /books/book/_search
{
	aggs:
	{
		num_of_authors: {
			cardinality: {field: "author"}
		}
	}
}
```
- Extended Stats aggregation is a multi-value metrics that calculates count, min, max, sum, sum_of_squares, variance, std_deviation, etc. over values of a numeric field across a set of documents:

```
GET /books/book/_search
{
	aggs:
	{
		book_stats: {
			extended_stats: {field: "book_count"}
		}
	}
}
```
- Geo-aggregation uses longitute and latitude from a set of documents to calculate geo bounds (box) enclosing all lon/lat locations:

```
GET /INDEX_NAME/TYPE_NAME/_search
{
	query: {
		match: {tags: "technology"}
	},
	aggs: {
		viewport: {
			geo_bounds: {field: "pin.location"}
		}
	}
}
```	
- Bucket Aggregation groups search results in buckets (for histograms)

Date Histogram:
```
GET /books/book/_search
{
	aggs: {
		pub_over_time: {"date_histogram: {field: "pub_date", interval: "month"}}
	}
}
```
- Histogram is applied to numerical values of certain fields grouped into buckets across a set of documents:
```
GET /books/book/_search
{
	aggs: {
		book_prices: 
		{histogram: 
		{
			field: "price", interval: 5, min_doc_count: 1
		}
	}
}
```
Example - average price of all books:

```
GET /books/book/_search
{
	size: 0
	aggs: {
		avg_book_price: 
		{
		avg: 
		{
			field: "price"
		}
	}
}
```
- Use Geo-bounds to form a perimeter (or box) around all of the bookstore in index:
```
GET /books/store/_search
{
	query: {
		"match_all: {}
	},
	size: 0,
	aggs: {
		viewport: {
			geo_bounds: {
				field: "pin.location",
				wrap_longitude: true
			}
		}
	}
}
```
- All books published within certain date range plotted in a histogram by month:
```
GET /books/book/_search
{
	query: {
		range: {
			pub_date: {
				gte: "2015-01-01", lt: "2016-12-31"
			}
		}
	},
	size: 0,
	aggs: {
		pub_over_time: {
			date_histogram: {
				field: "pub_date",
				interval: "month"
			}
		}
	}
}
```
ES is optimized for read performance.

Impediments of speed:

- Transactions
- Denormalization

##### ElasticSearch:

- lacks in security: authentication and authorization (X-Pack compensates for this)
- cannot cancel resource intensive queries
- prone & susceptible to OutOfMemory errors

ES is most effectively used in conjunction with another DB, allowing to leverage the full power of search while simultaneously enjoying the robustness of a mature (relational) DB system.  In this model the DB maintaines the master records while asynchronously pushing records to ES.  

It's perfect for write once, read many times (WORM) situations.

Good for: 

- Indexing and searching massive datasets
- Querying large datasets for exact matches in select fields
- Filtering
- Storing semi-structured WORM data
- Wide range searches
- Time-based aggregations
- Geo data points handling

ElasticSearch vs Druid: https://stackoverflow.com/questions/40287228/what-are-differences-between-druid-and-elasticsearch-whart-advantages-for-both


Not to be used for precision tasks, when real-time low latency data is required, when there's a need for transations, granular security features, integrity constraints (record uniqueness guarantees).

##### ElasticStack

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/Elastic%20Stack.PNG)

- Kibana https://www.elastic.co/products/kibana
- Logstash https://www.elastic.co/products/beats
- X-Pack https://www.elastic.co/products/x-pack
- Beats https://www.elastic.co/products/beats

##### Preparing for Log Analysis with LogStash

- Configure LogStash pipeline:

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/logstash%20pipeline.png)

- FileBeat Configuration:

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/filebeat%20config.PNG)

- LogStash Configuration:

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/logstash.conf%20skeleton.PNG)

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/logstash.conf%20input%20section.PNG)

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/logstash.conf%20filter%20section.PNG)

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/logstash.conf%20output%20section.PNG)

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/grok%20patterns.PNG)

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/grok%20complex%20pattern.PNG)

- Test LogStash Pipeline:

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/test%20logstash%20pipeline.PNG)

- Run LogStash Pipeline:

![](https://github.com/excelsiorsoft/learning-elastic-search-5.0-video/blob/master/run%20logstash%20pipeline.PNG)

https://www.safaribooksonline.com/library/view/learning-elasticsearch-50/9781783984589/video8_2.html

##### Sorting in Elastic Search:
```
GET /books/book/_search
{
	sort: [
		{pub_date: {order:desc}}
	],
	query: {
		term: {tags: "big data"}
	}
}
```
```
GET /books/book/_search
{
	from: 0, size: 30
	sort: [
		{pub_date: {order:asc}}
	],
	query: {
		match_all: {}
	}
}
```
##### Geo Searching

- Distance query example:

```
filter: {
	geo_distance: {
		distance: 20km,
		pin.location:
			{
				lat: 40,
				lon: -70
			}
	}
}
```

```
filter: {
	geo_distance_range: {
		from: 20km, to: 40km,
		pin.location:
			{
				lat: 40,
				lon: -70
			}
	}
}
```
```
sort:
[
	_geo_distance: {pin.location:[-64,35], order: asc, unit: km, mode: min, distance_type: sloppy_arc}
]
```
##### Synonims

- Skeleton in index:
```
PUT /books
{
	settings: {
		analysis: {
			filter: {}
			analyzer: {}
		}
	}
}
```

- Filter Part:
```
filter:
{
	the_es_filter: 
	{
		type: synonym, synonyms:
		[
			"AI, Artificial Intelligence, Deep Learning",
			"big data, massive datasets, petabytes"
		]
	}
}
```
- Analyzer Part:
```
analyzer:
{
	the_es:
	{
		tokenizer: standard,
		filter:
		[
			"lowercase", "the_es_filter"
		]
	}
}
```







