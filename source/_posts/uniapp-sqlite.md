---
title: 关于uni-app使用sqlite来做App的本地缓存
tags:
  - uni-app
  - sqlite
categories:
  - Web前端
---

> 对于开发项目使用到的本地存储，一般用缓存storage就足够了，但是如果碰到数据量比较大的情况下，就需要使用sqlite数据库来进行数据的存储了。
## 问题
比如我碰到的一个业务，因为操作环境是处于断网的情况下，所以用户的数据需要存储到本地，数据包括一系列文字内容和图片；所以，我采用了分开存储的方法，图片存在相册里，包括图片路径的其他数据存到sqlite。
## sqlite
开始写业务之前，我们得了解uni-app是怎么使用sqlite的，uni-app内置的sqlite方法目前一共有6个：  
***注意：HBuilderX1.7.2及以上版本支持此功能。***
### 打开/创建数据库
```js
void plus.sqlite.openDatabase(options)
```
这个api需要传入一个至少包含`name`和`path`的对象，`name`为数据库名称，`path`为数据库的地址，官方推荐的是以`_doc/`为开始，例如`doc/demo.db`。  
示例代码：
```js
function openDatabase () {
    return new Promise((resolve, reject) => {
        plus.sqlite.openDatabase({
            name: 'demo',
            path: '_doc/demo.db',
            success: () => {
                resolve('ok.')
            },
            fail: () => {
                reject('fail.')
            }
        })
    })
}
```
### 判断数据库是否打开
```js
Boolean plus.sqlite.isOpenDatabase(options)
```
这个api用于判断数据库是否开启，因为数据库打开的情况下，再次打开是会走失败的回调函数的，所以再打开数据前，应当判断当前状态；传入的对象也是包含`name`和`path`两个必填参数，返回值为`Boolean`。
示例代码：
```js
const isOpen = plus.sqlite.isOpenDatabase({
    name: 'demo',
    path: '_doc/demo.db'
})
```
### 关闭数据库
```js
void plus.sqlite.closeDatabase(options)
```
关闭数据库，传入必填参数`name`和选填参数`success`以及`fail`。
### 执行查询语句
```js
void plus.sqlite.selectSql(options)
```
这个api用于查询表数据用，除了`name`，`success`，`fail`等参数外，需要传入`sql`参数，`sql`是一条sql语句（基本同mysql的语法一致）。  
示例代码：
```js
function findData () {
    return new Promise((resolve, reject) => {
        plus.sqlite.selectSql({
            name: 'demo',
            sql: 'select * from users',
            success: data => {
                resolve(data)
            },
            fail: () => {
                reject('fail.')
            }
        })
    })
}
```
`selectSql`这个方法的`success`回调函数的返回参数是有值的，即数据库查询出来的数据，其格式为JSON对象数组，没有数据的话就是一个空数组。
### 执行增删改创建表等语句
```js
void plus.sqlite.executeSql(options)
```
和`selectSql`类似，也是需要传入`sql`字段，但是这里有所不同的是，`executeSql`传入的`sql`除了是一个字符串类型外，还可以是字符串数组，表示执行多次操作。  
示例代码：
```js
function createUsers () {
    return new Promise((resolve, reject) => {
        plus.sqlite.executeSql({
            name: 'demo',
            sql: `
            create table if not exists users (
                "id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL COMMENT "唯一ID",
                "name" varchar(32) NOT NULL COMMENT "名称"
            )
            `,
            success: () => {
                resolve('ok.')
            },
            fail: () => {
                reject('fail.')
            }
        })
    })
}
```
```js
return new Promise((resolve, reject) => {
        plus.sqlite.executeql({
            name: 'demo',
            sql: `insert into users (name) values ('lihua')`,
            success: () => {
                plus.sqlite.selectSql({
                    name: 'demo',
                    sql: 'select last_insert_rowid()',
                    success: data => {
                        // 获取插入后的主键id
                        resolve(data)
                    }
                })
            },
            fail: () => {
                reject('fail.')
            }
        })
    })
```
### 开启事务
这个就是保证数据的一致性和原子性的，简单点来说，就是在一个事务里面做的多次sql操作，要么都成功，要么都不成功。
```js
void plus.sqlite.transaction(options)
```
除`name`，`success`，`fail`外多一个`operation`参数，一共有三个值：begin（开始事务）、commit（提交）、rollback（回滚）。需要注意的是，开启事务后得记住关闭事务，要么提交要么回滚（回滚一般在异常情况下执行）。
## 结束
uni-app提供操作sqlite的api还是非常简单的，当需要缓存大量数据的时候，就可以考虑使用这个了，需要注意的是：一但卸载的App的话sqlite的数据库会一起丢失。