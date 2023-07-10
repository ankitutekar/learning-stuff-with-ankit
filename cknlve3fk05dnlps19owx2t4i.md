---
title: "Implementing auto-complete functionality in Elasticsearch - Part II: n-grams"
datePublished: Sat Apr 17 2021 15:03:10 GMT+0000 (Coordinated Universal Time)
cuid: cknlve3fk05dnlps19owx2t4i
slug: implementing-auto-complete-functionality-in-elasticsearch-part-ii-n-grams
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618159193827/JQWWeZRhd.jpeg
tags: tutorial, search, elasticsearch

---

This is part II of [my series](https://www.learningstuffwithankit.dev/series/auto-complete-es) on implementing auto-completion feature using Elasticsearch. In [first part](https://www.learningstuffwithankit.dev/implementing-auto-complete-functionality-in-elasticsearch-part-i-prefix-queries) we talked about using prefix queries, a query time approach for auto-completions. In this post, we will talk about *n-grams* - an index time approach which generates additional tokens after basic tokenization so that we can have faster prefix matches later at query time. But before that, let's see what an n-gram is. As per Wikipedia -

> an n-gram is a contiguous sequence of n items from a given sequence of text or speech

Yes, it is as simple as that, just a sequence of text. 'n' items here mean 'n' characters in case of *character level n-grams* and 'n' words in case of *word level n-grams.* *Word level n-grams* are also known as *shingles*. Further, based on value of 'n', these are categorized as uni-gram(n=1), bi-gram(n=2), tri-gram(n=3) and so on.

Below example will make it clearer: 
```json
Character n-grams for input string = "harry":
    n = 1 : ["h", "a", "r", "r", "y"]
    n = 2 : ["ha", "ar", "rr", "ry"]
    n = 3 : ["har", "arr", "rry"]

Word n-grams for input string = "harry potter and the goblet of fire":
    n = 1 : ["harry", "potter", "and", "the", "goblet", "of", "fire"]
    n = 2 : ["harry potter", "potter and", "and the", "the goblet",
           "goblet of", "of fire"]
    n = 3 : ["harry potter and", "potter and the", "and the goblet",
            "the goblet of", "goblet of fire"]
```

In this post, we will discuss two n-gram based approaches - first using *edge-n-gram tokenizer* and then using built-in *search-as-you-type* type, which also uses n-gram tokenization internally. These additional tokens are outputted into inverted index while indexing the document, which minimizes search time latency. Here, ES simply has to compare the input with these tokens unlike prefix query approach where it needed to check if individual token starts with given input.

### Edge-n-gram tokenizer

As we have already seen, text fields are analyzed and stored in inverted index. Tokenization is 2nd step in this 3 step [analysis process](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/analyzer-anatomy.html), ran after filtering characters but before applying token filters. [Edge-n-gram tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/analysis-edgengram-tokenizer.html) is one of the built-in tokenizers available in ES. It first breaks down given text into tokens, then generates character level n-grams for each of these tokens.

Let's create an index for movies, this time using edge-n-gram tokenizer:

```json
PUT /movies
{
    "settings": {
    		"analysis": {
    			"analyzer": {
    				"custom_edge_ngram_analyzer": {
    					"type": "custom",
    					"tokenizer": "customized_edge_tokenizer",
    					"filter": [
    						"lowercase"
    					]
    				}
    			},
    			"tokenizer": {
    				"customized_edge_tokenizer": {
    					"type": "edge_ngram",
    					"min_gram": 2,
    					"max_gram": 10,
    					"token_chars": [
    						"letter",
    						"digit"
    					]
    				}
    			}
    		}
    	},
    	"mappings": {
    		"properties": {
    			"title": {
    				"type": "text",
    				"analyzer": "custom_edge_ngram_analyzer"
    			}
    		}
      }
}
```

In prefix query examples, we weren't passing *analyzer* parameter to any of the fields in mapping, we were relying on default *standard* analyzer. Above, we have first created a custom analyzer *custom_edge_ngram_analyzer* by passing it customized tokenizer *customized_edge_tokenizer* of type *edge_ngram*.  Edge_ngram tokenizer can be customized using below parameters:

  - __min_gram__ ⇒ Minimum number of characters to put in a gram, defaults to 1, similar to uni-gram example seen above
  - __max_gram__ ⇒ Maximum number of characters to put in a gram, default to 2, similar to bi-gram example seen above
  - __token_chars__ ⇒ Characters that are to be kept in a token, if ES encounters any character that doesn't belong to the provided list, it uses that character as break-point for new token. Supported character classes include letter, digit, punctuation, symbols and white-space. In above mapping, we have kept letters and digits as part of the token. If we pass input string as "harry potter: deathly hallows", ES will generate ["harry", "potter", "deathly", "hallows"] by breaking on white-space and punctuation.

Let's use *_analyze* API to test how our custom edge-n-gram analyzer will behave:

```json
/**********Request**********/
GET /movies/_analyze
{
    {
    	"field": "title",
    	"text": "Harry Potter and the Order of the Phoenix"
    }
}

/**********Response*********/
[ha, har, harr, harry, po, pot, pott, potte, potter, an,
 and, th, the, or, ord, orde, order, of, th, the, ph, pho,
 phoe, phoen, phoeni, phoenix]
```

To keep it concise, I haven't included actual response which contains an array of objects, one object per gram, containing metadata about that gram. Anyhow, as it can be observed, our custom analyzer is working as designed - emitting grams for passed string, lower-cased and having length within min-max settings. Let's index some movies to test auto-complete functionality -

```json
POST /movies/_doc
{
      {
    		"title": "Harry Potter and the Half-Blood Prince"
      }
}

POST /movies/_doc
{
      {
    		"title": "Harry Potter and the Deathly Hallows – Part 1"
    	}
}
```

Edge-n-grammed fields support infix matches as well. i.e. you can match document with title 'harry potter and the deathly hallows' by passing 'har' and 'dead' too. This makes it suitable approach for auto-complete implementations where there is no fixed ordering of words in input text.

```json
/**Matches second document**/
GET /movies/_search
{
	{
	"query": {
		"match": {
			"title": {
				"query": "deathly "
			}
		}
	}
 }
}

/**Matches both the documents**/
GET /movies/_search
{
	{
	"query": {
		"match": {
			"title": {
				"query": "harry pot"
			}
		}
	}
  }
}

/**Also matches both the documents**/
GET /movies/_search
{
	{
	"query": {
		"match": {
			"title": {
				"query": "potter har"
			}
		}
	}
 }
}
```

By default, search queries on analyzed fields(title in above example) [run analyzer on search term as well.](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/specify-analyzer.html#specify-search-analyzer) If you specify search term as "deathly potter" hoping that it will match only the second document, you will be surprised because it matches both the documents. It's because the search term "deathly potter" will be tokenized too, outputting "deathly" and "potter" as separate tokens. Although "Harry Potter and the Deathly Hallows – Part 1" is matched with the highest score, input query tokens are matched separately giving us both the documents as result.  If you think this can cause issues, you can [specify an analyzer for the search query as well.](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/specify-analyzer.html#specify-search-query-analyzer)

Thus, edge-n-gram overcomes the limitations of prefix query by saving additional tokens in inverted index, minimizing query time latency. But, these additional tokens do take up extra space on nodes and can cause performance degradation. We should be careful while choosing the fields for n-gramming because values of some fields can have unbounded size and can bloat up your indices.

### Search_as_you_type

[Search_as_you type](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-as-you-type.html) datatype, which was introduced in Elasticsearch 7.2, is designed to provide out-of-the-box support for auto-complete functionality. Like edge-n-gram approach, this also does most of the work at index time by generating additional tokens to optimize auto-complete queries. When a particular field is mapped as search_as_you_type type, additional sub-fields are created for it internally. Let's change our title field type to search_as_you_type:

```json
PUT /movies
{
     "mappings": {
    		"properties": {
    			"title": {
    				"type": "search_as_you_type",
    				"max_shingle_size": 3
    			}
    		}
      }
}
```

For the title property in above index, three sub-fields will be created. These sub-fields use [shingle token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/analysis-shingle-tokenfilter.html). Shingles are nothing but groups of consecutive words(word n-grams as seen above).

  - __the root title field__ ⇒ analyzed using analyzer provided in the mapping, default one is used if not provided
  - __title._2gram__ ⇒ This will split up the title into parts having two words each, i.e. shingles of size 2.
  - __title._3gram__ ⇒ This will split up the title into parts having three words each.
  - __title._index_prefix__ ⇒ This will perform further edge-ngram tokenization on tokens generated under title._3gram.

We can test its behavior using our favourite *_analyze* API:

```json
Input string = "Harry Potter and the Goblet of Fire"

title._2gram: [ "harry potter", "potter and", "and the",
                "the goblet", "goblet of", "of fire" ]

title._3gram: [ "harry potter and", "potter and the",
                "and the goblet", "the goblet of", 
                "goblet of fire" ]

title._index_prefix on "goblet of fire" token from
title._3gram: [ "g", "go", "gob", "gobl", "goble", 
               "goblet", "goblet ", "goblet o", "goblet of" ]
```

How many sub-fields are to be created is decided by *max_shingle_size* parameter which defaults to 3 and can be set to 2, 3 or 4. *Search_as_you_type* is a text-like field, so [additional options](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/search-as-you-type.html#general-params) that we use for text fields(e.g. analyzer, index, store, search_analyzer) are also supported.

As you must've guessed it by now, it supports prefix as well as infix matches. While querying, we need to use [multi_match](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html) query as we need to target its sub-fields too:

```json
/**********Request**********/
GET /movies/_search
{
	"query": {
		"multi_match": {
			"query": "the goblet",
			"type": "bool_prefix",
			"analyzer": "keyword",
			"fields": [
				"title",
				"title._2gram",
				"title._3gram"
			]
		}
	}
}

/**********Response**********/
{
	"took": 14,
	"timed_out": false,
	"_shards": {...},
	"hits": {
		"total": {
			"value": 1,
			"relation": "eq"
		},
		"max_score": 3.0,
		"hits": [
			{
				"_index": "movies",
				"_type": "_doc",
				"_id": "r03hm3gBuCt11zp-Z_lC",
				"_score": 3.0,
				"_source": {
					"title": "Harry Potter and the Goblet of Fire"
				}
			}
		]
	}
}
```

We have set query type to [bool_prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-bool-prefix) here. The query will match documents having titles in any order, but documents having order that matches to the text in query will be ranked higher. In above example, we have passed "the goblet" as query text so documents having title as "the goblet of fire" will be ranked higher than documents having title "fire goblet".

Also, we have specified query analyzer to be keyword, so that our query text "the goblet" won't be analyzed and will be matched as it is. Without this, along with documents having title as "Harry Potter and the Goblet of Fire", documents having title "Harry Potter and the Deathly Hallows – Part 1"  would also match.

This is not the only way to query a *search_as_you_type* field, but certainly more suitable for our auto-complete use-case.

Like edge-n-gram, *search_as_you_type* overcomes the limitation of prefix query approach by storing data that is optimized for auto-completions. So in this approach too, we have to be careful about things we are storing using this field. Additional space is required for storing these n-grammed tokens.

In [part III](https://www.learningstuffwithankit.dev/implementing-auto-complete-functionality-in-elasticsearch-part-iii-completion-suggester), we will talk about completion suggesters, another index time approach which further speeds up queries by storing suggestions in-memory. Do let me know your feedback on this part in comments below.
