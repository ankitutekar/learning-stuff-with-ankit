---
title: "Implementing auto-complete functionality in Elasticsearch - Part I: Prefix queries"
datePublished: Sat Apr 17 2021 12:53:01 GMT+0000 (Coordinated Universal Time)
cuid: cknlqqpi904cqlps1d42eezhe
slug: implementing-auto-complete-functionality-in-elasticsearch-part-i-prefix-queries
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618062664835/YwMz5U0s0.jpeg
tags: tutorial, search, elasticsearch

---

Being a software engineer, I tend to judge products and companies behind those products based on how efficiently they have implemented technically challenging things. One of the things on the internet that fascinates me is blazing fast auto-complete implementations! Especially those, which load things asynchronously, from a large data-set in the back-end.

Auto-complete is not like search functionality - we are supposed to update auto-complete options as soon as the user types next character, hitting the database literally every second, filtering through millions of records, without causing any performance degradation!!!

A technology that makes it easy to implement such features is Elasticsearch - a search and analytics engine built on top of [Apache Lucene library](https://lucene.apache.org/). Elasticsearch has distributed, multi-tenant architecture with built-in routing and re-balancing, making it easy to scale. It's a widely used data store for storing, searching, and analyzing large volumes of data.

In this [three-part series of blog posts](https://www.learningstuffwithankit.dev/series/auto-complete-es), I will be going into details of how we can implement auto-complete functionality using various options available in Elasticsearch. In the first part(i.e. this post), we will talk about prefix queries. In the second part, we will have a look at n-grams and in the final part, we will discuss completion suggesters. I will be using [Elasticsearch 7.12](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/index.html), the *current* version at the time of writing this.

For example purposes, we will be using an index that stores data of movies. To keep it simple, *title* will be the only property present in this index. As Elasticsearch exposes REST interface for its operations, you can use any REST based tool to communicate with it. 

This series assumes basic familiarity with Elasticsearch. If you are new to Elasticsearch, I highly recommend reading [an article](https://www.knowi.com/blog/what-is-elastic-search/) or two on the basics of Elasticsearch.

 So let's get started, shall we?

### Prefix queries

Prefix queries are the simplest form of auto-complete implementation in Elasticsearch. We don't do anything special while storing the field, most of the work is done at query time. The field is indexed(stored!) as a simple text/keyword field and queries that allow us to match documents based on passed prefixes are used to query it.

Let's create an index to run prefix queries on:

```json
PUT /movies
{
     {
	  "mappings": {
		"properties": {
			"title": {
				"type": "keyword",
				"fields": {
					"analyzed_title": {
						"type": "text"
					}
				}
			}
		 }
	  }
    }
}
```
While creating an index, we need to provide mapping, indicating type of data we intend to store. For the purpose of examples below, the *title* is mapped as a keyword field and also as a text field for supporting full-text queries. A field can be mapped as more than one type using [multi-fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/multi-fields.html) feature of Elasticsearch.

The key difference between a keyword field and a text field is that the keyword fields are not analyzed, i.e. data we pass to a keyword field is stored as it is. Text fields are analyzed, i.e. tokenized, possibly transformed(e.g. lowercased, stemmed, etc.), and stored in an *inverted index*. [Inverted index](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up#inverted-indexes-and-index-terms) is a data structure that stores mappings from terms to the location of documents they appear in, enabling efficient full-text searches. 

To test how our data will be analyzed, we can use [_analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/indices-analyze.html).  Let's see how our main title field will be analyzed:

```json
/***********************
         Request
***********************/
GET /movies/_analyze
{
	"text": "Chamber of Secrets",
	"field": "title" 
}

/***********************
       Response
***********************/
{
	"tokens": [
		{
			"token": "Chamber of Secrets",
			"start_offset": 0,
			"end_offset": 18,
			"type": "word",
			"position": 0
		}
	]
}
```

So, it returned only a single token. Why? That's right, it's because it's a keyword field! Let's test how our analyzed title will behave:

```json
/***********************
       Request
***********************/
GET /movies/_analyze
{
	"text": "Chamber of Secrets",
	"field": "title.analyzed_title" 
}

/***********************
       Response
***********************/
{
	"tokens": [
		{
			"token": "chamber",
			"start_offset": 0,
			"end_offset": 7,
			"type": "<ALPHANUM>",
			"position": 0
		},
		{
			"token": "of",
			"start_offset": 8,
			"end_offset": 10,
			"type": "<ALPHANUM>",
			"position": 1
		},
		{
			"token": "secrets",
			"start_offset": 11,
			"end_offset": 18,
			"type": "<ALPHANUM>",
			"position": 2
		}
	]
}
```

As expected, it was broken down into three tokens. Moreover, the tokens are lower-cased. Why is that? Because, even if we don't specify any analyzer, default *standard analyzer* is applied to text fields which performs grammar-based tokenization and also lower-cases these tokens. [Text analysis](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/analysis.html) is a highly configurable process that consists of one or more character filters, a tokenizer, and one or more token filters, running in a pipeline. We can create our own analyzers and also can customize built-in analyzers.

Let's add some Harry Potter movies to our index, i.e. let's *index* some documents:

```json
POST /movies/_doc
{
	"title": "Harry Potter and the Chamber of Secrets"
}

POST /movies/_doc
{
	"title": "Harry Potter and the Prisoner of Azkaban"
}
```

Let's try to query our main title field(keyword) using [prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-prefix-query.html). The prefix query is a type of term level query which is used to query non-analyzed fields. We will try two different requests - first with prefix of first word in the title, another one with prefix of second word in the title:

```json
/************************
Query using prefix "Harr"
*************************/
GET /movies/_search
{
	"query": {
		"prefix": {
			"title": "Harr"
		}
	}
}

/***********************
Returns below response 
************************/
{
	"took": 6,
	"timed_out": false,
	"_shards": {...},
	"hits": {
		"total": {
			"value": 2,
			"relation": "eq"
		},
		"max_score": 1.0,
		"hits": [
			{
				"_index": "movies",
				"_type": "_doc",
				"_id": "qk1qlngBuCt11zp-N_lD",
				"_score": 1.0,
				"_source": {
					"title": "Harry Potter and the
                                Chamber of Secrets"
				}
			},
			{
				"_index": "movies",
				"_type": "_doc",
				"_id": "q01rlngBuCt11zp-GPl_",
				"_score": 1.0,
				"_source": {
					"title": "Harry Potter and the
                                Prisoner of Azkaban"
				}
			}
		]
	}
}

/***********************
Query using prefix "Pott"
************************/
GET /movies/_search
{
	"query": {
		"prefix": {
			"title": "Pott"
		}
	}
}

/*********************
Returns below response 
**********************/
{
	"took": 3,
	"timed_out": false,
	"_shards": {...},
	"hits": {
		"total": {
			"value": 0,
			"relation": "eq"
		},
		"max_score": null,
		"hits": []
	}
}
```
The title being a keyword field, we have to provide prefix with correct casing. If we pass 'harr' in query, it won't match. The first request returns both the documents indexed above, as it should. But the second request doesn't return us anything. That's because this query doesn't support infix(matching in the middle of the title) matches.

*__Note__: I will be keeping only the relevant part & removing other parts of responses by replacing it with '...', just to make it a bit shorter.*

If we want to match inside the title, we should be using [match_phrase_prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-match-query-phrase-prefix.html) - a type of query used for prefix matching on analyzed text fields:

```json
/**************************
 Query using prefix "pott" 
**************************/
GET /movies/_search
{
    "query": {
    		"match_phrase_prefix": {
    			"title.analyzed_title": {
    				"query": "pott"
    			}
    		}
    	}
 }

/***********************
 Returns below response 
************************/
{
    	"took": 5,
    	"timed_out": false,
    	"_shards": {...},
    	"hits": {
    		"total": {
    			"value": 3,
    			"relation": "eq"
    		},
    		"max_score": 0.1461155,
    		"hits": [
    			{
    				...
    				"_source": {
    					"title": "Harry Potter and the
                                    Chamber of Secrets"
    				}
    			},
    			{
    				...
    				"_source": {
    					"title": "Harry Potter and
                                    the Prisoner of Azkaban"
    				}
    			}
    		]
    	}
}
```

As we are searching on the analyzed title which is tokenized, "pott" prefix matches with token "potter", which belongs to both of our documents. So, both the documents are returned.

What about out-of-order prefixes? As words inside the title are tokenized, we will expect "potter harry" to match both the documents. But this being a *phrase prefix* query, it respects the order of input. If we want out-of-order matches, we can use [match_bool_prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-match-bool-prefix-query.html).

```json
/***********************************
Below query doesn't return anything
***********************************/
GET /movies/_search
{
	"query": {
		"match_phrase_prefix": {
			"title.analyzed_title": {
				"query": "potter harry"
			}
		}
	}
}

/************************************************
Below query DOES return both the documents,
similar to response in match_phrase_prefix above
*************************************************/
GET /movies/_search
{
	"query": {
		"match_bool_prefix": {
			"title.analyzed_title": {
				"query": "pott harr"
			}
		}
	}
}
```

So that's all I had to talk about auto-complete using prefix queries. There are a few things we need to consider while choosing this as an approach for implementing auto-complete functionality:

*    This one is the least recommended approach and considered to be the slowest one when compared to other auto-complete implementations is ES. The searches are slow because we are not doing any work while indexing the field that will help auto-complete queries. It is indexed as a simple text field, most of the work of matching the documents with queried text is done at search time. It will go to the inverted index and check if any token *starts with* text provided in the query, which is an expensive operation.
*    In recent versions of Elasticsearch, *index_prefixes* option has been added for term level prefix query that allows to [speed up prefix queries](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-prefix-query.html#prefix-query-index-prefixes) by storing prefixes in separate fields.
*    If you already have a working index and don't want update mapping, prefix queries will be a suitable approach for you, given that auto-complete is not one of the heavily used features of your system. But if it is, then you might run into performance issues. It will be better to use one of the approaches discussed in the next parts of this series and re-index the data.

In [part II](https://www.learningstuffwithankit.dev/implementing-auto-complete-functionality-in-elasticsearch-part-ii-n-grams) we will talk about n-grams, an index time approach for auto-completes in ES. Do let me know your feedback on this part in comments below!