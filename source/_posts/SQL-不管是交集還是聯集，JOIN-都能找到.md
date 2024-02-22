---
title: SQL - 不管是交集還是聯集，JOIN 都能找到
date: 2023-04-15 23:28:09
description: 學會 JOIN ，要找多難的集合資料都不是問題
---
# Joins

上一篇我們介紹了 includes 跟 joins 的差異，今天想介紹一下 Joins 的各式用法及不同

先來假設我們今天有兩個 TABLE1 跟 TABLE2

TABLE1 存放了 學生的資料(table name = Student)

| id | name | phone      |email      | course_id|
| -- | ---- | ---------- |--------   |----------|
| 1  | Ning | 0911123456 |ning@zz.zz |     1    |
| 2  | Amy  | 0922123789 |amy@zz.zz  |     2    |
| 3  | Jason| 0933456789 |jason@zz.zz|          |

TABLE2 存放了 課程的資料(table name = Course)

| id | course  | price |student_id |
| -- | ----    | ------|-------    |
| 1  | english | 150   |1          |
| 2  | japanese| 150   |2          |
| 3  | french  | 150   |           |

假設我們今天要看有哪個學生有買課程，我們來看一下各種 JOINS 的用法


### INNER JOIN

![](https://i.imgur.com/7jr1fKs.jpg)

情境：今天要看有哪個學生有買課程
需求：要顯示的欄位只需要 **有買課程的學生名字** / **課程金額**

這時候我們就可以用 INNER JOINS 去搜尋

* 可以把 INNER JOINS 想像成 Rails 的 Joins 方法

```sql=
SELECT TABLE1.id, TABLE1.name, TABLE2.price
FROM TABLE1
INNER JOIN TABLE2
ON TABLE1.id = TABLE2.student_id;
```

這邊解釋一下方法
* SELECT: 查詢的欄位
* FROM: 以哪個資料庫為基準
* ON: 放入篩選的條件，會根據這條件選出符合資格的資料

最終會篩出這些資料

| id | name|price|
| -- | ----| ---- | 
| 1  | Ning|150   |
| 2  | Amy |150   |


### LEFT JOIN

![](https://i.imgur.com/4PfO6wI.jpg)

情境：今天要看有哪個學生有買課程，哪個學生沒買課程
需求：要顯示的欄位需要 **所有學生的名字** / **課程金額**

這時候我們就可以用 LEFT JOIN 方法，可以把 LEFT 想像成 FROM TABLE1 的 TABLE1，是以 TABLE1 的資料欄位為主，保留 TABLE1 的每一筆資料，並把 SELECT 要的共同資料(TABLE2 的 price)篩選出來

```sql=
SELECT TABLE1.id, TABLE1.name, TABLE2.price
FROM TABLE1
LEFT JOIN TABLE2
ON TABLE1.id = TABLE2.student_id
```

所以 LEFT JOIN 會幫我們選出這些資料

| id | name |price|
| -- | ---- | ---- | 
| 1  | Ning |150   |
| 2  | Amy  |150   |
| 3  | Jason|      |


### RIGHT JOIN

![](https://i.imgur.com/09Yt3pl.jpg)

情境：哪個學生有買課程，也要看到哪些課程沒人買
需求：要顯示的欄位需要 **有買課程的學生** / **所有課程**

以 RIGHT JOIN 後面接的 TABLE2 為基準，保留 TABLE2 的每一筆資料，
並篩選出 SELECT 要的共同資料(TABLE1 的 name)篩選出來

```sql=
SELECT TABLE2.id, TABLE2.course, TABLE2.price, TABLE1.name, 
FROM TABLE1
RIGHT JOIN TABLE2
ON TABLE1.id = TABLE2.student_id
```

所以會做出

| id | course  |price|name |
| -- | ----    |---- |---- |
| 1  | english |150  |Ning |
| 2  | japanese|150  |Amy  |
| 3  | french  |150  |     |


### FULL OUTER JOIN

![](https://i.imgur.com/9z680N0.jpg)

情境：所有的學生跟所有課程的資料
需求：要顯示的欄位需要 **所有學生名字** / **所有課程資訊**

當兩個資料庫的資料都要全部保留的時候，並找出關聯時，就是 FULL OUTER JOIN 派上用場的時候了

```sql=
SELECT TABLE1.id, TABLE1.name, TABLE2.course, TABLE2.price
FROM TABLE1
FULL OUTER JOIN TABLE2
ON TABLE1.id = TABLE2.student_id
```

這時候就會做出

| id | name |course  |price |
| -- | ---- |----    |----  | 
| 1  | Ning |english |150   |
| 2  | Amy  |japanese|150   |
| 3  |Jason |        |      |


以上是 JOIN 比較基本的語法介紹，接下來我們可以來點進階的

### LEFT JOIN without data from table 2

![](https://i.imgur.com/dHPCtRC.jpg)

情境：有哪個學生**沒有買課程**
需求：要顯示的欄位需要 **沒有買課程的學生名字** / **沒有買課程的學生電話**

當我們需要的資料是沒有關聯的，並且只需要 TABLE1 (也就是 FROM 後面接的 TABLE)的資料時，就可以用 LEFT JOIN + WHERE

```sql=
SELECT TABLE1.id, TABLE1.name, TABLE1.phone
FROM TABLE1
LEFT JOIN TABLE2
ON TABLE1.id = TABLE2.student_id
WHERE TABLE2.student_id IS NULL
```

就會跑出

| id | name |phone      |
| ---| ---- |---------- |
| 3  |Jason |0933456789 |


### RIGHT JOINS without data from table 1

![](https://i.imgur.com/XGGUfee.jpg)

情境：哪個課程**沒有學生購買**
需求：要顯示的欄位需要 **沒有學生購買的課程名稱**

用法與 LEFT JOIN without data from table 2 一樣

```sql=
SELECT TABLE2.id, TABLE2.course
FROM TABLE1
RIGHT JOIN TABLE2
ON TABLE1.id = TABLE2.student_id
WHERE TABLE1.course_id IS NULL
```

這時候就會跑出

| id | course  |
| -- | ----    |
| 3  | french  |


### FULL OUTER JOINS without data from table 1 and 2

![](https://i.imgur.com/eFZMqEb.jpg)

情境：今天要看到**哪個課程沒有學生購買** 或者 **哪個學生沒買課程**
需求：要顯示的欄位需要 **沒買課程的學生名稱** 及 **沒學生購買的課程名稱**

尋找的條件就會使用 OR 來尋找，這些資料可以說是這兩個資料庫沒交集的資料群

```sql=
SELECT TABLE1.name, TABLE2.course
FROM TABLE1
FULL OUTER JOIN TABLE2
ON TABLE1.id = TABLE2.student_id
WHERE TABLE2.student_id IS NULL
OR TABLE1.course_id IS NULL
```

所以就會找出

| name | course  |
| --   | ----    |
| Jason|         |
|      | french  |