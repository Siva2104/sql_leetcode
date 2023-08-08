---
title: "Intermediate SQL course"
author: "Shiva"
date: "AUG-08-2023"
output:
  html_document:
    code_folding: hide
    toc: true
    number_sections: true
    toc_float:
      collapsed: true
      smooth_scroll: true
  theme: slate
  highlight: kate
---

```{r global_options, include=FALSE}
library(knitr)
knitr::opts_chunk$set(fig.width = 8, fig.height = 7, fig.path = 'Figs/', warning = FALSE, message = FALSE)
```

# Welcome

![](images/sql.svg){width=300px}



## Prerequisites

To successfully complete this tutorial you should have a basic understanding of SQL and R.

## Objectives

By finishing this tutorial, students will be able to:

1. Group and summarize the data together.
2. Combine the grouping mechanisms with aggregation functions.
3. Write queries that retrieve information by combining data from different tables.
4. Combine the results of multiple queries in various ways.

## Introduction and Refresher

**SQL(Structured Query Language)** is a database query language - a language specifically designed to interact with relational databases. It is possible to extract, update, filter, replace or insert data using SQL. Some common relational database management software which uses SQL to access the data are MySQL, SQLite, Microsoft SQL Server, PostgreSQL, Oracle, Sybase. 

### Load the library and database  

We are going to use the RSQLite package in the following tutorial and the Amazon stock price [dataset](data/AMZN.csv).  

```{r loadStockData, results='hide'}
library(DBI)
AMZN <- read.table("data/AMZN.csv", stringsAsFactors = FALSE, header = TRUE, sep = ",")
# Create a database connection
stock <- dbConnect(RSQLite::SQLite(), "stock.sqlite")
# Write the data in the table
dbWriteTable(stock, "amazon",AMZN, overwrite = TRUE)
```

#### SELECT Statement

`SELECT` statement is the core of the SQL language and is used to query the database to retrieve the selected rows that match the user condition. The `SELECT` statement has five main clauses, although `FROM ` is the only required clause.

**Syntax:**
```sql
SELECT [ALL | DISTINCT] column(s) FROM table [WHERE "condition(s)"] [GROUP BY column(s)] [HAVING "condition(s)"] [ORDER BY column(s) [ASC| DESC]] [LIMIT number]
```

**Clauses table**

| Name    | Purpose
|---------|:------------------------------------------------------
|SELECT   | Columns to include in the query result set
|FROM     | Identifies tables from which data has to be drawn
|WHERE    | Filters unwanted data
|GROUP BY | Groups rows based on a criteria
|HAVING   | Filters unwanted groups
|ORDER BY | Sorts the rows of the result set in increasing or decreasing order

**Example 1:** Look at all the columns for the first 5 rows of the dataset.

```{r look_1}
kable(dbGetQuery(stock, 'SELECT * FROM amazon LIMIT 5'))
```

**ALL** and **DISTINCT** are keywords used to select ALL (by default) or unique rows in the query by discarding the duplicate entries. 

**Example 2:** What are the first five DISTINCT years in the dataset?

```{r look_2}
dbGetQuery(stock, 'SELECT DISTINCT year FROM amazon LIMIT 5')
```

#### Aggregate functions  
SQL is excellent for aggregating data. Following are the **aggregate functions students should be comfortable with** before moving forward.  

`COUNT()` - It counts the total number of non-null rows in a column. It is possible to add *aliases* using `AS`. For example:

**Example 3:** Count the total number of rows in the dataset.

```{r count_1}
dbGetQuery(stock, 'SELECT COUNT(*) AS "total_rows" FROM amazon')
```

`SUM()` - It adds all the values of a given column. It treats nulls as 0. 

**Example 4:** Find the total volume traded for Amazon stock in the dataset.

```{r sum_1}
dbGetQuery(stock, 'SELECT SUM(Volume) AS "total_vol" FROM amazon')
```

`MIN()/MAX()` - It returns the minimum and the maximum numerical or non-numerical values in a column.  

**Example 5:** Return the first and last date Amazon stock was traded according to the dataset.

```{r min_max_1}
dbGetQuery(stock, 'SELECT MIN(Date) AS min_date, MAX(Date) AS max_date FROM amazon')
```

`AVG()` - It returns the average of the values from a selected group. It can only be used for numerical columns and ignores null values.

**Example 6:** Return the average open price for the stock.

```{r avg_1}
dbGetQuery(stock, 'SELECT AVG(Open) AS avg_open FROM amazon')
```

#### GROUP BY and ORDER BY clause  

Information retrieved from a database using SQL can be placed into separate categories using `GROUP BY` clause, which can then be aggregated independent of each other. The query proceeds in two stages. First, the rows are grouped according to grouping criteria. Then, the aggregate operations are performed on the rows of each category. The `GROUP BY` clause must precede the data items to be used for grouping. 

`ORDER BY` clause is used to sort the results of the query either in ascending (ASC) or descending order (DESC).  

##### `GROUP BY` single column

**Example 7:** Count number of days traded each year.

```{r count_2}
kable(dbGetQuery(stock, 'SELECT year, COUNT(*) AS days_traded FROM amazon GROUP BY year LIMIT 5'))
```

