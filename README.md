# Amazon Customer Reviews – AI Data Engineering Assignment

## Overview

This project demonstrates an end-to-end **data engineering and data preparation pipeline** built using **PySpark**. The goal is to ingest Amazon Customer Reviews data, perform data cleaning and quality checks, apply transformations, generate a curated training dataset, and document data quality issues and labeling logic.

The notebook `ai_de_assignment.ipynb` contains the complete implementation.

---

## Objectives

* Ingest raw Amazon Customer Reviews data
* Identify and handle common data quality issues
* Clean and standardize columns
* Apply business logic for aggregation and handle deduplication.
* Generate a final **training dataset** suitable for analytics or ML use cases
* Document assumptions and transformation logic.

---

## Technology Stack

* **Apache Spark (PySpark)**
* **Power BI**

---

## Input Data

The dataset contains Amazon product reviews with fields such as:

* `product_id`
* `product_name`
* `category`
* `discounted_price`
* `actual_price`
* `discount_percentage`
* `rating`
* `rating_count`
* `about_product`
* `user_id`
* `user_name`
* `review_id`
* `review_title`
* `review_content`
* `img_link`
* `product_link`

---

## Data Quality Issues Identified

The following data quality issues were observed and handled:

1. **Null Values**

   * Two records have null values in the Rating Count column

2. **Duplicate Observation**

   * Multiple rows per `product_id`
   * Some product IDs have multiple rows for the actual price and discounted price. These should either be summed or properly aggregated before calculating the discounted amount. Ex:B096MSW6CT ,B09MT84WV5
   * Some product IDs have multiple rows for the rating count. These should either be summed or the maximum value should be taken per product. ex:B00NH11KIK
   * In some cases, the review ID and user ID are swapped. We need to identify distinct user IDs or review IDs per product.
   * Some products have more than one image link, and review content may contain image or product links, which causes duplicate records.
   * Drive KPI from Product id. in some case different product id contain same product name.

3. **Invalid Characters**

   * product_name,about_product,review_content,user_name,review_title strings containing special characters '[^a-zA-Z0-9\s]'
   * review_content strings conating link for web hence need to remove HTML Tag.
   * use lower case for product_name,about_product,review_content,category,user_name,review_title,product_link strings to avoid incorrect filter.
   * during convertion,`ratings` column was found to contain the | character, which has been handled.
      
---

## Data Cleaning & Transformation Logic

### 1. Null Handling.

   * Two records have null values in the 'Rating Count' column

### 2. Datatype Convertion

* Remove unwanted characters  like ₹| and % and and try casting safely for column discounted_price,actual_price,discount_percentage,rating_count.
* during convertion,rating column was found to contain the | character, which has been handled.

### 2. Price Handling

* Aggregated `actual_price` and discounted price per product where duplicates exist
* Calculated `discount_amount`:

  ```
  discount_amount = actual_price - discount_price
  ```

### 3. Rating Aggregation

* Aggregated ratings at product level
* `rating_count`& `ratings` :  **max value** used to select rating count.


### 4. Category Cleaning

* Extracted parent category before delimiter (`|`)
* Example:

  ```
  home&kitchen|craftmaterials|scrapbooking|tape
  → home&kitchen
  ```

---

## Data Annotation Logic 

1. **Rating Rounding**

   * The original rating value is rounded to two decimal places.
   * This rounded value is stored temporarily to ensure consistent and accurate comparisons when applying sentiment rules.

2. **Sentiment Assignment**

   * Sentiment is derived based on the rounded rating value using the following rules:
   * Negative: If the rounded rating is less than 3
   * Neutral: If the rounded rating is greater than or equal to 3 and less than 4
   * Positive: If the rounded rating is greater than or equal to 4.
  

3. **Cleanup of Temporary Columns**

   * After the sentiment value is assigned, the temporary rounded rating column is removed.
   * This keeps the final dataset clean and avoids storing intermediate calculation fields.
      


---

## Output

### Final Dataset

* **File:** `training_dataset.csv`

### Notebook

* **File:** `ai_de_assignment.ipynb`
* Contains ingestion, transformation, and validation logic

### Power BI 

**Graph representation of KPI List**
* Top 10 Products by Popularity.
* Revenue Potential vs Discount %.
* % Positive Reviews per Category
* product Count per Sentiment

