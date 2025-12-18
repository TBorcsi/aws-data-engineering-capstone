# Project Overview and Goal

The objective of this capstone project is to architect and implement a serverless data engineering solution on AWS. The project focuses on processing historical fisheries data provided by the "Sea Around Us" initiative.

## Key Objectives

* Set up a cloud-based development environment (Cloud9).
* Ingest raw data in CSV format.
* Optimize data storage:
  * Transform row-based CSV files into column-based Parquet format Data
* Cataloging:
  * Use AWS Glue Crawlers to automatically infer the schema and populate the AWS Glue Data Catalog.
* Data Analysis:
  * Perform SQL queries using Amazon Athena to derive insights from the dataset.

## Architecture

The solution utilizes the following AWS services:

* AWS Cloud9: An Integrated Development Environment (IDE) used to run Python scripts for data transformation.
* Amazon S3: Object storage used for storing raw data, optimized Parquet files, and Athena query results.
* AWS Glue: A fully managed ETL service. We used the
* Glue Crawler to traverse the S3 bucket and create a metadata table in the Data Catalog.
* Amazon Athena: A serverless query service used to analyze the data using standard SQL.

---

# TASK 1

## 4. Step 1: Environment Setup

4. Step 1: Environment Setup - I created an AWS Cloud9 environment (`CapstoneIDE`) within the provided VPC and Public Subnet.
<img width="602" height="339" alt="Picture1" src="https://github.com/user-attachments/assets/e16824f7-f6d0-47a5-858d-6b473902c84a" />

## 5. Amazon S3 Buckets

5. Additionally, I provisioned two S3 buckets:

1. Data Source Bucket: `data-source-12101` (for storing Parquet files)  
2. Query Results Bucket: `query-results-12102` (for Athena outputs)

<img width="602" height="339" alt="Picture2" src="https://github.com/user-attachments/assets/91f5e8fd-9d85-4b78-9386-80f716652d23" />

---

## 6–9. Download, Inspect, Convert, Upload (GLOBAL)

6. Download the three .csv source data files, run the following commands in the terminal of your AWS Cloud9 IDE:
<img width="602" height="339" alt="Picture3" src="https://github.com/user-attachments/assets/926edd96-b49a-4bbd-a5d1-b207af150e09" />

7. To observe the column header row and the first five rows of data in the SAU-GLOBAL-1-v48-0.csv file, run the following command

CSV to parquet I did it with the given code
<img width="602" height="339" alt="Picture4" src="https://github.com/user-attachments/assets/4797b609-c698-4ad1-809d-65795ad146bc" />

8-9. Upload the SAU-GLOBAL-1-v48-0.parquet file to the data-source bucket

```bash
aws s3 cp SAU-GLOBAL-1-v48-0.parquet s3://data-source-12101/
```

![TASK 1 – Cloud9 terminal + upload command (page 3)](images/page-03.png)

---

# TASK 2

## 10. Inspect (HighSeas CSV)

10. To observe the column header row and first few lines of data from the SAU-HighSeas-71-v48-0.csv file, use the head command.
<img width="602" height="339" alt="Picture5" src="https://github.com/user-attachments/assets/c224f89a-7710-42de-b418-8f8362ca4b83" />

## 11. Convert & Upload (HighSeas)

11. Convert the SAU-HighSeas-71-v48-0.csv file to Parquet format and upload it to the data-source bucket.

```bash
aws s3 cp SAU-HighSeas-71-v48-0.parquet s3://data-source-12101/
```

## 12. Glue Database + Crawler

12. Create an AWS Glue database and an AWS Glue crawler with the following settings:

<img width="602" height="339" alt="Picture6" src="https://github.com/user-attachments/assets/1e5c8263-71ea-44d9-ab9a-f62c4ca3155e" />

---

## 13–14. Run Crawler + Verify in Athena

