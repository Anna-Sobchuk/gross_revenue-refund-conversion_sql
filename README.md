# gross_revenue-refund-conversion_sql
In BigQuery exists such database:

- event_date(date) - date of event
- subscription_name(string) - name of subscription (monthly/annual)
- subscriber_id(string) - unique user id
- customer_price(float) - amont paid for subscription
- refund(boolean) - is a transaction a refund (true/false)

## In the application, the monetization model is subscription.

- There are subscriptions with a trial period (that is, on day 0, the user takes a trial and there is a record in the database where the user had a transaction with a price = 0).
- There are trialless subscriptions, where the initial entry will be a transaction with a price > 0.
- Data on refund subscriptions are also recorded in the database. This is a normal transaction in which the refund field will have the value true.

## Write the following SQL queries:

1. Calculate as a cohort how much we earned each day. That is, if the user took the trial on 01/01 and we received money from him after 7 days (08/01), then this revenue should be credited to 01/01. Calculate Gross Revenue.
2. Calculate on which day the user makes a refund on average.
3. Calculate the conversion in the 2nd, 3rd, 4th, 5th and 6th payment according to the monthly rates. Take the monthly cohort. That is, it is important for us to know how people who first bought a subscription, for example, in October, then continued to pay for the following months.
