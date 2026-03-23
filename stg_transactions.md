{{ config(
    materialized='table', 
    unique_key='txn_id'
) }}

with raw_data as (
    select * from {{ source('transactions_source', 'transactions_batch_1') }}
)

select
    txn_id,
    cust_id,
    amount,
    is_member,
    points,
    status,
    txn_date,

    case 
        when txn_id is null or txn_id = '' then 'CORRUPT'
        when amount is null or amount < 0 then 'CORRUPT'
        when txn_date is null then 'CORRUPT'
        else 'CLEAN'
    end as _record_status,

    {{ audit_columns('bronze') }},
    
    cast(null as string) as _source_file 

from raw_data

{% if is_incremental() %}
  where txn_date > (select max(txn_date) from {{ this }})
{% endif %}