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
![alt text](https://github.com/Andreawqh/hw7/blob/master/ER%20Diagram.jpeg)
-- comment: 
* 1. CASE TABLE: include the serial id (a unique generated id) that uniquely identify each row on the table, where it is associated with 1 or more report_id, and event_date, created_date. it is used to conNect with other tables including: PRODUCT, TERMS, CASE_REPORT 

* 2. PRODUCT TABLE: has 1 to many relationship with CASE table since each product can be associated with multiple cases. the product attribute is associated with PRODUCT_DATA


* 3. PRODUCT_DATA: one to one relationshop with PRODUCT, each product has related product_type and product_code, product code is a foreign key that can identify related descriptions

* 4. DESCRIPTION TABLE provide the descriptions that can be identified by the product code

* 5. TERMS is identified by the unique serial _id, many ids can be associated with the same symptoms(separated from the terms). there are multiple symptoms that belong to the same terms-which is shown in the SYMPTOMS TABLE

* 6. CASE_REPORT TABLE is also identified by serial_id, it has the fields of patient age (which will be normalized), and sex, and outcome



## Views and Index
--small query : find oldest people in table
```
patient_age | age_units 
-------------+-----------
             | year(s)
         104 | year(s)
         104 | year(s)
         104 | year(s)
         102 | year(s)
         101 | year(s)
         100 | year(s)
         100 | year(s)
         100 | year(s)
         100 | year(s)
(10 rows)

```
EXPLAIN: 
```
  QUERY PLAN                                   
-------------------------------------------------------------------------------
 Limit  (cost=2455.38..2455.41 rows=10 width=12)
   ->  Sort  (cost=2455.38..2526.42 rows=28417 width=12)
         Sort Key: patient_age DESC
         ->  Seq Scan on caers_event  (cost=0.00..1841.30 rows=28417 width=12)
               Filter: ((age_units)::text ~~* '%year%'::text)
(5 rows)


```
EXPLAIN ANALYZE
```
`QUERY PLAN                                                          
------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=2455.38..2455.41 rows=10 width=12) (actual time=26.678..26.680 rows=10 loops=1)
   ->  Sort  (cost=2455.38..2526.42 rows=28417 width=12) (actual time=26.677..26.677 rows=10 loops=1)
         Sort Key: patient_age DESC
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Seq Scan on caers_event  (cost=0.00..1841.30 rows=28417 width=12) (actual time=0.034..21.593 rows=28379 loops=1)
               Filter: ((age_units)::text ~~* '%year%'::text)
               Rows Removed by Filter: 51645
 Planning Time: 0.426 ms
 Execution Time: 26.713 ms
(9 rows)


```
EXPLAIN (after add index):
```
QUERY PLAN                              
----------------------------------------------------------------------
 Limit  (cost=0.00..767.21 rows=10 width=63)
   ->  Seq Scan on caers_event  (cost=0.00..1841.30 rows=24 width=63)
         Filter: ((age_units)::text ~~* '%day%'::text)
(3 rows)

```
EXPLAIN ANALYZE (after add index)
```
 QUERY PLAN                                                    
-----------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.00..767.21 rows=10 width=63) (actual time=1.679..3.427 rows=10 loops=1)
   ->  Seq Scan on caers_event  (cost=0.00..1841.30 rows=24 width=63) (actual time=1.677..3.424 rows=10 loops=1)
         Filter: ((age_units)::text ~~* '%day%'::text)
         Rows Removed by Filter: 9335
 Planning Time: 0.087 ms
 Execution Time: 3.446 ms
(6 rows)

```
conclusion: the index worked! (much less time), this is because when ordring the age, it can use a binary tree instead of looking through the data

## join
The join is overall successful, the data is pretty much similar to the stage data, the only difference is the order of the columns, this is due to the inner join only combines the tables but don't order them

## GENERAL QUERIES
--1. Find the name and product code of all of the products (no duplicates) that give you nightmares 

```
product   | product_code |                                                           terms                                                           
-------------+--------------+---------------------------------------------------------------------------------------------------------------------------
 EXEMPTION 4 | 54           | {NIGHTMARE," INCREASED APPETITE"," HEADACHE"," CONFUSIONAL STATE"," COMPULSIONS"," BLOOD GLUCOSE INCREASED"," AGITATION"}
 EXEMPTION 4 | 54           | {NIGHTMARE," INCREASED APPETITE"," HEADACHE"," CONFUSIONAL STATE"," COMPULSIONS"," BLOOD GLUCOSE INCREASED"," AGITATION"}
 EXEMPTION 4 | 54           | {NIGHTMARE," INCREASED APPETITE"," HEADACHE"," CONFUSIONAL STATE"," COMPULSIONS"," BLOOD GLUCOSE INCREASED"," AGITATION"}
 EXEMPTION 4 | 54           | {NIGHTMARE," INCREASED APPETITE"," HEADACHE"," CONFUSIONAL STATE"," COMPULSIONS"," BLOOD GLUCOSE INCREASED"," AGITATION"}
 EXEMPTION 4 | 41           | {NIGHTMARE," INCREASED APPETITE"," HEADACHE"," CONFUSIONAL STATE"," COMPULSIONS"," BLOOD GLUCOSE INCREASED"," AGITATION"}
