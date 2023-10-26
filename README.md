# udemy-elasticsearch

## Section 1: Introduction

Github: https://github.com/codingexplained/complete-guide-to-elasticsearch

Demo kibana: https://demo.elastic.co/app/kibana#/dashboard/welcome_dashboard

Scope:
- Only Elasticsearch, not Elastic Stack.
- Allows more detail.

Learn:
- Write complex queries
- No specific use case

What can be done with ElasticSearch:
- Full text search
- Query & analyze structured data
- Application Performance Management (APM): Analyze application logs and system metrics
- Send events to Elasticsearch + Aggregates
- Analyzing lots of data
    - Forecast future values with ML
    - Anomality detection

How does it work?
- Data is stored as documents
- Document ~ Row in MySQL
- Document contains fields ~ Columns
- Document is a JSON object
- Documents are requested via REST API
- Written in Java, over Apache Lucene
- Easy to use
- Highly scalable

Elastic Stack:
- Elastic Search is the core of the stack
- Kibana: analytics & visualization platform
    - Dashboard
    - Create charts
    - Configure anomaly detection & predictions
    - Is a web UI for Elastic Search
- Logstash
    - born as a processor of logs from applications
    - Data processing pipeline
    - Input plugins <- sources of information
    - Filter plugins <- CSV / XML / JSON
    - Output plugin (stashes) -> elastic search or other
    - Configurable pipeline
- X-Pack
    - Adds additional features to the ElasticSearch & Kibana
    - Security: authentication and authorization
        - Integrate with authentication services
        - Control premissions with fine-grained authorization
    - Monitoring: how the Elastic Stack is running
        - Get notified if something goes wrong
    - Alerting
    - Reporting
        - Export Kibana visualization to PDFs (or other)
        - Generated on-demand or scheduled
        - Export data for CSV files
    - Machine Learning
        - Abnormality Detection
        - Forecasting
    - Graph: Analyze relationships in the data
        - Related songs,
        - Popular vs. Relevant
            - The uncommonly common signals
            - Only looking to popular is misleading
        - Integrate into application with API
        - Visualize relationships within Kibana
        - Works out of the box
    - Elastic SQL
        - Elasticsearch queries are written in Query DSL (json object)
        - Flexible but a bit verbose at first
        - Translates SQL to -> DSL (also an API)
- Beats
    - Collection of data shippers that send information to logstash
    - Filebeat
        - Useful Error logs
    - Metricbeat
        - CPU, memory, etc.

ELK stack:
    - Elasticserach
    - Logstash
    - Kibana

Architecture
- Example of an E-commerce application
- Data is stored in a relational database
- Fulltext is not gret in SQL
- Elastic search is included for that
    - But it means duplicating data
    - What if we already have data? script import
- Implement a dashboard
    - use Kibana
- Monitor server resources
    - Metricbeat ->[ingest node] -> Elasticsearch
    - Visualize in Kibana
- Optionally set up alerting
- Add monitoring for google analytics
- Identify bad deployments <- anomaly detection
    - Filebeat
- Advanced processing of events
    - Logstash
    - Centralizing processing
    - Ideally all events should go through Logstash
- Application should only be reading from ElasticSearch

## Section 2: Getting Started

Installation options
- Elastic Cloud -> free trial on: https://www.elastic.co/es/cloud/cloud-trial-overview
- Docker -> https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
- Linux
    - Elasticsearch: https://www.elastic.co/es/downloads/elasticsearch
    - Kibana: https://www.elastic.co/es/downloads/kibana

### Docker
After setting up `.env` file, run Elastic Stack using:
```
docker-compose up -d
```

Stop them using
```
docker-compose down
```

### Linux

Install:

```
mv elasticsearch{-8.10.2-linux-x86_64,}.tar.gz
mv kibana{-8.10.2-linux-x86_64,}.tar.gz

tar -zxf elasticsearch.tar.gz
tar -zxf kibana.tar.gz

mv elasticsearch{-8.10.2,}
mv kibana{-8.10.2,}

rm -rf *.tar.gz
```

Execute:

```
elasticsearch/bin/elasticsearch -E http.port=51450
kibana/bin/kibana -E http.port=51450
```

### Basic architecture

- Nodes
    - Elasticsearch set of nodes: instances that stores data.
    - More nodes -> more data
    - Each node stores a part of the data
    - Although not needed, in production it is recommended to have one node per machine, virtual machine or container.
- Each node belongs to a Cluster.
    - Clusters are independent from each other.
    - It is possible to perform cross-cluster, but not recommended.
    - Clusters respond to logical separations and allow different settings.
- A document is the unit of information stored in the nodes.
    - JSON object
    - Original data `_source` + metadata
    - Organized by indices
- Index is a collection of documents with similar structure.

### Inspecting the cluster

- In Kibana, go to the left menu > Management > Dev Tools
- Kibana uses the Elasticsearch API
- Using Kibana, it is not needed to specify the network address.
- GET {api_path}/{command}
- Examples:
    - `GET /_cluster/health`
    - `GET /_cat/nodes?v`
    - `GET /_cat/indices?v`
    - `GET /_cat/indices?v&expand_wildcards=all` -> shows "system indices" (hidden)

### Sending queries with cURL

- Kibana is the easiest way to send queries
- Without certificate
```
curl --insecure -X GET https://localhost:<port> ...
```
- With certificate
```
curl --cacert config/certs/http_ca.crt -u elastic -X GET -H "Content-Type:application/json" https://localhost:<port> -d '{ "query": { "match_all": {} } }'
```

### Sharding and scalability

- Sharding is a way to divide indices into smaller pieces, each called `shard`
- It is done at the index level
- This allows horizontal scaling the data volume
- A shard is an independent index
    - Apache Lucene Index
- A shard has no predefined size; it grows as new documents are added to it.
- A shard may store up to about two billion documents.
- Enables parallelization of queries
    - The same query can be executed in multiple shards, increasing the response time and the throughput.
- Primary shards
- Over-sharding -> having too many shards
- Elasticsearch < 7.0.0 indices had 5 shards
- Now 1 shard by default, using
    - Split API to increase the number of shards.
    - Shrink API to reduce the number of shards.
- Try to anticipate in a mid-near future the number of documents an index will contain.
- How many shards are optimal?
    - It depends, no correct answer
    - Factors, # nodes, # indices, # queries, sizes, etc.
    - Millions of documents -> 2 shards

### Understanding replication

- Replication
- An index consisting of one shard, might be totally lost if the hard disk drive fails
- Elasticsearch supports replication for fault tolerance natively.
- Replication is extremely easy in Elasticsearc
- Replication is configured at index level, copying shards.
- A shard that has been replicated, is a primary shard
- Primary and replicas -> replication group
- A replica shard can server search requests
- The number of replicas can be configured at index creation
    - One replica as default
- Replica shards are never stored in the same node as the primary shard
- Replication only makes sense when more than one node is available.
    - Elasticsearch won't allow replication with one single node.
- How many?
    - One or two replicas is usually enough
    - Is it OK to have data unavailable while restoring?
    - Critical systems -> 2+ replicas
- Snapshots is a way to create backups.
    - Index level and/or cluster
    - It does NOT provide high availability
    - Maybe useful before delicate queries
- Replication -> high availability + performance
    - Works with live data
    - Increases throughput -> allows the Nodes to parallelize the search queries among the replica shards.
- Create a new index:  `PUT /<index name>`
    - Changes the health status from `green` to `yellow` if there is only one node available. This is because replication has not been possible and the `yellow` status is a warning for potential data loss.
- List shards: `GET /_cat/shards?v`
- .kibana indices are configured in one `shard` and `zero`replicas:
    - auto_expand_replicas: will automatically expand replicas as new nodes are available.

### Adding more nodes to the cluster

- Both for data volume and for replication, eventually we'll need to add more nodes to the cluster
- Cloud deployments are handled via the web console.
- For local deployments:
    - `GET /_cluster/health` -> `number_of_nodes`
    - `GET /_cat/shards` -> check how shards have been distributed.
    - Simply *extract* the archive as many times as new nodes you want to add.
        - Do NOT copy the existing folders.
    - Edit `<node-folder>/config/elasticsearch.yml` file
        - Uncomment and set `node.name: <node-name>`
    - New nodes need an enrollment token
        - Open a terminal in the folder of the running node.
        - `bin/elasticsearch-create-enrollment-token --scope node` -> copy output
        - Go to the folder of the new node.
        - `bin/elasticsearch --enrollment-token <pasted-token>`
- As new nodes are added, the status of the cluster changes from `yellow` to `green` because it can start replicating the shards.
- Production environments are more complicated
- (!) After adding a third node, at least two nodes are required to run the cluster.
- Shutting down one node (ctrl+C)
    - Look at the `master node`
    - Shards are reorganized in about 60s

