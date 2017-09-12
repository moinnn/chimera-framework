# MongoDB API

Chimera-framework offer several API you can use in your Node.Js scripts. In order to use the API, you need to import `chimera-framewomongo-driver`. This API supporting row-versioning automatically.

Most function in Chimera-framework mongoDB API rely on dbConfig.

DbConfig is a javascript value, with several keys.

```javascript
const DEFAULT_DB_CONFIG = {
    'mongo_url' : '<mongoDb-connection-string>',
    'collection_name' : '<your-collection-name>',
    'history' : '_history',
    'deletion_flag_field' : '_deleted',
    'id_field' : '_id',
    'modification_time_field' : '_modified_at',
    'modification_by_field' : '_modified_by',
    'process_deleted' : false,
    'show_history' : false,
    'user_id' : '<user-id>',
    'persistence_connection' : false,
    'verbose' : false,
}
```

# Full Example

First you need a javascript file as a bridge:

```javascript
// file location:mongo-driver.js
const mongoDriver = require('chimera-framework/mongo-driver')
module.exports = mongoDriver
```

Then you can create a YAML file:

```yaml
out: out
vars:
    mongo_url: mongodb://localhost/project-template
    collection_name: person
    user_id: u001
series:
    ## create config
    - (mongo_url, collection_name, user_id) -> [mongo-driver.js createDbConfig] -> dbConfig

    ## insert new data
    - ('{"name":"Tono Stark","alias":"Ironman","age":30}') ->-> insert_data
    - (dbConfig, insert_data) -> [mongo-driver.js insert] -> out.insert_doc
    # { _id: '59b2b69a31ff37363b4e9a88',
    #   name: 'Tono Stark',
    #   alias: 'Ironman' }


    ## update inserted data with a newer one
    - (out.insert_doc._id) ->-> ironman_id
    - ('{"name":"Toni Stark"}') ->-> update_data
    - (dbConfig, ironman_id, update_data) -> [mongo-driver.js update] -> out.update_doc
    # { _id: '59b2b69a31ff37363b4e9a88',
    #   name: 'Toni Stark',
    #   alias: 'Ironman' }

    ## insert another data
    - (dbConfig, insert_data) -> [mongo-driver.js insert] -> out.another_insert_doc
    # { _id: '59b2b69b31ff37363b4e9a89',
    #   name: 'Tono Stark',
    #   alias: 'Ironman' }

    ## delete the last one
    - (dbConfig, out.another_insert_doc._id) -> [mongo-driver.js remove] -> out.remove_doc
    # { _id: '59b2b69b31ff37363b4e9a89',
    #   name: 'Tono Stark',
    #   alias: 'Ironman',
    #   _deleted: 1 }

    ## insert bulk
    - ('[{"name":"Steve Roger","alias":"Captain America","age":31},{"name":"Bruce Banner","alias":"Hulk","age":32}]') ->-> bulk_insert_data
    - (dbConfig, bulk_insert_data) -> [mongo-driver.js insert] -> out.insert_bulk_docs
    # [ { _id: '59b2b69b31ff37363b4e9a8a',
    #     name: 'Steve Roger',
    #     alias: 'Captain America' },
    #   { _id: '59b2b69b31ff37363b4e9a8b',
    #     name: 'Bruce Banner',
    #     alias: 'Hulk' } ]

    ## update bulk
    - ('{"affiliation":"Avenger"}') ->-> bulk_update_data
    - (dbConfig, "{}", bulk_update_data) -> [mongo-driver.js update] -> out.update_bulk_docs
    # [ { _id: '59b2b69a31ff37363b4e9a88',
    #     name: 'Toni Stark',
    #     alias: 'Ironman',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8a',
    #     name: 'Steve Roger',
    #     alias: 'Captain America',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8b',
    #     name: 'Bruce Banner',
    #     alias: 'Hulk'

    ## insert superman
    - ('{"name":"Clark Kent","alias":"Superman","age":33,"affiliation":"Justice League"}') ->-> superman_data
    - (dbConfig, superman_data) -> [mongo-driver.js insert] -> out.superman_doc
    # { _id: '59b2b69b31ff37363b4e9a8c',
    #   name: 'Clark Kent',
    #   alias: 'Superman' }

    ## get ironman
    - (dbConfig, ironman_id) -> [mongo-driver.js find] -> out.ironman_doc
    # { _id: '59b2b69a31ff37363b4e9a88',
    #   name: 'Toni Stark',
    #   alias: 'Ironman',
    #   affiliation: 'Avenger' }

    ## get ironman, but only show name and alias (affiliation hidden)
    - (dbConfig, ironman_id, 'name alias') -> [mongo-driver.js find] -> out.ironman_doc_with_name_1
    # { _id: '59b2b69a31ff37363b4e9a88',
    #   name: 'Toni Stark',
    #   alias: 'Ironman' }
    - (dbConfig, ironman_id, '{"name":1,"alias":1}') -> [mongo-driver.js find] -> out.ironman_doc_with_name_2
    # { _id: '59b2b69a31ff37363b4e9a88',
    #   name: 'Toni Stark',
    #   alias: 'Ironman' }
    - (dbConfig, ironman_id, '{"fields":{"name":1,"alias":1}}') -> [mongo-driver.js find] -> out.ironman_doc_with_name_3
    # { _id: '59b2b69a31ff37363b4e9a88',
    #   name: 'Toni Stark',
    #   alias: 'Ironman' }

    ## get ironman, but only show name and affilication (name hidden)
    - (dbConfig, ironman_id, '-name -alias') -> [mongo-driver.js find] -> out.ironman_doc_no_name_1
    # { _id: '59b2b69a31ff37363b4e9a88', affiliation: 'Avenger' }
    - (dbConfig, ironman_id, '{"name":0,"alias":0}') -> [mongo-driver.js find] -> out.ironman_doc_no_name_2
    # { _id: '59b2b69a31ff37363b4e9a88', affiliation: 'Avenger' }
    - (dbConfig, ironman_id, '{"name":0,"alias":0}') -> [mongo-driver.js find] -> out.ironman_doc_no_name_3
    # { _id: '59b2b69a31ff37363b4e9a88', affiliation: 'Avenger' }

    ## get all data
    - (dbConfig) -> [mongo-driver.js find] -> out.find_docs
    # [ { _id: '59b2b69a31ff37363b4e9a88',
    #     name: 'Toni Stark',
    #     alias: 'Ironman',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8a',
    #     name: 'Steve Roger',
    #     alias: 'Captain America',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8b',
    #     name: 'Bruce Banner',
    #     alias: 'Hulk',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8c',
    #     name: 'Clark Kent',
    #     alias: 'Superman' } ]

    ## get all data affiliate to Avenger
    - (dbConfig, '{"affiliation":"Avenger"}') -> [mongo-driver.js find] -> out.find_avenger_docs
    # [ { _id: '59b2b69a31ff37363b4e9a88',
    #     name: 'Toni Stark',
    #     alias: 'Ironman',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8a',
    #     name: 'Steve Roger',
    #     alias: 'Captain America',
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8b',
    #     name: 'Bruce Banner',
    #     alias: 'Hulk',
    #     affiliation: 'Avenger' } ]

    ## get the data, limited by 2, skipped one document, and sorted by name
    - (dbConfig, '{}', '{"sort":"name", "limit":2, "skip":1}') -> [mongo-driver.js find] -> out.find_limited_skipped_sorted_docs
    # [ { _id: '59b2b69b31ff37363b4e9a8c',
    #     name: 'Clark Kent',
    #     alias: 'Superman' },
    #   { _id: '59b2b69b31ff37363b4e9a8a',
    #     name: 'Steve Roger',
    #     alias: 'Captain America',
    #     affiliation: 'Avenger' } ]

    ## get the data affiliated with Avenger, limited by 2, sorted by alias, only show alias
    - (dbConfig, '{"affiliation":"Avenger"}', '{"sort":"alias", "limit":2, "alias":1}') -> [mongo-driver.js find] -> out.find_limited_sorted_filtered_docs
    # [ { _id: '59b2b69b31ff37363b4e9a8a', alias: 'Captain America' },
    #   { _id: '59b2b69b31ff37363b4e9a8b', alias: 'Hulk' } ]

    ## Try aggregation
    - (dbConfig, '[{"$group":{"_id":"count","count":{"$sum":1}}}]') -> [mongo-driver.js aggregate] -> out.aggregation_result
    # [ { _id: 'count', count: 4 } ]

    ## Try sum
    - (dbConfig, 'age') -> [mongo-driver.js sum] -> out.sum_all_result
    # 126
    - (dbConfig, 'age', '{"affiliation":"Avenger"}') -> [mongo-driver.js sum] -> out.sum_avenger_result
    # 93
    - (dbConfig, 'age', '{}', 'affiliation') -> [mongo-driver.js sum] -> out.sum_by_affiliation_result
    # { 'Justice League': 33, Avenger: 93 }

    ## Try avg
    - (dbConfig, 'age') -> [mongo-driver.js avg] -> out.avg_all_result
    # 31.5
    - (dbConfig, 'age', '{"affiliation":"Avenger"}') -> [mongo-driver.js avg] -> out.avg_avenger_result
    # 31
    - (dbConfig, 'age', '{}', 'affiliation') -> [mongo-driver.js avg] -> out.avg_by_affiliation_result
    # { 'Justice League': 33, Avenger: 31 }

    ## Try max
    - (dbConfig, 'age') -> [mongo-driver.js max] -> out.max_all_result
    # 33
    - (dbConfig, 'age', '{"affiliation":"Avenger"}') -> [mongo-driver.js max] -> out.max_avenger_result
    # 32
    - (dbConfig, 'age', '{}', 'affiliation') -> [mongo-driver.js max] -> out.max_by_affiliation_result
    # { 'Justice League': 33, Avenger: 32 }

    ## Try min
    - (dbConfig, 'age') -> [mongo-driver.js min] -> out.min_all_result
    # 30
    - (dbConfig, 'age', '{"affiliation":"Avenger"}') -> [mongo-driver.js min] -> out.min_avenger_result
    # 30
    - (dbConfig, 'age', '{}', 'affiliation') -> [mongo-driver.js min] -> out.min_by_affiliation_result
    # { 'Justice League': 33, Avenger: 30 }

    ## Try count
    - (dbConfig) -> [mongo-driver.js count] -> out.count_all_result
    # 4
    - (dbConfig, '{"affiliation":"Avenger"}') -> [mongo-driver.js count] -> out.count_avenger_result
    # 3
    - (dbConfig, '{}', 'affiliation') -> [mongo-driver.js count] -> out.count_by_affiliation_result
    # { 'Justice League': 1, Avenger: 3 }

    ## get "sharingan" configuration
    - (dbConfig) ->-> sharingan_config
    - ("true") ->-> sharingan_config.process_deleted
    - ("true") ->-> sharingan_config.show_history
    ## get all data, including the deleted one, plus it's histories
    - (sharingan_config) -> [mongo-driver.js find] -> out.find_sharingan_docs
    # [ { _id: '59b2b69a31ff37363b4e9a88',
    #     name: 'Toni Stark',
    #     alias: 'Ironman',
    #     _history: [Object],
    #     _deleted: 0,
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a89',
    #     name: 'Tono Stark',
    #     alias: 'Ironman',
    #     _history: [Object],
    #     _deleted: 1 },
    #   { _id: '59b2b69b31ff37363b4e9a8a',
    #     name: 'Steve Roger',
    #     alias: 'Captain America',
    #     _history: [Object],
    #     _deleted: 0,
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8b',
    #     name: 'Bruce Banner',
    #     alias: 'Hulk',
    #     _history: [Object],
    #     _deleted: 0,
    #     affiliation: 'Avenger' },
    #   { _id: '59b2b69b31ff37363b4e9a8c',
    #     name: 'Clark Kent',
    #     alias: 'Superman',
    #     _history: [Object],
    #     _deleted: 0 } ]

    ## permanent remove
    - (dbConfig) -> [mongo-driver.js permanentRemove] -> out.permanent_remove_result
    # { ok: 1, n: 5 }
```

