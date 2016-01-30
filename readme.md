[![NPM version][npm-version-image]][npm-url] [![NPM downloads][npm-downloads-image]][npm-url] [![MIT License][license-image]][license-url]

# A very helpful ORM for node.js

`npm install sqlagent`

Then is needed to install a database driver: `npm install pg` or `npm install mysql` or `npm install mssql`

---

- Currently supports __PostgreSQL__, __MySQL__, __SQL Server__ and MongoDB
- Simple and powerful
- Best use with [total.js - web application framework for node.js](https://www.totaljs.com)

__IMPORTANT__:

- code is executed as it's added
- rollback is executed automatically when is transaction enable
- SQL Server: pagination works only in SQL SERVER >=2012
- `SqlBuilder` is a global object

## Initialization

### Basic initialization

#### PostgreSQL

```javascript
var Agent = require('sqlagent/pg');
var sql = new Agent('connetion-string-to-postgresql');
```

#### MySQL

```javascript
var Agent = require('sqlagent/mysql');
// var sql = new Agent('mysql://user:password@127.0.0.1/database');
var sql = new Agent({ host: '...', database: '...' });
```

#### SQL Server (MSSQL)

```javascript
var Agent = require('sqlagent/sqlserver');
// var sql = new Agent('mssql://user:password@127.0.0.1/database');
var sql = new Agent({ server: '...', database: '...' });
```

#### MongoDB

```javascript
var Agent = require('sqlagent/mongodb');
var nosql = new Agent('connetion-string-to-mongodb');
```

### Initialization for total.js

```javascript
// Below code rewrites total.js database prototype
require('sqlagent/pg').init('connetion-string-to-postgresql', [debug]); // debug is by default: false
require('sqlagent/mysql').init('connetion-string-to-mysql', [debug]); // debug is by default: false
require('sqlagent/sqlserver').init('connetion-string-to-sqlserver', [debug]); // debug is by default: false
require('sqlagent/mongodb').init('connetion-string-to-mongodb', [debug]); // debug is by default: false

// var sql = DATABASE([ErrorBuilder]);
var sql = DATABASE();
// sql === SqlAgent
```

## Usage

### Select

```plain
instance.select([name], table, [columns])
```

- `name` (String) is an identificator for results, optional (default: internal indexer)
- `table` (String) table name, the library automatically creates SQL query
- `colums` (String, Array or Object (keys will be as columns))
- __returns__ SqlBuilder

```javascript
sql.select('users', 'tbl_user').make(function(builder) {
    builder.where('id', '>', 5);
    builder.page(10, 10);
});

sql.select('orders', 'tbl_order').make(function(builder) {
    builder.where('isremoved', false);
    builder.page(10, 10);
    builder.fields('amount', 'datecreated');
});

sql.select('products', 'tbl_products').make(function(builder) {
    builder.between('price', 30, 50);
    builder.and();
    builder.where('isremoved', false);
    builder.limit(20);
    builder.fields('id', 'name');
});

sql.exec(function(err, response) {
    console.log(response.users);
    console.log(response.products);
    console.log(response.admin);
});
```

### Save

```plain
instance.save([name], table, isINSERT, prepare(builder, isINSERT));
```

```javascript
sql.save('user', 'tbl_user', somemodel.id === 0, function(builder, isINSERT) {

    builder.set('name', somemodel.name);

    if (isINSERT) {
        builder.set('datecreated', new Date());
        return;
    }

    builder.inc('countupdate', 1);
    builder.where('id', somemodel.id);
});
```

### Insert

```plain
instance.insert([name], table, [value])
```

- `name` (String) is an identificator for results, optional (default: internal indexer)
- `table` (String) table name, the library automatically creates SQL query
- `value` (Object) optional (value can be SqlBuilder)
- __returns__ if value is undefined then __SqlBuilder__ otherwise __SqlAgent__

```javascript
sql.insert('user', 'tbl_user').make(function(builder) {
    builder.set({ name: 'Peter', age: 30 });
});

sql.insert('log', 'tbl_logs').make(function(builder) {
    builder.set('message', 'Some log message.');
    builder.set('created', new Date());
});

sql.exec(function(err, response) {
    console.log(response.user); // response.user.identity (INSERTED IDENTITY)
    console.log(response.log); // response.log.identity (INSERTED IDENTITY)
});
```

### Update

```plain
instance.update([name], table, [value])
```

- `name` (String) is an identificator for results, optional (default: internal indexer)
- `table` (String) table name, the library automatically creates SQL query
- `value` (Object) optional (value can be SqlBuilder)
- __returns__ if value is undefined then __SqlBuilder__ otherwise __SqlAgent__

```javascript
sql.update('user1', 'tbl_user').make(function(builder) {
    builder.set({ name: 'Peter', age: 30 });
    builder.where('id', 1);
});

// is same as
sql.update('user2', 'tbl_user').make(function(builder) {
    builder.where('id', 1);
    builder.set('name', 'Peter');
    builder.set('age', 30);
});

sql.exec(function(err, response) {
    console.log(response.user1); // returns {Number} (count of changed rows)
    console.log(response.user2); // returns {Number} (count of changed rows)
});
```

### Delete

```plain
instance.delete([name], table)
instance.remove([name], table)
```

- `name` (String) is an identificator for results, optional (default: internal indexer)
- `table` (String) table name, the library automatically creates SQL query
- __returns__ SqlBuilder

```javascript
sql.remove('user', 'tbl_user').make(function(builder) {
    builder.where('id', 1);
});

sql.exec(function(err, response) {
    console.log(response.user); // returns {Number} (count of deleted rows)
});
```

### Query

```plain
instance.query([name], query, [params])
```

- `name` (String) is an identificator for results, optional (default: internal indexer)
- `query` (String) SQL query
- `params` (Object Array) optional
- __returns__ if params is undefined then __SqlBuilder__ otherwise __SqlAgent__

```javascript
sql.query('user', 'SELECT * FROM tbl_user').make(function(builder) {
    builder.where('id', 1);
});

sql.exec(function(err, response) {
    console.log(response.user);
});
```

### Aggregation

```plain
instance.count([name], table)
```

- __returns__ SqlBuilder

```javascript
var count = sql.count('users', 'tbl_user');
count.between('age', 20, 40);

sql.exec(function(err, response) {
    console.log(response.users); // response.users === number
});
```

---

```plain
instance.max([name], table, column)
instance.min([name], table, column)
instance.avg([name], table, column)
```

- __returns__ SqlBuilder

```javascript
var max = sql.max('users', 'tbl_user', 'age');
max.where('isremoved', false);

sql.exec(function(err, response) {
    console.log(response.users); // response.users === number
});
```

### Exists

```plain
instance.exists([name], table)
```

- __returns__ SqlBuilder

```javascript
var exists = sql.exists('user', 'tbl_user');
exists.where('id', 35);

sql.exec(function(err, response) {
    console.log(response.user); // response.user === Boolean (in correct case otherwise undefined)
});
```

---

```plain
instance.max([name], table, column)
instance.min([name], table, column)
instance.avg([name], table, column) // doesn't work with Mongo
```

- __returns__ SqlBuilder

```javascript
var max = sql.max('users', 'tbl_user', 'age');
max.where('isremoved', false);

sql.exec(function(err, response) {
    console.log(response.users); // response.users === number
});
```

### Transactions

- doesn't work with MongoDB
- rollback is performed automatically

```javascript
sql.begin();
sql.insert('tbl_user', { name: 'Peter' });
sql.commit();
```

## Special cases

### How to set the primary key?

- doesn't work with MongoDB

```javascript
// instance.primary('column name') is same as instance.primaryKey('column name')

instance.primary('userid');
instance.insert('tbl_user', ...);

instance.primary('productid');
instance.insert('tbl_product', ...);

instance.primary(); // back to default "id"
```

- default `primary key name` is `id`
- works only in PostgreSQL because INSERT ... RETURNING __must have specific column name__

### How to use latest primary id value for relations?

```javascript
// primary key is id + autoincrement
var user = sql.insert('user', 'tbl_user');
user.set('name', 'Peter');

var address = sql.insert('tbl_user_address');
address.set('id', sql.$$);
address.set('country', 'Slovakia');

sql.exec();
```

### How to use latest primary id value for multiple relations?

- doesn't work with MongoDB

```javascript
// primary key is id + autoincrement
var user = sql.insert('user', 'tbl_user');
user.set('name', 'Peter');

// Lock latest inserted identificator
sql.lock();
// is same as
// sql.put(sql.$$);

var address = sql.insert('tbl_user_address');
address.set('iduser', sql.$$); // adds latest primary id value
address.set('country', 'Slovakia');

var email = sql.insert('tbl_user_email');
email.set('iduser', sql.$$); // adds locked value
email.set('email', 'petersirka@gmail.com');
sql.unlock();

sql.exec();
```

### If not or If exists

```javascript
instance.ifnot('user', function(error, response) {
    // error === ErrorBuilder
    // It will be executed when the results won't be contain `user` property
    // Is executed in order
});

instance.ifexists('user', function(error, response) {
    // error === ErrorBuilder
    // It will be executed when the results will contain `user` property
    // Is executed in order
});
```

### Default values

- you can set default values
- values are bonded immediately (not in order)

```javascript
sql.default(function(response) {
    response.count = 0;
    response.user = {};
    response.user.id = 1;
});

// ...
// ...

sql.exec(function(err, response) {
    console.log(response);
});
```

### Modify results

- values are bonded in an order

```javascript
sql.select(...);
sql.insert(...);

sql.modify(function(response) {
    response.user = {};
    response.user.identity = 10;
});

// ...
// ...

// Calling:
// 1. select
// 2. insert
// 3. modify
// 4. other commands
sql.exec(function(err, response) {
    console.log(response);
});
```

### Preparing (dependencies)

- you can use multiple `sql.prepare()`

```javascript
var user = sql.update('user', 'tbl_user');
user.where('id', 20);
user.set('name', 'Peter');

var select = sql.select('address', 'tbl_address');
select.where('isremoved', false);
select.and();
select.where('city', 'Bratislava');
select.limit(1);

// IMPORTANT:
sql.prepare(function(error, response, resume) {
    // error === ErrorBuilder
    sql.builder('address').set('idaddress', response.address.id);
    resume();
});

var address = sql.update('address', 'tbl_user_address');
address.where('iduser', 20);

sql.exec();
```

### Validation

- you can use multiple `sql.validate()`

```plain
sql.validate(fn)
```

```javascript
var select = sql.select('address', 'tbl_address');
select.where('isremoved', false);
select.and();
select.where('city', 'Bratislava');
select.limit(1);

// IMPORTANT:
sql.validate(function(error, response, resume) {

    // error === ErrorBuilder

    if (!response.address) {
        error.push('Sorry, address not found');
        // cancel pending queries
        return resume(false);
    }

    sql.builder('user').set('idaddress', response.id);

    // continue
    resume();
});

var user = sql.update('user', 'tbl_user');
user.where('id', 20);
user.set('name', 'Peter');

sql.exec();
```

---

```plain
sql.validate([result_name_for_validation], error_message, [reverse]);
```

- `result_name_for_validation` (String) a result to compare.
- `error_message` (String) an error message
- `reverse` (Boolean) a reverse comparison (false: result must exist (default), true: result must be empty)
__

If the function throw error then SqlAgent cancel all pending queris (perform Rollback if the agent is in transaction mode) and executes callback with error.

```javascript
var select = sql.select('address', 'tbl_address');
select.where('isremoved', false);
select.and();
select.where('city', 'Bratislava');
select.limit(1);

// IMPORTANT:
sql.validate('Sorry, address not found');

var user = sql.select('user', 'tbl_user');
user.where('id', 20);

sql.validate('Sorry, user not found');
sql.validate('Sorry, address not found for the current user', 'address');

sql.exec();
```

## Global

### Skipper

```javascript
sql.select('users', 'tbl_users');
sql.skip(); // skip orders
sql.select('orders', 'tbl_orders');

sql.bookmark(function(error, response) {
    // error === ErrorBuilder
    // skip logs
    sql.skip('logs');
});

sql.select('logs', 'tbl_logs');

sql.exec(function(err, response) {
    console.log(response); // --- response will be contain only { users: [] }
});
```

### Bookmarks

Bookmark is same as `sql.prepare()` function but without `resume` argument.

```javascript
sql.select('users', 'tbl_users');

sql.bookmark(function(error, response) {
    // error === ErrorBuilder
    console.log(response);
    response['custom'] = 'Peter';
});

sql.select('orders', 'tbl_orders');

sql.exec(function(err, response) {
    response.users;
    response.orders;
    response.custom; // === Peter
});
```

### Error handling

```javascript
sql.select('users', 'tbl_users');

sql.validate(function(error, response, resume) {

    // error === ErrorBuilder

    if (!response.users || respone.users.length === 0)
        error.push(new Error('This is error'));

    // total.js:
    // error.push('error-users-empty');

    resume();
});

sql.select('orders', 'tbl_orders');

// sql.validate([error message], [result name for validation])
sql.validate('error-orders-empty');
// is same as:
// sql.validate('error-orders-empty', 'orders');

sql.validate('error-users-empty', 'users');
```

### Predefined queries

- doesn't work with MongoDB

```plain
Agent.query(name, query);
```

```javascript
Agent.query('users', 'SELECT * FROM tbl_users');
Agent.query('allorders', 'SELECT * FROM view_orders');

sql.query('users').where('id', '>', 20);
sql.query('orders', 'allorders').limit(5);

sql.exec(function(err, response) {
    console.log(response[0]); // users
    console.log(response.orders); // orders
});
```

### Waiting for specified values

- `+3.1.0`

```javascript
sql.when('users', function(error, response) {
    console.log(response.users);
});

sql.when('orders', function(error, response) {
    console.log(response.orders);
});

sql.select('users', 'tbl_users');
sql.select('orders', 'tbl_orders');
sql.exec();
```

## Bonus

### How to get latest inserted ID?

- doesn't work with MongoDB

```javascript
sql.insert('user', 'tbl_user').set('name', 'Peter');

sql.bookmark(function() {
    console.log(sql.id);
});

sql.exec();
```

### Expected values? No problem

```plain
sql.expected(name, index, property); // gets a specific value from the array
sql.expected(name, property);
```

```javascript
sql.select('user', 'tbl_user').where('id', 1).first();
sql.select('products', 'tbl_product').where('iduser', sql.expected('user', 'id'));

sql.exec();
```

### Measuring time

```javascript
sql.exec(function(err, response) {
    console.log(sql.time + ' ms');
    // or
    // console.log(this.time)
});
```

### Events

```
sql.on('query', function(name, query, params){});
sql.on('data', function(name, response){});
sql.on('end', function(err, response, time){});
```

### Generators in total.js

```javascript
function *some_action() {
    var sql = DB();

    sql.select('users', 'tbl_user').make(function(select) {
        select.where('id', '>', 100);
        select.and();
        select.where('id', '<', 1000);
        select.limit(10);
    });

    sql.select('products', 'tbl_product').make(function(select) {
        select.where('price', '<>', 10);
        select.limit(10);
    });

    // get all results
    var results = yield sync(sql.$$exec())();
    console.log(results);

    // or get a specific result:
    var result = yield sync(sql.$$exec('users'))();
    console.log(result);
}
```

### Priority

Set a command priority, so the command will be processed next round.

```javascript
sql.select('... processed as second')
sql.select('... processed as first');
sql.priority(); // --> takes last item in queue and inserts it as first (sorts it immediately).
```


### Debug mode

Debug mode writes each query to console.

```javascript
sql.debug = true;
```

### We need to return into the callback only one value from the response object

```javascript
sql.exec(callback, 0); // --> returns first value from response (if isn't error)
sql.exec(callback, 'users'); // --> returns response.users (if is isn't error)

sql.exec(function(err, response) {
    if (err)
        throw err;
    console.log(response); // response will contain only orders
}, 'orders');
```

## SqlBuilder

- automatically adds `and` if is not added between e.g. 2x where

```javascript
// Creates SqlBuilder
var builder = sql.$;

builder.where('id', '<>', 20);
builder.set('isconfirmed', true);

// e.g.:
sql.update('users', 'tbl_users', builder);
sql.exec(function(err, response) {
    console.log(response.users);
})
```

---

#### builder.set()

```plain
builder.set(name, value)
```

adds a value for update or insert

- `name` (String) column name
- `value` (Object) value

---

#### builder.raw()

```plain
builder.raw(name, value)
```

adds a raw value for update or insert without SQL encoding

- `name` (String) column name
- `value` (Object) value

---

```plain
builder.set(obj)
```
adds an object for update or insert value collection

```javascript
builder.set({ name: 'Peter', age: 30 });
// is same as
// builder.set('name', 'Peter');
// builder.set('age', 30);
```

---

#### builder.inc()

```plain
builder.set(name, [type], value)
```

adds a value for update or insert

- `name` (String) column name
- `type` (String) increment type (`+` (default), `-`, `*`, `/`)
- `value` (Number) value

```javascript
builder.inc('countupdate', 1);
builder.inc('countview', '+', 1);
builder.inc('credits', '-', 1);

// Short write
builder.inc('countupdate', '+1');
builder.inc('credits', '-1');
```

---

#### builder.rem()

```plain
builder.rem(name)
```
removes an value for inserting or updating.

```javascript
builder.set('name', 'Peter');
builder.rem('name');
```

---

#### builder.sort()

```plain
builder.sort(name, [desc])
builder.order(name, [desc])
```
adds sorting

- `name` (String) column name
- `desc` (Boolean), default: false

---

#### builder.skip()

```plain
builder.skip(value)
```
skips records

- `value` (Number or String), string is automatically converted into number

---

#### builder.take()

```plain
builder.take(value)
builder.limit(value)
```
takes records

- `value` (Number or String), string is automatically converted into number

---

#### builder.page()

```plain
builder.page(page, maxItemsPerPage)
```
sets automatically sql.skip() and sql.take()

- `page` (Number or String), string is automatically converted into number
- `maxItemsPerPage` (Number or String), string is automatically converted into number

---

#### builder.first()

```plain
builder.first()
```
sets sql.take(1)

---

#### builder.join()

- doesn't work with MongoDB

```plain
builder.join(name, on, [type])
```

adds a value for update or insert

- `name` (String) table name
- `on` (String) condition
- `type` (String) optional, inner type `inner`, `left` (default), `right`

```javascript
builder.join('address', 'address.id=user.idaddress');
```

---

#### builder.where()

```plain
builder.where(name, [operator], value)
builder.push(name, [operator], value)
```
add a condition after SQL WHERE

- `name` (String) column name
- `operator` (String), optional `>`, `<`, `<>`, `=` (default)
- `value` (Object)

---

#### builder.group()

- doesn't work with MongoDB

```plain
builder.group(name)
builder.group(name1, name2, name3); // +v2.9.1
```
creates a group by in SQL query

- `name` (String or String Array)

---

#### builder.having()

- doesn't work with MongoDB

```plain
builder.having(condition)
```
adds having in SQL query

- `condition` (String), e.g. `MAX(Id)>0`

---

#### builder.and()

```plain
builder.and()
```
adds AND to SQL query

---

#### builder.or()

```plain
builder.or()
```
adds OR to SQL query

---

#### builder.in()

```plain
builder.in(name, value)
```
adds IN to SQL query

- `name` (String), column name
- `value` (String, Number or String Array, Number Array)

---

#### builder.between()

```plain
builder.between(name, a, b)
```
adds between to SQL query

- `name` (String), column name
- `a` (Number)
- `b` (Number)

---

#### builder.like()

```plain
builder.like(name, value, [where])
```
adds like command

- `name` (String) column name
- `value` (String) value to search
- `where` (String) optional, e.g. `beg`, `end`, `*` ==> %search (beg), search% (end), %search% (*)

---

#### builder.sql()

- doesn't work with MongoDB

```plain
builder.sql(query, [param1], [param2], [param..n])
```
adds a custom SQL to SQL query

- `query` (String)

```javascript
builder.sql('age=? AND name=?', 20, 'Peter');
```

---

#### builder.scope()

```plain
builder.scope(fn);
```
adds a scope `()`

```javascript
builder.where('user', 'person');
builder.and();
builder.scope(function() {
    builder.where('type', 20);
    builder.or();
    builder.where('age', '<', 20);
});
// creates: user='person' AND (type=20 OR age<20)
```

#### builder.define()

```plain
builder.define(name, SQL_TYPE_LOWERCASE);
```
- __only for SQL SERVER__
- change the param type

```javascript
var insert = sql.insert('user', 'tbl_user');

insert.set('name', 'Peter Širka');
insert.define('name', 'varchar');
insert.set('credit', 340.34);
insert.define('credit', 'money');
sql.exec();
```

---

#### builder.schema()

- doesn't work with MongoDB

```plain
builder.schema()
```
sets current schema for `join`, `where`, `in`, `between`, `field`, `fields`, `like`

```javascript
builder.schema('b');
builder.fields('name', 'age'); // --> b."name", b."age"
builder.schema('a');
builder.fields('name', 'age'); // --> a."name", a."age"
builder.fields('!COUNT(id) as count') // --> a.COUNT()
```

#### builder.escape()

- doesn't work with MongoDB

```plain
builder.escape(string)
```
escapes value as prevention for SQL injection

#### builder.fields()

```plain
builder.fields()
```
sets fields for data selecting.

```javascript
builder.fields('name', 'age'); // "name", "age"
builder.fields('!COUNT(id)'); // Raw field: COUNT(id)
builder.fields('!COUNT(id) --> number'); // Raw field with casting: COUNT(id)::int (in PG), CAST(COUNT(id) as INT) (in SQL SERVER), etc.
```

#### builder.replace()

```plain
builder.replace(builder)
```
replaces current instance of SqlBuilder with new.

- `builder` (SqlBuilder) Another instance of SqlBuilder.


---

#### builder.toString()

- doesn't work with MongoDB

```plain
builder.toString()
```
creates escaped SQL query (internal)

[license-image]: https://img.shields.io/badge/license-MIT-blue.svg?style=flat
[license-url]: license.txt

[npm-url]: https://npmjs.org/package/sqlagent
[npm-version-image]: https://img.shields.io/npm/v/sqlagent.svg?style=flat
[npm-downloads-image]: https://img.shields.io/npm/dm/sqlagent.svg?style=flat
