+ ## Business Requirement:

Hi Daniel,

Thank you for the insight shared on the DVD Movie Rental dataset, your findings helped the senior executives to decide what to do next.

Could you please take a look at the `User_logs data` in the Health schema and let me know what you find. I will appreciate if I can get it before noon tomorrow for review since the meeting with the client is scheduled for Monday. Thank you

Kind Regards,

Danny Ma.   

##  SOLUTION:
The table has the following columns and datatypes:

- id ----> varchar(40) (strings)
- log_date ----> date
- measure ----> text (strings)
- measure_value ----> numeric (int)
- systolic ----> numeric (int)
- diabolic ----> numeric (int)

> The only difference between TEXT and VARCHAR(n) is that you can limit the maximum length of a VARCHAR column, for example, VARCHAR(255)does not allow inserting a string more than 255 characters long.

Let's see the first 10 rows of the table.

**Query:**

```sql
SELECT *
FROM health.user_logs
LIMIT 10;
```
**Result:**

|id|log_date|measure|measure_value|systolic|diastolic|
|:----|:----|:----|:----|:----|:----|
|fa28f948a740320ad56b81a24744c8b81df119fa|2020-11-15T00:00:00.000Z|weight|46.03959| | |
|1a7366eef15512d8f38133e7ce9778bce5b4a21e|2020-10-10T00:00:00.000Z|blood_glucose|97|0|0|
|bd7eece38fb4ec71b3282d60080d296c4cf6ad5e|2020-10-18T00:00:00.000Z|blood_glucose|120|0|0|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-10-17T00:00:00.000Z|blood_glucose|232|0|0|
|d14df0c8c1a5f172476b2a1b1f53cf23c6992027|2020-10-15T00:00:00.000Z|blood_pressure|140|140|113|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-10-21T00:00:00.000Z|blood_glucose|166|0|0|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-10-22T00:00:00.000Z|blood_glucose|142|0|0|
|87be2f14a5550389cb2cba03b3329c54c993f7d2|2020-10-12T00:00:00.000Z|weight|129.060012817|0|0|
|0efe1f378aec122877e5f24f204ea70709b1f5f8|2020-10-07T00:00:00.000Z|blood_glucose|138|0|0|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-10-04T00:00:00.000Z|blood_glucose|210| | |

The snapshot of the first 10 rows showed that the `Health.user_logs` table has 6 columns. To know the total record count on this table, I ran the following query

**Query:**

```sql
SELECT COUNT(*)
FROM health.user_logs;
```
|count|
|:----|
|43891|

This showed that there are **43,891** records in the table. It will be okay to know the unique `id` values there are in this dataset. This will give us a feel of how many unique users there are whilst we continue getting a better picture of the dataset. We get this by running the following query,

```sql
SELECT COUNT(DISTINCT id)
FROM health.user_logs;
```
The query showed that there are **554** unique id values in the `Health.user_logs` table. It will be okay to know the earliest and the lastest date in the `log_date` column.

```sql
SELECT
  MIN(log_date) AS min_log_date,
  MAX(log_date) AS max_log_date
FROM health.user_logs;
```
|min_log_date|max_log_date|
|:----|:----|
|2015-01-11T00:00:00.000Z|2020-11-15T00:00:00.000Z|

The `log_date` column revealed that the records were logged between **2015-01-11** and **2020-11-15**, which is almost 6 years.
Let's inspect the `measure` column in the table and take a look at the most frequent values within this column. See the query below,

```sql
SELECT
  measure,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY frequency DESC;
```
|measure|frequency|percentage|
|:----|:----|:----|
|blood_glucose|38692|88.15|
|weight|2782|6.34|
|blood_pressure|2417|5.51|

As seen in the table above, most of the `measure` value are for `blood_glucose`. It will be nice if we look at the `id` column as well,

