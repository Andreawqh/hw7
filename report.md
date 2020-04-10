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
