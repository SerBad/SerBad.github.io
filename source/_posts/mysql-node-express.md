---
title: 翻译：如何在Node.js和Express中使用MySql
date: 2016-04-15 22:57:37
tags:
  - Node.js
  - mysql
  - Express
comments: false
---

## 连接到mysql数据库

在这之前先安装正确的npm包：
```JavaScript
    npm install mysql
```
**mysql** 是一个非常容易使用的模块，它提供了你可能需要的所有功能。安装好后，你需要连接数据库的方法是：
```JavaScript
var mysql = require('mysql')

var connection = mysql.createConnection({
  host: 'localhost',
  user: 'your_user',
  password: 'some_secret',
  database: 'database'
})

connection.connect(function(err) {
  if (err) throw err
  console.log('You are now connected...')
})
```
<!--more-->
现在，你可以读写你的数据库了。

## 读写mysql数据库

你已经知道如何去连接你的数据库了，现在让我们看一个简单的例子：
```JavaScript
var mysql = require('mysql')

var connection = mysql.createConnection({
  host: 'localhost',
  user: 'your_user',
  password: 'some_secret',
  database: 'database'
})

connection.connect(function(err) {
  if (err) throw err
  console.log('You are now connected...')

  connection.query('CREATE TABLE people(id int primary key, name varchar(255), age int, address text)', function(err, result) {
    if (err) throw err
    connection.query('INSERT INTO people (name, age, address) VALUES (?, ?, ?)', ['Larry', '41', 'California, USA'], function(err, result) {
      if (err) throw err
      connection.query('SELECT * FROM people', function(err, results) {
        if (err) throw err
        console.log(results[0].id)
        console.log(results[0].name)
        console.log(results[0].age)
        console.log(results[0].address)
      })
    })
  })
})
```
你能够看到在上面的占位符`?`，它不仅仅使使用起来更容易，而且也是安全的。

## 更换你的 *db.js* 文件

