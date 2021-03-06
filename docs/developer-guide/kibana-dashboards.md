# Creating and Importing own Kibana Elements in the SecureCodeBox

This article shows how you can save your own kibana searches, visualizations and dashboards and load them at startup of the SecureCodeBox. 

In order to save your own visualizations, after creating them in Kibana, you need the `elasticdump` tool, available [here](https://github.com/taskrabbit/elasticsearch-dump/blob/master/bin/elasticdump). This is a very simple and easy to use tool to export your elasticsearch data into a json file. 

For our own visualizations and dashboards we exported the whole `.kibana` index with `elasticdump`. The command looks as follows: 

```
elasticdump --input=http://<eshost:port>/.kibana --output=kibana-imports.json
```
 where `<eshost:port>` needs to be replaced with the elasticsearch address. 
 
 This command produces the `kibana-imports.json` file in the current directory which includes all information about the `.kibana` index. 
 
Unfortunately this file needs a little bit of refactoring due to the fact that elasticdump produces more than one top-level JSON elements. For the import mechanism in the SecureCodeBox to work properly, the file has to be converted into a JSON array including all the elements. 

The conversion is really simple and works as following: 

Originally the file looks like this: 

```
{"_index":".kibana", "_type":"doc", "_source":{"type":"visualization", "more":"stuff"}}
{"_index":".kibana", "_type":"doc", "_source":{"type":"index-pattern", "more":"stuff"}}
```

We wrap everything inside an array and put a comma at the end of each line.
The final result looks like this: 

```
[
{"_index":".kibana", "_type":"doc", "_source":{"type":"visualization", "more":"stuff"}},
{"_index":".kibana", "_type":"doc", "_source":{"type":"index-pattern", "more":"stuff"}}
]
```

The resulting file needs to be put into the resources folder of the `elasticsearch-persistenceprovider` directory (*scb-persistenceproviders/elasticsearch-persistenceprovider/src/main/resources*) and must have the name `kibana-imports.json`, otherwise it will be ignored. 

After a new start of the SecureCodeBox the visualizations and dashboards will be loaded with the first initialization of the `ElasticSearchPersistenceProvider` (that means at least one scan must be run and there must be data inside elasticsearch).
