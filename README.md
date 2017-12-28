# SQLX

> Database driver with extended features like mysql changelog/oplog, connection auto release.

[![NPM version][npm-image]][npm-url]
[![Build status][travis-image]][travis-url]
[![Coverage Status][coveralls-image]][coveralls-url]
[![License][license-image]][license-url]
[![Downloads][downloads-image]][downloads-url]

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Usage](#usage)
  - [database list](#database-list)
  - [feature list](#feature-list)
  - [interface](#interface)
    - [overall](#overall)
    - [action whitelist: operator_info.actions](#action-whitelist-operator_infoactions)
    - [config_or_interface](#config_or_interface)
    - [method](#method)
    - [where](#where)
  - [mysql](#mysql)
    - [extend](#extend)
- [Development](#development)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Usage
database driver with extended features.

## database list
* mysql
* redis
* define custom function to use any database or service

## feature list
* changelog/oplog
* auto release timeout connection

## interface


### overall
```javascript
const sqlx = require('sqlx')
const client = sqlx.createClient({
  connection_timeout: 1000, // destroy connection on timeout
  logAction: logAction,     // if a log function is provided,
                            // the function will be called everytime a action 
                            // is done with a object describing this action.
})

// client.define(table, config_or_interface)
client.define(['table1'], config1)
client.define(['table2'], config2)
client.define('logic_table3', InterfaceOne)
client.define('logic_table4', InterfaceTwo)
client.define('*', config3) // match all other tables

// for changelog
var operator_info = {
  user: '101,23',
  actions: [ // action whitelist
    'select',
    'update',
  ]
}
const conn = client.getConnection(operator_info)

// callback
conn.insert('table7', {a:1, b:2}, function(err, rows, info) {
  if (err) throw err
  console.log(rows, info)
})

// Promise/async/await
let result
try {
  result = await conn.insert('table7', {a:1, b:2})
  console.log(result.rows)  // rows
  console.log(result.info)  // info
} catch (err) {
  throw err
}

```

### action whitelist: operator_info.actions

```javascript
operator_info.actions = '*'  // allow all
operator_info.actions = ['select']  // allow: select
```


### config_or_interface
```javascript
var config1 = {
    type: 'mysql',
    config: {
      host: '1.1.1.1',
      database: 'db1'
      user: 'root',
      password: '',
    },
  }

var config2 = {
    type: 'mysql',
    config: {
      host: '2.2.2.2',
      database: 'db2'
      user: 'root',
      password: '',
    },
  }

var config2 = {
  type: 'redis',
  config: {
    host: '127.0.0.1',
    port: '6379'
  },
}

const InterfaceOne = {

  // All methods are optional
  // Do NOT change definition of any method

  initialize: function(callback) {
    // 1. 'initialize' is called whenever client.define is called
    // 2. 'this' is created whenever 'initialize' is called
    // 3. 'this' is shared between all methods of InterfaceOne

    this._client = require('some-db-drvier').createClient({
      config1: 'value1',
      config2: 'value2',
    })
    this._client.on('connected', callback)
  },

  selectEx: function(table, query, callback) { },
  insert: function(table, sets, callback) { },
  delete: function(table, where, callback) { },
  update: function(table, sets, where, callback) { },
  select: function(table, fields, where, callback) { },
  release: function() {},

}

const InterfaceTwo = {

  select: function(table, fields, where, callback) {

    const request = require('request')
    var p = {
      url: 'http://example.com/table/insert',
      method: 'post',
      json: true,
      body: {
        field: fields,
        query: where,
      }
    }
    request(p, function(err, res, body) {
      // callback must be called with 3 parameters! callback(err, rows, info)
      callback(err, body.rows, null)
    })

  },

}
```


### method
```javascript
// callback
conn.selectEx(
  /* table */ 'table0',
  /* custom sql */ 'select ... join ...where field1 = ? and field2 = ?',
  /* where */ [1,2]
  function(err, rows, info) {
  })

conn.insert(
  /* table */ 'table1',
  /* set   */ {field1: 20},
  function(err, rows, info) {
  })

conn.update(
  /* table */ 'table2',
  /* set   */ {field1: 20},
  /* where */ {field1: 10},
  function(err, rows, info) {
  })

conn.select(
  /* table */ 'table3',
  /* field */ ['field1', 'field2'],
  /* where */ {field1: 10},
  function(err, rows, info) {
  })

conn.select(
  /* table */ 'table3',
  /* field */ 'field1',
  /* where */ {field1: 10},
  function(err, rows, info) {
  })

conn.delete(
  /* table */ 'table4',
  /* where */ {field1: 10},
  function(err, rows, info) {
  })

// Promise/async/await
let result
try {
  result = await conn.insert(
    /* table */ 'table0',
    /* set   */ {field1: 20})
  console.log(result.rows)  // rows
  console.log(result.info)  // info
} catch (err) {
  throw err
}
// other functions also works.
```


### where
where is mongo-like JSON object, examples:

```javascript
// a == 1 || b == 2
{ $or: {a:1, b:2} }
```


## mysql
### extend
```js
var config_with_extend = {
    type: 'mysql',
    config: {
      host: '1.1.1.1',
      database: 'db1'
      user: 'root',
      password: '',
    },
    extend: {
      insert: function(table, sets, callback) {
        if (sets === undefined) {
          return new Error('find some error before call sqlx')
        }
        this.constructor.prototype.insert.apply(this, arguments)
      },
    },
  }
```


# Development

```
git clone https://github.com/yinrong/node-sqlx.git
cd node-sqlx
npm i
npm test
```


[npm-image]: https://img.shields.io/npm/v/sqlx.svg?style=flat-square
[npm-url]: https://npmjs.org/package/sqlx
[travis-image]: https://img.shields.io/travis/yinrong/node-sqlx/master.svg?style=flat-square
[travis-url]: https://travis-ci.org/yinrong/node-sqlx
[license-image]: http://img.shields.io/npm/l/sqlx.svg?style=flat-square
[license-url]: LICENSE
[downloads-image]: http://img.shields.io/npm/dm/sqlx.svg?style=flat-square
[downloads-url]: https://npmjs.org/package/sqlx
[coveralls-image]: https://img.shields.io/coveralls/github/yinrong/node-sqlx/master.svg?style=flat-square
[coveralls-url]: https://coveralls.io/github/yinrong/node-sqlx