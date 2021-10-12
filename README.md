# SQL TRICKS

## Summary

### Beginner
* [6 SQL Queries Every Data Engineer Should Be Aware Of](#6-sql-queries-every-data-engineer-should-be-aware-of)
### Initiate
### Intermediate
### Expert

## Beginner

### 6 SQL Queries Every Data Engineer Should Be Aware Of

> From: https://betterprogramming.pub/6-sql-queries-every-data-engineer-should-be-aware-of-2d0a2cc5986e
> 
> Author: [Cinto](https://cinto-sunny.medium.com/)
> 
> Rewriter: Renaud Racinet

It might be more than 45 years old, but SQL still gets the job done

Whether you are a beginner starting your engineering career, or you are an experienced data engineer or data analyst, knowledge of advanced SQL syntax is a must.
With exponential growth in data, it has become more important to analyze these data super quickly.

![](https://miro.medium.com/max/1230/1*SMZcPiBLxN2pJs-wVJlNBg.png)
Source: [Statista](https://www.statista.com/statistics/871513/worldwide-data-created/)

The units in this graph are zettabytes.


|    Units    | Acronym   | Values in bytes     |
|-------------|:----:|-------------------------:|
|1 KiloBytes  | Kb | 1,024                      |
|1 MegaBytes  | Mb | 1,048,576                  |
|1 GigaBytes  | Gb | 1,073,741,824              |
|1 TeraBytes  | Tb | 1,099,511,627,776          |
|1 PetaBytes  | Pb | 1,125,899,906,842,624      |
|1 ExaBytes   | Eb | 1,152921504606847e18       |
|1 ZettaByte  | Zb | 1,18059620717411e21        |
|1 YottaBytes | Yb | 1,208925819614629e24       |

And people may say that SQL is dead, but the reality is that there is no system currently to replace it currently. There are many very capable NoSQL stores that do their jobs very well, supporting massive scale out with low costs. However, they don’t replace high-quality SQL-based stores — they complement them. The ACID properties of SQL make it a highly reliable way to model data relatively naturally.

As a data engineer myself, I have been using SQL for a while, and I know the importance of writing complex queries faster. So, here is some advanced SQL syntax that will surely come in handy.

For the below examples, I have used the below table content:

| id    | Month     |   Type    |   Amount  |
|:-----:|:---------:|:----------|:----------|
|   1   |2021-01-01 |   M       |2000       |
|   2   |2021-01-01 |   F       |1200       |
|   3   |2021-01-02 |   M       |3000       |
|   4   |2021-01-02 |   F       |1500       |
|   5   |2021-01-02 |   F       |2200       |
|   6   |2021-01-03 |   M       |1800       |

##### Running Totals
You often come across scenarios where you have to calculate a running total from a table. This is to know what each value is, against a running total.

A running total refers to the sum of values in all cells of a column that precedes the next cell in that particular column.

Here is a query to do that.

```sql
SELECT id, month, Amount, SUM(Amount) OVER (ORDER BY id) as total_sum FROM Bill
```

|   id  |   month   |   Amount  | total_sum |
|:------|:---------:|:----------|:----------|
|   1   |2021-01-01 | 2000      | 2000      |
|   2   |2021-01-01 | 1200      | 3200      |
|   3   |2021-01-02 | 3000      | 6200      |
|   4   |2021-01-02 | 1500      | 7100      |
|   5   |2021-01-02 | 2200      | 9900      |
|   6   |2021-01-03 | 1800      | 11700     |

#### Common Table Expressions
The Common Table Expressions or CTE’s for short are used to simplify the readability of complex joins and subqueries.

It is basically a temporary named result set that you can reference within a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement.

Consider this simple query,

```sql
SELECT *
FROM bill
WHERE id in
    (
        SELECT DISTINCT id
        FROM id
        WHERE country = "US"
        AND status = "Y"
    )
```
git for cte. Added by [author](https://cinto-sunny.medium.com/)

Now imagine if we are using this subquery multiple times in the subsequent query. Won’t it be easier if we can use that as a temporary table? CTW solves this exact problem.

```sql
WITH istempp as (
    SELECT id as id
    FROM id
    WHERE country = "US"
    AND status = "Y"
)

SELECT *
FROM bill
WHERE id in (SELECT id FROM idtempp)
```

This is a small example, but this generally can be really useful for larger and complex subqueries.

#### Ranking the Data
Data engineers and analysts would agree that it is very common to rank values based on some parameter like salaries or expense, etc. And having the knowledge of ranking data at your fingertips can save you a lot of time finding out the exact query.

```sql
SELECT id, Amount, RANK() OVER (ORDER BY Amount desc)
FROM bill
```
A snippet of ranking. Added by the [author](https://cinto-sunny.medium.com/)

In the below query, I have ranked the dataset based on the amount column.

You can also use `DENSE_RANK()` which is similar to `RANK()` except that it doesn't skip the subsequent rank if two rows have the same value.

#### Adding Subtotals

Again, a super important query for data engineers and analysts. In my 10 year career working as a business/data analyst, I have used this query for a lot of analysis. Having a subtotal helps you put the data in perspective of the total.

It’s an extension of a GROUP BY clause with the ability to add subtotals and grand totals to your data.

```sql
SELECT Type, id, SUM(Amount) AS total_amount
FROM bill
GROUP BY Type, id WITH ROLLUP
```
A snippet of rollup. Added by [author](https://cinto-sunny.medium.com/)

| Type  | id     | total_amount |
|:------|:------:|:-------------|
|   F   |   2    | 1200         |
|   F   |   4    | 1500         |
|   F   |   5    | 2200         |
|   F   |   null | 4900         |
|   M   |   1    | 2000         |
|   M   |   3    | 3000         |
|   M   |   3    | 3000         |
|   M   |   null | 6800         |
|  null |   null | 11700        |

> **Note:** the above query is in MySQL. The rollup syntax may vary for others.

In the above query, the rows where both type and id are null is the one that is the total. You also have subtotals irrespective of the id column. That is represented by the 4th row and the second last row.

#### Temporary functions
Temporary functions allow you to modify the data easily without writing huge case statements.

In the below example, a temporary function is used to convert type to gender. This can be done using case statement inline in the query, but it would have been messy to read

```sql
CREATE TEMPORARY FUNCTION get_gender(type varchar) AS (
    CASE WHEN type = "M" THEN "male"
         WHEN type = "F" THEN "female"
         ELSE "n/a"
    END
)

SELECT name, get_gender(Type) as gender
FROM bill
```
A snippet of temporary function. Added by [author](https://cinto-sunny.medium.com/)

#### Variance and Standard Deviation
For data scientists and analysts, having the ability to get the variance and the standard deviation is crucial. Thankfully, there are functions to get these values.

The `VARIANCE`, `VAR_POP`, and `VAR_SAMP` are aggregate functions i.e they group the data. These are utilized to determine the variance, group variance, and sample variance of a collection of data individually.

```sql
SELECT 
    VARIANCE(amount) AS var_amount,
    VAR_POP(amount) AS pop_amount,
    VAR_SAMP(amount) AS samp_amount,
    STDDEV_SAMP(amount) AS stddev_amount,
    STDDEV_POP(amount) AS stddev_amount,
FROM bill
```
A snippet of variance and std deviation. Added by [author](https://cinto-sunny.medium.com/)

* `VAR_POP`    : This is the sample variance
* `VAR_SAMP`   : This is the population variance
* `STDDEV_SAMP`: This is the sample standard deviation
* `STDDEV_POP` : This is the population standard deviation

These are some of the top SQL commands that I have constantly used in my data engineering career. These have come in very handy for solving a lot of business problems. [Stats](https://thenewstack.io/sql-is-dead-right/) say that the ecosystem of SQL tools that includes anything from Excel and Tableau to [SparkSQL](https://scalegrid.io/blog/2019-database-trends-sql-vs-nosql-top-databases-single-vs-multiple-database-use/) is used by more than 60% of organizations. A very impressive feat, especially considering its age.

So, if you are a data engineer, I am sure you will find these commands useful. Let me know in the comments section if I have missed anything.


## Initiate

## Intermediate

## Expert