```sql
SELECT
  id,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY id
ORDER BY frequency DESC
LIMIT 10;
```
|id|frequency|percentage|
|:----|:----|:----|
|054250c692e07a9fa9e62e345231df4b54ff435d|22325|50.86|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|1589|3.62|
|ee653a96022cc3878e76d196b1667d95beca2db6|1235|2.81|
|abc634a555bbba7d6d6584171fdfa206ebf6c9a0|1212|2.76|
|576fdb528e5004f733912fae3020e7d322dbc31a|1018|2.32|
|87be2f14a5550389cb2cba03b3329c54c993f7d2|747|1.70|
|46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9|651|1.48|
|fba135f6a50a2e3f371e47f943b025705f9d9617|633|1.44|
|d696925de5e9297694ef32a1c9871f3629bec7e5|597|1.36|
|6c2f9a8372dac248192c50219c97f9087ab778ba|582|1.33|

The result above showed that about 51% of the total records are from one  `id` or one user. Let's also take a quick look at the most frequent values for each column:

### Measure_value column
```sql
SELECT
  measure_value,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY measure_value
ORDER BY frequency DESC
LIMIT 10;
```
|measure_value|frequency|
|:----|:----|
|0|572|
|401|433|
|117|390|
|118|346|
|123|342|
|122|331|
|126|326|
|120|323|
|128|319|
|115|319|

It will be okay if we have a descriptive statistics of this column too. Let's check it out with the folowing query

```sql
SELECT
  MIN(measure_value) AS min_measure_value,
  MAX(measure_value) AS max_measure_value,
  ROUND(
  AVG(measure_value),
  2
  ) AS avg_measure_value,
  ROUND(
  STDDEV(measure_value),
  2) AS stdev_measure_value
FROM health.user_logs;
```
|min_measure_value|max_measure_value|avg_measure_value|stdev_measure_value|
|:----|:----|:----|:----|
|-1|39642120|1986.23|267610.87|

The minimum value is **-1** and the maximum is **39,642,120** with an average of **1,986**. With a standard deviation of **267,610.87**, the values are greatly disparsed. However, since the `measure_value` is based on different `measure`, let look at this description for each categories in the `measure` column using the query below:

```sql
SELECT
  measure,
  MIN(measure_value) AS min_measure_value,
  MAX(measure_value) AS max_measure_value,
  ROUND(
  AVG(measure_value),
  2
  ) AS avg_measure_value,
  ROUND(
  STDDEV(measure_value),
  2) AS stdev_measure_value
FROM health.user_logs
GROUP BY measure;
```
|measure|min_measure_value|max_measure_value|avg_measure_value|stdev_measure_value|
|:----|:----|:----|:----|:----|
|blood_glucose|-1|227228|177.35|1157.32|
|blood_pressure|0|189|95.40|54.89|
|weight|0|39642120|28786.85|1062759.55|

From the table shown above, we have a better description of each of the categories in the `measure` column based on their values in the `measure_value` columns. We can tell that the larger values are from the `weight` category.

## Systolic
```sql
SELECT
  systolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```
|systolic|frequency|
|:----|:----|
| |26023|
|0|15451|
|120|71|
|123|70|
|128|66|
|127|64|
|130|60|
|119|60|
|135|57|
|124|55|

There seems to me a lot of nulls. Let's consider the descriptive statistics as well with the following query

```sql
SELECT
  MIN(systolic) AS min_systolic,
  MAX(systolic) AS max_systolic,
  ROUND(
  AVG(systolic),
  2
  ) AS avg_systolic,
  ROUND(
  STDDEV(systolic),
  2) AS stdev_systolic
FROM health.user_logs;
```
|min_systolic|max_systolic|avg_systolic|stdev_systolic|
|:----|:----|:----|:----|
|-1|204|17.09|43.79|

### Diastolic
```sql
SELECT
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```
|diastolic|frequency|
|:----|:----|
| |26023|
|0|15449|
|80|156|
|79|124|
|81|119|
|78|110|
|77|109|
|73|109|
|83|106|
|76|102|

Noticed how many 0 values there were for the `measure_value` field and `null` values for both `systolic` and `diastolic` columns? This also contain a lot of null values. Let's also look at the descriptive statistics for this column;

```sql
SELECT
  MIN(diastolic) AS min_diastolic,
  MAX(diastolic) AS max_diastolic,
  ROUND(
  AVG(diastolic),
  2
  ) AS avg_diastolic,
  ROUND(
  STDDEV(diastolic),
  2) AS stdev_diastolic
FROM health.user_logs;
```
|min_diastolic|max_diastolic|avg_diastolic|stdev_diastolic|
|:----|:----|:----|:----|
|-1|1914|10.75|30.71|

