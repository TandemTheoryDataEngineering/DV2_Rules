
# In progress ...

# DV2_Rules
Best Practices and Rules for Querying Hubs, Satellites, and Links in Data Vault 2.0

# DV2_Rules
Best Practices and Rules for Querying Hubs, Satellites, and Links in Data Vault 2.0
## Introduction
Data Vault 2.0 (DV 2.0) is an advanced data modeling methodology designed to enable agile and scalable data warehousing. It supports the integration of data from multiple sources and provides a robust framework for historical data tracking. The key components of a Data Vault are hubs, satellites, and links. Each plays a crucial role in organizing data for efficient storage and retrieval. This paper provides a comprehensive guide to querying these components effectively, ensuring high performance and accuracy in data analysis.
## Understanding Data Vault Components
### Hubs
Hubs store unique business keys and the metadata related to their origin and load. Each hub represents a distinct business entity, such as a customer or product.

### Satellites
Satellites store descriptive attributes and their historical versions, linked to hubs or links. They capture the changes in the business entity's attributes over time.

### Links
Links establish and store relationships between business keys from different hubs. They represent associations such as transactions, memberships, or hierarchies.

## General Best Practices
### Data Model Familiarity
Understand the Architecture: Familiarize yourself with the Data Vault components and their relationships.
Surrogate Keys: Use surrogate keys for joins to improve performance, as they are usually indexed and smaller than business keys.
### Query Strategy
Avoid Direct Queries on Raw Vault: Prefer querying Business Vault or Information Marts which are optimized for analysis.
SQL Optimization: Write efficient SQL queries using proper indexing, partitioning, and window functions to improve performance.
## Querying Hubs
### Direct Hub Queries
Retrieve Business Keys and Metadata: Fetch unique business keys and their load dates.

SELECT HubBusinessKey, LoadDate
FROM HubTable
WHERE HubBusinessKey = 'XYZ';

### Joining Hubs and Satellites
Join with Satellites for Descriptive Attributes: Combine hubs with related satellites to retrieve comprehensive entity information.

SELECT h.HubBusinessKey, s.Attribute1, s.Attribute2, s.LoadDate
FROM HubTable h
JOIN SatelliteTable s ON h.HubBusinessKey = s.HubBusinessKey
WHERE h.HubBusinessKey = 'XYZ';


## Querying Satellites
### Retrieve Current Attributes
Latest Record Selection: Use window functions to filter the latest satellite records.

SELECT s.HubBusinessKey, s.Attribute1, s.Attribute2, s.LoadDate
FROM (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY HubBusinessKey ORDER BY LoadDate DESC) as rn
  FROM SatelliteTable
) s
WHERE s.rn = 1;
### Historical Attribute Queries
Filter by Date Range: Retrieve historical data by filtering based on date ranges.

SELECT HubBusinessKey, Attribute1, Attribute2, LoadDate
FROM SatelliteTable
WHERE HubBusinessKey = 'XYZ'
  AND LoadDate BETWEEN '2022-01-01' AND '2022-12-31';
## Querying Links
### Direct Link Queries
Retrieve Relationships: Fetch relationships between business keys.

SELECT HubBusinessKey1, HubBusinessKey2, LoadDate
FROM LinkTable;
### Joining Links with Hubs and Satellites
Detailed Relationship Information: Join links with hubs and satellites to get detailed relationship data.

SELECT l.HubBusinessKey1, l.HubBusinessKey2, s1.Attribute1, s2.Attribute2, l.LoadDate
FROM LinkTable l
JOIN SatelliteTable1 s1 ON l.HubBusinessKey1 = s1.HubBusinessKey
JOIN SatelliteTable2 s2 ON l.HubBusinessKey2 = s2.HubBusinessKey
WHERE l.LoadDate = (SELECT MAX(LoadDate) FROM LinkTable WHERE HubBusinessKey1 = 'Key1' AND HubBusinessKey2 = 'Key2');