##### `GROUP BY` multiple columns

**Example 8:** Count number of days traded in months of each year.

```{r count_3}
kable(dbGetQuery(stock, 'SELECT year, month, COUNT(*) AS days_traded FROM amazon GROUP BY year, month LIMIT 5'))
```
  
##### `GROUP BY` column numbers  

We can substitute column names with column numbers in the `GROUP BY` clause. 

**Example 9:** `GROUP BY` Column 1 and 2

```{r count_4}
kable(dbGetQuery(stock, 'SELECT year, month, COUNT(*) AS days_traded FROM amazon GROUP BY 1, 2 LIMIT 5'))
```

##### `ORDER BY` column names or numbers

**Example 10:** Sort the results of the previous query in descending order of the year.

```{r count_5}
kable(dbGetQuery(stock, 'SELECT year, month, COUNT(*) AS days_traded FROM amazon GROUP BY 1, 2 ORDER BY year DESC LIMIT 5'))
```

#### HAVING Clause

`HAVING` - Similar to `WHERE` but it allows us to filter on aggregate columns. The `HAVING` clause always follows `GROUP BY` clause.
**Note:** The properties that are tested in the `GROUP BY` clause must be the properties of the groups, not individuals.

**Example 11:**

Find the average open price of Amazon for each month of each year in the dataset. 

```{r having_1}
kable(dbGetQuery(stock,
'SELECT year,month,avg(open) AS avg_open_price FROM amazon GROUP BY year,month LIMIT 5'))
```

**Example 12:**

Find every month of each year where Amazon stock price was above $1500/share.

```{r having}
kable(dbGetQuery(stock, 'SELECT year, month, MAX(high) AS month_high FROM amazon GROUP BY year, month HAVING month_high > 1500 ORDER BY year, month'))
```

### Exercise 1

**1. Show all the columns from table amazon where year = 2014 or date > 2016-01-01 and sort the results in such a manner that the same days are displayed together despite different years or months. For example, data from all days with day 31 should appear together.**

_Sample Output_   

```{r ex1}
kable(dbGetQuery(stock,'SELECT * FROM amazon WHERE (year = 2014 OR date > "2016-01-01") ORDER BY day LIMIT 5'))
```

**2. Select all rows between the dates 2007-01-01 and 2009-01-01.**

_Sample output_  
```{r ex2}
kable(dbGetQuery(stock,'SELECT * FROM amazon WHERE date BETWEEN "2007-01-01" AND "2009-01-01" LIMIT 5'))
```

**3. Select all rows where the year is either 2007,2008 or 2018.**

_Sample output_  
```{r ex3}
kable(dbGetQuery(stock, 'SELECT * FROM amazon WHERE year IN (2007,2008,2018) LIMIT 5'))
```

**4. SELECT all days when Amazon stock was traded after 10th (including) for the month of January for all years after 2000 (including). **

_Sample output_  
```{r ex4}
kable(dbGetQuery(stock,'SELECT * FROM amazon WHERE date LIKE "2%-01-1_" LIMIT 5'))
```

*Do it after you have done JOINS.*

**5. Select all the days when the open price was above the average open price for that month. For example, if the average open price for Jan 1999 was 2.5 USD then select all the days for Jan 1999 where the open price was above 2.5 USD.** 

_Sample output_  
```{r ex5}
kable(dbGetQuery(stock,'SELECT a.year,a.month,a.day,a.open,b.avg_open FROM amazon AS a INNER JOIN (SELECT year,month,avg(open) AS avg_open FROM amazon GROUP BY year,month) AS b ON a.year = b.year AND a.month = b.month AND a.open > b.avg_open LIMIT 5'))
```

**6. Count the number of days now.**

_Sample output_  
```{r ex6}
kable(dbGetQuery(stock,'SELECT a.year,a.month,COUNT(day) AS num_days, b.avg_open FROM amazon AS a INNER JOIN (SELECT year,month,avg(open) AS avg_open FROM amazon GROUP BY year,month) AS b ON a.year = b.year AND a.month = b.month AND a.open > b.avg_open GROUP BY a.year, a.month LIMIT 5'))
```

**Disconnect from the stock database**
```{r disconnect_1, results='hide'}
dbDisconnect(stock)
```

## Intermediate SQL

-------------------------------------------------------------------

