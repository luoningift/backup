---
title: MySQL的json操作
date: 2016-12-20 18:40:16
tags:
- MySQL
- Json
categories:
- MySQL
---
#### 创建表
```sql
CREATE TABLE json_test(id INT(11) AUTO_INCREMENT PRIMARY KEY,person_desc JSON)ENGINE INNODB;
```
#### 插入数据：
```sql
insert into json_test (person_desc) values ('{"name":"adfasdfa","luoning":{"t1":"sdfd","t2":"sdfj"}}');
```
#### 更新数据：
```sql
UPDATE json_test SET person_desc = json_set(person_desc,'$.name','罗宁');
UPDATE json_test SET person_desc = json_set(person_desc,'$.luoning.t1','罗宁');
```
#### 合并json数据
```sql
UPDATE json_test SET person_desc = json_merge(person_desc,'{"luoning1":{"id":"asdasdfasd","name":"asdfasdfasdfas"}}') where id = 1
```
#### 查询数据
```sql
SELECT * FROM user WHERE person_desc ->'$.name' = 'adfasdfa';
SELECT * FROM user WHERE JSON_EXTRACT(person_desc,'$.name') = 'adfasdfa';
SELECT * FROM user WHERE JSON_EXTRACT(person_desc,'$.luoning.t1') = 'sdfa';
```
#### 参考资料
- 1.[JSON Function Reference](https://dev.mysql.com/doc/refman/5.7/en/json-function-reference.html)
- 2.[The JSON Data Type](http://dev.mysql.com/doc/refman/5.7/en/json.html)