## Performance Optimization
### Indexing
Primary Keys and Foreign Keys: Ensure that primary and foreign keys are indexed.
Date Columns: Index date columns used in filtering or joining.
### Partitioning
Large Tables: Use partitioning strategies (e.g., range partitioning on date columns) for large satellite tables to improve query performance.
### Materialized Views
Frequent Queries: Create materialized views for frequently accessed and complex queries to enhance performance.
##Conclusion
Querying hubs, satellites, and links in a Data Vault 2.0 environment requires a deep understanding of the model and strategic use of SQL techniques. By following best practices such as leveraging surrogate keys, indexing appropriately, and utilizing window functions, one can achieve efficient and accurate data retrieval. These practices ensure that the Data Vault remains a powerful and flexible solution for modern data warehousing needs.








# The Don'ts When Querying Data Vaults
Querying Data Vaults efficiently requires not only an understanding of best practices but also an awareness of common pitfalls and practices to avoid. The following are the key "don'ts" to keep in mind when querying Data Vaults:

## General Don'ts
##Directly Querying the Raw Vault for Business Reports
Avoid Direct Access: The raw vault is not optimized for end-user queries and reporting. Use the Business Vault or Information Marts instead.
Complex Queries: Raw vault queries can become overly complex and inefficient due to the nature of normalized data storage.
## Ignoring the Use of Surrogate Keys
Business Keys: Using business keys for joins can lead to performance issues, especially in large datasets. Always prefer surrogate keys.
## Overlooking Indexing and Partitioning
No Indexes: Failing to index primary and foreign keys, as well as frequently queried columns, can significantly degrade performance.
No Partitioning: Not partitioning large tables can lead to slow queries and difficulty in managing data at scale.
## Don'ts When Querying Hubs
Including Irrelevant Columns
Select Only Needed Columns: Fetching all columns from a hub when only the business key and load date are needed can waste resources.

-- Avoid this
SELECT * FROM HubTable WHERE HubBusinessKey = 'XYZ';

-- Instead, do this
SELECT HubBusinessKey, LoadDate FROM HubTable WHERE HubBusinessKey = 'XYZ';
Joining Hubs without Filtering
No Filters: Joining hubs without appropriate filters can result in large intermediate result sets, which can be inefficient.

-- Avoid this
SELECT * FROM Hub1 h1 JOIN Hub2 h2 ON h1.HubBusinessKey = h2.HubBusinessKey;

-- Instead, do this with filters
SELECT * FROM Hub1 h1 JOIN Hub2 h2 ON h1.HubBusinessKey = h2.HubBusinessKey WHERE h1.HubBusinessKey = 'XYZ';
## Don'ts When Querying Satellites
Ignoring the Load Date in Queries
Historical Data: Not considering the load date when querying satellites can lead to retrieving irrelevant or incorrect historical data.

-- Avoid this
SELECT * FROM SatelliteTable WHERE HubBusinessKey = 'XYZ';

-- Instead, do this
SELECT * FROM SatelliteTable WHERE HubBusinessKey = 'XYZ' AND LoadDate <= '2023-12-31';
## Not Using Window Functions for Latest Records
All Records: Selecting all records when only the latest record is needed can increase query complexity and execution time.

-- Avoid this
SELECT * FROM SatelliteTable WHERE HubBusinessKey = 'XYZ';

-- Instead, do this
SELECT s.* FROM (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY HubBusinessKey ORDER BY LoadDate DESC) as rn
  FROM SatelliteTable
) s WHERE s.rn = 1;
## Don'ts When Querying Links
Joining Links without Appropriate Filters
Full Table Scans: Joining links with hubs or other links without appropriate filters can lead to full table scans and inefficient queries.

-- Avoid this
SELECT * FROM LinkTable l JOIN HubTable h ON l.HubBusinessKey1 = h.HubBusinessKey;

-- Instead, do this with filters
SELECT * FROM LinkTable l JOIN HubTable h ON l.HubBusinessKey1 = h.HubBusinessKey WHERE h.HubBusinessKey = 'XYZ';

