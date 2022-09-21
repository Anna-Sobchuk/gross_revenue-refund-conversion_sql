# gross_revenue-refund-conversion_sql

**1.1 Calculate as a cohort how much we earned each day. That is, if the user took the trial on 01/01 and we received money from him after 7 days (08/01), then this revenue should be credited to 01/01. Calculate Gross Revenue.**

```sql
with prev_pay as( --adding column with month shifted for 1
  select 
    *,
    lag(customer_price) over (partition by subscriber_id order by event_date) as prev_pay_amount
  from my_table
  where refund = false
),

add_pay_cohorts as( --if trial change date for previous or else leave current day
  select 
    *,
    case 
      when prev_pay_amount = 0 then 
        lag(event_date) over (partition by subscriber_id order by event_date)
      else 
        event_date 
    end as pay_cohort
  from prev_pay
)

select
  pay_cohort,
  sum(customer_price)
from add_pay_cohorts
group by 1
order by 1
```

**1.2 Calculate on which day the user makes a refund on average.**

```sql
with day_refund as(
  select
    subscriber_id,
    first_value(event_date) over (partition by subscriber_id order by event_date) as ref
  from my_table
  where true
		and refund = true
),

day_pay as(
  select
    subscriber_id,
    first_value(event_date) over (partition by subscriber_id order by event_date) as pay
  from my_table
  where true
    and refund = false
),

day_of_refund as(
  select
    subscriber_id,
    date_diff(ref, pay, day) as when_refunded
  from day_refund 
    left join day_pay using(subscriber_id)
)

select
  sum(when_refunded) / count(when_refunded) as average_day
from day_of_refund
```

**1.3 Calculate the conversion in the 2nd, 3rd, 4th, 5th and 6th payment according to the monthly rates. Take the monthly cohort. That is, it is important for us to know how people who first bought a subscription, for example, in October, then continued to pay for the following months.**

```sql
with add_day_cohort as( --choosing date of first payment
  select 
    *,
    first_value(event_date) over (partition by subscriber_id order by event_date) as cohort_day
  from my_table
  where true
    and subscription_name = 'monthly'
    and customer_price > 0
),

add_month_cohort as(
  select 
    *,
    format_timestamp('%b_%y', cohort_day) as month_cohort, --grooping by month
    row_number() over (partition by subscriber_id order by event_date) as num_pay, --counting which transaction it is
  from add_day_cohort
),

pivoted as(
  select *
  from( --choosing cohort month and number of transaction
        select 
          num_pay,
          month_cohort
        from add_month_cohort
  ) 
  pivot (count(*) for month_cohort in ('Jan_18', 'Feb_18')) --adding all to pivot for cohorts in array
)

select
  num_pay, --month of retention
  Jan_18/(first_value(Jan_18) over (order by num_pay)) as Jan_2018_conversion, --percent of people who were coming next months and are in cohort of january of 18
  Feb_18/(first_value(Feb_18) over (order by num_pay)) as Feb_2018_conversion
from pivoted
order by num_pay
limit 5 offset 1
```