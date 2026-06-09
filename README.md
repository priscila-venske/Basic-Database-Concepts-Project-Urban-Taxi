# 💻 Basic Database Concepts Project – Urban Taxi
 
## Objective
 
Perform investigation and validation activities using command-line tools and SQL queries to analyze application logs, identify system issues, and validate business data stored in a relational database for an urban taxi application.
 
---
 
## Scope of Testing
 
- Log analysis and filtering
- Error investigation and incident isolation
- File and directory management in Linux environments
- Database querying and data validation
- Business rule verification using SQL
- Weather and trip data analysis
- Data aggregation and reporting
---
 
## Tools Used
 
- **Cygwin** – Linux terminal emulator for command-line operations
- **PostgreSQL** – Relational database management system
- **SQL** – Query language for data extraction and validation
---
 
## Activities Performed
 
- Accessed and navigated remote Linux servers
- Analyzed application logs using command-line utilities
- Filtered records based on IP addresses, timestamps, and HTTP status codes
- Created and organized directories and log files for incident investigation
- Isolated and categorized server errors (400 and 500)
- Connected to a PostgreSQL database environment
- Executed SQL queries to validate operational and business data
- Performed data aggregation, filtering, grouping, and sorting operations
- Validated relationships between database tables
- Investigated data inconsistencies reported by stakeholders
- Generated datasets to support debugging and defect analysis
---
 
## Features Tested
 
### 🖥️ Log Analysis & Incident Investigation
 
- Identification of requests from specific IP ranges
- Filtering of application logs by date and error type
- Extraction and categorization of HTTP errors
- Validation of server-side events
### 📂 Linux File Management
 
- Directory creation and organization
- Log extraction and storage
- Error segregation into dedicated files
- Command-line operations for troubleshooting
### 🗄️ Database Validation
 
- Verification of taxi fleet availability
- Validation of company vehicle distribution
- Analysis of ride volume by company
- Data consistency verification across related tables
### 🌦️ Business Rule Validation
 
- Classification of weather conditions using SQL `CASE` expressions
- Validation of ride and weather datasets
- Verification of reporting logic used by business applications
---
 
## Types of Testing Executed
 
- Database Testing
- Data Validation Testing
- Backend Testing
- Log Analysis
- Incident Investigation
- Business Logic Validation
---
 
## Database Schema
 
| Table | Key Columns |
|---|---|
| `neighborhoods` | `neighborhood_id`, `name` |
| `cabs` | `cab_id`, `vehicle_id`, `company_name` |
| `trips` | `trip_id`, `cab_id`, `start_ts`, `end_ts`, `duration_seconds`, `distance_miles`, `pickup_location_id`, `dropoff_location_id` |
| `weather_records` | `record_id`, `ts`, `temperature`, `description` |
 
> `trips` and `weather_records` have no direct FK — they are joined via `trips.start_ts = weather_records.ts`.
 
---
 
## Test Cases & Results
 
### Part 1 – Console & Server Logs
 
#### Task 1 – Filter Logs by IP Address
 
**Objective:** Identify all requests from IP addresses starting with `233.201.` across all log files in `logs/2019/12`.
 
```bash
cd ~/logs/2019/12
grep -R '^233.201'
```
 
**Result:**
 
```
apache_2019-12-18.txt:233.201.188.154 - - [18/12/2019:21:46:01 +0000] "DELETE /events HTTP/1.1" 403 3971
apache_2019-12-21.txt:233.201.182.9 - - [21/12/2019:21:56:20 +0000] "PATCH /users HTTP/1.1" 400 4118
```
 
---
 
#### Task 2 – Isolate and Classify Bug Logs
 
**Objective:** A bug was active on `30/12/2019` generating HTTP `400` and `500` errors. Isolate and organize those logs by error type.
 
**Directory structure created:**
 
```
~/bug1/
├── main.txt       ← all 400 and 500 errors from 30/12/2019
└── events/
    ├── 400.txt    ← 179 lines
    └── 500.txt    ← 160 lines
```
 
