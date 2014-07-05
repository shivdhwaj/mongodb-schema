mongodb-schema
==============

Infer probabilistic schema of javascript objects or a MongoDB collection. 

This package is dual-purpose. It serves as a [node.js module](#usage-with-nodejs) and can also be used with [MongoDB](#usage-with-mongodb) directly, where it extends the `DBCollection` shell object.

_mongodb-schema_ is an early prototype. Use at your own risk.

<br>

## Usage with Node.js

### Installation
Install the script with:

```
npm install mongodb-schema
```

### Usage 

Then load the module and use call `schema( documents, options, callback )`, which will call `callback(err, res)` with an error or the result once it's done analysing the documents.

```js
var schema = require('mongodb-schema');

// define some documents
var documents = [
    {a: 1},
    {a: {b: "hello"}}
];

// define options
var options = {flat: true};

// define callback function
var callback = function(err, res) {
    // handle error
    if (err) {
        return console.error( err );
    }
    // else pretty print to console
    console.log( JSON.stringify( res, null, '\t' ) );
}

// put it all together
schema( documents, options, callback );
```

This would output:
```json
{
    "$count": 2,
    "a": {
        "$count": 2,
        "$type": {
            "number": 1,
            "object": 1
        },
        "$prob": 1
    },
    "a.b": {
        "$count": 1,
        "$type": "string",
        "$prob": 0.5
    }
}
```

<br>

## Usage with MongoDB

### Installation

There are two ways to load the script, one-time (for testing) and permanent (for frequent use).

#### 1. Load the script directly (one-time usage)

This will first load `mongodb-schema.js` and the open the shell as usual. You will have to add the script every time you open the shell. 

```
mongo <basepath>/lib/mongodb-schema.js --shell
```

Replace the `<basepath>` part with the actual path where the `mongodb-schema` folder is located.

#### 2. Load the script via the `.mongorc.js` file (permanent usage)

You can also add the following line to your `~/.mongorc.js` file to always load the file on shell startup (unless started with `--norc`):

```js
load('<basepath>/lib/mongodb-schema.js')
```

Replace the `<basepath>` part with the actual path where the `mongodb-schema` folder is located.


### Usage

##### Basic Usage

The script extends the `DBCollection` object to have another new method: `.schema()`. On a collection called `foo`, run it with:

```js
db.foo.schema()
```

This will use the first 100 (by default) documents from the collection and calculate a probabilistic schema based on these documents.

##### Usage with options

You can pass in an options object into the `.schema()` method. Currently it supports 2 options: `samples` and `flat`.

```js
db.foo.schema( {samples: 20, flat: true} )
```

This will use the first 20 documents to calculate the schema and return the schema as flat object (all fields are collapsed to the top with dot-notation). See the [Examples](#examples) section below for nested vs. flat schemata. 

<br>

## Examples 

The schema generated by this method annotates most of the fields in the object with schema information. These can be counts (`$count`), type information (`$type`), a flag whether or not the field was an array in at least one instance (`$array`) and the probability of a field appearing given it's parent field (`$prob`).

#### Example of nested schema (default)

100 documents were passed to the schema function (in MongoDB-mode, this is the default if the `samples` option is not specified).

```json
{
    "$c": 100,
    "_id": {
        "$count": 100,
        "$prob": 1, 
        "$type": "ObjectId"
    },
    "a": {
        "$count": 100,
        "$prob": 1,
        "b": {
            "$count": 100,
            "$prob": 1,
            "$type": "number"
        },
        "c": {
            "$count": 70,
            "$prob": 0.7,
            "$type": "string"
        }
    }
}
```


#### Example of flat schema

20 documents were passed to the schema function, as well as the option `{flat: true}`.

```json
{
    "$count": 20,
    "_id": {
        "$count": 20,
        "$prob": 1, 
        "$type": "ObjectId"
    },
    "a.b": {
        "$count": 20,
        "$prob": 1,
        "$type": "number"
    },
    "a.c": {
        "$count": 13,
        "$prob": 0.65,
        "$type": "string"
    }
}
```