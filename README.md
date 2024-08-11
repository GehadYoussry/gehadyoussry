## Introduction

In many fintech companies, particularly those that need to transfer money to their users as loans, there is a common issue where users change their payment methods to ones not under their names. To address this problem, a demo code has been created to help these fintech companies determine if users have changed their payment methods.

### Steps:
**two tables has been created**

- The first table, named payout, retrieves the users who will receive money this month.

- The second table, named payout_updated, contains information about users who received money in the previous month.

**Query Explanation**

The query aims to compare user payment methods and the dates associated with them between two tables: payout (for the current month) and payout_updated (for the previous month). It identifies if users have changed their payment method and checks if the date difference between the two records is significant.

**1. Common Table Expressions (CTEs)**
- RankedPayout: Selects records from the payout table, assigning a row number rn for each ID, which is useful for ensuring correct row matching in the join.

- RankedPayoutUpdated: Selects records from the payout_updated table, also assigning a row number rn for each ID.

These CTEs help to create a consistent view of both tables with ranking to ensure correct comparisons.

**2. Main Query**

- SELECT Clause: Chooses the relevant columns from both CTEs. It calculates the Date_Difference between Original_Date and Updated_Date in days using julianday.
  
- CASE Statement:
  
a. WHEN julianday(rp.Original_Date) - julianday(rpu.Updated_Date) < 180 THEN 'Need Confirmation call':

If the date difference is less than 180 days, it indicates that a confirmation call might be needed.

b. WHEN julianday(rp.Original_Date) - julianday(rpu.Updated_Date) > 180 THEN 'More than 6 months':

If the date difference is more than 180 days, it indicates that the dates are too far apart.

c. WHEN rp.Original_Method = rpu.Updated_Method THEN 'Match':

If the payment methods are the same, it’s a match.

d. WHEN Updated_Date = 'New_joiner' THEN 'New_joiner':

This condition checks if the Updated_Date is marked as 'New_joiner', which should be handled as a special case.

**3.Join Condition**

-INNER JOIN: Joins RankedPayout and RankedPayoutUpdated on ID and ensures that the rows with the same rn are compared.

**4. WHERE Clause** 

-Ensures that only rows with the same rn (row number) in both CTEs are considered.

## Table Creation
**Create Pyout Table:**
```sql
-- Create the table with AUTOINCREMENT for ID
CREATE TABLE payout (
    ID INTEGER PRIMARY KEY AUTOINCREMENT,
    Method VARCHAR(50),
    NationalID TEXT,
    PhoneNumber TEXT
);
INSERT INTO payout (Method, NationalID, PhoneNumber)
VALUES 
('bank', '123456789012', '1234567890'),
('digital wallet', '987654321098', '2345678901'),
('prepaid card', '564738291046', '3456789012'),
('bank', '192837465091', '4567890123'),
('digital wallet', '564738291047', '5678901234'),
('prepaid card', '102938475610', '6789012345'),
('bank', '564738291048', '7890123456'),
('digital wallet', '987654321099', '8901234567'),
('prepaid card', '192837465092', '9012345678'),
('bank', '102938475611', '0123456789'),
('digital wallet', '564738291049', '1234567891'),
('prepaid card', '987654321100', '2345678902'),
('bank', '192837465093', '3456789013'),
('digital wallet', '102938475612', '4567890124'),
('prepaid card', '564738291050', '5678901235');

ALTER TABLE payout
ADD COLUMN date DATE;

UPDATE payout
SET date = '2024-08-15';
```
**Create Pyout_updated Table:**
```sql
CREATE TABLE IF NOT EXISTS payout_updated (
    ID INTEGER PRIMARY KEY,
    Method VARCHAR(50),
    NationalID TEXT,
    PhoneNumber TEXT
);
INSERT OR REPLACE INTO payout_updated (ID, Method, NationalID, PhoneNumber)
SELECT ID, 'digital wallet', NationalID, PhoneNumber
FROM payout
WHERE Method = 'bank';

INSERT OR REPLACE INTO payout_updated (ID, Method, NationalID, PhoneNumber)
SELECT ID, 'prepaid card', NationalID, PhoneNumber
FROM payout
WHERE Method = 'digital wallet';

INSERT OR REPLACE INTO payout_updated (ID, Method, NationalID, PhoneNumber)
SELECT ID, 'prepaid card', NationalID, PhoneNumber
FROM payout
WHERE Method = 'prepaid card';

UPDATE payout_updated
SET payout_date = CASE
    WHEN ID = 1 THEN '2023-08-15'
    WHEN ID = 2 THEN '2023-08-15'
    WHEN ID = 3 THEN '2023-07-15'
    WHEN ID = 4 THEN '2024-01-15'
    WHEN ID = 5 THEN '2024-02-15'
    WHEN ID = 6 THEN '2024-03-15'
    WHEN ID = 7 THEN '2024-04-15'
    WHEN ID = 8 THEN '2024-05-15'
    WHEN ID = 9 THEN '2024-06-15'
    WHEN ID = 10 THEN '2024-07-15'
    ELSE 'New_joiner'
```

**Joining the two tables**

```sql
WITH RankedPayout AS (
    SELECT 
        ID, 
        Method AS Original_Method, 
        NationalID,
        PhoneNumber,
        date AS Original_Date,
        ROW_NUMBER() OVER (PARTITION BY ID ORDER BY ID) AS rn
    FROM 
        payout
),
RankedPayoutUpdated AS (
    SELECT 
        ID, 
        Method AS Updated_Method, 
        NationalID,
        PhoneNumber,
        payout_date AS Updated_Date,
        ROW_NUMBER() OVER (PARTITION BY ID ORDER BY ID) AS rn
    FROM 
        payout_updated
)
SELECT 
    rp.ID,
    rp.Original_Method,
    rpu.Updated_Method,
    rp.NationalID,
    rp.PhoneNumber,
    rp.Original_Date,
    rpu.Updated_Date,
   julianday(rp.Original_Date)- julianday(rpu.Updated_Date)AS Date_Difference,
    CASE 
        WHEN julianday(rp.Original_Date)- julianday(rpu.Updated_Date) < 180 THEN 'Need Confirmation call'
        WHEN julianday(rp.Original_Date)- julianday(rpu.Updated_Date) > 180 Then 'More than 6 months'
        WHEN rp.Original_Method = rpu.Updated_Method THEN 'Match'
        WHEN Updated_Date='New_joiner' Then 'New_joiner'
    END AS Method_Comparison
FROM 
    RankedPayout rp
INNER JOIN 
    RankedPayoutUpdated rpu
ON 
    rp.ID = rpu.ID
WHERE 
    rp.rn = rpu.rn;
```
## Summary:
- RankedPayout and RankedPayoutUpdated create ordered views of the payout and payout_updated tables, respectively.
- Main Query compares methods and dates, and categorizes the results based on the date difference and method match.
-  CASE conditions are evaluated to determine if a confirmation call is needed or if there is a mismatch, with a special check for 'New_joiner'.
