{{ config(materialized='incremental', unique_key='txn_id') }}

with source as (
    select * from {{ ref('stg_transactions') }}
)

select 
    *,
    current_timestamp() as _quarantined_at
from source
where _record_status = 'CORRUPT'
   or amount is null
   or points is null