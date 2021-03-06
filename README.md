[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![David deps][david-image]][david-url]
[![node version][node-image]][node-url]
[![npm download][download-image]][download-url]
[![npm license][license-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/@leizm/sql.svg?style=flat-square
[npm-url]: https://npmjs.org/package/@leizm/sql
[travis-image]: https://img.shields.io/travis/leizongmin/leizm-sql.svg?style=flat-square
[travis-url]: https://travis-ci.org/leizongmin/leizm-sql
[coveralls-image]: https://img.shields.io/coveralls/leizongmin/leizm-sql.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/leizongmin/leizm-sql?branch=master
[david-image]: https://img.shields.io/david/leizongmin/leizm-sql.svg?style=flat-square
[david-url]: https://david-dm.org/leizongmin/leizm-sql
[node-image]: https://img.shields.io/badge/node.js-%3E=_6.0-green.svg?style=flat-square
[node-url]: http://nodejs.org/download/
[download-image]: https://img.shields.io/npm/dm/@leizm/sql.svg?style=flat-square
[download-url]: https://npmjs.org/package/@leizm/sql
[license-image]: https://img.shields.io/npm/l/@leizm/sql.svg

# @leizm/sql

SQL 查询构造器

## 安装

```bash
npm install @leizm/sql --save
```

## 使用方法

```typescript
import { table, expr, query } from "@leizm/sql";
import * as mysql from "mysql";

// 普通查询
table("test")
  .select("a", "b")
  .where({ a: 1 })
  .and("b=?", [2])
  .orderBy("b DESC")
  .offset(10)
  .limit(5)
  .build();
// SELECT `a`, `b` FROM `test` WHERE `a`=1 AND `b`=2 ORDER BY b DESC LIMIT 10,5

// 连表查询
table("hello")
  .select("*")
  .as("A")
  .leftJoin("world")
  .as("B")
  .on("A.id=B.id")
  .where("1")
  .and("2")
  .offset(2)
  .limit(3)
  .build();
// SELECT `A`.*, `B`.* FROM `hello` AS `A` LEFT JOIN `world` AS `B` ON A.id=B.id WHERE 1 AND 2 LIMIT 2,3

// 插入数据
table("test1")
  .insert({
    a: 123,
    b: 456,
  })
  .build();
// INSERT INTO `test1` (`a`, `b`) VALUES (123, 456);

// 批量插入数据
table("test1")
  .insert([
    {
      a: 123,
      b: 456,
    },
    {
      a: 789,
      b: 110,
    },
  ])
  .build();
// INSERT INTO `test1` (`a`, `b`) VALUES (123, 456),
// (789, 110)");

// 插入数据，忽略已经存在记录
table("test1")
  .insert({ a: 123, b: 456 })
  .ignore()
  .build();
// INSERT IGNORE INTO `test1` (`a`, `b`) VALUES (123, 456)

// 插入数据，如果记录已经存在则改为更新
table("test1")
  .insert({ a: 123, b: 456 })
  .onDuplicateKeyUpdate()
  .set({ a: "xxx" })
  .build();
// INSERT INTO `test1` (`a`, `b`) VALUES (123, 456) ON DUPLICATE KEY UPDATE `a`='xxx'

// 替换数据
table("test1")
  .replace({
    a: 123,
    b: 456,
  })
  .build();
// REPLACE INTO `test1` (`a`, `b`) VALUES (123, 456);

// 更新数据
table("test1")
  .update({
    a: 123,
    b: 456,
  })
  .where({
    b: 777,
  })
  .limit(12)
  .build();
// UPDATE `test1` SET `a`=123, `b`=456 WHERE `b`=777 LIMIT 12

// 删除数据
table("test1")
  .delete()
  .where({
    b: 777,
  })
  .limit(12)
  .build();
// DELETE FROM `test1` WHERE `b`=777 LIMIT 12

// 子查询
table("test1")
  .select("*")
  .where("a=? AND b IN ???", [
    123,
    table("test2")
      .select("id")
      .where({ id: { $lt: 10 } })
      .limit(100),
  ])
  .build();
// SELECT * FROM `test1` WHERE a=123 AND b IN (SELECT `id` FROM `test2` WHERE `id`<10 LIMIT 100)

// 查询克隆
const q = table("test1")
  .select("*")
  .where({ a: 123 });
q.clone()
  .where({ b: 456 })
  .offset(10)
  .limit(20)
  .build();
// SELECT * FROM `test1` WHERE `a`=123 AND `b`=456 LIMIT 10,20
q.clone()
  .where({ b: 789, c: 666 })
  .orderBy("a DESC")
  .build();
// SELECT * FROM `test1` WHERE `a`=123 AND `b`=789 AND `c`=666 ORDER BY a DESC

// 条件表达式
table("test")
  .select("*")
  .where(
    expr()
      .and("a=?", [123])
      .or({ b: 456 })
      .and({ c: { $in: [789] } })
      .or("d=:d", { d: 666 }),
  )
  .build();
// SELECT * FROM `test` WHERE (a=123 OR `b`=456 AND `c` IN (789) OR d=666)

// 快捷执行MySQL查询
const conn = mysql.createConnection({});
query(
  conn,
  table("test")
    .select("*")
    .limit(1),
)
  .then(ret => console.log(ret))
  .catch(err => console.error(err));
```

## License

```text
MIT License

Copyright (c) 2018 Zongmin Lei <leizongmin@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