13. Run the crawler to create a table that contains metadata in the AWS Glue database.  
Verify that the expected table is created.
<img width="602" height="339" alt="Picture7" src="https://github.com/user-attachments/assets/1da4d682-1740-4065-88eb-57d329c16e3e" />

14. To confirm that the table properly categorized the data, use Athena to run SQL queries against each column in the new table.

Important: Before you run the first query in Athena, configure the Athena Query Editor to output data to the query-results bucket.

<img width="602" height="339" alt="Picture8" src="https://github.com/user-attachments/assets/261ce9a3-6b41-4384-9a07-d7edc9f93237" />

---

## 15. Challenge Query Requirement

15. Now that your data table is defined, run queries to confirm that it provides useful results.

Challenge: Find the value in US dollars of all fish caught by the country Fiji from all high seas areas since 2001, organized by year. In your output results, name the US dollar value column ValueAllHighSeasCatch

<img width="602" height="339" alt="Picture9" src="https://github.com/user-attachments/assets/a32f5a1b-3bbc-444b-8b64-c45e4c9bf8b3" />
<img width="602" height="339" alt="Picture10" src="https://github.com/user-attachments/assets/ecc85bd5-c78e-49b2-a1da-35bda34e6081" />
---

## Challenge SQL

```sql
CREATE OR REPLACE VIEW challenge AS
SELECT
year,
CAST(CAST(SUM(landed_value) AS DOUBLE) AS DECIMAL(38,2)) AS
valueAllHighSeasCatch
FROM "fishdb"."data_source_xxxxx"
WHERE fishing_entity = 'Fiji'
AND year > 2001
AND area_name IS NOT NULL
GROUP BY year
ORDER BY year;
```

<img width="602" height="339" alt="Picture11" src="https://github.com/user-attachments/assets/3f797e59-f3a7-4010-98f8-c47a03353528" />
<img width="602" height="339" alt="Picture12" src="https://github.com/user-attachments/assets/54335aba-f13e-41ef-a4f4-49da045473ad" />

---

## Challenge Results Analysis

Challenge Results Analysis: The data shows a clear trend for Fiji between 2002 and 2011. Income started at $58 million in 2002 and grew to a high of nearly $90 million in 2007. After this peak, the earnings became unstable, dropping to $67 million in 2010. This confirms that while high seas fishing is very profitable, the income is not consistent and changes significantly every year.

---

# TASK 3

## 17. Fix Column Names + Convert EEZ to Parquet (pandas)

17. Use the Python data analyst library, which is called pandas, to fix the column names. In addition, convert the EEZ file to the Parquet format.

```bash
# Make a backup of the file before you modify it in place
cp SAU-EEZ-242-v48-0.csv SAU-EEZ-242-v48-0-old.csv

# Start the python interactive shell
python3
```

```python
import pandas as pd

# Load the backup version of the file
data_location = 'SAU-EEZ-242-v48-0-old.csv'

# Use Pandas to read the CSV into a dataframe
df = pd.read_csv(data_location)

# View the current column names
print(df.head(1))

# Change the names of the 'fish_name' and 'country' columns to match the column
# names where this data appears in the other data files already in your data-source
# bucket
df.rename(columns = {"fish_name": "common_name", "country": "fishing_entity"}, inplace
= True)

# Verify the column names have been changed
print(df.head(1))

# Write the changes to disk
df.to_csv('SAU-EEZ-242-v48-0.csv', header=True, index=False)
df.to_parquet('SAU-EEZ-242-v48-0.parquet')

exit()
```

<img width="602" height="339" alt="Picture13" src="https://github.com/user-attachments/assets/b65d9c1a-2e8f-4e74-a996-3e85d7ab5e6d" />
---

## 18. Upload EEZ Parquet to S3

18. Upload the new EEZ data file to the data-source bucket

```python
df.to_parquet('SAU-EEZ-242-v48-0.parquet')
exit()
```

```bash
aws s3 cp SAU-EEZ-242-v48-0.parquet s3://data-source-12101/
```

