## Quality Check №1 - Completeness
This check verifies data completeness - it should show us if some essential values are missing. Failure of this check
requires corrective action. Lack of data in columns where values are expected
can lead to incorrect analytics and reporting.

##### Data quality dimension - Completeness

##### SQL logic or condition
`SELECT * FROM ecommerce WHERE customerId is null;`
We can check any important columns in such way, but in this particular table only these fields
contain nulls.
##### Severity level - critical (requires actions)

## Quality Check №2 - Quantity validation
Typically, quantity is positive, and only in case of cancellation it should be negative.
Cancellation InvoiceNo values are marked with letter 'C' at the beginning.
Any other behavior violates business logic.

##### Data quality dimension - Validity

##### SQL logic or condition
`SELECT * FROM ecommerce WHERE (quantity >= 0 AND InvoiceNo LIKE 'C%') OR (quantity <= 0 AND InvoiceNo NOT LIKE 'C%');`

##### Severity level - critical (requires actions)

## Quality Check №3 - Uniqueness of InvoiceNo and StockCode
Duplicate data can worsen conclusions, by skewing observations and making them biased.

##### Data quality dimension - Uniqueness

##### SQL logic or condition
```
SELECT * FROM ecommerce
WHERE (InvoiceNo, StockCode) IN (
    SELECT InvoiceNo, StockCode
    FROM ecommerce
    WHERE InvoiceNo IS NOT NULL AND StockCode IS NOT NULL
    GROUP BY InvoiceNo, StockCode
    HAVING COUNT(*) > 1
)
ORDER BY InvoiceNo, StockCode;
```

##### Severity level - warning

## Quality Check №4 - Timeliness of invoices
Outdated data may reduce analytical relevance and business value, as can future (invalid) dates.
If records fall outside the expected date range, the check can be considered failed.


##### Data quality dimension - Timeliness

##### SQL logic or condition
```
SELECT
    datetime(AVG(julianday(InvoiceDate))) as avg_invoice_date,
    MIN(InvoiceDate) as earliest_date,
    MAX(InvoiceDate) as latest_date,
    COUNT(*) as total_records,
    SUM(CASE WHEN InvoiceDate > '2011-12-10' THEN 1 ELSE 0 END) as records_after_threshold,
    SUM(CASE WHEN InvoiceDate < '2010-12-01' THEN 1 ELSE 0 END) as records_before_threshold
FROM ecommerce
WHERE InvoiceDate IS NOT NULL;
```

##### Severity level - warning

## Quality Check №5 - Country validation

##### Data quality dimension - Validity, Consistency
Simple check that country belongs to a predefined list of countries is validation check -
"country field is really a country". However, "USA vs United States" would be Consistency check.
The nature of the check depends on data requirements.

##### SQL logic or condition
We should have some predefined list or map of countries and their notation. Any country not in this list
should be considered invalid. This logic is implemented in real-data-checks.ipynb
```
    SELECT e.Country, COUNT(*) AS invalid_count
    FROM ecommerce e
    LEFT JOIN countries_references r
    ON e.Country = r.name
    WHERE r.name IS NULL
    GROUP BY e.Country;
```

##### Severity level - critical