### Overview of node roles

- Each node can have one or more roles
    - `Master-eligible`: cluster-wide actions, creating/deleting indices, etc.
        - the cluster elects the master node, it is not manually-assignable
    - `Data`: Enables a role to store data and perform queries.
        - dedicated master nodes will not have this role
    - `Ingest`: Enables a role to perform ingest pipelines.
        - Processors
        - Simplified version Logstash
        - Useful for relatively simple data transformations
    - `Machine learning`:
        - `node.ml`: enables the node to run ML jobs
        - `xpack.ml.enabled`: enables the node to respond to ML API requests
        - Useful for dedicated ML nodes.
    - `coordination`: Refers to the distribution of queries. Does not search any data.
        - There is not any specific role for this. Only disabling all previous nodes.
    - `Voting-only`: Rarely used. Cannot be elected master, but can vote for it.
- Use `GET /_cat/nodes?v` to see the roles under the columns:
    - `node.role`: abbreviated 'dim' means `Data`, `Ingest` and `Master`.
    - `master`: indicates which ones is the master
- When to change node roles:
    - It depends
    - Useful for large clusters
    - Typically done when optimizing the cluster to scale the number of requests
    - Better understand what hardwar resources are used for
    - Only change them if you know what you are doing.


## Section 3: Managing Documents

### Creating & deleting indices

- To delete an index use: `DELETE /<index-name>`
- To create an index with options: ```PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}```

### Indexing documents

- To add/index an object: ```POST /<index-name>/_doc
{
    "name": "Coffee Maker",
    "price": 64,
    "in_stock": 10
}```
- In the response,
    - `_shards` key indicates the replication of the new inserted data.
    - `_id`: autogenerated if not specified in the request.
- To create a document specifying its ID: ```PUT /<index-name>/_doc/<id>
{
    // body
}```
- The setting `action.auto_create_index`: enables index creation when adding documents.
    - default: true

### Retrieving documents by ID

- Use `GET /<index-name>/_doc/<id>`
- The object is under `_source` key. Also `found` key indicates if the element was found or not.

### Updating documents

- To update a document: ```POST /<index-name>/_update/<id>
{
    "doc": {
        "<field_name>": <new-value>
    }
}```
- The response `result` key is `updated`. It can also be `no-op`
- New fields can be added as well.
- Documents are immutable
    - Update requests *replaced* the document

### Scripted updates