(5 rows)


```
--2.Create a list of event dates, report ids and product, regardless of whether or not there's a product name
```
event_date |    report_id    |                               product                               
------------+-----------------+---------------------------------------------------------------------
 2013-09-29 | 172989          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172990          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172991          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172992          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172993          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172994          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172995          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 172996          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-29 | 173006          | RESER'S STONEMILL KITCHENS COUNTRY RED POTATO SALAD
 2013-09-12 | 173016          | MOVE FREE
 2013-09-13 | 173073          | NEW ZEALAND GRANNY SMITH APPLES
 ...
--THIS IS ONLY PART OF THE LIST
```

--3.What are the 3 most common symptoms
```
 symptom     | count 
----------------+-------
 Ovarian cancer | 11632
 OVARIAN CANCER |  8194
  Death         |  4787
(3 rows)
```
--sort by alphebetical order
```
    symptom                         | count 
--------------------------------------------------------+-------
  ABASIA                                                |   149
  ABDOMINAL ABSCESS                                     |     5
  ABDOMINAL ADHESIONS                                   |    12
  ABDOMINAL DISCOMFORT                                  |   785
  ABDOMINAL DISTENSION                                  |   749
  ABDOMINAL HERNIA                                      |     2
  ABDOMINAL HERNIA PERFORATION                          |     2
  ABDOMINAL INFECTION                                   |     1
  ABDOMINAL INJURY                                      |     6
  ABDOMINAL MASS                                        |    11
  ABDOMINAL NEOPLASM                                    |     2
  ABDOMINAL PAIN                                        |  2440
  ABDOMINAL PAIN LOWER                                  |    75
  ABDOMINAL PAIN UPPER                                  |  2124
  ABDOMINAL RIGIDITY                                    |    15
  ABDOMINAL SEPSIS                                      |     2
  ABDOMINAL SYMPTOM                                     |     2
  ABDOMINAL TENDERNESS                                  |    20
  ABNORMAL BEHAVIOUR                                    |    89
  ABNORMAL CLOTTING FACTOR                              |     4
  ABNORMAL DREAMS                                       |     9
  ... ONLY PART OF THE LIST
  ```
--4.How afraid should you be of yogurt? 
```
product                   | patient_age | age_units 
---------------------------------------------+-------------+-----------
 BLUE BUNNY FROZEN YOGURT NO SUGAR ADDED     |          81 | year(s)
 DANNON BLUEBERRY YOGURT                     |          74 | year(s)
 DANNON ACTIVIA LIGHT FAT FREE PEACH YOGURT  |          73 | year(s)
 GENERAL MILLS, YOPLAIT LOW FAT PEACH YOGURT |          72 | year(s)
 YO GOOD PEACH FLAVORED DRINKABLE YOGURT     |          71 | year(s)
(5 rows)
```

--5.Show a comma separated list of symptoms / terms for every event
```
serial_id |                               string_agg                                
-----------+-------------------------------------------------------------------------
         1 | NAUSEA
         2 | VOMITING
         3 | SLEEP DISORDER, ANXIETY, SALIVARY GLAND DISORDER
         4 |  PAIN, CAUSTIC INJURY, BURNING SENSATION,TENDERNESS, MUCOSAL ULCERATION
         5 |  DYSGEUSIA,HYPERSENSITIVITY
(5 rows)
```

extra credit:
additional query
--Find the product that has is most reported
```
 product                                                                                                  | case_count 
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------
 EXEMPTION 4                                                                                                                                                                                              |      26668
 SUPER BETA PROSTATE                                                                                                                                                                                      |       1076
 VITAMIN D                                                                                                                                                                                                |        605
 MULTIVITAMIN                                                                                                                                                                                             |        429
 FISH OIL                                                                                                                                                                                                 |        416
 
 ```
 ## Conclusions
 --why this semi-normalized form is better
 * Semi-normalized form is better when running analysis on specific attribute, it is much more straightforward since it is only related to the primary key. For example, when analyzing the most common symptoms, after being cleaned out from the terms into atomic form, it is much easier to use query to extract the specific data.
 
--why this semi-normalized form is worse / difficult to use?
* when exploring relationship between multiple attributes it is much harder to since we need to do multiple joins to have a full table before conducting analysis. For example, the question that wants name and product code of all of the products that give nightmare need to combine product table, terms table  before running queries. If the question also wanted patient name, gender, etc, we will have to combine more tables in 1 query (containing multiple subqueries), which makes the code very complex
