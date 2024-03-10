# 十四、数据库

## MongoDB

### 问题

你需要与一个 MongoDB 数据库连接的接口。

### 解决方案

#### 对于 Node.js

##### 安装

*   如果你的计算机中还没有 [MongoDB](http://www.mongodb.org/display/DOCS/Quickstart) ，需要安装。

*   [安装本地 MongoDB 模块](https://github.com/christkv/node-mongodb-native)。

##### 保存记录

```
mongo = require 'mongodb'

server = new mongo.Server "127.0.0.1", 27017, {}

client = new mongo.Db 'test', server, {w:1}

 # save() updates existing records or inserts new ones as needed

exampleSave = (dbErr, collection) ->
    console.log "Unable to access database: #{dbErr}" if dbErr
    collection.save { _id: "my_favorite_latte", flavor: "honeysuckle" }, (err, docs) ->
        console.log "Unable to save record: #{err}" if err
        client.close()

client.open (err, database) ->
    client.collection 'coffeescript_example', exampleSave
```

##### 查找记录

```
mongo = require 'mongodb'

server = new mongo.Server "127.0.0.1", 27017, {}

client = new mongo.Db 'test', server, {w:1}

exampleFind = (dbErr, collection) ->
    console.log "Unable to access database: #{dbErr}" if dbErr
    collection.find({ _id: "my_favorite_latte" }).nextObject (err, result) ->
        if err
            console.log "Unable to find record: #{err}"
        else
            console.log result # => {  id: "my_favorite_latte", flavor: "honeysuckle" }
        client.close()

client.open (err, database) ->
    client.collection 'coffeescript_example', exampleFind
```

##### 对于浏览器

一个[基于 REST 的接口](https://github.com/tdegrunt/mongodb-rest)在工程中，会提供基于 AJAX 的访问通道。

### 讨论

这个方法将 save 和 find 分开进单独的实例，其目的是分散 MongoDB 指定的连接任务的关注点以及回收任务。[async 模块](https://github.com/caolan/async)可以帮助这样的异步调用。

## SQLite

### 问题

你需要 Node.js 内部与 [SQLite](http://www.sqlite.org/) 数据库连接的接口。

### 解决方案

使用 [SQLite 模块](http://code.google.com/p/node-sqlite/)。

```
sqlite = require 'sqlite'

db = new sqlite.Database

 # The module uses asynchronous methods,

 # so we chain the calls the db.execute

exampleCreate = ->
    db.execute "CREATE TABLE snacks (name TEXT(25), flavor TEXT(25))",
        (exeErr, rows) ->
            throw exeErr if exeErr
            exampleInsert()

exampleInsert = ->
    db.execute "INSERT INTO snacks (name, flavor) VALUES ($name, $flavor)",
        { $name: "Potato Chips", $flavor: "BBQ" },
        (exeErr, rows) ->
            throw exeErr if exeErr
            exampleSelect()

exampleSelect = ->
    db.execute "SELECT name, flavor FROM snacks",
        (exeErr, rows) ->
            throw exeErr if exeErr
            console.log rows[0] # => { name: 'Potato Chips', flavor: 'BBQ' }

 # :memory: creates a DB in RAM

 # You can supply a filepath (like './example.sqlite') to create/open one on disk

db.open ":memory:", (openErr) ->
    throw openErr if openErr
    exampleCreate()
```

### 讨论

你也可以提前准备你的 SQL 查询语句。

```
sqlite = require 'sqlite'
async = require 'async' # Not required but added to make the example more concise

db = new sqlite.Database

createSQL = "CREATE TABLE drinks (name TEXT(25), price NUM)"

insertSQL = "INSERT INTO drinks (name, price) VALUES (?, ?)"

selectSQL = "SELECT name, price FROM drinks WHERE price < ?"

create = (onFinish) ->
    db.execute createSQL, (exeErr) ->
        throw exeErr if exeErr
        onFinish()

prepareInsert = (name, price, onFinish) ->
    db.prepare insertSQL, (prepErr, statement) ->
        statement.bindArray [name, price], (bindErr) ->
            statement.fetchAll (fetchErr, rows) -> # Called so that it executes the insert
                onFinish()

prepareSelect = (onFinish) ->
    db.prepare selectSQL, (prepErr, statement) ->
        statement.bindArray [1.00], (bindErr) ->
            statement.fetchAll (fetchErr, rows) ->
                console.log rows[0] # => { name: "Mia's Root Beer", price: 0.75 }
                onFinish()

db.open ":memory:", (openErr) ->
    async.series([
        (onFinish) -> create onFinish,
        (onFinish) -> prepareInsert "LunaSqueeze", 7.95, onFinish,
        (onFinish) -> prepareInsert "Viking Sparkling Grog", 4.00, onFinish,
        (onFinish) -> prepareInsert "Mia's Root Beer", 0.75, onFinish,
        (onFinish) -> prepareSelect onFinish
    ])
```

[SQL 的 SQLite 版本](http://www.sqlite.org/lang.html)的以及 [node-SQLite](https://github.com/orlandov/node-sqlite#readme) 模块文档提供了更完整的信息。