# createDbConfig

Creating a dbConfig object, which is required for `find` `insert` `update` `remove` and `permanentRemove`

If callback is empty, then the created dbConfig will be shown in stdout.

## Usage

* `createDbConfig(<mongoUrl>, <collectionName>, <userId>, <callback>)`
* `createDbConfig(<obj>, <collectionName>, <userId>, <callback>)`

## Parameters
* `mongoUrl`: string, MongoDB connection string (e.g: `mongodb://localhost/test`
* `obj`: Object with `mongo_url` key. Instead of literal string, you can pass an object with `mongo_url` key instead
* `collectionName`: string, name of collection
* `userId`: string, userId. Used for row versioning to fill up `_modified_by` column
* `callback`: callback function, require 3 parameters:
    - __dbConfig__: A newly created dbConfig object. This object is required for `find` `insert` `update` `remove` and `permanentRemove`
    - __success__: boolean, contains `true` if the operation succeed
    - __errorMessage__: string, error message

# closeConnection
Close database connection manually

## Usage
* `closeConnection()`

# find
Get document(s) based on query and projection

## Usage
* `find(<dbConfig>, <query>, <projection_and_options>, <callback>)`
* `find(<dbConfig>, <query>, <callback>)`
* `find(<dbConfig>, <callback>)`

## Parameters
* `dbConfig`: DbConfig object contains `mongo_url`, `collection_name`, `user_id` and other configurations for document manipulation
* `query`: object, [https://docs.mongodb.com/manual/tutorial/query-documents/](MongoDB query)
* `projection_and_options`: object or string, [https://automattic.github.io/monk/docs/collection/find.html](Options and projection)
* `callback`: callback function, require 3 parameters:
    - __docs__: Array of object or an object. If you put primary key value as `query`, a single object representing the document will be returned, otherwise an array containing list of documents matching the `query` will be returned
    - __success__: boolean, contains `true` if the operation succeed
    - __errorMessage__: string, error message

# insert
Insert new document(s) into collection

## Usage
* `insert(<dbConfig>, <data>, <options>, <callback>)`
* `insert(<dbConfig>, <data>, <callback>)`

## Parameters
* `dbConfig`: DbConfig object contains `mongo_url`, `collection_name`, `user_id` and other configurations for document manipulation
* `data`: array of object or object, the document(s) you want to insert
* `options`: object, insert options
* `callback`: callback function, require 3 parameters:
    - __docs__: Array of object or an object. If you put primary key value as `query`, a single object representing the document will be returned, otherwise an array containing list of documents matching the `query` will be returned
    - __success__: boolean, contains `true` if the operation succeed
    - __errorMessage__: string, error message

# update
Update document(s) based on `query` and `data`

## Usage
* `update(<dbConfig>, <query>, <data>, <options>, <callback>)`
* `update(<dbConfig>, <query>, <data>, <callback>)`

## Parameters
* `dbConfig`: DbConfig object contains `mongo_url`, `collection_name`, `user_id` and other configurations for document manipulation
* `query`: object, [https://docs.mongodb.com/manual/tutorial/query-documents/](MongoDB query)
* `data`: array of object or object, the document(s) you want to insert
* `options`: object, update options
* `callback`: callback function, require 3 parameters:
    - __docs__: Array of object or an object. If you put primary key value as `query`, a single object representing the document will be returned, otherwise an array containing list of documents matching the `query` will be returned
    - __success__: boolean, contains `true` if the operation succeed
    - __errorMessage__: string, error message

# remove
Put deletion-flag into document/documents in collection

## Usage
* `remove(<dbConfig>, <query>, <options>, <callback>)`
* `remove(<dbConfig>, <query>, <callback>)`

## Parameters
* `dbConfig`: DbConfig object contains `mongo_url`, `collection_name`, `user_id` and other configurations for document manipulation
* `query`: object, [https://docs.mongodb.com/manual/tutorial/query-documents/](MongoDB query)
* `options`: object, update options
* `callback`: callback function, require 3 parameters:
    - __docs__: Array of object or an object. If you put primary key value as `query`, a single object representing the document will be returned, otherwise an array containing list of documents matching the `query` will be returned
    - __success__: boolean, contains `true` if the operation succeed
    - __errorMessage__: string, error message

# permanentRemove
Remove document(s) from collection

## Usage
* `permanentRemove(<dbConfig>, <query>, <options>, <callback>)`
* `permanentRemove(<dbConfig>, <query>, <callback>)`

## Parameters
* `dbConfig`: DbConfig object contains `mongo_url`, `collection_name`, `user_id` and other configurations for document manipulation
* `query`: object, [https://docs.mongodb.com/manual/tutorial/query-documents/](MongoDB query)
* `options`: object, delete options
* `callback`: callback function, require 3 parameters:
    - __result__: Deletion result object. Typically contains something like `{'ok':1, 'n':4}` depending on your server version.
    - __success__: boolean, contains `true` if the operation succeed
    - __errorMessage__: string, error message
