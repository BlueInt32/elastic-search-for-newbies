#Elasticsearch Basics

Elasticsearch (ES) is a kind of NoSQL distributed full-text database engine. It is designed to store data as **Documents**. It is based on Lucene (written in Java) and exposes an API layer.
The key features of ES are :
* Speed
* Search relevancy
* Statistical analysis tools

ES is better used with systems that need a lot of search or analysis.

## Setting Up
- Java
- setting up the JAVA_HOME environnemnt variable
- Elastic search binary cf http://ruilopes.com/elasticsearch-setup
- Maven plugin

## Nodes and Clusters
A **Node** is a physical server on which elastic search is installed. It is created with the installation of ES on the computer.

A **Cluster** is a group of **Nodes**, with **Shards** distributed among them equally.

By default, ES uses the TCP port 9200 and exposes an API on it.

Therefore, on a given ES **Node**, we can make the following requests : 

* Get basic infos on the **Node**, the **Cluster**, its state and the version of ES installed
        GET /

* Get a list of all the **Indices** which have been created on the **Node**
        GET /_cat/indices 
        
## Creating an Index and its Mapping
An **Index** relates to the concept of *Database* in the SQL world. Thus, it references a group of **Types** (SQL *Tables*) provided via a **Mapping** (SQL *Schema*).

When we want to create an **Index** called "my_blog", he needs to provide its structure via a POST request in the form of a JSON **Mapping** giving the details of the **Types** it contains :

        POST my_blog
        {
          "mappings":{
            "post":{
              "properties":{
                "user_id":{
                  "type":"integer"
                },
                "post_text":{
                  "type":"string"
                },
                "post_date":{
                  "type":"date"
                }
              }
            }
          }
        }

A **Type** contains **Properties** (SQL *columns*) and each **Property** has its own datatype (integer, string, date, etc..).

The mapping can be retrieved hereafter using the following GET request
        
        GET my_blog/_mapping

The **Index** can be deleted like so 

        DELETE my_blog  

## Shards
A **Shard** is a logical subdivision of the **Index**. It is managed automatically by ES. If a **Cluster** has several **Nodes**, an **Index** will have its **Shards** spread equally on all the **Nodes** to maximize the power of parallellism. 
An index can be given a setting about its number of shards during its creation. 

        POST my_blog
        {
          "settings":{
            "index":{
              "number_of_shards":5
            }
          },
          "mappings":{
            ...
          }
        }  

## Documents

### Creating Documents
Once the mapping has been created, we want to insert data into the **Index**. An entry of one **Type** is called a **Document**. To insert a **Document**, we use the following POST request :

        POST my_blog/post
        {
          "post_date":"2015-08-30",
          "post_text":"This is a blog post",
          "user_id":1
        }

### Searching for Documents
To retrieve all the documents of one Type, we use the following request : 

        GET my_blog/post/_search

This will return something like the following JSON :

        {
          "took": 1,
          "timed_out": false,
          "_shards": {
              "total": 5,
              "successful": 5,
              "failed": 0
          },
          "hits": {
              "total": 1,
              "max_score": 1,
              "hits": [
                {
                    "_index": "my_blog",
                    "_type": "post",
                    "_id": "AU964wxetHCdRLXP4ncX",
                    "_score": 1
                }
              ]
          }
        }

### Id generation and custom Ids

We can see that ES generated an _id for us. We can retrieve the document back with the following request : 

        GET my_blog/post/AU964wxetHCdRLXP4ncX
returning :
        
        {
          "_index": "my_blog",
          "_type": "post",
          "_id": "AU964wxetHCdRLXP4ncX",
          "_version": 1,
          "found": true,
          "_source": {
              "post_date": "2015-08-30",
              "post_text": "This is a blog post",
              "user_id": 1
          }
        }
        
But instead we could create our own id, preventing ES to generate it 
        
        POST my_blog/post/my_id
        {
          "post_date":"2015-08-30",
          "post_text":"This is a blog post with my id",
          "user_id":2
        }

## Customizing the Mapping of Indices

### Forcing the date format on date Properties
A **Property** of datatype date can be given a specific format, leading to errors if other formats are provided during the creation of **Documents**

        "post_date":{
          "type":"date",
          "format":"YYYY-MM-DD"
        }
### Specifying Property Storage behaviour
ES is often used simultaneously with other database engines to provide search functionality on its side. By default, ES will store all the data defined in the mapping, but this can lead to a lot of redundancy. We can tell ES to disable the storage by default and chose which property to store to reduce the memory allocated by the index at the **Type** level :

        POST my_blog
        {
          "mappings":{
            "post":{
              "_source":{
                "enabled":false
              },
              "properties":{
                "user_id":{
                  "type":"integer",
                  "store":true
                },
                ...
              }
            }
          }
        }
This setting is useful especially when disk space is a real concern, but it makes documents trickier to retrieve and debug.

### The _all setting
By default, ES stores an additional property containing all the data from the properties, in our case "user_id", "user_text" and "user_date". This can be disabled to decrease the size of the index using the following setting, at the **Type** level like the storage setting :

        POST my_blog
        {
          "mappings":{
            "post":{
              "_all":{
                "enabled":false
              },
              "properties":{
                ...
              }
            }
          }
        }

## Routing and Aliases

## Querying

### Simple Query

### Query DSL


