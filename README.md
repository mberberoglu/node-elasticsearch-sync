# node-elasticsearch-sync [![Build Status](https://travis-ci.org/mberberoglu/node-elasticsearch-sync.svg?branch=master)](https://travis-ci.org/mberberoglu/node-elasticsearch-sync) [![Coverage Status](https://coveralls.io/repos/github/mberberoglu/node-elasticsearch-sync/badge.svg?branch=master)](https://coveralls.io/github/mberberoglu/node-elasticsearch-sync?branch=master)
ElasticSearch and MongoDB sync module for node without environment variables


## What does it do?
elasticsearch-sync package keeps your mongoDB collections and elastic search cluster in sync. It does so by tailing the mongo oplog and replicate whatever crud operation into elastic search cluster without any overhead.
Please note that a replica set is needed for the package to tail mongoDB.


## How to use
```bash
$ npm install mberberoglu/node-elasticsearch-sync --save
```


## Sample usage (version >= 1.0.0)

```javascript

// initialize package as below
var ESMongoSync = require('node-elasticsearch-sync');

let transformFunction = (watcher, document, callBack) => {
  document.name = document.firstName + ' ' + document.lastName;
  callBack(document);
}

let sampleWatcher = {
  collectionName: 'users',
  index: 'person',
  type: 'users',
  transformFunction: transformFunction, // can be null
  fetchExistingDocuments: true,
  priority: 0
};

// the "collectionName" and "type" fields in watchers MUST be the same. This might change in later versions.

let watcherArray = [];
watcherArray.push(sampleWatcher);

let config = {
  mongo: {
		data: "mongodb://127.0.0.1:27017/db-name", // mongoDB url where data will be pulled from
		oplog: "mongodb://127.0.0.1:27017/oplog", // mongoDB oplog url which is the local DB of replica-set
		batch: 1000 // Number of documents to be indexed in a single batch indexing
	},
	elastic: "localhost:9200" // ElasticSearch cluster url
}

ESMongoSync.init(config, watcherArray, null);


/*
 * The init function takes two (2) arguments in all, as follows
 * 1. Array of wather objects specifying which mongoDB collections to pull from and keep in sync with ES cluster
 * 2. ELasticSearch object (can be null) - an already defined ElasticSerach object (returned from elasticsearch cluster connection) can be passed into the init() function to ensure that the same
 *    object used in cluster connection is used in package. This only reduces the number of connections to elasticsearch by one and might offer no practical performance engancement.
 *    If null is passed, then node-elasticsearch-sync will create and maintain its own internal elasticsearch object and uses that for data pull and real-tim sync.
 *    It is recommended that "null" is passed if the above explanation is not elaborate enough...
 */


```

All other configurations are as they were in previous versions.


## More usage info

Below is more info about sample watcher:

```javascript
let sampleWatcher = {
  collectionName: 'users',
  index: 'person',
  type: 'users',
  transformFunction: transformFunction,
  fetchExistingDocuments: true,
  priority: 0
};
```

- **collectionName** - MongoDB collection to watch.

- **index** - ElasticSearch index where documents from watcher collection is saved.

- **type** - ElasticSearch type given to documents from watcher collection

- **transformFunction** - Function that gets run each time watcher document is processed. Takes 3 parameters (watcher object, document and callBack function). The callBack function is to be called with processed document as argument. (can be null)

- **fetchExistingDocuments** - Specifies if existing documents in collection are to be pulled on initialization

- **priority** - Integer (starts from 0). Useful if certain watcher depends on other watchers. Watchers with lower priorities get processed before watchers with higher priorities.



## Extra APIs

#### Reindexing

If you have a cron-job that runs at specific intervals and you need to reindex data from your mongoDB database to ElasticSearch cluster, there is a reIndex function that takes care of fetching the new documents and reindexing in ElasticSearch.
This can also come in handy if there is an ElasticSearch mappings change and there is a need to reindex data. It should be noted that calling reIndex overwrites previously stored data in ElasticSearch cluster. It also doesn't take into consideration the size of documents to reindex and ElasticSearch cluster specs.

```javascript

ESMongoSync.reIndex();

```


#### MongoDB and Oplog connection destruction

If for any reason there is a need to disconnect from MongoDB and MongoDB Oplog, then the `destroy` or `disconnect` functions handle that.

```javascript

// completely destroy both conenctions
ESMongoSync.destroy();

// only disconnect from MongoOplog.
ESMongoSync.disconnect();

```

To resume syncing after mongoDB and Oplog connection has been destroyed or stopped,

```javascript

ESMongoSync.resume();

```

will reconnect to Mongo and Oplog and re-enable real time Mongo to ElasticSearch sync.


## Dynamic watchers
You can dynamically add watchers by calling:
```javascript

ESMongoSync.addWatchers(watchersArray);  

// this takes an array of watchers as argument, even if only 1 watcher is to be included.
// note that existing documents won't be pulled for the new watcher, but oplog activities will be tailed in real time.

```

Below is more info about sample watcher:

```javascript
let sampleWatcher = {
  collectionName: 'users',
  index: 'person',
  type: 'users',
  transformFunction: transformFunction,
  fetchExistingDocuments: true,
  priority: 0
};
```

- **collectionName** - MongoDB collection to watch.

- **index** - ElasticSearch index where documents from watcher collection is saved.

- **type** - ElasticSearch type given to documents from watcher collection

- **transformFunction** - Function that gets run each time watcher document is processed. Takes 3 parameters (watcher object, document and callBack function). The callBack function is to be called with processed document as argument. (can be null)

- **fetchExistingDocuments** - Specifies if existing documents in collection are to be pulled on initialization

- **priority** - Integer (starts from 0). Useful if certain watcher depends on other watchers. Watchers with lower priorities get processed before watchers with higher priorities.


## Dynamic watchers
You can dynamically add watchers by calling:
```javascript

ESMongoSync.addWatchers(watchersArray);  

// this takes an array of watchers as argument, even if only 1 watcher is to be included.
// note that existing documents won't be pulled for the new watcher, but oplog activities will be tailed in real time.

```


## Contributing

Contributions are **welcome** and will be fully **credited**.

We accept contributions via Pull Requests on [Github](https://github.com/mberberoglu/node-elasticsearch-sync).


### Pull Requests

- **Document any change in behaviour** - Make sure the `README.md` and any other relevant documentation are kept up-to-date.

- **Consider our release cycle** - We try to follow [SemVer v2.0.0](http://semver.org/). Randomly breaking public APIs is not an option.

- **Create feature branches** - Don't ask us to pull from your master branch.

- **One pull request per feature** - If you want to do more than one thing, send multiple pull requests.

- **Send coherent history** - Make sure each individual commit in your pull request is meaningful. If you had to make multiple intermediate commits while developing, please [squash them](http://www.git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Changing-Multiple-Commit-Messages) before submitting.


## Issues

Check issues for current issues.

## Credits

- [Mustapha Babatunde](https://twitter.com/iAmToystars)
- [Andy Collins](https://github.com/andymcollins)
- [Mustafa Berberoglu](https://github.com/mberberoglu)

## License

The MIT License (MIT). Please see [LICENSE](LICENSE) for more information.