For the next few lessons you are going to use three datasets _viz._ [drugs names](data/drugs/drugs_names), [drugs data](data/drugs/drugs_data) and [drugs target](data/drugs/drugs_target). These datasets were collected from [DRUGBANK](https://www.drugbank.ca/) website.

-------------------------------------------------------------------

### Load the data and create the database connection

```{r loadDrugsData, results='hide'}
drugs_data <- read.table("data/drugs/drugs_data", sep = "\t",header = T, stringsAsFactors = F)

drugs_target <- read.table("data/drugs/drugs_target", sep = ",",header = T, stringsAsFactors = F, quote = "\"")

drugs_names <- read.table("data/drugs/drugs_names", sep = ",",header = T, stringsAsFactors = F, quote = "\"")


# Create a database connection
drugs <- dbConnect(RSQLite::SQLite(), "drugs.sqlite")
# Write the data in the table
dbWriteTable(drugs, "drugs_data",drugs_data, overwrite = TRUE)
dbWriteTable(drugs, "drugs_target",drugs_target, overwrite = TRUE)
dbWriteTable(drugs, "drugs_names",drugs_names, overwrite = TRUE)
```

**Example:**

Display the records in each table

```{r showDrugsData}
kable(dbGetQuery(drugs, 'SELECT * FROM drugs_names LIMIT 5'))
kable(dbGetQuery(drugs, 'SELECT * FROM drugs_data LIMIT 5'))
kable(dbGetQuery(drugs, 'SELECT * FROM drugs_target LIMIT 5'))
```

All the displayed tables are self-explanatory. However, a brief description is given below:

1. The **drugs_names** table is the first table in the _drugs database_. It contains drugbank id, the common name of the drug and PubChem compound id (cid).

2. The **drugs_data** table is the second table in the database. It contains the PubChem compound id (cid) and various molecular properties of drugs like molecular weight, number of rotatable bonds, the total charge on the drug etc.

3. The **durgs_target** table is the third table. It contains drugbank id, name, type of drug and the name of the protein or receptor the drug binds to.

### Subqueries 

A subquery (also called as an inner or nested query) is a SQL query that is inside a larger query and allows the user to perform operations in multiple steps. The inner query is executed first and the results are passed on to the outer query. Subqueries can occur at several places inside a query. For example:

1. In `SELECT` clause
2. In `FROM` clause
3. In `WHERE` clause

* Subquery with `WHERE` clause 

**Syntax:**

```sql
SELECT table1.column_name(s) FROM table1 WHERE search_condition ( Subquery);
```

**Example 1:**

Select drugbank id, names of drugs with molecular weight less than equal to 500 from *drugs_names* and *drugs_data* tables.

```{r sub_1}
kable(dbGetQuery(drugs,"SELECT dn.drugBankID, dn.name FROM drugs_names AS dn WHERE dn.cid IN ( SELECT dd.cid FROM drugs_data AS dd WHERE dd.molecular_weight <= 500) LIMIT 5"))
```

**Example 2a:**

Select the count of drugs that bind to a protein target and sort the results by drugs count (no subquery required). 

```{r sub_2_1}
kable(dbGetQuery(drugs,
"SELECT  uniprotID, proteinName, COUNT(*) as drugs_count FROM drugs_target GROUP BY uniprotID, proteinName ORDER BY drugs_count DESC LIMIT 5"))
```

**Example 2b:**

Select drugbank id, protein id and protein name from *drugs_target* table where a protein target binds to more than 40 drugs. (subquery required).

```{r sub_2_2}
kable(dbGetQuery(drugs,"SELECT drugBankID,uniProtID,proteinName FROM drugs_target WHERE uniprotID IN(SELECT uniProtID FROM drugs_target GROUP BY uniprotID HAVING COUNT(uniprotID) >40) ORDER BY drugBankID LIMIT 5"))
```

**Note:**There are many instances where a protein target binds to more than one drug (like Gamma-aminobutyric acid receptor subunit alpha-1 binds 89 drugs) however, there are also drugs which bind to many protein targets. For example, **DB00154** shown in the table above binds to two different protein targets. 

**Example 3a:**

Let us now count the number of protein targets each drug binds to and sort the results by protein count (no subquery required).  

```{r sub_3_1}
kable(dbGetQuery(drugs,"SELECT  drugBankID, COUNT(*) AS protein_count FROM drugs_target GROUP BY drugBankID ORDER BY protein_count DESC LIMIT 5"))
```

There are five drugs **DB03147**, **DB00898**, **DB00334**, **DB00543** and **DB01049** which binds to more than 40 protein targets. Pretty amazing..!!
These drugs not only bind there proposed protein target but also others. This may explain why certain drugs have side effects.

**Example 3b:**

Now let us find all the protein targets these drugs bind to.

```{r sub_3_2}
kable(dbGetQuery(drugs,"SELECT drugBankID,uniProtID,proteinName FROM drugs_target WHERE drugBankID IN (SELECT drugBankID FROM drugs_target GROUP BY drugBankID HAVING COUNT(drugBankID) > 40) LIMIT 5"))
```

* Subquery with `FROM` clause

**Syntax:**

```sql
SELECT sub.column_name(s) FROM (Subquery);
```

**Example 4:**

The results from query 2a can also be obtained with the combination of subquery and an `INNER JOIN`.

```{r sub_4_1}
kable(dbGetQuery(drugs,
"SELECT DISTINCT dt.uniProtID, dt.proteinName,sub.drugs_count FROM drugs_target AS dt INNER JOIN(SELECT uniProtID, COUNT(drugBankID) AS drugs_count FROM drugs_target GROUP BY uniProtID) AS sub USING(uniProtID) ORDER BY sub.drugs_count DESC LIMIT 5"))
```

**Example 5:**

Select UniProt id, and count of drugs that bind to various *opioid receptors* (the type of proteins) from *drugs_target* table.

```{r sub_5_1}
kable(dbGetQuery(drugs,"SELECT A.uniProtID,A.proteinName, COUNT(A.drugBankID) AS drug_count FROM (SELECT * FROM drugs_target WHERE proteinName LIKE '%opioid%') AS A GROUP BY uniProtID,proteinName"))
```

### Exercise 2

**1. Find all drugs (drugbank id, UniProt id and protein name) which bind to only one protein target.**

_Sample Output_  

```{r ex2_1}
kable(dbGetQuery(drugs,"SELECT drugBankID, uniProtID, proteinName FROM drugs_target WHERE drugBankID IN (SELECT drugBankID FROM drugs_target GROUP BY drugBankID HAVING COUNT(uniProtID) = 1) LIMIT 5"))
```

**2. Find average molecular weight, hydrogen bond acceptors, hydrogen bond donors and rotatable bonds for drugs that bind _Kinase proteins_.**

_Sample Output_ 

```{r ex2_2}
kable(dbGetQuery(drugs,"SELECT AVG(molecular_weight), AVG(hbond_acceptors), AVG(hbond_donors), AVG(rotatable_bond) FROM drugs_data WHERE cid IN(SELECT cid FROM drugs_names WHERE drugBankID IN (SELECT DISTINCT drugBankID FROM drugs_target WHERE proteinName LIKE '%kinase%'))"))
```

## JOINS

The real power of SQL comes from fetching information by combining multiple tables. In fact, the term "relational" in *R*elational *D*atabase refers to the fact that tables within it are related to one another and contain a common key through which multiple tables can be combined on the fly. **The JOIN is a core concept in the Relational Databases and hence SQL.** The ability to relate information from different tables is an essential skill for a data scientist.     

To list data from two or more tables following is required:  
1. FROM clause should list all the tables. Order of listing does not matter however it may make a difference in the speed of execution. 
2. JOIN operator should be used to coordinate the records in one table with the records in another table. Most common types of joins are:

* INNER JOIN
* LEFT JOIN
* RIGHT JOIN
* FULL JOIN
* CROSS JOIN
* SELF JOIN

### INNER JOIN
In the animation below you can see that the **ID** field matches for 1 and 4 (shaded rows). INNER JOIN selects all the matching rows from both the tables. Mismatched rows are not shown.  

![INNER_JOIN](images/inner.gif)

**Syntax:**

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1 INNER JOIN table2 ON table1.column_name = table2.column_name  
```
**Example 1:**

Select drugbank id, drug name and drug properties using *drugs_name* and *drugs_data* table. 

```{r inner_1}
kable(dbGetQuery(drugs, "SELECT drugs_names.drugBankID, drugs_names.name, drugs_data.molecular_formula, drugs_data.molecular_weight, drugs_data.hbond_acceptors, drugs_data.hbond_donors FROM drugs_names INNER JOIN drugs_data ON drugs_names.cid = drugs_data.cid LIMIT 5"))
```

**Alias:**

Instead of writing a full table name, it is possible to use alias as a shortcut for example, the above query can also be run in the following manner: 

```{r inner_2}
kable(dbGetQuery(drugs, "SELECT dn.drugBankID, dn.name, dd.molecular_formula, dd.molecular_weight, dd.hbond_acceptors, dd.hbond_donors FROM drugs_names AS dn INNER JOIN drugs_data AS dd ON dn.cid = dd.cid LIMIT 5"))
```

**JOIN multiple tables**

**Example 2:**

Select drugbank id, drug name, molecular formula and the molecular weight of the drug and the protein name to which drug binds using *drugs_names*, *drugs_data* and *drugs_target* table. 

```{r inner_3}
kable(dbGetQuery(drugs, "SELECT dn.drugBankID, dn.name, dd.molecular_formula, dd.molecular_weight, dt.proteinName FROM drugs_names AS dn INNER JOIN drugs_data AS dd ON dn.cid = dd.cid INNER JOIN drugs_target AS dt ON dn.drugBankID = dt.drugBankID LIMIT 10"))
```

**USING Keyword**

If the key name is the same in the two tables being joined we can also use **USING** keyword instead of **ON** keyword. For example:

**Example 3:**

```{r inner_4}
kable(dbGetQuery(drugs, "SELECT dn.drugBankID, dn.name, dd.molecular_formula, dd.molecular_weight, dd.hbond_acceptors, dd.hbond_donors FROM drugs_names AS dn INNER JOIN drugs_data AS dd USING(cid) LIMIT 5"))
```

### LEFT JOIN
LEFT JOIN **keeps all the original rows of the left table** in addition to the matching rows from both tables. The values that do not match in the right table are marked as missing. In the animation below there are no values for ID 3 and 6 in the right table and therefore, are missing in the resulting LEFT JOIN table (grey shaded area). 

![LEFT_JOIN](images/left.gif)

**Syntax:**

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1 LEFT JOIN table2 ON table1.column_name = table2.column_name  
```

**Example 4:**

Select Drugbank ID, Protein name from *drugs_target* (left_table), Drug name and Drug cid from *drugs_names* (right_table).  

```{r left_1}
kable(dbGetQuery(drugs, "SELECT dt.drugBankID, dt.proteinName,dn.name,dn.cid FROM drugs_target AS dt LEFT JOIN drugs_names AS dn ON dt.drugBankID = dn.drugBankID ORDER BY dt.drugBankID LIMIT 5"))
```

**Note:** that the DrugBank ID starts from DB00115 in *drugs_names* table (table on the right in above query) and hence the corresponding values are NULL in the resulting table. The  `USING` keyword can also be used in a left join similar to an inner join.

### RIGHT JOIN

`RIGHT JOIN` **keeps all the original rows of the right table** in addition to the matching rows from both the tables. The values that do not match in the left table are marked as missing. In the animation below there are no values for ID 2 and 5 in the left table and therefore, are missing in the resulting RIGHT JOIN table (grey shaded area). **Note:** `RIGHT JOIN` is equivalent to `LEFT JOIN` with tables reversed.  

![RIGHT_JOIN](images/right.gif)

**Syntax:**

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1 RIGHT JOIN table2 ON table1.column_name = table2.column_name  
```
In SQLite there is no `RIGHT JOIN`. However, we can emulate `RIGHT JOIN` as shown in the syntax below:

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table2 LEFT JOIN table1 ON table1.column_name = table2.column_name  
```

**Example 5:**

Select Drugbank ID, Protein name from *drugs_target* (left_table), Drug name and Drug cid from *drugs_names* (right_table).  

```{r rigth_1}
kable(dbGetQuery(drugs, "SELECT dt.drugBankID, dt.proteinName,dn.name,dn.cid FROM drugs_names AS dn  LEFT JOIN drugs_target AS dt  ON dt.drugBankID = dn.drugBankID ORDER BY dt.drugBankID LIMIT 5"))
```

**Note:** This time the drugbank id and protein names columns have `NULL` values because there are 114 drugs in the left_table (drugs_names table) which do not have any corresponding information in the right table (drugs_target table). It means there is no information available on what and how many kinds of proteins these drugs bind.

### FULL JOIN

The `FULL JOIN` returns all the records with or without a match in left and right table. The missing values will be filled as NULL on either side. It can potentially return a very large dataset.

![FULL_JOIN](images/full.gif)

**Syntax:**

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1 FULL JOIN table2 ON table1.column_name = table2.column_name.
```
In SQLite and MySQL there is no `FULL JOIN`. However, we emulate `FULL JOIN` using the syntax below:

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1 LEFT JOIN table2 ON table1.column_name = table2.column_name 
UNION ALL 
SELECT table1.column_name(s), table2.column_name(s) FROM table1 RIGHT JOIN table2 ON table1.column_name = table2.column_name.
```

**Example 6:**

Select Drugbank ID, Protein name from *drugs_target* (left_table), Drug name and Drug cid from *drugs_names* (right_table).  

```{r full_1}
kable(dbGetQuery(drugs, "SELECT dt.drugBankID as id_from_drugs_target, dt.proteinName, dn.drugBankID AS id_from_drugs_names, dn.name FROM drugs_names AS dn LEFT JOIN drugs_target AS dt ON dt.drugBankID = dn.drugBankID 
  UNION ALL 
  SELECT dt.drugBankID, dt.proteinName, dn.drugBankID,dn.name FROM drugs_target AS dt LEFT JOIN drugs_names AS dn ON dt.drugBankID = dn.drugBankID 
  ORDER BY id_from_drugs_names,id_from_drugs_target LIMIT 5"))
```

### CROSS JOIN

The `CROSS JOIN` or Cartesian join results in all possible combinations of rows from the joined tables. Suppose table1 has `M` rows and table2 has `N` rows The `CROSS JOIN` would produce a result set of `M x N` rows, therefore, `CROSS JOIN` should be used with caution.

![CROSS_JOIN](images/cross.gif)

**Syntax:**

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1 CROSS JOIN table2
```

OR

```sql
SELECT table1.column_name(s), table2.column_name(s) FROM table1, table2
```

The `CROSS JOIN` query does not make sense in this dataset however, it was run for demonstration purpose:

**Example 7:**

Select Drugbank ID, Drug name, Protein name from *drugs_target* and *drugs_names* table.  

```{r cross_1}
kable(dbGetQuery(drugs,"SELECT dt.drugBankID, dn.name, dt.proteinName FROM drugs_names AS dn CROSS JOIN drugs_target AS dt LIMIT 5"))
```

### SELF JOIN

In a `SELF JOIN` a table is joined to itself. 

**Syntax:**

```sql
SELECT t1.column_name(s), t2.column_name(s) FROM table1 AS t1, table2 AS t2 WHERE t1.column = t2.column
```

**Example 8:**

Select drugbank ids from *drugs_target* table which bind to both the proteins with id 'P28223' and 'P46098'. 

The following query will not work because it is not possible for the value of a single column in a single row to contain two values at the same time. 

```{r self_1}
kable(dbGetQuery(drugs,"SELECT dt.drugBankID FROM drugs_target AS dt WHERE dt.uniProtID = 'P28223' AND dt.uniProtID = 'P46098'"))
```

This query will require a `SELF JOIN`. The correct query is as follows:

```{r self_2}
kable(dbGetQuery(drugs,
"SELECT DISTINCT dt1.drugBankID FROM drugs_target dt1, drugs_target dt2 WHERE dt1.drugBankID = dt2.drugBankID AND dt1.uniProtID = 'P28223' AND dt2.uniprotID = 'P46098'"))
```

### Exercise 3

**1. Select cid, drugBankID, uniProtID and count the number of drugs that bind to the protein target from *drugs_data*, *drugs_names* and *drugs_target* tables.**

_Sample Output_ 

```{r ex3_1 }
kable(dbGetQuery(drugs,"SELECT cid,B.drugBankID,B.uniProtID, B.count FROM (SELECT drugBankID, A.uniProtID, A.count FROM drugs_target INNER JOIN (SELECT uniProtID, COUNT(uniProtID) AS count FROM drugs_target GROUP BY uniProtID) AS A USING(uniProtID) ORDER BY uniProtID) AS B LEFT JOIN drugs_names USING(drugBankID) ORDER BY B.uniProtID LIMIT 5"))
```

**2. Find those proteins and drugs which have only 1 binding partner and drug type in not _BiotechDrug_. For example, a drug should bind to a single protein target and a protein target should bind to a single drug.**

_Sample Output_

```{r ex3_2}
kable(dbGetQuery(drugs,"SELECT drugBankID, name, sub.uniProtID, sub.proteinName FROM (SELECT drugBankID, uniProtID, proteinName, type FROM drugs_target WHERE uniProtID IN (SELECT uniProtID FROM drugs_target WHERE uniProtID IN (SELECT uniProtID FROM drugs_target WHERE drugBankID IN (SELECT drugBankID FROM drugs_target GROUP BY drugBankID HAVING COUNT(uniprotID) = 1)) GROUP BY uniProtID HAVING COUNT(drugBankID) = 1) AND type !='BiotechDrug') AS sub LEFT JOIN drugs_names USING(drugBankID) LIMIT 5"))
```

## Set theory clauses

### UNION

The `UNION` keyword allows us to combine the related information from two tables. The `UNION` will count the values only once *i.e.* will eliminate the duplicate rows. **Note:** Column numbers and data types must be same for the columns to combine.

![](images/union.jpg){width=300px}

**Syntax:**

```sql
SELECT table1.column_name(s) FROM table1 
UNION 
SELECT table2.column_name(s) FROM table2
```

**Example 1:**

Let us count the total Drugbank IDs from *drugs_target* and *drugs_names* tables individually and then using a `UNION`.

Count `DISTINCT` drugBankIDs in *drugs_names* and *drugs_target* table individually.

```{r union_1}
kable(dbGetQuery(drugs,"SELECT COUNT(DISTINCT(drugs_target.drugBankID)) AS drugs_target_ids, COUNT(DISTINCT(drugs_names.drugBankID)) AS drugs_names_ids FROM drugs_target,drugs_names"))
```

**Example 2:**

Count `DISTINCT` drugBankIDs in combined tables using `UNION`.

```{r union_2}
kable(dbGetQuery(drugs,
"SELECT COUNT(drugBankID) AS union_drugbank_ids FROM (SELECT dn.drugBankID FROM drugs_names AS dn UNION  SELECT dt.drugBankID FROM drugs_target AS dt) AS sub"))
```

### UNION ALL

Like the `UNION` the `UNION ALL` keyword allows us to combine the related information from two tables however, the `UNION ALL` will return all the rows from the `SELECT` statement *i.e.* will not eliminate the duplicate rows and combine all rows from each table into a single table. **Note:** Column numbers and data types must be same for the columns to combine.

![](images/union_all.jpg){width=300px}

**Syntax:**

```sql
SELECT table1.column_name(s) FROM table1 
UNION ALL
SELECT table2.column_name(s) FROM table2
```

**Example 3:**

Let us count the total Drugbank IDs from *drugs_target* and *drugs_names* tables using `UNION ALL` keyword.

```{r union_all_1}
kable(dbGetQuery(drugs,
"SELECT COUNT(drugBankID) AS union_all_drugbank_ids FROM (SELECT dn.drugBankID FROM drugs_names AS dn UNION ALL SELECT dt.drugBankID FROM drugs_target AS dt) AS sub"))
```

### INTERSECT

The `INTERSECT` keyword produces rows that are common between the two tables. 

![](images/intersect.jpg){width=300px}


**Syntax:**

```sql
SELECT table1.column_name(s) FROM table1 
INTERSECT
SELECT table2.column_name(s) FROM table2
```

**Example 4:**

Let us count Drugbank IDs common in *drugs_target* and *drugs_names* tables using `INTERSECT` keyword.

```{r intersect_1}
kable(dbGetQuery(drugs,
"SELECT COUNT(drugBankID) AS intersect_drugbank_ids FROM (SELECT dn.drugBankID FROM drugs_names AS dn INTERSECT SELECT dt.drugBankID FROM drugs_target AS dt) AS sub"))
```

In MySQL `INTERSECT` keyword is not available, however, the effect can be simulated through `IN` operator.  

```{r intersect_2}
kable(dbGetQuery(drugs,
"SELECT COUNT(drugBankID) AS in_drugbank_ids FROM (SELECT dn.drugBankID FROM drugs_names AS dn WHERE dn.drugBankID IN (SELECT dt.drugBankID FROM drugs_target AS dt)) AS sub"))
```

### EXCEPT 

The `EXCEPT` keyword is used to return all the rows in the first table which are not in the second table. In Oracle `MINUS` is used instead of `EXCEPT`. 

![](images/except.jpg){width=300px}


**Syntax:**

```sql
SELECT table1.column_name(s) FROM table1 
EXCEPT
SELECT table2.column_name(s) FROM table2
```

**Example 5:**

Let us count Drugbank IDs in *drugs_names* table which are not present in *drugs_target* table using `EXCEPT` keyword.

```{r except_1}
kable(dbGetQuery(drugs,"SELECT COUNT(drugBankID) AS except_id FROM (SELECT dn.drugBankID FROM drugs_names AS dn EXCEPT SELECT dt.drugBankID FROM drugs_target AS dt) AS sub"))
```

MySQL does not support `EXCEPT` or `MINUS` keyword. However, the same result can be achieved using the `NOT IN` operator.  

```{r except_2}
kable(dbGetQuery(drugs,
"SELECT COUNT(drugBankID) AS not_in_id FROM (SELECT dn.drugBankID FROM drugs_names AS dn WHERE dn.drugBankID NOT IN (SELECT dt.drugBankID FROM drugs_target AS dt)) AS sub"))
```

### Exercise 4

**1. Show all the drugbank ids in drugs_names and drugs_target table.**

_Sample Output_   

```{r ex4_1}
kable(dbGetQuery(drugs,'SELECT dn.drugBankID FROM drugs_names AS dn UNION ALL SELECT dt.drugBankID FROM drugs_target AS dt LIMIT 5'))
```

**2. Select drugbank id, cid, name and molecular weight of drugs that are present in drugs_names table, however, missing from the drugs_target table. Do an `INNER JOIN` of the resulting table with the drugs_data table.** 

_Sample Output_

```{r ex4_2}
kable(dbGetQuery(drugs,"SELECT sub.drugBankID, sub.cid, sub.name,molecular_weight FROM drugs_data INNER JOIN (SELECT dn.drugBankID,dn.cid,dn.name FROM drugs_names AS dn WHERE dn.drugBankID NOT IN (SELECT dt.drugBankID FROM drugs_target AS dt)) AS sub USING(cid) LIMIT 5"))
```


**3. Select drugbank id, cid, name and molecular weight of drugs that are present in both drugs_names and drugs_target table.**

_Sample Output_

```{r ex4_3}
kable(dbGetQuery(drugs,
"SELECT sub.drugBankID, sub.cid, sub.name,molecular_weight FROM (SELECT dn.drugBankID,dn.cid,dn.name FROM drugs_names AS dn WHERE dn.drugBankID IN (SELECT dt.drugBankID FROM drugs_target AS dt)) AS sub LEFT JOIN drugs_data USING(cid) LIMIT 5"))
```

**4. Devise a query that will generate a sequence of numbers from 0 .. 99**

_Hint:_ Use `UNION ALL` and `CROSS JOIN` 

_Sample Output_

```{r ex4_4}
kable(dbGetQuery(drugs,"SELECT ones.num + tens.num AS seq FROM (SELECT 0 num UNION ALL SELECT 1 num UNION ALL SELECT 2 num UNION ALL SELECT 3 num UNION ALL SELECT 4 num UNION ALL SELECT 5 num UNION ALL SELECT 6 num UNION ALL SELECT 7 num UNION ALL SELECT 8 num UNION ALL SELECT 9 num) ones CROSS JOIN (SELECT 0 num UNION ALL SELECT 10 num UNION ALL SELECT 20 num UNION ALL SELECT 30 num UNION ALL SELECT 40 num UNION ALL SELECT 50 num UNION ALL SELECT 60 num UNION ALL SELECT 70 num UNION ALL SELECT 80 num UNION ALL SELECT 90 num) tens ORDER BY seq LIMIT 5"))
                 
```

## The SQL CASE Statements
The `CASE` statement in SQL is a way to handle if/else logic. The `CASE` statement is followed by at least one pair of `WHEN` and `THEN` statements and ends by `END` keyword.  THE `ELSE` statement is optional and captures values not mentioned in `WHEN` and `THEN` statements.

### Simple CASE Statements 

**Syntax 1:**

```sql
CASE value 
WHEN compare_value1 THEN result1
WHEN compare_value2 THEN result2
...
ELSE result END
```

**Syntax 2:**

```sql
CASE 
WHEN condition1 THEN result1
WHEN condition2 THEN result2
...
ELSE result END
```

**Example 1:**

Suppose you want to find the drugs which are present in *drugs_target* table but not in *drugs_names* table.

```{r case_1}
kable(dbGetQuery(drugs,"SELECT dn.drugBankID, dt.name, CASE WHEN dn.drugBankID IS NULL THEN 'not' ELSE 'in' END AS in_drugs_names FROM drugs_target AS dt LEFT JOIN drugs_names AS dn USING(drugBankID) ORDER BY dt.name LIMIT 10"))
```

### Adding multiple conditions 

**Example 2:**

Select cid, drug name, molecular formula and molecular weight of all drugs and group them into three categories as follows:

1. small - drugs with the molecular weight of less than 200
2. medium - drugs with the molecular weight of less than 400
3. big - drugs with the molecular weight of above 400 

```{r case_2}
kable(dbGetQuery(drugs, 'SELECT cid,name, molecular_formula, molecular_weight, 
  CASE WHEN molecular_weight < 200 
  THEN "small"
  WHEN molecular_weight < 400
  THEN "medium"  
  ELSE "big" 
  END AS size 
FROM drugs_data INNER JOIN drugs_names USING (cid) LIMIT 5'))
```

**Explanation:**

1. The `CASE` statement checks the molecular weight of a compound in each row.
2. For any compound, if the molecular weight is less than 200, the word "small" get inserted to the column we created as `size`. Similarly, "medium" and "big" are inserted accordingly.
3. At the same time, JOIN takes place between drugs_data and drugs_names table.


### Using aggregate functions with CASE Statements

**Example 3:**

Now count the drugs in each group from the query above.

```{r case_3}
kable(dbGetQuery(drugs,"SELECT CASE WHEN molecular_weight < 200    THEN 'small' WHEN molecular_weight < 400   THEN 'medium' ELSE 'big'  END AS size,COUNT(1) AS num_drugs FROM drugs_data INNER JOIN drugs_names USING (cid) GROUP BY size"))
```


### Exercise 5 

**1. Group protein targets into three categories based on the number of drugs they bind. Three groups of protein targets are as follows:**

1. less_than_3_drugs - bind less than 3 drugs
2. four_to_10_drugs - bind 4 or more but less than 10 drugs
3. more_than_11_drugs - bind more than 11 drugs

*Hint:* In order to perform this query you have to use `INNER JOIN`, Subquery and `Case statements`.

_Sample Output_

```{r ex5_1}
kable(dbGetQuery(drugs,"SELECT SUM(CASE WHEN A.count<3 THEN 1 ELSE 0 END) AS less_than_3_drugs, SUM(CASE WHEN A.count>=4 AND A.count <= 10 THEN 1 ELSE 0 END) AS four_to_10_drugs, SUM( CASE WHEN A.count >= 11 THEN 1 ELSE 0 END) AS more_than_11_drugs FROM 
                 (SELECT DISTINCT dt.uniProtID, dt.proteinName,sub.count FROM drugs_target AS dt INNER JOIN (SELECT uniProtID, COUNT(drugBankID) AS count FROM drugs_target GROUP BY uniProtID) AS sub USING(uniProtID) ORDER BY sub.count) AS A"))
```

**Note:** There are a large number of protein targets that bind up to 3 different drugs compared to protein targets which bind 4 or more different drugs. Based on this data we may conclude that protein targets are quite specific for the kind of drugs they bind.

**2. Now group drugs into three categories based on the number of protein targets they bind. Three groups of drugs are as follows:**

1. less_than_3_protein_targets - bind less than 3 protein targets
2. four_to_10_protein_targets - bind 4 or more but less than 10 targets
3. more_than_11_protein_targets - bind more than 11 protein targets

_Sample Output_

```{r ex5_2}
kable(dbGetQuery(drugs,"SELECT SUM(CASE WHEN A.count<3 THEN 1 ELSE 0 END) AS less_than_3_protein_targets, SUM(CASE WHEN A.count>=4 AND A.count <= 10 THEN 1 ELSE 0 END) AS four_to_10_protein_targets, SUM( CASE WHEN A.count >= 11 THEN 1 ELSE 0 END) AS more_than_11_protein_targets FROM 
(SELECT DISTINCT dn.drugBankID, dn.name,sub.count FROM drugs_names AS dn INNER JOIN (SELECT drugBankID, COUNT(uniProtID) AS count FROM drugs_target GROUP BY drugBankID) AS sub USING(drugBankID) ORDER BY sub.count) AS A"))
```

**Note:** There are 609 drugs that bind 3 or fewer protein targets. We again observe high specificity for drugs and protein targets.

**3. Find those biotech drugs from _drugs_target_ table that end with 'ab' and use the case statement to form two groups as follows: **

1. Antibody - drugs ending with 'ab' and of type 'BiotechDrug'
2. Not antibody - all other drugs of type 'BiotechDrug'

_Sample Output_

```{r ex5_3}
kable(dbGetQuery(drugs,"SELECT DISTINCT drugBankID, CASE WHEN name LIKE '%ab' THEN 'Antibody' ELSE 'Not Antibody' END class FROM drugs_target WHERE type = 'BiotechDrug' LIMIT 10"))
```

## Summary

**Congratulations**, on completing the tutorial you are an SQL ninja now!! In this tutorial you specifically learned:

1. Various types of Subqueries
2. Advanced usage of JOINS
3. Set operators and their use in SQL statements
4. CASE Statements for If/Else logic

In case you have any questions, comments please [mail me](mailto:varun[DOT]napster[AT]gmail[DOT]com). 
