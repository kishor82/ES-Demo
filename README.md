
## Basic 
Elasticsearch is a distributed search engine that uses Apache Lucene under the hood to manage and store data.  It's an ideal solution for various use cases such as full-text search, analytics, and logging. Some reasons why you might consider using Elasticsearch over SQL databases or MongoDB include:

1.  Scalability: Elasticsearch is built to scale horizontally, meaning that you can easily add more nodes to the cluster to handle increased traffic.
    
2.  Full-Text Search: Elasticsearch has powerful full-text search capabilities, making it easy to search through large amounts of text data.
    
3.  Real-Time Analytics: With its built-in aggregation capabilities, Elasticsearch allows you to perform real-time analytics on large amounts of data.
    
4.  Easy to use: Elasticsearch provides a simple, RESTful API for indexing, searching, and managing data, making it easy to get started.
    
5.  High Performance: Elasticsearch is designed for high performance and can handle thousands of queries per second, making it suitable for real-time search and analytics.
6. NoSQL Data Model: Elasticsearch has a NoSQL data model, which allows you to store structured, semi-structured, and unstructured data without having to define a fixed schema. This makes it a good choice for use cases that require flexible data models.

Internally, Elasticsearch uses an inverted index to store the data, which is optimized for fast search operations. When a document is added to an index in Elasticsearch, the inverted index is updated to include the new information. This allows Elasticsearch to quickly search through all the data in an index to find the relevant results.

An inverted index is a data structure that stores the mapping between the content of documents and their location. In Elasticsearch, an inverted index is used to search for documents that match a query.

Each document is processed and split into terms (words or tokens), and for each term, the inverted index contains a list of document IDs that contain that term. When a search query is executed, the terms in the query are also looked up in the inverted index, and the intersection of the lists of document IDs is returned as the search result.

In this way, the inverted index enables fast searching by eliminating the need to scan through every document in the database. Instead, only the relevant documents, as determined by the inverted index, need to be examined.

In comparison to SQL databases and MongoDB, Elasticsearch is designed specifically for searching and analyzing large amounts of data in real-time. It also has a powerful query language, known as the Query DSL, which allows for complex search operations to be performed easily. SQL databases and MongoDB can also be used for search, but they are typically not as optimized for search operations as Elasticsearch is.

Overall, if you have a use case that requires full-text search, scalability, real-time search, analytics, and flexible data modeling, then Elasticsearch may be the right choice for you.



## 1. Match Query

The match query is used to search for a specific string in one or multiple fields. The search results are ranked based on relevance.

```bash
GET /index/_search
{
    "query": {
        "match": {
            "field_name": "search_string"
        }
    }
}

```

## 2. Term Query

The term query is used to search for exact match of a value in a field. It is not analyzed, so it's useful for searching for specific values like numbers or dates.

```bash
GET /index/_search
{
    "query": {
        "term": {
            "field_name": "search_value"
        }
    }
}

```


## 3. Range Query

The range query is used to search for values within a range of values in a field.
```bash
GET /index/_search
{
    "query": {
        "range": {
            "field_name": {
                "gte": "lower_value",
                "lte": "upper_value"
            }
        }
    }
}

```

## 4. Wildcard Query

The wildcard query is used to search for values that match a pattern. The `?` symbol matches one character and the `*` symbol matches zero or more characters.

```bash
GET /index/_search
{
    "query": {
        "wildcard": {
            "field_name": "search_pattern"
        }
    }
}
```

## 5. Fuzzy Query

The fuzzy query is used to search for values that are similar to the search term. The `fuzziness` parameter can be set to specify how similar the values should be.
```bash
GET /index/_search
{
    "query": {
        "fuzzy": {
            "field_name": {
                "value": "search_value",
                "fuzziness": "AUTO"
            }
        }
    }
}
```



## Search all documents

```bash
GET /_search
{
    "query": {
        "match_all": {}
    }
}

```

## Search documents with multiple fields
```bash
GET /_search
{
    "query": {
        "multi_match": {
            "query": "search_term",
            "fields": ["field_name_1", "field_name_2", ...]
        }
    }
}

```

## Search documents using bool query

```bash
GET /_search
{
    "query": {
        "bool": {
            "must": [
                { "match": { "field_name_1": "search_term_1" } },
                { "range": { "field_name_2": { "gte": "start_range", "lte": "end_range" } } }
            ]
        }
    }
}	
```

## Facet Search
A common use case for facet search in Elasticsearch is e-commerce product search. For example, consider a scenario where you have a large product catalog with different types of products and various attributes such as color, brand, price range, etc.

```bash
POST /product_catalog/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "categories": {
      "terms": {
        "field": "category.keyword"
      }
    }
  }
}

```

This query returns the aggregated count of documents in the `product_catalog` index, grouped by the `category` field. The result would look something like this:

```bash
{
  ...
  "aggregations": {
    "categories": {
      "buckets": [
        {
          "key": "electronics",
          "doc_count": 20
        },
        {
          "key": "books",
          "doc_count": 15
        },
        {
          "key": "furniture",
          "doc_count": 10
        }
      ]
    }
  }
}
```

Here, the faceting result shows the number of documents in the `product_catalog` index that belong to each of the three categories: electronics, books, and furniture.

# Analyzer
Suppose you have an e-commerce website and you want to implement a search functionality for your product catalog.

One of the use cases is that a user may search for a product using misspelled keywords or using different variants of the same word (e.g. "shirt" vs "shirts"). To handle such cases, you can use an analyzer that normalizes the user's query and matches it against the normalized product names in your catalog.	

```bash
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "product_name": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "product_name": "Shirt for Men"
}

PUT my_index/_doc/2
{
  "product_name": "Shirt for Women"
}

GET my_index/_search
{
  "query": {
    "match": {
      "product_name": "shirts"
    }
  }
}
```
In this example, we used an analyzer that performs the following operations on the product names:

-   Tokenizes the text into terms using the "standard" tokenizer.
-   Transforms all terms to lowercase using the "lowercase" filter.
-   Removes diacritics (such as accents) from the terms using the "asciifolding" filter.

In Elasticsearch, there are several types of analyzers available, including:

1.  Standard Analyzer: It is the default analyzer and breaks text into terms based on whitespace and punctuation.
    
2.  Simple Analyzer: It breaks text into terms based on whitespace only.
    
3.  Whitespace Analyzer: It breaks text into terms based on whitespace.
    
4.  Stop Analyzer: It is similar to the Standard Analyzer but removes common words such as "the," "and," and "a."
    
5.  Keyword Analyzer: It treats the entire input as a single term and does not perform any tokenization.
    
6.  Pattern Analyzer: It uses a regular expression to break text into tokens.
    
7.  Language Analyzers: These are language-specific analyzers that are used to handle text written in specific languages.
    
8.  Custom Analyzer: It allows users to combine various character filters, tokenizers, and token filters to create a custom analyzer.