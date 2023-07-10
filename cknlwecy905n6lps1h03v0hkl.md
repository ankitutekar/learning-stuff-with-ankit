---
title: "Implementing auto-complete functionality in Elasticsearch - Part III: Completion suggester"
datePublished: Sat Apr 17 2021 15:31:22 GMT+0000 (Coordinated Universal Time)
cuid: cknlwecy905n6lps1h03v0hkl
slug: implementing-auto-complete-functionality-in-elasticsearch-part-iii-completion-suggester
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618371474944/zgrFJN0L1.jpeg
tags: tutorial, search, elasticsearch

---

This is part III of [my series](https://www.learningstuffwithankit.dev/series/auto-complete-es) on designing auto-complete feature in Elasticsearch. In this part, we will talk about [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-suggesters.html#completion-suggester)  -  a type of [suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-suggesters.html) which is optimized for auto-complete functionality and considered to be faster than the approaches we have discussed so far.

Completion suggesters use a data structure known as [Finite State Transducer](https://stackoverflow.com/questions/4872115/what-is-a-finite-state-transducer) which is similar to the Trie data structure and is optimized for faster look-ups. These data structures are stored in-memory on nodes to enable faster searches. Like edge-n-gram and search_as_you_type, this also does most of the work at index time by updating in-memory FSTs with the input that we provide.

A special type of ES type - *completion,* is used for implementing it -

```json
PUT /movies
{
	"mappings": {
		"properties": {
			"title": {
				"type": "completion"
			}
		}
	}
}
```

Mapping also supports *analyzer, search analyzer, max_input_length* parameters for the completion field. Analyzer value defaults to *simple* analyzer which lower-cases the input and tokenizes on any non-letter character such as number, space, a hyphen, etc. Analyzers on completion types behave differently than analyzers on other text fields. After analysis, tokens are not available separately - they are put together and inserted into FST, based on their order in the input text. Also, we can't test our mappings using *_analyze* endpoint in this approach. If we try to do so ES throws an error saying '*Can't process field [title], Analysis requests are only supported on tokenized fields*'.

While indexing a document, we specify *input* and an optional *weight* parameter -

```json
POST /movies/_doc/1001
{
	"title": [
		{"input": "Harry Potter and the Goblet of Fire", "weight": 5},
		{"input": "Goblet of Fire", "weight": 10}
	]
}

POST /movies/_doc/1002
{
	{
	"title": {
		"input": ["Harry Potter and the Goblet of Fire",
			      "Goblet of Fire"],
		"weight": 2
		}
	}
}

```

We can specify multiple matches for a single document using input parameter. The weight parameter controls the ranking of documents in search results. It can be specified per input as shown in the first document(1001) above, or can be kept same for all the inputs as shown in the second document(1002).

Suggester fields are queried using *suggest* clause inside the request body of *_search* endpoint. Before ES version 5.0, there was a separate endpoint - *_suggest* for suggesters. Many examples on the internet use *_suggest.* Since version 5, *_search* endpoint itself has been updated to support suggesters too.

By default, Elasticsearch returns entire matching document. If we are only interested in the suggestion text, we can use *_source* option and set it to "suggest". This way, we minimize disk fetch and transport overhead:

```json
GET /movies/_search
{
	"_source": "suggest",
	"suggest": {
		"harry_suggest": {
			"prefix": "goblet of f",
			"completion": {
				"field": "title"
			}
		}
	}
}
```

Above query returns both the documents - 1001 and 1002, as both the documents contain "Goblet of Fire" as one of the suggestions for title. First document is ranked higher as it has more weight i.e. 10. This can be observed in the response of above query:

```json
{
	"took": 10,
	"timed_out": false,
	"_shards": {...},
	"hits": {...},
	"suggest": {
		"harry_suggest": [
			{
				"text": "goblet of f",
				"offset": 0,
				"length": 11,
				"options": [
					{
						"text": "Goblet of Fire",
						"_index": "movies",
						"_type": "_doc",
						"_id": "1001",
						"_score": 10.0,
						"_source": {}
					},
					{
						"text": "Goblet of Fire",
						"_index": "movies",
						"_type": "_doc",
						"_id": "1002",
						"_score": 2.0,
						"_source": {}
					}
				]
			}
		]
	}
}
```

"Goblet of Fire" is returned twice in suggestions as we had provided this text as input in both the documents. This can be avoided by using [*skip_duplicates*](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-suggesters.html#skip_duplicates) option.

In case of completion suggester, ES matches the documents one character at a time starting from the first character, moving ahead one position as a new character is typed in. As discussed above, it preserves the order of input in FST. So, it won't be able to match in the middle of the input like n-gram based approaches. i.e. if you have a movie named "Harry Potter and the Goblet of Fire" and you type in "goblet of fire", it won't return the document as a match. You can, however, use the input option to provide multiple matches. You can manually tokenize your input string and pass the tokens to Elasticsearch in the input option, like how we have done in the examples above by providing "Goblet of Fire" as additional input.

Completion suggester supports [fuzzy queries](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-suggesters.html#fuzzy) that allow us to consider typos while searching documents. You can also specify prefix text as [regex query](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-suggesters.html#regex). Both the queries in example below return "Goblet of Fire" as suggestions -

```json
/******************
    Fuzzy query 
******************/
GET /movies/_search
{
    	"_source": "suggest",
    	"suggest": {
    		"harry_suggest": {
    			"prefix": "gobet of f",
    			"completion": {
    				"field": "title",
    				"fuzzy": {
    					"fuzziness": 2
    				}
    			}
    		}
      }
}

/******************
   regex query 
******************/
GET /movies/_search
{
      "_source": "suggest",
    	"suggest": {
    		"harry_suggest": {
    			"regex": "g[aieou]b",
    			"completion": {
    				"field": "title"
    			}
    		}
       }
}
```

### Adding Context to searches

Unlike other queries, completion suggesters don't support adding filters in your queries. i.e. you can't filter out suggestions based on values of other fields in the document. Suppose, we have an index that stores Movies and we are developing auto-complete based on title field. Let's say we have mapped title as completion type and there are other fields like genres, ratings, production companies, etc. There is a document with title "Goblet of Fire" which has genre as "action". Now, if we try to filter out auto-complete suggestion based on genre = "romance",  we expect that it shouldn't return "Goblet of Fire":

```json
GET /movies/_search
{
  	"query": {
    		"bool": {
    			"filter": [
    				{
    					"term": {
    						"genre": "romance"
    					}
    				}
    			]
    		}
    	},
    	"suggest": {
    		"harry_suggest": {
    			"prefix": "goblet",
    			"completion": {
    				"field": "title"
    			}
    		}
      }
}
```

This doesn't work as we expect - it returns "Goblet of Fire" as a suggestion even though it belongs to the "action" genre. The main reason behind this limitation is its design. As already discussed, suggestions are stored in a separate data-structure - in-memory FST, whereas other fields are stored on disk. This design facilitates faster searches through in-memory FST. Queries like above go against this design.

However, Elasticsearch does provide [context suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-suggesters.html#context-suggester) to circumvent this issue up to some extent. To use context suggester, we have to provide contexts while creating mapping for the index:

```json
PUT /movies
{
   	"mappings": {
        		"properties": {
        			"title": {
        				"type": "completion",
        				"contexts": [
        					{
        						"name": "genre",
        						"type": "category"
        					}
        				]
        		  }
        	}
      }
}
```

For a particular completion field, we can define multiple contexts having unique names. There are two types of contexts supported:
  - __Category__ => Category of the thing you are indexing, e.g. genre of a movie/song
  - __Geo__ => Geo-points for the documents you are indexing, allows filtering suggestions based on lat-long.

Each context type above supports some advanced parameters such as precision, neighbours for geo context, boost while querying so that documents having particular category are scored higher. Note that, for context enabled completion fields, contexts parameter is mandatory while indexing a document as well as querying it.

Let's index some documents in the index created above:

```json
POST /movies/_doc/2001
{
     "title": {
        		"input": "Harry Potter and the Chamber of Secrets",
        		"contexts": {
        			"genre": "mystery"
        		}
       }
}

POST /movies/_doc/2002
{
        	"title": {
        		"input": "Harry Potter and the Prisoner of Azkaban",
        		"contexts": {
        			"genre": "crime"
        		}
       }
}
```

Above, we have indexed "Harry Potter and the Prisoner of Azkaban" as a movie in "crime" genre and "Harry Potter and the Chamber of Secrets" in "mystery" genre. Let's try to get suggestions for prefix "harry":

```json
/******************
      Request
******************/
GET /movies/_search
{
       {
        	"_source": "suggest",
        	"suggest": {
        		"harry_suggest": {
        			"prefix": "harry",
        			"completion": {
        				"field": "title",
        				"contexts": {
        					"genre": "crime"
        				}
        			}
        		}
        	}
       }
}
/******************
      Response
******************/
{
        	"took": 25,
        	"timed_out": false,
        	"_shards": {...},
        	"hits": {...},
        	"suggest": {
        		"potter_suggest": [
        			{
        				"text": "harry",
        				"offset": 0,
        				"length": 5,
        				"options": [
        					{
        						"text": "Harry Potter
                                 and the Prisoner of Azkaban",
        						"_index": "movies",
        						"_type": "_doc",
        						"_id": "2002",
        						"_score": 1.0,
        						"_source": {},
        						"contexts": {
        							"genre": [
        								"crime"
        							]
        						}
        					}
        				]
        			}
        		]
         }
}
```

As it can be observed in the response above, only "Harry Potter and the Prisoner of Azkaban" from the "crime" genre is returned even though the passed prefix matches both the documents indexed above.

That was all about our third approach for implementing auto-complete in ES. So how does completion suggester compare with other approaches seen so far? It is definitely the fastest one as data to be searched is available in memory, but there are some things we need to keep in mind if we decide to implement auto-complete using it:
  - Have to be mindful of the size of the index as suggestions are stored in-memory.
  - Infix matching, e.g. matching by middle-name, is not supported.
  - Advanced filtering for suggestions by other fields in the document is not supported.

So to conclude this series, we can say that following factors should be considered while choosing an approach for implementing auto-complete functionality in Elasticsearch:
  - Is the data already indexed? In what format? Can we re-index it to make it more suitable for auto-complete functionality? If the data is already indexed as *text* field and we can't re-index it, we will need to go with query time approach - i.e. prefix queries!
  - In what ways can this field be queried? Does it make sense to store it in more than one way?
  - Does it need to support infix matches? Is the order of words in text is fixed? Is the order well known to users? Completion suggesters don't support infix matches and are not suitable for fields having highly known order.
  - What can be the maximum size of the text that will be supplied as value to our field? Can it create problems if it's saved in-memory? Completion suggesters save data in memory, n-gram based approaches create additional tokens after basic tokenization for faster matches.
  - Do we need to have a separate index for this field? If all of the three approaches mentioned here don't satisfy your requirements, then you will need to create another index. In that index, only fields needed for auto-complete functionality will be stored as unique documents, instead of saving them with other data in the same index. This will minimize the chances of bloating up nodes and also can provide faster suggestions. But yes, it is a separate index after all, you will have to keep data in sync between your main index and new index. And there is overhead of managing another index too.

I hope you enjoyed this series and it added something to your knowledge. Do let me know your feedback in the comments.

> May Elasticsearch be the Firebolt for your search-engine game!!!