It will be best to further inspect the rows a bit further to check if this only happens for certain `measure` values when the condition measure_value = 0 is met and the systolic and diastolic columns are null.

```sql
SELECT 
  measure,
  COUNT(*) AS frequency
FROM health.user_logs
WHERE measure_value = 0
GROUP BY measure
ORDER BY frequency DESC;
```
|measure|frequency|
|:----|:----|
|blood_pressure|562|
|blood_glucose|8|
|weight|2|

Let's also compare this with the overall frequency percentage calculated earlier:

|measure|frequency|percentage|
|:----|:----|:----|
|blood_glucose|38692|88.15|
|weight|2782|6.34|
|blood_pressure|2417|5.51|

Even though the `measure= blood_pressure` is just about 5.51% of the total records, it's also the measure with most of the `measure_value = 0`. Let's filter with these two conditions,

```sql
SELECT *
FROM health.user_logs
WHERE measure_value = 0
AND measure = 'blood_pressure';
```
|id|log_date|measure|measure_value|systolic|diastolic|
|:----|:----|:----|:----|:----|:----|
|ee653a96022cc3878e76d196b1667d95beca2db6|2020-03-18T00:00:00.000Z|blood_pressure|0|115|76|
|ee653a96022cc3878e76d196b1667d95beca2db6|2020-03-15T00:00:00.000Z|blood_pressure|0|115|76|
|ee653a96022cc3878e76d196b1667d95beca2db6|2020-02-03T00:00:00.000Z|blood_pressure|0|105|70|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-02-24T00:00:00.000Z|blood_pressure|0|136|87|
|c7af488f4c8efc0ecdfd6d0c427e7c133bf2f2d9|2020-02-06T00:00:00.000Z|blood_pressure|0|164|84|
|c7af488f4c8efc0ecdfd6d0c427e7c133bf2f2d9|2020-02-10T00:00:00.000Z|blood_pressure|0|190|94|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-02-07T00:00:00.000Z|blood_pressure|0|125|79|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-02-19T00:00:00.000Z|blood_pressure|0|136|84|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-02-15T00:00:00.000Z|blood_pressure|0|135|89|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-02-27T00:00:00.000Z|blood_pressure|0|138|85|

It appears that when the blood pressure is measured sometimes the `measure_value` field is empty and the relevant `systolic` and `diastolic` fields are recorded. It is okay to also look at what happened when the blood pressure is measured and the `measure_value` is not 0.

```sql
SELECT *
FROM health.user_logs
WHERE measure_value != 0
AND measure = 'blood_pressure'
LIMIT 10;
```
|id|log_date|measure|measure_value|systolic|diastolic|
|:----|:----|:----|:----|:----|:----|
|d14df0c8c1a5f172476b2a1b1f53cf23c6992027|2020-10-15T00:00:00.000Z|blood_pressure|140|140|113|
|9fef7a7b06dea13eac08b2b609a008d6a178d0b7|2020-10-02T00:00:00.000Z|blood_pressure|114|114|80|
|0b494d455a27f8a2709d7da6c98796ea0e629690|2020-10-19T00:00:00.000Z|blood_pressure|132|132|94|
|ee653a96022cc3878e76d196b1667d95beca2db6|2020-10-09T00:00:00.000Z|blood_pressure|105|105|68|
|46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9|2020-04-12T00:00:00.000Z|blood_pressure|149|149|85|
|46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9|2020-04-10T00:00:00.000Z|blood_pressure|156|156|88|
|46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9|2020-04-29T00:00:00.000Z|blood_pressure|142|142|84|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-04-07T00:00:00.000Z|blood_pressure|131|131|71|
|0f7b13f3f0512e6546b8d2c0d56e564a2408536a|2020-04-08T00:00:00.000Z|blood_pressure|128|128|77|
|abc634a555bbba7d6d6584171fdfa206ebf6c9a0|2020-03-09T00:00:00.000Z|blood_pressure|114|114|76|

It looks like that systolic value is sometimes being used at that measure_value and sometimes the measure_value is 0!

