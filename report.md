 ## Exploring Data  
 
 --1. Describe the table to understand their relationship, below reveals the data type whether one is primary key
 
```
                                          
                                          Table "public.staging_caers_events"
    Column    |          Type          | Collation | Nullable |                         Default                         
--------------+------------------------+-----------+----------+---------------------------------------------------------
 report_id    | character varying(100) |           |          | 
 created_date | date                   |           |          | 
 event_date   | date                   |           |          | 
 product_type | text                   |           |          | 
 product      | text                   |           |          | 
 product_code | text                   |           |          | 
 description  | text                   |           |          | 
 patient_age  | integer                |           |          | 
 age_units    | character varying(10)  |           |          | 
 sex          | character varying(10)  |           |          | 
 terms        | text                   |           |          | 
 outcomes     | text                   |           |          | 
 serial_id    | integer                |           | not null | nextval('staging_caers_events_serial_id_seq'::regclass)
Indexes:
    "staging_caers_events_pkey" PRIMARY KEY, btree (serial_id)

```
result analysis: This shows that the datatypes are set based on the CREATE TABLE statement, and the created serial_id has the constraint notnull and is a primary key.


 --2. Find the maximum number of values in serial_id to find the maximum distinct values

```
count 
-------
 80024
(1 row)

```
result: This shows the primary key has the maximum number- which is 80024

--3. Check whether there are unique attributes that may qualify it as a candidate key

```
distinct_id | d_term | d_create | d_event | d_producttype |   p   | pc | descript | age | d_term | doutcome 
-------------+--------+----------+---------+---------------+-------+----+----------+-----+--------+----------
       46198 |  19626 |     1641 |    3099 |             2 | 28109 | 46 |       42 | 104 |  19626 |      168
(1 row)

```
result: None of the columns have distinct rows, so none of them can be candidate key

--4. --4. since report_id and product are most unique, try to see if their combination is unique
```
created_date |                                             product                                             
--------------+-------------------------------------------------------------------------------------------------
 2018-08-07   | BLUE DIAMOND ALMOND BREEZE
 2014-04-15   | GNC GENETIX HD CYCLO-SHRED 90 TABLETS
 2016-08-19   | BROMELAIN
 2015-08-06   | LOBSTER AND CRAB
 2015-12-28   | ACETYL L-CARNITINE CAPS
 2015-04-10   | NATURE'S BOUNTY FLEXAMIN TRIPLE STRENGTH WITH JOINT FLEX PLUS VITAMIN D3 2000 IU COATED TABLETS
 2014-06-18   | CITRACAL PETITES CHOLECALCIFEROL+CALCIUM
 2015-10-29   | SEA BEST SHRIMP
 2016-08-01   | NATURE MADE ADULT GUMMIES MULTI FOR HER PLUS OMEGA-3S (DIETARY SUPPLEMENT) GUMMY
 2016-12-23   | D3 LIQUID
(10 rows)
```
result: this shows the combination of the two attributes should be unique - composite key

## Database Design
![alt text](https://github.com/Andreawqh/hw7/blob/master/ER%20Diagram.jpg)
-- comment: 1. CASE TABLE: include the serial id (a unique generated id) that uniquely identify each row on the table, where it is associated with 1 
