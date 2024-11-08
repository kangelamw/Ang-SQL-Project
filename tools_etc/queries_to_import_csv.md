# QUERIES to insert tables into the db

### No Primary Keys to start with.
I added `SKU`/`productSKU` as Primary Key for `Products`

#### Foreign Keys:
- <details>
    - `productSKU` on:
        - `all_sessions`
        - `sales_by_sku`
        - `sales_report`

    </details>

### `all_sessions.csv`
```sql
CREATE TABLE all_sessions (
    fullVisitorId TEXT, 
    channelGrouping TEXT,
    time BIGINT,
    country TEXT,
    city TEXT,
    totalTransactionRevenue BIGINT,
    transactions INTEGER,
    timeOnSite INTEGER,
    pageviews INTEGER,
    sessionQualityDim INTEGER,
    date DATE,
    visitId TEXT,
    type TEXT,
    productRefundAmount NUMERIC,
    productQuantity INTEGER,
    productPrice NUMERIC,
    productRevenue NUMERIC,
    productSKU TEXT,
    v2ProductName TEXT,
    v2ProductCategory TEXT,
    productVariant TEXT,
    currencyCode TEXT,
    itemQuantity INTEGER,
    itemRevenue NUMERIC,
    transactionRevenue NUMERIC,
    transactionId TEXT,
    pageTitle TEXT,
    searchKeyword TEXT,
    pagePathLevel1 TEXT,
    eCommerceAction_type INTEGER,
    eCommerceAction_step INTEGER,
    eCommerceAction_option TEXT
);
```

<br>

#### `analytics.csv`
```
CREATE TABLE analytics (
    visitNumber INTEGER,
    visitId TEXT,
    visitStartTime BIGINT,
    date DATE,
    fullVisitorId TEXT,
    userId TEXT,
    channelGrouping TEXT,
    socialEngagementType TEXT,
    units_sold INTEGER,
    pageviews INTEGER,
    timeOnSite INTEGER,
    bounces INTEGER,
    revenue NUMERIC,
    unit_price NUMERIC
);
```

<br>

#### `products.csv`
```
CREATE TABLE products (
    SKU VARCHAR(50),
    name TEXT,
    orderedQuantity INTEGER,
    stockLevel INTEGER,
    restockingLeadTime INTEGER,
    sentimentScore NUMERIC(3, 2),
    sentimentMagnitude NUMERIC(3, 2)
);
```

<br>

#### `sales_report.csv`
```
CREATE TABLE sales_report (
    productSKU VARCHAR(50),
    total_ordered INTEGER,
    name TEXT,
    stockLevel INTEGER,
    restockingLeadTime INTEGER,
    sentimentScore NUMERIC(3, 2),
    sentimentMagnitude NUMERIC(3, 2),
    ratio NUMERIC(6, 5)
);
```

<br>

#### `sales_by_sku.csv`
```CREATE TABLE sales_by_sku (
    productSKU VARCHAR(50),
    total_ordered INTEGER
);
```