虽然你看到的mysql模块非常容易，但是真实的应用有着更加复杂的需求。这就是为什么在[Connecting and Working with MongoDB](https://www.terlici.com/2015/04/03/mongodb-node-express.html)中，我们创建一个分开的文件db.js来帮助我们管理我们的连接。
它的作用是无论何时我们不用经常键入凭证来很容易地连接数据库。它的第二个目标就是很容易运行测试。

```JavaScript
var mysql = require('mysql')
  , async = require('async')

var PRODUCTION_DB = 'app_prod_database'
  , TEST_DB = 'app_test_database'

exports.MODE_TEST = 'mode_test'
exports.MODE_PRODUCTION = 'mode_production'

var state = {
  pool: null,
  mode: null,
}

exports.connect = function(mode, done) {
  state.pool = mysql.createPool({
    host: 'localhost',
    user: 'your_user',
    password: 'some_secret',
    database: mode === exports.MODE_PRODUCTION ? PRODUCTION_DB : TEST_DB
  })

  state.mode = mode
  done()
}

exports.get = function() {
  return state.pool
}

exports.fixtures = function(data) {
  var pool = state.pool
  if (!pool) return done(new Error('Missing database connection.'))

  var names = Object.keys(data.tables)
  async.each(names, function(name, cb) {
    async.each(data.tables[name], function(row, cb) {
      var keys = Object.keys(row)
        , values = keys.map(function(key) { return "'" + row[key] + "'" })

      pool.query('INSERT INTO ' + name + ' (' + keys.join(',') + ') VALUES (' + values.join(',') + ')', cb)
    }, cb)
  }, done)
}

exports.drop = function(tables, done) {
  var pool = state.pool
  if (!pool) return done(new Error('Missing database connection.'))

  async.each(tables, function(name, cb) {
    pool.query('DELETE * FROM ' + name, cb)
  }, done)
}
```
这个db.js有点儿比我们之前的要复杂的多。

首先，它提供一种方法去连接数据库。在生产模块和测试模块都能
去连接。测试模块仅在自动化测试时运行。

然后，有一个`get`方法能够一直提供给你有效的连接，也可以用来查询数据库。而且，你可以去设置密码，其他模块也可以调用这个方法。

最后，有两个方法`fixtures`和`drop`，很容易地去退出测试。

**drop** 清空数据，它将会帮助你的测试数据库在测试之前总是空的。

**fixtures** 使用json对象，加载数据库中的数据，看一个例子：
```JavaScript
var data = {
  tables: {
    people: [
     {id: 1, name: "John", age: 32},
     {id: 2, name: "Peter", age: 29},
    ],
    cars: [
      {id: 1, brand: "Jeep", model: "Cherokee", owner_id: 2},
      {id: 2, brand: "BMW", model: "X5", owner_id: 2},
      {id: 3, brand: "Volkswagen", model: "Polo", owner_id: 1},
    ],
  },
}

var db = require('./db')
db.connect(db.MODE_PRODUCTION, function() {
  db.fixtures(data, function(err) {
    if (err) return console.log(err)
    console.log('Data has been loaded...')
  })
})
```

## 使用SQL搭建模块

所有事情准备好了，接下来看看如何在你的应用中使用db.js。来看一个例子：
```JavaScript
controllers/
  comments.js
  users.js
models/
  comment.js
  user.js
views/
app.js
db.js
package.json
```
**app.js** 是应用的入口，也是我们将要设置数据库连接的地方。来看一个例子：
```JavaScript
var db = require('./db')

app.use('/comments', require('./controllers/comments'))
app.use('/users', require('./controllers/users'))

// Connect to MySQL on start
db.connect(db.MODE_PRODUCTION, function(err) {
  if (err) {
    console.log('Unable to connect to MySQL.')
    process.exit(1)
  } else {
    app.listen(3000, function() {
      console.log('Listening on port 3000...')
    })
  }
})
```
你的应用将会通过这个模块来与数据库交互。来看一个例子：
```JavaScript
var db = require('../db.js')

exports.create = function(userId, text, done) {
  var values = [userId, text, new Date().toISOString()]

  db.get().query('INSERT INTO comments (user_id, text, date) VALUES(?, ?, ?)', values, function(err, result) {
    if (err) return done(err)
    done(null, result.insertId)
  })
}

exports.getAll = function(done) {
  db.get().query('SELECT * FROM comments', function (err, rows) {
    if (err) return done(err)
    done(null, rows)
  })
}

exports.getAllByUser = function(userId, done) {
  db.get().query('SELECT * FROM comments WHERE user_id = ?', userId, function (err, rows) {
    if (err) return done(err)
    done(null, rows)
  })
}
```
正如你看到的db.js是你完全不用带薪连接数据库的问题，甚至不用只用是否是生产环境还是测试环境。

## 为读和写使用不一样的实例

随着你的应用需求的增长，今天，很多应用都是面向读的。这就意味着，人们读数据少于写数据。例如，所有这一类应用的社交网站，通常你的状态只被读一次，但是然后被成千上万的人去读。在这种情况下是通常有一个主要的数据库将接受所有的写,然后同步到一些MySQL实例将只读取请求。当你有更多的用户可以很容易地添加更多数据库去读，你的应用将仍然是非常快的。

但是，如何在我们的初始化中解决这个问题呢？mysql模块 **PoolCluster** 特性非常有用,它可以配置几个实例,然后连接到所有的人。看一个db.js中的例子：
```JavaScript
var mysql = require('mysql')
  , async = require('async')

var PRODUCTION_DB = 'app_prod_database'
  , TEST_DB = 'app_test_database'

exports.MODE_TEST = 'mode_test'
exports.MODE_PRODUCTION = 'mode_production'

var state = {
  pool: null,
  mode: null,
}

exports.connect = function(mode, done) {
  if (mode === exports.MODE_PRODUCTION) {
    state.pool = mysql.createPoolCluster()

    state.pool.add('WRITE', {
      host: '192.168.0.5',
      user: 'your_user',
      password: 'some_secret',
      database: PRODUCTION_DB
    })

    state.pool.add('READ1', {
      host: '192.168.0.6',
      user: 'your_user',
      password: 'some_secret',
      database: PRODUCTION_DB
    })

    state.pool.add('READ2', {
      host: '192.168.0.7',
      user: 'your_user',
      password: 'some_secret',
      database: PRODUCTION_DB
    })
  } else {
    state.pool = mysql.createPool({
      host: 'localhost',
      user: 'your_user',
      password: 'some_secret',
      database: TEST_DB
    })
  }

  state.mode = mode
  done()
}

exports.READ = 'read'
exports.WRITE = 'write'

exports.get = function(type, done) {
  var pool = state.pool
  if (!pool) return done(new Error('Missing database connection.'))

  if (type === exports.WRITE) {
    state.pool.getConnection('WRITE', function (err, connection) {
      if (err) return done(err)
      done(null, connection)
    })
  } else {
    state.pool.getConnection('READ*', function (err, connection) {
      if (err) return done(err)
      done(null, connection)
    })
  }
}

exports.fixtures = function(data) {
  var pool = state.pool
  if (!pool) return done(new Error('Missing database connection.'))

  var names = Object.keys(data.tables)
  async.each(names, function(name, cb) {
    async.each(data.tables[name], function(row, cb) {
      var keys = Object.keys(row)
        , values = keys.map(function(key) { return "'" + row[key] + "'" })

      pool.query('INSERT INTO ' + name + ' (' + keys.join(',') + ') VALUES (' + values.join(',') + ')', cb)
    }, cb)
  }, done)
}

exports.drop = function(tables, done) {
  var pool = state.pool
  if (!pool) return done(new Error('Missing database connection.'))

  async.each(tables, function(name, cb) {
    pool.query('DELETE * FROM ' + name, cb)
  }, done)
}
```
这里有三处和之前的不一样。

1. 当连接在连接方法在测试模式下,做你之前所做的。然而,当你在生产模式下,添加3个更多的servers。每一个server都有一个名字，第一个是`WRITE`，另外两个是`READ1`和`READ2`。应该很清楚每一个的目的是什么。`READ1`和`READ2`是`WRITE`的奴隶,所以每次有人写数据到`WRITE`,然后很快地对`READ1`和`READ2`可用。
2. 第二处变化是`get`方法。之前，它会立即返回一个连接。现在，它返回一个连接到回调函数作为第二个参数提供。
3. 最后一处变化还是在`get`方法。在你连接数据库的时候，你需要告诉连接的类型 **READ** 和 **WRITE**。然后根据名称设置为服务器将提供正确的类型的连接。在这种情况下你可以看到`getConnection`提供一种模式系统,这样您就可以选择适当的数据库

然后让我们看一个使用 **db.js** 的实例：
```JavaScript
var db = require('../db.js')

exports.create = function(userId, text, done) {
  var values = [userId, text, new Date().toISOString()]

  db.get(db.WRITE, function(err, connection) {
    if (err) return done('Database problem')

    connection.query('INSERT INTO comments (user_id, text, date) VALUES(?, ?, ?)', values, function(err, result) {
      if (err) return done(err)
      done(null, result.insertId)
    })
  })
}

exports.getAll = function(done) {
  db.get(db.READ, function(err, connection) {
    if (err) return done('Database problem')

    connection.query('SELECT * FROM comments', function (err, rows) {
      if (err) return done(err)
      done(null, rows)
    })
  })
}

exports.getAllByUser = function(userId, done) {
  db.get(db.READ, function(err, connection) {
    if (err) return done('Database problem')

    connection.query('SELECT * FROM comments WHERE user_id = ?', userId, function (err, rows) {
      if (err) return done(err)
      done(null, rows)
    })
  })
}
```
你也可以创建一个代理连接对象,基于可以查询了解这是一个写或读,然后选择适当的连接。这样你甚至不会改变你的模块。

## 最后

本文翻译了<https://www.terlici.com/2015/08/13/mysql-node-express.html>的大部分，如果有错误，可以留言。