## 19. Re-run Glue Crawler

19. To update the table metadata with the additional columns that are now part of your dataset, run the AWS Glue crawler again.

I ran it again

## 20. Verify area_name

20. To verify the values in the area_name column, as you did before, use the following query:

<img width="534" height="300" alt="Picture14" src="https://github.com/user-attachments/assets/5e78c7b1-b261-44c8-a1fd-13f4c7f015ca" />

---

## 20. b (Fiji EEZ since 2001)

20. b To find the value in US dollars of all fish caught by Fiji from the Fiji EEZ since 2001, organized by year, use the following query:

This query looks at the value of fish caught specifically inside Fiji's own waters (EEZ) since 2001. The results show that the income changes a lot from year to year. It reached a high point of $86.8 million in 2007 but then dropped significantly to a low of $50.1 million in 2013. Since then, the numbers have improved slightly, stabilizing around $64.2 million in 2018.

<img width="602" height="339" alt="Picture15" src="https://github.com/user-attachments/assets/3ec7343c-653b-4274-ae9f-516e95762fd8" />

<img width="602" height="339" alt="Picture16" src="https://github.com/user-attachments/assets/ac5f7579-215b-4248-ac53-e1639bad9cdf" />

---

## 20/c (Open seas since 2001)

20/c To find the value in US dollars of all fish caught by Fiji from the open seas since 2001, organized by year, use the following query:

This query focuses on fishing in the "open seas" (international waters) by looking for records without a specific area name. The results show that income from this sector is quite stable, usually staying between $61 million and $74 million per year. The best years were 2004 and 2017, both earning over $73 million, while the lowest point was in 2013 with $61.9 million. This proves that international waters provide a steady and reliable source of income for Fiji.


<img width="602" height="339" alt="Picture17" src="https://github.com/user-attachments/assets/1f30c93f-04aa-468a-b4f9-bf2a4edc4860" />

<img width="602" height="339" alt="Picture18" src="https://github.com/user-attachments/assets/3ebe6bad-2550-4258-800d-ac424949d6c4" />

---

## 20/d (Fiji EEZ OR open seas since 2001)

20/d To find the value in US dollars of all fish caught by Fiji from either the Fiji EEZ or the open seas since 2001, organized by year, use the following query:

This query combines the value of fish caught in both Fiji's local waters and the open ocean to give a complete picture. The results show that the industry is very large, consistently earning over $112 million every year since 2001. The best year was 2007 with a total of $154.2 million, while the lowest point was in 2013 at $112 million. Since then, the numbers have recovered well, reaching $131.5 million by 2018.


<img width="602" height="339" alt="Picture19" src="https://github.com/user-attachments/assets/4e835c73-d51b-477f-8350-e0f6bb3494d8" />

<img width="602" height="339" alt="Picture20" src="https://github.com/user-attachments/assets/932cb995-7f13-47c7-949a-4c940fa6042c" />

---

## 21. Create a View in Athena

21. Create a view in Athena, which will be useful to review the data in the next section of this capstone.


<img width="602" height="339" alt="Picture21" src="https://github.com/user-attachments/assets/0040b457-6974-4b3f-9c54-91460b0e9097" />

## 22. Tonnes of mackerel caught by year by country

22. To view the following data Tonnes of mackerel caught by year by country  
Return to the Athena query editor, and run the following SQL query to identify the countries with the highest mackerel catch each year.


<img width="602" height="339" alt="Picture22" src="https://github.com/user-attachments/assets/d1254261-27b7-4634-a06b-31189f78a971" />
<img width="602" height="339" alt="Picture23" src="https://github.com/user-attachments/assets/4c6e5965-4cb6-490a-8586-1c5d113219cd" />

<img width="602" height="339" alt="Picture24" src="https://github.com/user-attachments/assets/2ff723c9-e9d8-4a07-8e9c-3dd0008ff4c2" />