- Subtract the field value by one
- It can contain if statements
- Request example: ```POST /<index-name>/_update/<id>
{
    "script": {
        "source": "ctx._source.in_stock--"
    }
}```
    - `ctx` is a variable
    - """ <- multi line statements
- Request with parameters: ```POST /<index-name>/_update/<id>
{
    "script": {
        "source": "ctx._source.in_stock -= params.quantity",
        "params": {
            "quantity": 4
        }
    }
}```
- By default, using scripted updates, the `result` key in the response will always be `updated`.
    - Unless explicitly set, for example: ```POST /<index-name>/_update/<id>
{
    "script": {
        "source": """
            if (ctx._source.in_stock == 0) {
                ctx.op = 'noop';
            }
            cxt._source.in_stock--;
        """
    }
}```
- It can also be deleted by setting `ctx.op = 'delete'`.

### Upserts

- If the document already exists, an update is done, otherwise it is inserted.
- Example: ```POST /<index-name>/_update/<id>
{
    "script": {
        "source": "ctx._source.in_stock++"
    },
    "upsert": {
        "name": "foo",
        "price": 999,
        "in_stock": 5
    }
}```
- The response's `result` key is `created` if the document didn't exist, `updated` otherwise.

### Replacing documents
- Replacing existing documents is the same query as creating them: ```PUT /<index-name>/_doc/<existing-id>
{
    // new body
}```

### Deleting documents
- To delete a document use: ```DELETE /<index-name>/_doc/<id>```

### Understanding routing
- How does Elasticsearch know the shards to be read or modified: routing
- Routing is the process of resolving a shard for a document.
    - `shard_num = hash(_routing) % num_primary_shards`
    - This is why the number of shards cannot be changed once an index is created
    - Changing this number requires Shrink and Split APIs, to recreate the whole index
- Routing is 100% transparent.
- Default routing strategy can be changed.
- Metafields:
    - `_routing` field is only specified if routing strategy is customized.

### How Elasticsearch reads data
1. A node receives the read request, this will be the coordinator
2. The coordinator resolves the replication group (routing)
3. Then uses an "adaptive replica selection" to choose a member of the replication group
    - ARS reduces response time
    - ARS is an intelligent load balancer
4. Sends the read request to the shards
5. Collects the request from the shard and responses its own coordinator.

### How Elasticsearch writes data
1. A node receives the write request, this will be the coordinator
2. The coordinator resolves the replication group (routing)
3. Write requests are routed to the primary shard
4. Primary shard validates the query
5. Peforms the write operation locally, then propagates the data to its replicas, async.

- How failures in replication are handled?
    - Recovery process
    - A new replica is elected primary shard, it might not have the last version of data.
    - Primary terms:
        - A way to distinguish between old and new primary shards
        - A counter of how many times the primary was changed
        - Each operation is given a primary term
        - Available within responses
    - Sequence number:
        - The primary shard increases the sequence number
        - It is used to know the order of operations
        - Available within responses
    - Checkpoints: For large indices, this process is expensive
        - Local and global checkpoints
        - Sequence number that all active shards within a replication group ahve been aligned _at least up to_.
        - When a failure happens, only operations performed since the checkpoint need to be applied to the failed shards.

### Understanding document versioning

- Not a revision history of documents.
- Versioning does not include reverting to a past version.
- `_version` is metadata field of the documents:
    - integer value, incremented by one when modifying a document.
    - retained for 60 seconds when deleting a document. (`index.gc_deletes`)
- Why?
    - How many times a document is modified
    - Not many used anymore
    - Used in the optimistic concurrency control
        - now there is a better way

### Optimistic concurrency control

- Prevent an old version of a document overriding a newer version.
- Possible due to concurrency
- Example:
    - 2 users of an e-commerce purchaes items at the same time.
    - `in_stock` field needs to be decremented twice, but it could be updated wrongly.
- In `GET /<index-name>/_doc/<id>` results, `_seq_no` and `_primary_term` are included in the results.
- Then, use the previous values in the following query:
```
POST /<index-name>/_update/<id>?if_primary_term=X&if_seq_no=Y
{
    "doc": {
        "in_stock": <new-value>
    }
}
```
- Only the first execution of the previous update will succeed. The second executing will fail.
- How to handle the failure?
    - Retrieve document again
    - Repeat calculations for the `new-value`
    - Send update request again.
    - Repeat until no error occurs.

### Update by query

- Similar `UPDATE WHERE` equivalent in SQL
- The query uses three previous concepts:
    - Primary terms
    - Sequence numbers
    - Optimistic concurrency control
- Query:
```
POST /<index-name>/_update_by_query
{
    "script": {
        "source": "ctx._source.in_stock--"
    },
    "query": {
        "match_all": {}
    }
}
```
- In the result, `updated` key shows the number of updated rows.
- Write process of a `update_by_query`:
    1. Requests gets to a coordinator node.
    2. A snapshot of the index is created
    3. A search query is sent to all the replication groups *sequentially*
        - If documents matched, a bulk request is sent to all matched documents.
    4. Errors are retried up to 10 times.
        - Updates done until the error will be kept.
        - Not a transaction-like update.
        - If all retries fail, the main query is aborted and non-updated shards won't get updated.
- The snapshot taken in step 2 prevents overwriting changes made *after* the snapshot was taken.
    - The query may take a while to finish if updating many documents
- Each document's primary term and sequence number is used
    - A document is only updated if the values match the ones from the snapshot
    - Optimistic concurrency control
    - If a version conflict occurs, the whole query is aborted
- The number of version conflicts is returned within the `version_conflicts` key.
- To avoid aborting, use `"conflict": "proceed"` in the query body.

### Delete by query

- Similar to `DELETE WHERE`
- Query:
```
POST /<index-name>/_delete_by_query
{
    "query": {
        "match_all": {}
    }
}
```
- In the result, `deleted` key shows the number of deleted rows.
- Internally works in the same way as `_update_by_query`.

### Batch processing

- Bulk API allows performing Index, Update, and Delete documents in a single query.
- It expects a NDJSON specification
- Query:
```
POST /_bulk
{ "index": { "_index": <index-name>, "_id": <id> } }
{ <field-name>: <field-value> }
{ "create": { "_index": <index-name>, "_id": <id> } } // this will fail if the document exists
{ <field-name>: <field-value> }
{ "update": { "_index": <index-name>, "_id": <id> } }
{ "doc": { <field-name>: <new-value> } }  // it allows script syntax as regular _update
{ "delete": { "_index": <index-name>, "_id": <id> } }
```
- Results will show the result of every operation in the bulk
- If `<index-name>` is the same in all operations, we can do it this way:
```
POST /<index-name>/_bulk
{ "index": { "_id": <id> } }
{ <field-name>: <field-value> }
{ "create": { "_id": <id> } } // this will fail if the document exists
{ <field-name>: <field-value> }
{ "update": { "_id": <id> } }
{ "doc": { <field-name>: <new-value> } }  // it allows script syntax as regular _update
{ "delete": { "_id": <id> } }
```
- When using `_bulk`, the header `Content-Type:application/x-ndjson`
    - Console, SDKs handles this automatically
- Each line (including the last one) must end with `\n`.
    - Especially when using files as input
- If an action fails, it does not affect other actions.
- When to use the Bulk API
    - Lots of write actions to avoid network roundtrips
    - Script that generates the request.
- Routing is done for each action.
- Optimistic concurrency control is supported using `if_primary_term` and `if_seq_no`.

### Importing data with cURL

- Let's import the `products-bulk.json` file as follows:
```bash
curl -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@path/to/products-bulk.json"
```
    - `--data-binary` is important so that the last `\n` in the file is considered.
- See shards:
```
GET /_cat/shards?v
```

## Section 4: Mapping & Analysis

### Introduction to Analysis

- AKA: Text Analysis
- Only applicable to text value
- Purpose: structure the text for efficient searching.
- The `_source` object is *not* used when searching for documents.
- Document -> Analyzer -> Storage
    - The analyzer consists of:
        - Character filters: Adds, removes or changes characters.
        - Tokenizer: Split the text into tokens. Characters may be removed during this process. Recalls the character index for each token.
        - Token filters:
            - Receives the output of the tokenizer
            - Adds, removes or modifies tokens
            - Multiple filters can be specified, in an ordered way
            - E.g. lowercase filter
    - Custom analyzers are possible
- Default analyzer:
    - Char filter - `none`: No character filter
    - Tokenizer - `standard`: Whitespace and special character clean up
    - Tokenizer filter - `lowercase`

### Using the Analyze API

- Use the `_analyze` API as follows:
```
POST /_analyze
{
    "text": "2 guys walk into    a bar, but the third... DUCKS! :-)",
    "analyzer": "standard"
}
```
- The result contains the tokens emitted by the analyzer
    - Metadata
    - Type: "<ALPHANUM>|<NUM>"
- Specify each step of the analyzer like this:
```
POST /_analyze
{
    "text": "2 guys walk into    a bar, but the third... DUCKS! :-)",
    "char_filter": [],
    "tokenizer": "standard",
    "filter": ["lowercase"]
}
```

### Understanding inverted indices

- Field values are stored differently depending on their data type.
    - E.g. Numeric, dates & geo are stored in BKD trees
- Searches are handled by Apache Lucene, not Elasticsearch itself.
- Inverted Indices
    - Mapping between terms/tokens <-> documents containing them
    - Relevance scoring is beyond these indices.
    - One for each text "field"

### Introduction to Mapping

- Mapping defines the structure of documents, how they are indexed and stored.
- Equivalent to table's schema in relational databases
- Explicit vs. dynamic mapping
    - Explicit: manually defined
    - Dynamic: automatically generated

### Overview of data types

- Data types: object, long, integer, boolean, text, double, short, date, float
- Special data types, like `IP`, are usually linked to some features/use cases of ElasticSearch.
- `object`
    - Any json object
    - nested objects
    - mapped using the `properties` parameter
    - objects are NOT stored as objects in Apache Lucene
        - ES transforms them to ensure they are indexed.
        - objects are flattened, i.e. `{a: {b: {c: 'd'}}} = a.b.c =='d'`
        - Arrays of objects are flattened as well grouping all properties in the top-level.
            - This causes problems when searching (see slides)
- `nested`
    - Maintains object relationships
    - Query objects independently
    - Use `nested` query
    - Nested objects are stored as hidden documents
- `keyword`
    - Only for exact matching
    - Filtering, sorting & aggregations
    - E.g. Status `READY`, `FAILED`, etc.
    - Use `text` for full-text searches

### How the `keyword` data type works

- `keyword` fields use `keyword` Analyzer
- It is a no-op analyzer
- It can be changed to add a `filter`, like `lowercase`.

### Understanding type coercion

- Data types are validated when indexing documents.
    - Some invalid values are rejected
    - E.g. index object in a `text` field
- When creating the index, the fields are created with the dynamic mapping.
- After that, new documents may trigger the type coercion.
    - Infer the type of new document's values
    - Map them to the expected data type
    - If a mapping exists, apply and continue, fail otherwise.
    - The `_source` key keeps the original value.
- Coercion is enabled by default, but can be disabled.

### Understanding arrays

- It does not exist
- Any field may contain zero or more values
    - No configuraiton or mapping needed
- We can store `tags: "A"` or `tags: ["B", "C"]` indistinguishably.
- Text arrays are concatenated with a ' '.
- Array values should be of the same data type
    - Or can be coerced into the same data type
    - Coercion needs to be created first
- Nested arrays are also possible, but they will be flattened

### Adding explicit mappings

- Field mappings can be created:
    - When creating a new index
    - Afterwards
- Query example:
```
PUT /reviews
{
    "mappings": {
        "properties": {
            "rating": { "type": "float" },
            "content": { "type": "text" },
            "product_id": { "type": "integer" },
            "author": {
                "properties": {
                    "first_name": { "type": "text" },
                    "last_name": { "type": "text" },
                    "email": { "type": "keyword" }
                }
            }
        }
    }
}
```

### Retrieving mappings

- Query:
```
GET /<index-name>/_mappings
GET /<index-name>/_mappings/field/<field-name>
GET /<index-name>/_mappings/field/author.email // object dot notation
```

### Using dot notation in field names

- Query example:
```
PUT /reviews_dot_notation
{
    "mappings": {
        "properties": {
            "rating": { "type": "float" },
            "content": { "type": "text" },
            "product_id": { "type": "integer" },
            "author.first_name": { "type": "text" },
            "author.last_name": { "type": "text" },
            "author.email": { "type": "keyword" }
        }
    }
}
```
- It can also be used in search queries

### Adding mapping to existing indices

- Add a field to existing mapping
- Example Query:
```
PUT /review/_mapping
{
    "properties": {
        "created_at": { "type": "date" }
    }
}
```

### How dates work in Elasticsearch

- Specified in one of these ways:
    - Specially formatted strings
    - Milliseconds since epoch (long)
    - Seconds since epoch (integer)
- Default behavior
    - Formats:
        - Date without time
        - Date with time
        - Millis from epoch
    - UTC timezone if not specified
    - ISO 8601 specification
- Dates stored
    - Converted to Millis from epoch (long)
    - Returned inside `_source` as they were specified when created
- When adding dates DO NOT specify Unix timestamps (seconds since epoch), because Elasticsearch will interpret them as milliseconds. The query won't fail, but the data will be corrupted.

### How missing fields are handled

- All fields in Elasticsearch are optional
- You can leave out a field when indexing documents
- Integrity checks need to be done at the application level
- Adding a field mapping does NOT make the field required

### Overview of mapping parameters

- Not an exhaustive list of parameters, to be used in `{ "type": "...", <parameter>: <value>}`
- `format`
    - use to customize the format for `date` fields.
    - `"strict_date_optional_time||epoch_millis"`
    - using Java's `DateFormatter` syntax: `dd/MM/yyy`
    - built-in formats: `epoch_second`
- `properties`
    - used for `object` and `nested` fields.
- `coerce`
    - enable/disable type coercion of values
    - value: <boolean>
    - `index.mapping.coerce` to enable it in the index-level.
- `doc_values`
    - used to specify the data structure used by Apache Lucene
    - "uninverted" inverted index.
    - used for sorting, aggregations, and scripting
    - additional data structure
    - set `doc_values: false` to save disk space and increase indexing throughput and only if you know the data for aggregation, sorting, or scripting.
    - cannot be changed without reindexing documents
- `norms`
    - normalization factors use for relevance scoring
    - filter results, but also _rank_ them
    - set `norms: false` to save disk space, but only if you know that it will be used in rank
- `index`
    - disables indexing for a field
    - will still be in the `_source` object
    - saves disk space and increases indexing speed.
    - often used in timeseries data.
- `null_value`
    - `NULL` values cannot be indexed or searched
    - Used to replace `NULL` values with another value
    - Only works for explicit `NULL` values.
    - The replacement value must be of the same data type as the field.
- `copy_to`
    - copy values into another field `group field`
    - e.g. `first_name` and `last_name`
    - the group field won't appear in the `_source` object.

### Update existing mappings

- In general we cannot update field mappings
    - Only certain mapping parameters allow it.
- Documents may already been analyzed and it would mean rebuilding the whole data structure.

### Reindexing documents with the Reindex API

- Create a new index with the altered mapping.
- Query:
```
POST /_reindex
{
    "source": {
        "index": "source_index",
        // Optional
        "query": {
            "match_all": {}
        },
        "_source": [<field1>, <field2>, ...]
    },
    "dest": {
        "index": "destination_index"
    },
    // Optional
    "script": {
        "source": """
            // Type conversion
            if (ctx._source_product_id != null) {
                ctx._source.product_id = ctx._source.product_id.toString();
            }
            // Renaming
            ctx._source.comment = ctx._source.remove("content");
            // Delete
            ctx.op = "delete";
        """
    }
}
```
- `_source` object does not reflect how values were indexed
- We can modify the `_source` value while reindexing using the `script` key.
- We can select a subset of documents using the `source.query` field.
- We can remove fields by using the `source._source` filter.
- We can remove entire documents by using `ctx.op` in the `script.source` key.
- Reindex API uses Scroll API internally to perform massive operations without affecting the general performance.

### Defining field aliases

- Just renaming a field might not be enough reason to reindex all documents.
- Aliases is an easier alternative
- They are defined in the mapping as follows:
```
PUT /<index-name>/_mapping
{
    "properties": {
        "<alias-name>": {
            "type": "alias",
            "path": "<field-name>""
        }
    }
}
```
- Aliases can be updated to point to a different field
- Aliases can also be applied to indices

### Multi-field mappings

- A field can be mapped into different mappings
- We cannot run aggregations on `text` fields. `keyword` needs to be used.
- This is specified as followed:
```
PUT /<index-name>
{
    "mappings": {
        "properties": {
            "<field-name>": {
                "type": "text",
                "fields": {
                    "keyword": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
}
```
- To use the new mapping:
```
GET /<index-name>/_search
{
    "query": {
        "term": {
            "<field-name>.keyword": "<value>"
        }
    }
}
```
- This can be used for adding synonyms in the same index of the content

### Index templates

- Defines settings and mappings to indices that match one or more patterns.
- Patterns may include wildcards (*)
- Templates only apply to new indices.
- Example query:
```
PUT /_template/<template-name>
{
    "index_patterns": [
        "<indexname-withwildcards-*>",
    ],
    "settings": {
        "number_of_shards": 2,
        "index.mapping.coerce": false
    },
    "mappings": {...}
}
```
- Time series usually create an index for each day or month.
- After creating the template, if a new index is created:
```
PUT /<index-name>
```
- New indices will merge their configuration with the template's.
    - Request's settings have more priority.
- Index templates can be updated, but they will only affect new indices.
- Use `DELETE` verb to delete templates.

### Introduction to the Elastic Common Schema (ECS)

- A specification of common fields and how they should be mapped
- Before ECS there was no cohesion between field names.
- Example: `@timestamp`
- Use-case independent
- Group of fields are called `field set`.
- Documents are called `events`.
- Documentation: https://www.elastic.co/guide/en/ecs/current/ecs-reference.html

### Introduction to dynamic mapping

- It indends to make Elasticsearch easier to use.
- Field mapping is created when a document is added to a newly created index.
- Text fields are automatically created with `keyword` multi-field mapping.
- Typical mappings:
    - string -> text + keyword mapping | date | float | long
    - integer -> long
    - float -> float
    - boolean -> boolean
    - object -> object
    - array -> depends on the first non-null value

### Combining explicit and dynamic mapping

- Both mapping types can be combined, they are note exclusive.

### Configuring dynamic mapping

- Disable dynamic mapping when creating an index:
```
PUT /<index-name>
{
    "mappings": {
        "dynamic": false,
        ...
    }
}
```
- Documents are indexed, even though they include more fields than the ones defined in the mapping.
- The data is available in `_source`.
- But the documents won't be found by the new fields.
- New fields must be mapped explicitly, or else they will be _ignored_
- For a document containing extra fields to be _rejected_, set the setting to `dynamic: "strict"`.
- The `dynamic` setting can be set up in multiple levels of the mapping hierarchy.
- Numeric detection: Use `numeric_detection` setting
- Date detection:
    - Use `date_detection` to enable/disable
    - Use `dynamic_date_formats` to set up the date format

### Dynamic Templates

- They are used to configure the default behavior of the dynamic mapping when encountering a new field.
- Set it up:
```
PUT /<index-name>
{
    "mappings": {
        "dynamic_templates": [
            {
                "integers": {
                    "match_mapping_type": "long",
                    "mapping": {
                        "type": "integer"
                    }
                },
                "strings_only_text": {
                    "match_mapping_type": "string",
                    "match": "text_*",
                    "unmatch": "*_keyword",
                    "mapping": {
                        "type": "text"
                    }
                },
                "strings_only_keyword": {
                    "match_mapping_type": "string",
                    "match": "*_keyword",
                    "mapping": {
                        "type": "keyword"
                    }
                },
                "strings": {
                    "match_mapping_type": "string",
                    "mapping": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 512 // example value different than the default.
                            }
                        }
                    }
                },
                "no_doc_values": {
                    "match_mapping_type": "*",
                    "mapping": {
                        "type": "{dynamic_type}",
                        "index": false
                    }
                }
            }
        ]
    }
}
```
- The `match` and `unmatch` parameters are used to set up conditions based on the name of the fields.
    - Support wildcards (*)
    - `match_pattern: regex` would change the behavior of `match` to accept regexes.
- The `path_match` and `path_unmatch` extend the functionality to dot-notation path.
- Placeholders:
    - `{dynamic_type}` is replaced by the type that was detected by dynamic mapping.
- Index templates vs. dynamic templates
    - Index template apply settings to matching indices
    - Dynamic templates appply settings to matching fields within an index.

### Mapping recommendations

- Explicit mapping for production
- Save disk space with optimized mappings when storing many documents
- Setting `dynamic` to `strict` avoids surprises
- Don't always map strings as `text` and `keyword`.
    - `text` -> full-text searches
    - `keyword` -> aggregations, sorting, filtering on exact values
- Disable coercion, provide the correct data types
- Use appropriate numeric data types
- Mapping parameters when dealing with lots of parameters
    - `doc_values: false` if sorting, aggregations and scripting not needed
    - `norms: false` if relevance score is not needed
    - `index: false` if filtering is not needed

### Stemming & stop words

- Standard analyzer does not do stemming nor stop words
- Stemming is the process to reduce words to their root form, e.g. "loves" -> "love"
- Stopwords are filtered out during text analysis, e.g. "a" , "the", "at", ...
- Do not remove them by default.

### Analyzers and search queries

- Analyzers are also used during search time
- When filtering by a text field, the same analyzer is used.
- Searcn & indexer analyzer can be set up differently, but it is very rare.

### Built-in analyzers

- `standard`: splits texts at word boundaries and removes punctuation
    - `standard` tokenizer
    - lowercase
- `simple`:
    - tokenizer: splits by anything not a letter
    - `lowercase`tokenizer
- `whitespace`
    - not lowercase
- `keyword`:
    - no operation
- `pattern`
    - a regex is used for separator
    - default to (\W+)
    - lowercases by default

### Creating custom analyzers

- Query:
```
PUT /<index-name>
{
    "settings": {
        "analysis": {
            "filter": {
                "danish_stop": {
                    "type": "stop",
                    "stopwords": "_danish_"
                }
            },
            "char_filter": {},
            "tokenizer": {},
            "analyzer": {
                "my_custom_analyzer": {
                    "type": "custom",
                    "char_filter": [
                        "html_strip"
                    ],
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "danish_stop", // custom token filter
                        "asciifolding"
                    ]
                }
            }
        }
    }
}
```
- Remember, to test analyzer settings use:
```
POST /_analyze
{
    "analyzer": "my_custom_analyzer"
    "text": "..."
}
```

### Adding analyzers to existing indices

- Query:
```
PUT /<index-name>/_settings
{
    "analysis": {
        // Same format as in previous section
        "analyzer": ...
    }
}
```
- An open index is an index that is available for indexing and search requests
- A close index will refuse requests
    - Read and write requests are blocked
- Dynamic & static settings
    - Dynamic can be changed without closing the index first -> No downtime
    - Static require the index to be closed
- Analysis settings are static settings
- How to close the index?
```
POST /<index-name>/_close
```
- How to open the index?
```
POST /<index-name>/_open
```
- Open & close might not be acceptable -> Alternative: Reindex document into a new index
    - Create new index with the updated settings
    - Use an index alias for the transition

### Updating analyzers

- Analyzer can be overriden when searching, e.g.
```
GET /<index-name>/_search
{
    "query": {
        "match": {
            "description": {
                "query": "that",
                "analyzer": "keyword"
            }
        }
    }
}
```
- To update the analyzer of an index, use the same syntax as in the previous section.
- When updating the analyzer, some documents may not be indexed with the latest version of the analyzer. Therefore, they need to be reindexed. To do so:
```
POST /<index-name>/_update_by_query?conflicts=proceed
```

## Section 5: Introduction to Searching

### Introduction to searching

- URI Searches, example: `GET /products/_search?q=name:tshirt AND tags:red`
    - Apache Lucene syntax
- Query DSL (preferred way)
```
GET /products/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "name": "tshirt"
                    }
                },
                {
                    "match": {
                        "tags": "red"
                    }
                }
            ]
        }
    }
}
```
- Simplest query:
```
GET /products/_search
{
    "query": {
        "match_all": {}
    }
}
```
- Result keys:
    - `took`: execution time in ms
    - `_shards`:
        - `total`: number of shards queried to get the results
        - `skipped`: a shard is skipped if Elasticsearch considers that it does not contain any relevant documents.
        - `successful`: number of shards that ran the query correctly.
        - `failed`: number of shards that could NOT run the query correctly.
    - `hits`: elements that matched the query
        - `total`:
            - `value`:
            - `relation`: usually "eq"
        - `max_score`: maximum relevance score calculated by ElasticSearch
        - `hits`: actual documents that matched the query
            - `_index`: index where the document is stored. Useful when searching in multiple indices at the same time.
            - `_id`: identifier of the document
            - `_score`: relevance score
            - `_ignored`: name of the fields that were ignored when the document was indexed
            - `_source`: original document

### Introduction to term level queries

- One group of queries used to search for exact values (filtering)
- Term level queries are not analyzed
    - The value is used as-is
- Example
```
GET /<index-name>/_search
{
    "query": {
        "term": {
            <field>: <value>
        }
    }
}
```
- Case sensitive
- Can be used with: `keyword`, numbers, dates, etc.
- Do not use them with `text` fields (unless they are also indexed as `keyword`)
    - Results can seem strange and unpredictable
    - Can be hard to debug if you do not understand how the query works

### Searching for terms

- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "term": {
      "DestAirportID": "BCN"
    }
  }
}

GET /kibana_sample_data_flights/_search
{
  "query": {
    "term": {
      "dayOfWeek": 2
    }
  }
}

GET /kibana_sample_data_flights/_search
{
  "query": {
    "term": {
      "OriginCityName": { // verbose syntax needed for parameters
        "value": "barcelona",
        "case_insensitive": true // needed to get results
      }
    }
  }
}

GET /kibana_sample_data_flights/_search
{
  "query": {
    "terms": { // See that `terms` is in plural to allow multiple values AND/OR
      "OriginCityName": ["Barcelona", "Madrid"]
    }
  }
}
```

### Retrieving documents by IDs

- Equivalent to `SELECT * FROM table where ID in (<values>)`
- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "ids": {
      "values": ["not-an-id", "ciGmMIsBg9E5kuAqpAMO", "oCGmMIsBg9E5kuAqpAMO"]
    }
  }
}
```

### Range searches

- The `range` query is used to perform range searches
    - between dates
- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "range": {
      "timestamp": {
        // optional with dates:
        // "format": "dd/MM/yyyy",
        // "time_zone": "+01:00",
        // Dates in UTC by default
        "gte": "2023-10-08 00:00:00", // gt - greater than (strict)
        "lte": "2023-10-09 23:59:59" // lt - lower than (strict)
      }
    }
  }
}
```

### Prefixes, wildcards & regular expressions

- Term queries allow prefixes, wildcards, and regular expressions to match beyond exact matching
    - Still use them with `keyword` fields
- All these examples are case *sensitive* by default. To change that, use `case_insensitive: true` parameter.
- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "prefix": {
      "DestRegion": "ES-"
    }
  }
}
```
- Wildcards allow `?` (matches one character) and `*` (zero or more characters)
- Avoid placing a whildcard at the beginning of a pattern.
- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "wildcard": {
      "DestRegion": "ES-?"
    }
  }
}
```
- `regexp` query matches terms that match a regular expression
    - Anchor operators (^$) are not supported
- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "regexp": {
      "FlightNum": "[0-9]{2}.{3}[0-9]{2}"
    }
  }
}
```

### Querying by field existence

- Equivalent to `IS NOT NULL` in MySQL
- Examples in demo.elastic.co
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "exists": {
      "field": "DestLocation"
    }
  }
}
```
- Reasons for no indexed value
    - Empty value provided (`NULL` or `[]`)
    - No value was provided for the field
    - The `index` mapping parameters is set to `false` for the field
    - The value's length is greater than the `ignoer_above` parameter
    - Malformed value with the `ignore_malformed` mapping parameter set to `true`.
- Inverting the query use the `query.bool.must_not`:
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "DestLocation"
          }
        }
      ]
    }
  }
}
```

### Introduction to full text queries

- Searches on unstructured text data
    - Often used for long texts
- Full text queries are analyzed
- Do not use them in `keyword` fields
- Examples in demo.elastic.co:
```
GET /kibana_sample_data_flights/_search
{
  "query": {
    "match": {
        <field>: <value>
    }
  }
}
```

### The match query

- When specifying multiple terms, all terms are looked in the inverted index
    - `operator` parameter can be set to `OR` or `AND`.
    - by default it is set to `OR`
- Examples in demo.elastic.co:
```
GET /flights-2/_search
{
  "query": {
    "match": {
      "Carrier": {
        "query": "kibana airlines",
        "operator": "and"
      }
    }
  }
}
```

### Introduction to relevance scoring

- Relevance score helps ordering results by relevance.
- The more terms in common, the higher the score.
- `term` queries will only produce results with score 1.0
- `match` (full-text queries) will produce a different range of scores.

### Searching multiple fields

- Allows searching in multiple fields
- Examples in demo.elastic.co:
```
GET /flights-2/_search
{
  "query": {
    "multi_match": {
      "query": "international",
      // ^2 is a boost given to that field so that matches with that field have a higher score
      "fields": ["Origin^2", "Dest"],
      "tie_breaker": 0.3
    }
  }
}
```
- Elastic search splits these queries into N different `match` queries.
    - The score for each document is the best score for all the fields
- The `type` parameter allows adjusting the relevance scoring
    - Documentation: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html
- Tie breaker
    - Only one field is used for calculating a document's relevance score
    - We can "reward" documents where multiple fields match with the `tie_breaker` parameter
        - Non-best scores are multiplied by the tie_breaker value and added to the best score

### Phrase searches

- `match` or `multi_match` queries match if *any* of the fields matches the contents of the `query`, no matter the order
- `match_phrase` is analyzed in the same way, but the terms need to appear in the same order and without other terms in-between.
- Inverted indices not only contain information about the existence of a term in a document, but also about its position(s) in the document.
- When running a `phrase` query, Elasticsearch uses these positions to check if a document matches the query.
- Examples in demo.elastic.co:
```
GET /flights-2/_search
{
  "query": {
    "match": {
      "Dest": "airport international" // 1000 results
    }
  }
}

GET /flights-2/_search
{
  "query": {
    "match_phrase": {
      "Dest": "airport international" // 0 results
    }
  }
}

GET /flights-2/_search
{
  "query": {
    "match_phrase": {
      "Dest": "international airport" // 7844 results
    }
  }
}
```

### Leaf and compound queries

- All previous queries are called _leaf_ queries.
- Leaf queries can be used by themselves.
- Compound queries wrap other queries to produce a result.
    - Compound can wrap other compounds.
- `_search.query` field can only contain a single query.

### Querying with boolean logic

- `bool` query is used to group other queries:
    - `must`: obtain the documents matching with the subqueries
    - `must_not`: obtain the documents not matching with the subqueries
    - `filter`: is similar to `must`, but won't contribute to relevance score
    - `should`: boost the score of documents matching the subqueries, but do not discard those that doesn't.
        - Except if a bool query only contains a `should`, at least one subquery must match.
        - This behavior can be forced with `minimium_should_match`

|Occurrence type|Required to match?|Affects relevance scores?|Can be cached?|
|---|---|---|---|
|`must`|Yes|Yes|No|
|`filter`|Yes|No|Yes|
|`must_not`|No|No|Yes|
|`should`|Conditional|Yes|No|

- Examples in demo.elastic.co:
```
GET /flights-2/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "DestCountry.keyword": "US"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "OriginCountry.keyword": "US"
          }
        }
      ],
      "should": [
        {
          "term": {
            "DestWeather": "Sunny"
          }
        },
        {
          "match": {
            "Carrier": "kibana"
          }
        }
      ]
    }
  }
}
```
- `match` queries are translated into `bool` queries under the hood.
    - From: `{"match": { "field": "QUERY" } }`
    - To: `{"bool": { "must": [ {"term": {"field": "query"} } ] } }
    - The translated query uses the output of the Analyzer process.
    - In the case of multiple terms (without `operator: and`):
        - From: `{"match": { "field": "A B" } }`
        - To: ```
            {"bool": { "should": [
                {"term": {"field": "a"} },
                {"term": {"field": "b"} },
            ] } }
            ```
    - In the case of multiple terms (with `operator: and`):
        - From: `{"match": { "field": "A B", "operator": "and" } }`
        - To: ```
            {"bool": { "must": [
                {"term": {"field": "a"} },
                {"term": {"field": "b"} },
            ] } }
            ```

### Query execution contexts

- Query execution context
    - Document matches?
    - How well does it match? -> `_score`
    - Results in `_score` descending order
    - `"query"` keyword
- Filter execution context
    - Document matches?
    - No relevance score computed
    - Typically on structured data
    - Improves performance (no scores computed)
    - Results can be cached
- Queries that suppport changing the execution context usually have a `filter` parameter.

### Boosting query

- With `bool` query we increase relevance scores with `should`.
- `boosting` query allows _decreasing_ the relevance score.
    - `positive` narrows down the number of results
    - `negative` key reduces the relevance of those results matching the `positive` query
    - `negative_boost` indicates how much the relevance scores are reduced.
- Examples in demo.elastic.co:
```
GET /flights-2/_search
{
  "query": {
    "boosting": {
      "positive": {
        "bool": {
          "filter": [
            { "match_all": {} }
          ],
          "should": [
            {
              "match": {
                "DestWeather": "sunny"
              }
            }
          ]
        }
      },
      "negative": {
        "match": {
          "DestWeather": "cloudy"
        }
      },
      "negative_boost": 0.5 // multiplier for those documents matching the `negative` query.
    }
  }
}
```

### Disjunction max (dix_max)

- Another compound query
- The `dis_max.queries` contains a list of queries. Any document in the results must match _at least_ one of the queries.
- `tie_breaker` parameter is used to reward documents matching multiple queries
- `multi_match` query uses `dis_max` behind the hoods.
- Examples in demo.elastic.co:
```
GET /flights-2/_search
{
  "query": {
    "dis_max": {
      "tie_breaker": 0.3,
      "queries": [
        { "match": { "Dest": "international" } },
        { "match": { "Origin": "international" }  }
      ]
    }
  }
}
```

### Querying nested objects

- Remember that nested fields are stored "flattened". This means that a list of nested object like:
```
    {
        "products": [
            {
                "name": "A",
                "price": 12
            },
            {
                "name": "B",
                "price": 34
            },
            {
                "name": "C",
                "price": 56
            }
        ]
    }
```
- Will be stored/indexed like:
```
    {
        "products.name": ["A", "B", "C"],
        "products.price": [12, 34, 56]
    }
```
- This way, when querying the two fields, the query searches for all the values, regardless which object they belong to.
- In order to query values in nested objects we need to:
    1. Use `nested` data type in the mapping
    2. use the `nested` query type
- How is relevant score calculated?
    - `score_mode`, function to be applied to all the child object's score (avg, sum, max, etc.)
- Examples in demo.elastic.co:
```
DELETE /orders
PUT /orders
{
    "mappings": {
        "properties": {
            "products: "{
                "type": "nested",
                "properties": {
                    "name": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword"
                            }
                        }
                    },
                    "price": {
                        "type": float
                    }
                }
            }
        }
    }
}


GET /orders/_search
{
  "query": {
    "nested": {
        "path": "products",
        "query": {
            "bool": {
                "must": [
                    {
                        "match": {
                            "products.name": "A"
                        }
                    },
                    {
                        "range": {
                            "products.price": {
                                "gte": 20
                            }
                        }
                    }
                ]
            }
        }
    }
  }
}
```

### Nested inner hits

- With `nested` query, matches are "root documents", but we might want to know which nested objects actually matched -> `inner_hits`
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "query": {
    "nested": {
      "path": "products",
      "inner_hits": { // Enable inner hits
        "name": "name-of-inner_hits-subkey", // optional
        "size": <int> // how many innerhits to be returned, default: 3
      }
      "query": {
        ...
      }
    }
  }
}
```
- This adds a `inner_hits` key at the same level as `_source` in the results.

### Nested fields limitations

- Indexing and querying `nested` fields is more expensive than for other data types.
- An Apache Lucene document is created for each nested object.
- Important to remember for large datasets
- Elasticsearch provides safeguards to reduce the risk of performance bottlenecks
    - We need to use a specialized data type (`nested`) and query (`nested`)
    - Max 50 `nested` fields per index. Can be increased with `index.mapping.nested_fields.limit`
- 10k nested objects per document (across all `nested` fields)
    - Can be increased with `index.mapping.nested_objects.limit`

## Section 6: Joining Queries

### Introduction to this section

- Query relationships between documents
- Foreing keys in Relational Databases
- It is not recommended to use ElasticSearch as a primary data store
- ElasticSearch supports simple joins, but they are very expensive

### Add departments test data

- https://github.com/codingexplained/complete-guide-to-elasticsearch/blob/master/Joining%20Queries/add-departments-test-data.md
- Examples in demo.elastic.co, let's try it with `GET /reindex-kb-8.7/_mapping`

### Mapping document relationships

- Relationship is defined in the `_mapping` as shown below:
```
PUT /department
{
    "mappings": {
        "_doc": {
            "properties": {
                "join_field": {
                    "type": "join",
                    "relations": {
                        "department": "employee"
                    }
                }
            }
        }
    }
}
```

### Adding documents

- After adding a join field, when adding a new document, we need to specify the values of the relation.
- Example:
```
PUT /department/_doc/1
{
    "name": "Development",
    "join_field": "department"
}
PUT /department/_doc/2
{
    "name": "Marketing",
    "join_field": {         // equivalent syntax
        "name": "department"
    }
}

PUT /department/_doc/3?routing=1 // same as "parent", to store both documents in the same shard
{
    "name": "Bo Andersen",
    "age": 28,
    "gender": "M",
    "join_field": {
        "name": "employee",
        "parent": 1  // the ID of the parent document
    }
}
PUT /department/_doc/4?routing=2 // same as "parent", to store both documents in the same shard
{
    "name": "John Doe",
    "age": 44,
    "gender": "M",
    "join_field": {
        "name": "employee",
        "parent": 2  // the ID of the parent document
    }
}

...
```

### Querying by parent ID

- Example:
```
GET /department/_search
{
    "query": {
        "parent_id": {
            "type": "employee",
            "id": 1 // parent_id
        }
    }
}
```

### Querying child documents by parent

- The query type to use is `has_parent`. By default it does not support relevance scoring, a flat score of 1.0
    - Can be changed with an option: `score: true`. The relevance score of the matching of the parent is reflected to the child documents.
- Example:
```
GET /department/_search
{
    "query": {
        "has_parent": {
            "parent_type": "department",
            "score": true,
            "query": {
                // same syntax for regular queries
                "term": {
                    "name.keyword": "Development"
                }
            }
        }
    }
}
```

### Querying parent by child documents

- The query type to use is `has_child`.
    - This type of query, by default, does not consider the relevance score of the child documents.
    - In order to change it use `score_mode`, with the following possible values:
        - `min`: The lowest score of matching child documents is mapped into the parent.
        - `max`: The highest score of matching child documents is mapped into the parent.
        - `sum`: The sum score of matching child documents is mapped into the parent.
        - `avg`: The average score of matching child documents is mapped into the parent.
        - `none`: Default value, ignore child scores.
    - It also allows the minimum and maximum number of child nodes for a parent to appear in the results, use `min_children` and `max_children`
- Example:
```
GET /department/_search
{
    "query": {
        "has_child": {
            "type": "employee",
            "score_mode": "sum",
            "min_children": 2,
            "max_children": 5,
            "query": {
                // same syntax for regular queries
                "bool": {
                    "must": [
                        {
                            "range": {
                                "age": {
                                    "gte": 50
                                }
                            }
                        }
                    ],
                    "should": [
                        {
                            "term": {
                                "gender.keyword": "M"
                            }
                        }
                    ]
                }
            }
        }
    }
}
```

### Multi-level relations

- Example:
```
PUT /company
{
    "mappings": {
        "_doc": {
            "properties": {
                "join_field": {
                    "type": "join",
                    "relations": {
                        "company": ["department", "supplier"],
                        "department": "employee"
                    }
                }
            }
        }
    }
}
```
- Adding data:
```
PUT /company/_doc/1
{
    "name": "My Company Inc.",
    "join_field": "company"
}
PUT /company/_doc/2?routing=1 // same as "parent", to store both documents in the same shard
{
    "name": "Development",
    "join_field": {         // equivalent syntax
        "name": "department",
        "parent": 1         // the ID of the parent document
    }
}

PUT /company/_doc/3?routing=1 // the ID of the grandparent to store all documents of the same company in the same shard.
{
    "name": "Bo Andersen",
    "join_field": {
        "name": "employee",
        "parent": 2
    }
}
...
```
- Searching is the same as in the previous examples.
- Example:
```
ET /company/_search
{
    "query": {
        "has_child": {
            "type": "department",
            "query": {
                "has_child": {
                    "type": "employee",
                    "query": {
                        "match": {
                            "term": {
                                "name.keyword": "John Doe"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

### Parent/child inner hits

- `has_child` query allows setting the `inner_hits: {}` option to see which children matched the query.
- `has_parent` query allows setting the `inner_hits: {}` option to see which parents matched the query.

### Terms lookup mechanism

- If you want to look up for too many terms, the query would end up being too large. That's where the Terms lookup mechanism enters.
    - Fetch the terms from a document
- Example:
```
PUT /users/_doc/1
{
    "name": "Andrew",
    "following": [2, 3]
}
PUT /users/_doc/2
{
    "name": "Bob",
    "following": [1]
}
PUT /users/_doc/3
{
    "name": "Carl",
    "following": [2, 1]
}
PUT /stories/_doc/1
{
    "user": 1,
    "content": "The quick brown fox"
}
PUT /stories/_doc/2
{
    "user": 1,
    "content": "jumps over the lazy dog"
}
PUT /stories/_doc/3
{
    "user": 2,
    "content": "Lorem ipsum dolor sit amet"
}
PUT /stories/_doc/4
{
    "user": 3,
    "content": "This is a test"
}

GET /stories/_search
{
    "query": {
        "terms": {
            "user": {
                // The following fields help resolving the data without having it to transfer to the main application
                "index": "users",
                "type": "_doc",
                "id": "1",
                "path": "following"
            }
        }
    }
}
```

### Join limitations

- Joined documents must be stored within the same index
- Parent & child documents must be on the same shard
- Only one join field per index, with as many relations as needed
- New relations can be added to a created index
- Child relations can only be added to existing parents
- One parent per document

### Join field performance considerations

- Using join fields is slow!
- Avoid join fields whenever possible, except for a few scenarios
- The more child documents pointing to unique parents, the slower the `has_child` query is
    - The same to `has_parent`.
- Each level of document relations adds an overhead to queries
- "Allowed" scenarios:
    - One to many relationship between two document types, where one type has many more documents than the other, e.g. recipes <-> ingredients
- How should document relationships be mapped properly?
    - Nested data type
    - Don't do relationships, store data thinking about how it will be searched.

## Section 7: Controlling Query Results

### A word on document types

I need to steal your attention for a moment to talk about _mapping_ types. These were a thing for older versions of Elasticsearch, but will be removed in version 8.x. They are already deprecated and are only available when actively opting into using them. However, this means that pretty much all endpoints for the Elasticsearch API has changed. Whenever you see `default` as part of an endpoint, this refers to the mapping type.

We have been using queries that are compatible with Elasticsearch 7.0.0 and later until now, but you are about to see some queries that use a type of `default`. The reason for that is that those lectures were recorded before `_doc` was a thing. Some APIs now use `_doc` instead of `default`, while others completely got rid of that part of the endpoint. For the queries that contain `default`, please check the [GitHub repository for the updated queries](https://github.com/codingexplained/complete-guide-to-elasticsearch). The difference is just semantics, so hopefully it won't be an issue for you.

For instance, when you see the following:
```
GET /products/default/_search
```
... you should replace it with:
```
GET /products/_search
```
I am gradually updating the course with the latest queries, but please understand that re-recording and editing 10+ hours of content takes a long time, so there will be a period in which you will see the old document type being specified for parts of the course. This is a work in progress, and this document will be pushed forward as I make progress. 🙂 If in doubt, please refer to the [GitHub repository](https://github.com/codingexplained/complete-guide-to-elasticsearch).

Thank you for your understanding, and happy searching!

### Specifying the result format

- The output format of a search query can be changed by using the query parameter `?format=yaml`
- When using json, `?pretty` can be used to get a formatted json

### Source filtering

- Filtering what is returned inside the `_source` key.
- Use: `_source: false` at the root level of the body.
- `_source` key can contain one or multiple fields using dot-notation.
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "_source": ["geoip.*", "shipping_city", "products.product_name", "products.price"],
  "query": {
    "match_all": {}
  }
}

GET /orders/_search
{
  "_source": {
    "includes": ["products.*"],
    "excludes": "products.price"
  },
  "query": {
    "match_all": {}
  }
}
```

### Specifying the result size

- Control the number of hits
- Query param: `?size=<int>`
    - The total is still shown
- Body param: "size"
- Default size: 10
- Examples in demo.elastic.co:
```
GET /orders/_search?size=3
{
  "_source": false,
  "query": {
    "match_all": {}
  }
}

GET /orders/_search
{
  "_source": false,
  "size": 3,
  "query": {
    "match_all": {}
  }
}
```

### Specifying an offset

- For pagination, besides the `size` we need an `offset`.
- Body param: `from`
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "_source": false,
  "size": 2,
  "from": 0,
  "query": {
    "match_all": {}
  }
}
```

### Pagination

- total_pages = ceil(total_hits / page_size)
- from = (page_size * (page_number - 1))
- Limit of 10k elements of pagination
    - parameter `search_after`
- There are not "cursors" in ElasticSearch when obtaining the results. Once responded to a request, the elastic search cluster does not leave anything open.
- Beware of pagination and real time changes!

### Sorting results

- Use the `sort` key to configure how hits are sorted
- Besides field names, `_score` can also be specified
- Examples in demo.elastic.co:
```
GET /flights/_search
{
  "_source": false,
  "query": {
    "match_all": {}
  },
  "sort": [
    "AvgTicketPrice"
  ]
}


GET /flights/_search
{
  "_source": ["OriginCityName", "DestCityName"],
  "query": {
    "match_all": {}
  },
  "sort": [
    {"AvgTicketPrice": "desc"}
  ]
}

GET /flights/_search
{
  "_source": ["OriginCityName", "DestCityName", "AvgTicketPrice"],
  "query": {
    "prefix": {
      "OriginCityName.keyword": "C"
    }
  },
  "sort": [
    {"OriginCityName.keyword": "asc"},
    {"DestCityName.keyword": "desc"}
  ]
}
```

### Sorting by multi-value fields

- We can sort by fields containing more than one value
- Sorting mode: avg, sum, min, max
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "_source": ["billing_country_name", "products.product_name", "products.price"],
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "products.price": {
        "order": "desc",
        "mode": "sum"
      }
    }
  ]
}
```

## Section 8: Aggregations

### Introduction to aggregations

- Aggregations are a way of grouping and extracting statistics from documents.

### Metric aggregations

- Similar to relational databases' aggregations
- Two types of aggregations:
    - Single-value numeric metric aggregations: output a single value, e.g. a sum of fields, etc.
    - Multi-value numeric metric aggregations: output multiple values
- A query can be specified for aggregations, but, for simplicity, `query` is left outside the request, all elements will be used.
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_prices": {
      "sum": {
        "field": "products.price"
      }
    },
    "avg_price": {
      "avg": {
        "field": "products.price"
      }
    },
    "max_price": {
      "max": {
        "field": "products.price"
      }
    },
    "min_price": {
      "min": {
        "field": "products.price"
      }
    }
  }
}

GET /orders/_search
{
  "size": 0,
  "aggs": {
    "count_products": {
      "cardinality": { // Produces "approximate results" for performance's seek.
        "field": "category.keyword"
      }
    },
    "total_categories": {
      "value_count": {
        "field": "category.keyword"
      }
    }
  }
}

GET /orders/_search
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": {        // Produces count, min, max, avg, sum
        "field": "products.price"
      }
    }
  }
}
```

### Introduction to bucket aggregations

- Creates sets of documents, i.e. buckets
- The `status_terms` creates a bucket for the documents matching a certain criteria
    - The results include:
        - `sum_other_doc_count`: sum of documents not fitting into each of the buckets
        - `buckets.doc_count`: the number of documents in each bucket
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "category.keyword",
        "missing": "N/A",     // Default bucket for elements without the "field"
        "min_doc_count":0,   // Minimum number of occurrences for a bucket to appear in the results,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}
```

### Document counts are approximate

- The document count for the `terms`
- The reason is the distributed nature of the index.
    - Each shard takes the top N statistics and sends it to the "coordinating" node.
    - There could be appearances of a group in a shard that does not enter in its top N, but it is part of the top N of another shard. Then, the coordinator will never know it.
- The size parameter controls the N, defaults to 10
    - The higher the value, the less performance.
- Single shards would be 100% accurate.
- `doc_count_error_upper_bound`: this would be the sum of the last elements in all top N groups, meaning that a group not appearing in the final list may have, at most, `doc_count_error_upper_bound` appearances.

### Nested aggregations

- Bucket aggregations can contain metric and other bucket aggregations
    - Top-level aggregations run on the context of the query
    - Sub aggregations run on the context of their parent aggregations
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "taxful_total_price": {
        "lte": 10
      }
    }
  },
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "category.keyword",
        "order": {
          "_key": "asc"
        }
      },
      "aggs": {
        "price_stats": {
          "stats": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

### Filtering out documents

- Filter out documents before running an aggregation using
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "cheap_orders": {
      "filter": {
        "range": {
          "taxful_total_price": {
            "lte": 50
          }
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "taxful_total_price"
          }
        }
      }
    },
    "other_orders": {
      "filter": {
        "range": {
          "taxful_total_price": {
            "gt": 50
          }
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

### Defining bucket rules with filters

- Defines rules for each bucket to contain documents
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "my_filter": {
      "filters": {
        "filters": {  // Yes, 'filters' key is repeated
          "cheap": {
            "range": {
              "taxful_total_price": {
                "lte": 50
              }
            }
          },
          "regular": {
            "range": {
              "taxful_total_price": {
                "gt": 50,
                "lte": 100
              }
            }
          },
          "expensive": {
            "range": {
              "taxful_total_price": {
                "gt": 100
              }
            }
          }
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

### Range aggregations

- Two aggregations `range` and `date_range`.
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "price_distribution": {
      "range": {
        "field": "taxful_total_price",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}

GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_per_year": {
      "date_range": {
        "field": "order_date",
        "format": "yyyy-MM-dd",
        "keyed": true,
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+1y",
            "key": "year 2016"
          },
          {
            "from": "2017-01-01",
            "to": "2017-01-01||+1y",
            "key": "year 2017"
          },
          {
            "from": "2018-01-01",
            "to": "2019-01-01",
            "key": "year 2018"
          }
        ]
      },
      "aggs": {
        "price_stats": {
          "stats": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

### Histograms

- Aggregation types: `histogram` and `date_histogram`
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "taxful_total_price": {
        "lte": 500
      }
    }
  },
  "aggs": {
    "price_distribution": {
      "histogram": {
        "field": "taxful_total_price",
        "interval": 100,
        "min_doc_count": 0,  // minimum number of documents for a bucket to appear,
        "extended_bounds": {  // forces buckets to be created even if no documents fall into them
          "min": 500,
          "max": 1000
        }
      }
    }
  }
}

GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "field": "order_date",
        "format": "yyyy-MM",
        "keyed": true,
        "calendar_interval": "month" // valid values: day, month, year, ...
      }
    }
  }
}
```

### Global aggregation

- Even if a query filtered down the number of documents, a global aggregation allows accessing to all documents in the index.
- See the difference between `hits.total.value` and `aggregations.all_orders.doc_count`
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "taxful_total_price": {
        "lte": 20
      }
    }
  },
  "aggs": {
    "cheap_orders": {
      "stats": {
        "field": "taxful_total_price"
      }
    },
    "all_orders": {
      "global": {},
      "aggs": {
        "price_stats": {
          "stats": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

### Missing field values

- What if documents do not contain or contain null values on the aggregated field?
- The aggregation type is: `missing`.
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      }
    }
  }
}
```

### Aggregating nested objects

- `nested` objects can be grouped together by specifying the `nested` aggregation type
- Examples in demo.elastic.co:
```
GET /reindex-kb-8.7/_search
{
  "size": 0,
  "aggs": {
    "alerts": {
      "nested": {
        "path": "alerts.actions.actionRef"
      },
      "aggs": {
        "count_actions": {
          "cardinality": {
            "field": "alerts.actions.actionRef"
          }
        }
      }
    }
  }
}
```

## Section 9: Improving Search Results

### Proximity searches

- The `match_phrase` algorithm can be less strict in terms of the order of elements.
    - Allow a numer of terms inbetween the terms: `slop`
- `slop:1` allows matching `spicy sauce` with `spicy tomato sauce`
- The terms can be moved around and change the order of the terms if enough `slop` (edit distance)
- Examples in demo.elastic.co:
```
GET /orders/_search
{
  "_source": ["category"],
  "query": {
    "match_phrase": {
      "category": {
        "query": "men's shoes",
        "slop": 1
      }
    }
  }
}
```

### Affecting relevance scoring with proximity

- The lower the edit distance -> the higher the score.
- We can add multiple algorithms together to boost the relevance score
- Examples in demo.elastic.co:
```
GET /flights/_search
{
  "_source": ["DestCityName"],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "DestCityName": {
              "query": "San Juan"
            }
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "DestCityName": {
              "query": "San Juan",
              "slop": 5
            }
          }
        }
      ]
    }
  }
}
```

### Fuzzy match query (handling typos)

- Add the `fuzziness` parameter to the `match` query
    - It represents the edit distance (Levenshtein's distance)
    - Transpositions count as a single edit, can be disabled with `fuzzy_transpositions: false`
    - Can be set to `"auto"`, depending on the term-length
        - 1-2 -> 0 (exact match)
        - 3-5 -> 1
        - 5+ -> 2
        - Maximum edit distance: 2
- Examples in demo.elastic.co:
```
GET /flights/_search
{
  "_source": ["DestCityName"],
  "query": {
    "match": {
      "DestCityName": {
        "query": "Sant Joan",
        "fuzziness": "auto"
      }
    }
  }
}
```

### Fuzzy query

- Avoid it. It is NOT the same as `match` + `fuzziness: auto`.
- `fuzzy` type of query is a term level query
    - keyword
    - it also considers an edit distance
- Examples in demo.elastic.co:
```
GET /flights/_search
{
  "_source": ["DestCityName"],
  "query": {
    "fuzzy": {
      "DestCityName": {
        "value": "JUAN",
        "fuzziness": "auto"
      }
    }
  }
}
```

### Adding synonyms

- Synonyms are set in the `mapping` of an index
- Synonyms replace the terms in both the documents and the query.
- When replacing the term by multiple terms, the replacements are indexed in the same position.
- Synonyms need to be defined taking into account its position in the `settings.analyzer.<analizer-name>.filter`.
- Example:
```
PUT /synonyms
{
    "settings": {
        "analysis": {
            "filter": {
                "synonym_test": {
                    "type": "synonym",
                    "synonyms": [
                        "awful => terrible",
                        "awesome => great, super",
                        "elasticsearch, logstash, kibana => elk",
                        "weird, strange"
                    ]
                }
            },
            "analyzer": {
                "my_analyzer": {
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "synonym_test"
                    ]
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "description": {
                "type": "text",
                "analyzer": "my_analyzer"
            }
        }
    }
}
```

Test it:
```
POST /synonyms/_analyze
{
    "analyzer": "my_analyzer",
    "text": "awesome"
}
```

### Adding synonyms from file

- Replace `synonyms` by `synonyms_path: /path/to/synonyms.txt`
- Place the file in the `config` file, of all nodes
- It can contain comments using `#`
- If the document is edited, the indexed documents may no longer be matchable.
    - Solution: `POST /synonyms/_update_by_query` to reindex the documents.

### Highlighting matches in fields

- `highlight` object at the top level of the request body:
- The results include a `highlight` key with the highlighted terms surrounded by `<em></em>`.
- The original offsets of the terms are saved within the indexation
```
GET /<index-name>/_search
{
    "query": {
        "match": {
            "<field-name>": "<some text to match>"
        }
    },
    "highlight": {
        "pre_tags": [ "<strong>" ],
        "post_tags": [ "</strong>" ],
        "fields": {
            "<field-name>": {}
        }
    }
}
```

### Stemming

-
```
PUT /stemming_test
{
    "settings": {
        "analysis": {
            "filter": {
                "synonym_test": {
                    "type": "synonym",
                    "synonyms": [
                        "firm => company",
                        "love, enjoy"
                    ]
                },
                "stemmer_test": {
                    "type": "stemmer",
                    "name": "english"
                }
            },
            "analyzer": {
                "my_analyzer": {
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "synonym_test",
                        "stemmer_test"
                    ]
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "description": {
                "type": "text",
                "analyzer": "my_analyzer"
            }
        }
    }
}
```

Test it:
```
POST /stemming_test/_doc/1
{
    "description": "I love working for my firm!"
}

GET /stemming_test/_search
{
    "query": {
        "match": {
            "description": "enjoy work"
        }
    },
    "highlight": {
        "fields": {
            "description": {}
        }
    }
}
```

## Section 10: Conclusion

Discount links:
- https://www.udemy.com/course/processing-events-with-logstash/?couponCode=ESBONUS202310
- https://www.udemy.com/course/data-visualization-with-kibana/?couponCode=ESBONUS202310