**Filter all errors into `main.txt`:**
 
```bash
grep '[45]00' ~/logs/2019/12/apache_2019-12-30.txt > ~/bug1/main.txt
```
 
**Split by error type:**
 
```bash
grep '400' ~/bug1/main.txt > ~/bug1/events/400.txt
grep '500' ~/bug1/main.txt > ~/bug1/events/500.txt
```
 
**Sample from `400.txt` (179 lines total):**
 
```
# First 3 lines
80.57.170.51 - - [30/12/2019:21:35:12 +0000] "DELETE /users HTTP/1.1" 400 3623
204.235.176.118 - - [30/12/2019:21:35:13 +0000] "POST /users HTTP/1.1" 400 4704
82.95.203.67 - - [30/12/2019:21:35:19 +0000] "DELETE /lists HTTP/1.1" 400 3737
 
# Last 3 lines
203.106.235.105 - - [30/12/2019:22:12:38 +0000] "DELETE /events HTTP/1.1" 400 4158
18.211.28.150 - - [30/12/2019:22:12:40 +0000] "DELETE /collectors HTTP/1.1" 400 2212
229.16.123.45 - - [30/12/2019:22:12:54 +0000] "GET /auth HTTP/1.1" 400 2397
```
 
**Sample from `500.txt` (160 lines total):**
 
```
# First 3 lines
64.250.112.189 - - [30/12/2019:21:35:13 +0000] "PUT /parsers HTTP/1.1" 500 4639
193.253.101.180 - - [30/12/2019:21:35:31 +0000] "PATCH /alerts HTTP/1.1" 500 2944
197.106.117.194 - - [30/12/2019:21:35:31 +0000] "PATCH /parsers HTTP/1.1" 500 3519
 
# Last 3 lines
207.6.210.203 - - [30/12/2019:22:12:37 +0000] "PATCH /events HTTP/1.1" 500 4298
33.13.118.148 - - [30/12/2019:22:12:38 +0000] "PUT /alerts HTTP/1.1" 500 4711
107.188.33.199 - - [30/12/2019:22:12:59 +0000] "POST /parsers HTTP/1.1" 500 2833
```
 
---
 
### Part 2 – SQL Database Queries
 
**Connection:** `psql -U morty -d chicago_taxi`
 
---
 
#### Task 1 – Total Number of Taxis
 
**Objective:** Verify the actual taxi fleet size. The planned capacity was 10,550 vehicles.
 
```sql
SELECT COUNT(*) AS total_cars
FROM cabs;
```
 
**Result:**
 
```
 total_cars
------------
      5529
(1 row)
```
 
> ⚠️ Only **5,529** of the planned **10,550 vehicles** were available — approximately **52% of expected capacity**, directly contributing to user complaints about car shortages.
 
---
 
#### Task 2 – Companies with Fewer than 100 Cars
 
**Objective:** Identify which companies failed to provide adequate fleet coverage.
 
```sql
SELECT COUNT(*) AS cnt, company_name
FROM cabs
GROUP BY company_name
HAVING COUNT(*) < 100
ORDER BY cnt DESC;
```
 
**Result (51 rows — sample):**
 
```
 cnt |              company_name
-----+----------------------------------------------
  97 | Nova Taxi Affiliation Llc
  89 | Patriot Taxi Dba Peace Taxi Associat
  85 | Blue Diamond
  81 | Checker Taxi Affiliation
  80 | Chicago Medallion Management
  69 | Chicago Independents
  67 | 24 Seven Taxi
  60 | Checker Taxi
  55 | American United
  53 | Chicago Medallion Leasing INC
  49 | Top Cab Affiliation
  48 | KOAM Taxi Association
  38 | Chicago Taxicab
  34 | Norshore Cab
  20 | Gold Coast Taxi
   1 | 1085 - 72312 N and W Cab Co
   1 | 3591 - 63480 Chuks Cab
   ...
(51 rows)
```
 
---
 
#### Task 3 – Weather Condition Classification
 