<img width="602" height="339" alt="Picture25" src="https://github.com/user-attachments/assets/4a0bdd99-7cb4-4eb4-b5cb-3a06d78783f6" />

<img width="602" height="339" alt="Picture26" src="https://github.com/user-attachments/assets/863e4e15-faae-4264-9479-f26f4ed5f7dd" />
---

## Mackerel Results Analysis

The results show that two countries dominate mackerel fishing: the USA and Fiji. They catch far more fish than any other nation. The USA was the leader in 2015, 2016, and 2018, reaching a high of 859 tonnes in 2015. However, Fiji took first place in 2017 with 649 tonnes. There is a huge gap between these two leaders and other countries like Indonesia or Vietnam, which catch much less.



---

## 23. Country-specific mackerel (e.g China)

23. To view the MackerelsCatch for a particular country, e.g China, run the following query.

The data shows that China fishes for mackerel in two main areas: the high seas of the "Pacific, Western Central" region and inside Fiji's waters. Their best year was 2017. In that year, they caught 96.4 tonnes in the Pacific region and 77.1 tonnes in Fiji's waters.


<img width="602" height="339" alt="Picture27" src="https://github.com/user-attachments/assets/2fc568a8-43f4-4fc8-9257-9c2730f42cd1" />

<img width="602" height="339" alt="Picture28" src="https://github.com/user-attachments/assets/039cb1fd-b8dd-4464-a775-433c494c04bd" />
---

# My 3 queries

## Query 1

```sql
SELECT common_name,
 CAST(SUM(landed_value) AS DECIMAL(38,2)) AS Total_Value_USD
FROM "fishdb"."data_source_12101"
WHERE common_name IS NOT NULL
GROUP BY common_name
ORDER BY Total_Value_USD DESC
LIMIT 5;
```

<img width="602" height="339" alt="Picture29" src="https://github.com/user-attachments/assets/8ff8e3e4-87b5-4a9b-8c08-41b9757baa9b" />

I wanted to find out which fish species makes the most money globally. The results show that Skipjack tuna is the clear winner. It generated a total value of over 2 trillion, which is much higher than any other fish. This proves that Skipjack tuna is the most important target for the industrial fishing industry worldwide.


---

## Query 2

```sql
SELECT reporting_status,
 CAST(SUM(tonnes) AS DECIMAL(38,2)) AS Total_Tonnes,
 CAST(SUM(landed_value) AS DECIMAL(38,2)) AS Total_Value_USD
FROM "fishdb"."data_source_12101"
WHERE reporting_status IS NOT NULL
GROUP BY reporting_status
ORDER BY Total_Tonnes DESC;
```
<img width="602" height="339" alt="Picture30" src="https://github.com/user-attachments/assets/94dddc84-f783-4f37-af5c-740b3d81e242" />

I wanted to check how much of the global catch is officially recorded versus how much is unreported. The results are surprising. While there are 426 million tonnes of officially reported fish, a huge amount, 200 million tonnes, remains unreported. This means that roughly one-third of all fishing activity happens illegally. That makes more difficult to get to know the ocean and protect it.



---

## Query 3

```sql
SELECT catch_type,
 CAST(SUM(tonnes) AS DECIMAL(38,2)) AS Total_Weight_Tonnes
FROM "fishdb"."data_source_12101"
WHERE catch_type IS NOT NULL
GROUP BY catch_type
ORDER BY Total_Weight_Tonnes DESC;
```

<img width="602" height="339" alt="Picture31" src="https://github.com/user-attachments/assets/d9757f6b-b6b0-4afc-935b-a3840bc8b43d" />

I wanted to see how efficient fishing really is by comparing "Landings" (fish that are kept and sold) versus "Discards" (fish that are thrown back into the sea). The results show a significant amount of waste. While 554 million tonnes of fish were successfully landed, over 72 million tonnes were discarded. Ideally, we want the "Discards" number to be as low as possible.