Let’s finish this off by also looking at the null systolic and diastolic values in the same manner:
```sql
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE systolic IS NULL
GROUP BY 1;
```
|measure|count|
|:----|:----|
|weight|443|
|blood_glucose|25580|

This result confirms that systolic only has non-null records when measure = 'blood_pressure'

Can we confirm this is the fact for the diastolic column too?

```sql
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE diastolic IS NULL
GROUP BY 1;
```
|measure|count|
|:----|:----|
|weight|443|
|blood_glucose|25580|

Yes we can therefore confirm this!

## My Feedback to Danny Ma!

Hi Danny,

The dataset has **6** features (columns) and **43,891**  records (rows), there are also **554** unique `id` values which suggest the number of unique users. The records were logged between **11th January, 2015** and **15th November, 2020**. 

Taking a look at the `measure` column in the table, **88%** of the total records are a measure of the `blood_glucose`, other measures includes `weight` and `blood_pressure`. 

I also discovered that of the **554** unique `id` in the dataset about **51%** of the records were from a single id in the `id` column.

Even though `blood_pressure` has the least number of the total records, about **562 (23% of 2,417)** of the total records  for `blood_pressure` measure have their `measure_value` to be equal to 0, a further look into the dataset showed that when the `measure_value=0` the relevant `systolic` and `diastolic` fields are recorded. Also considering the instance when the  `measure_value` is not equal to zero for the `blood_pressure` measure, the `systolic` value is the same as the `measure_value`. 

In conclusion, I found out that the `systolic` and `diastolic` columns contains null value for `weight` and `blood_glucose` measures only. The minimum value for these two columns are -1. Let me know if these findings are helpful.

Best regards,
 
Daniel Ayangbile 


## Dealing with Duplicates

### Detecting Duplicates
The first ingredient for this recipe is the basic record count for our table - plain and simple using the COUNT(*).
```sql
SELECT COUNT(*)
FROM deduplic_user_logs;
```
|count|
|:----|
|43891|

In order to know the number of distinct records in a table, one of these methods can be used;
-----------------------------------------
--COMMON TABLE EXPRESSION (CTE)
-----------------------------------------
```sql
WITH deduplic_user_logs AS (
  SELECT DISTINCT *
  FROM health.user_logs
)
SELECT COUNT(*)
FROM deduplic_user_logs
```
|count|
|:----|
|31004|
----------------------------------------
--SUBQUERY
----------------------------------------
```sql
SELECT COUNT (*)
FROM (SELECT DISTINCT *
FROM health.user_logs) AS dup_count
```
|count|
|:----|
|31004|
---------------------------------------
--TEMPORARY TABLE
---------------------------------------
```sql
DROP TABLE IF EXISTS deduplicated_logs;
CREATE TEMP TABLE deduplicated_logs AS
SELECT DISTINCT *
FROM health.user_logs;

SELECT COUNT(*)
FROM deduplicated_logs;
```

|count|
|:----|
|31004|

We now have the row counts of the original table 43,891 and of our deduplicated table 31,004
It’s pretty safe to say that we have some deuplicate records!
By comparing the counts of the original and the deduplicated, we can prove the presence of duplicates.


#### How do I know which to use?
The answer to this is based on your need for this table later,

If yes you would need it later - opt for temporary tables.

If no - CTEs are your friend.

Usually we would not recommend subqueries as they are less readable than CTEs - making it more difficult for others to quickly understand your code!