**Objective:** Classify each hour of `2017-11-05` as `Good` or `Bad` to support ride cost coefficient validation. Conditions containing `rain` or `storm` are classified as `Bad`.
 
```sql
SELECT
    ts,
    CASE
        WHEN description LIKE '%rain%' OR description LIKE '%storm%' THEN 'Bad'
        ELSE 'Good'
    END AS weather_conditions
FROM weather_records
WHERE ts BETWEEN '2017-11-05 00:00:00' AND '2017-11-05 23:59:59';
```
 
**Result:**
 
```
        ts          | weather_conditions
---------------------+--------------------
 2017-11-05 00:00:00 | Good
 2017-11-05 01:00:00 | Bad
 2017-11-05 02:00:00 | Good
 2017-11-05 03:00:00 | Good
 2017-11-05 04:00:00 | Bad
 2017-11-05 05:00:00 | Bad
 2017-11-05 06:00:00 | Good
 2017-11-05 07:00:00 | Good
 2017-11-05 08:00:00 | Good
 2017-11-05 09:00:00 | Good
 2017-11-05 10:00:00 | Good
 2017-11-05 11:00:00 | Good
 2017-11-05 12:00:00 | Good
 2017-11-05 13:00:00 | Good
 2017-11-05 14:00:00 | Bad
 2017-11-05 15:00:00 | Good
 2017-11-05 16:00:00 | Bad
 2017-11-05 17:00:00 | Good
 2017-11-05 18:00:00 | Bad
 2017-11-05 19:00:00 | Bad
 2017-11-05 20:00:00 | Bad
```
 
---
 
#### Task 4 – Trip Count per Company (Nov 15–16, 2017)
 
**Objective:** After a software update, companies reported revenue discrepancies. Validate trip volume per company on the affected dates to identify potential data inconsistencies.
 
```sql
SELECT
    cabs.company_name,
    COUNT(trips.trip_id) AS trips_amount
FROM trips
JOIN cabs ON trips.cab_id = cabs.cab_id
WHERE trips.start_ts BETWEEN '2017-11-15 00:00:00' AND '2017-11-16 23:59:59'
GROUP BY cabs.company_name
ORDER BY trips_amount DESC;
```
 
**Result (64 rows — sample):**
 
```
              company_name               | trips_amount
-----------------------------------------+--------------
 Flash Cab                               |        19558
 Taxi Affiliation Services               |        11422
 Medallion Leasin                        |        10367
 Yellow Cab                              |         9888
 Taxi Affiliation Service Yellow         |         9299
 Chicago Carriage Cab Corp               |         9181
 City Service                            |         8448
 Sun Taxi                                |         7701
 Star North Management LLC               |         7455
 Blue Ribbon Taxi Association Inc.       |         5953
 ...
 3556 - 36214 RC Andrews Cab             |            2
(64 rows)
```
 
---
 
## Artifacts Produced
 
| Artifact | Description |
|---|---|
| ✔️ SQL Queries | [Database Tasks](https://docs.google.com/document/d/1BVC68LdmsQyUm7KKB5O6zx_HCmU1ltMkvZXpqOr8xCA/edit?usp=sharing) |
| ✔️ Log Analysis Reports | [Command-Line Tasks](https://docs.google.com/document/d/1V6jkRFYL9CC1MYAzfOqpn8GOuyCFdlU3ghKlU1wSS_Y/edit?usp=sharing) |
 
---
 
## Technologies and Concepts
 
| Category | Details |
|---|---|
| **CLI** | Linux Commands, Log Analysis, `grep`, file redirection |
| **Database** | PostgreSQL, SQL Queries, Joins, Aggregations, `CASE` Statements |
| **Concepts** | Data Filtering, Database Relationships, HTTP Status Codes |
 
---
 
## Skills Developed
 
- SQL Query Development
- Database Testing & Data Validation
- Linux Command Line
- Log Analysis & Root Cause Investigation
- Backend Validation
- Business Rule Testing
- Quality Assurance
---
 
## Author
 
Project developed as part of the **QA Engineering Bootcamp** at TripleTen.
