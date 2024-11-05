# COPY AND PASTE INTO:
[https://dbdiagram.io/d](https://dbdiagram.io/d)

```sql
TABLE sales_by_sku {
  productSKU VARCHAR(50)
  total_ordered INTEGER
}

TABLE all_sessions {
  fullVisitorId TEXT
  channelGrouping TEXT
  time BIGINT
  country TEXT
  city TEXT
  totalTransactionRevenue BIGINT
  transactions INTEGER
  timeOnSite INTEGER
  pageviews INTEGER
  sessionQualityDim INTEGER
  date DATE
  visitId TEXT
  type TEXT
  productRefundAmount NUMERIC
  productQuantity INTEGER
  productPrice NUMERIC
  productRevenue NUMERIC
  productSKU TEXT
  v2ProductName TEXT
  v2ProductCategory TEXT
  productVariant TEXT
  currencyCode TEXT
  itemQuantity INTEGER
  itemRevenue NUMERIC
  transactionRevenue NUMERIC
  transactionId TEXT
  pageTitle TEXT
  searchKeyword TEXT
  pagePathLevel1 TEXT
  eCommerceAction_type INTEGER
  eCommerceAction_step INTEGER
  eCommerceAction_option TEXT
  timeonsite_proper INTERVAL
  time_proper TIMEWITHOUTTIMEZONE
}

TABLE sales_report {
  productSKU VARCHAR(50)
  total_ordered INTEGER
  name TEXT
  stockLevel integer
  restockingLeadTime INTEGER
  sentimentScore NUMERIC(3, 2)
  sentimentMagnitude NUMERIC(3, 2)
  ratio NUMERIC(6, 5)
}

TABLE products {
  SKU VARCHAR(50)
  name TEXT
  orderedQuantity INTEGER
  stockLevel INTEGER
  restockingLeadTime INTEGER
  sentimentScore NUMERIC(3, 2)
  sentimentMagnitude NUMERIC(3, 2)
}

TABLE analytics {
  visitNumber INTEGER
    visitId TEXT
    visitStartTime BIGINT
    date DATE
    fullVisitorId TEXT
    userId TEXT
    channelGrouping TEXT
    socialEngagementType TEXT
    units_sold INTEGER
    pageviews INTEGER
    timeOnSite INTEGER
    bounces INTEGER
    revenue NUMERIC
    unit_price NUMERIC
    timeonsite_proper INTERVAL
    visitstarttime_proper TIMEWITHOUTTIMEZONE
}

Ref: "all_sessions"."productSKU" > "products"."SKU"

Ref: "products"."SKU" < "sales_by_sku"."productSKU"

Ref: "products"."SKU" < "sales_report"."productSKU"
```