## Identifying Duplicate Records
```sql
SELECT
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
ORDER BY frequency DESC
LIMIT 10;
```
|id|log_date|measure|measure_value|systolic|diastolic|frequency|
|:----|:----|:----|:----|:----|:----|:----|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-06T00:00:00.000Z|blood_glucose|401| | |104|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-05T00:00:00.000Z|blood_glucose|401| | |77|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-04T00:00:00.000Z|blood_glucose|401| | |72|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-07T00:00:00.000Z|blood_glucose|401| | |70|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-09-30T00:00:00.000Z|blood_glucose|401| | |39|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-09-29T00:00:00.000Z|blood_glucose|401| | |24|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-10-02T00:00:00.000Z|blood_glucose|401| | |18|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-10T00:00:00.000Z|blood_glucose|140| | |12|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-11T00:00:00.000Z|blood_glucose|220| | |12|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-04-15T00:00:00.000Z|blood_glucose|236| | |12|
... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ... ...
|id|log_date|measure|measure_value|systolic|diastolic|frequency|
|:----|:----|:----|:----|:----|:----|:----|
|abc634a555bbba7d6d6584171fdfa206ebf6c9a0|2020-05-17T00:00:00.000Z|blood_pressure|105|105|78|1|
|c7af488f4c8efc0ecdfd6d0c427e7c133bf2f2d9|2020-01-22T00:00:00.000Z|blood_glucose|172| | |1|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-05-07T00:00:00.000Z|blood_glucose|189| | |1|
|576fdb528e5004f733912fae3020e7d322dbc31a|2019-12-15T00:00:00.000Z|blood_pressure|0|124|72|1|
|46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9|2019-12-24T00:00:00.000Z|blood_pressure|0|183|161|1|
|8b130a2836a80239b4d1e3677302709cea70a911|2019-12-31T00:00:00.000Z|blood_glucose|109.799995| | |1|
|d41d5b2fa4d9a2caf3a3e839432da76450bc2041|2019-11-11T00:00:00.000Z|blood_glucose|176|0|0|1|
|6c2f9a8372dac248192c50219c97f9087ab778ba|2019-12-30T00:00:00.000Z|blood_glucose|207|0|0|1|
|ef84a9ac5319687e5fba6174cd1db3d7f1cfca8f|2020-08-20T00:00:00.000Z|blood_glucose|206|0|0|1|
|f3596442e990f21f929015a1c2d3514706e5d73d|2020-06-25T00:00:00.000Z|blood_glucose|244|0|0|1|

The table above show the first and last 10 records counts for each records in the dataset. Notice how the frequency for some of these values is 1 - whilst some are greater than 1? Let;s try to remove the duplicate records only

```sql
DROP TABLE IF EXISTS unique_duplicate_records;
CREATE TEMPORARY TABLE unique_duplicate_records AS
SELECT *
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1;

SELECT *
FROM unique_duplicate_records
LIMIT 10;
```
Let’s say for example - we want to know which exact records are duplicated - but also how many times they appeared. We can use the CTE approach with a WHERE filter condition to do this.
```sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT *
FROM groupby_counts
WHERE frequency > 1
ORDER BY frequency DESC
LIMIT 10;
```
|id|log_date|measure|measure_value|systolic|diastolic|frequency|
|:----|:----|:----|:----|:----|:----|:----|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-06T00:00:00.000Z|blood_glucose|401| | |104|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-05T00:00:00.000Z|blood_glucose|401| | |77|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-04T00:00:00.000Z|blood_glucose|401| | |72|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-07T00:00:00.000Z|blood_glucose|401| | |70|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-09-30T00:00:00.000Z|blood_glucose|401| | |39|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-09-29T00:00:00.000Z|blood_glucose|401| | |24|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-10-02T00:00:00.000Z|blood_glucose|401| | |18|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-10T00:00:00.000Z|blood_glucose|140| | |12|
|054250c692e07a9fa9e62e345231df4b54ff435d|2019-12-11T00:00:00.000Z|blood_glucose|220| | |12|
|054250c692e07a9fa9e62e345231df4b54ff435d|2020-04-15T00:00:00.000Z|blood_glucose|236| | |12|

## Do we actually need to care about duplicate values?

The context of your dataset will determine if you should care about duplicates or not.

> For the context of this dataset, this real world messy dataset captures data taken from individuals logging their health measurements via an online portal throughout the day.

For example, multiple measurements can be taken on the same day at different times, but you may notice this information is missing as the log_date column does not show timestamp values!.

A further analysis will the to answer the following questions,
1. Which id value has the most number of duplicate records in the health.user_logs table?
2. Which log_date value had the most duplicate records after removing the max duplicate id value from question 1?
3. Which measure_value had the most occurences in the health.user_logs value when measure = 'weight'?
4. How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table?
5. What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition?
6. What percentage of records are duplicates in the health.user_logs table?
