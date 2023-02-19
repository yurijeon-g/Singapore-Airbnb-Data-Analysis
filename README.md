# :house: Singapore Airbnb Case Study :house:

## Introduction
Airbnb is a popular platform that offers various types of accommodations for people all around the world. It provides an alternative to traditional hotel stays by allowing individuals to rent out their homes, apartments or rooms to travelers for a short period. This Airbnb dataset is a collection of data from Airbnb listings in Singapore and was last updated in December 2022. The source of this dataset is [InsideAirbnb](http://insideairbnb.com/get-the-data) . This dataset contains information such as location, price, availability, and number of reviews. This project will focus on analyzing the dataset using PostgreSQL queries to explore the pricing, hosts, and locations of Airbnb properties in Singapore.

## Data Description
The dataset contains 18,358 rows and 17 columns. The columns are as follows: id, name, host_id, host_name, neighbourhood_group, neighbourhood, latitude, longitude, room_type, price, minimum_nights, number_of_reviews, last_review, reviews_per_month, calculated_host_listings_count, availability_365, number_of_reviews_ltm, and license. A data dictionary is provided below.
| Field | Type | Calculated | Description |
| --- | --- | --- | --- |
| id | integer |  | Airbnb's unique identifier for the listing |
| name | string |  |  |
| host_id | integer |  |  |
| host_name | string |  |  |
| neighbourhood_group | text | Yes | The neighbourhood group as geocoded using the latitude and longitude against neighborhoods as defined by open or public digital shapefiles. |
| neighbourhood | text | Yes | The neighbourhood as geocoded using the latitude and longitude against neighborhoods as defined by open or public digital shapefiles. |
| latitude | numeric |  | Uses the World Geodetic System (WGS84) projection for latitude and longitude. |
| longitude |  |  | Uses the World Geodetic System (WGS84) projection for latitude and longitude. |
| room_type | string |  |  |
| price | currency |  | daily price in local currency. Note, $ sign may be used despite locale |
| minimum_nights | integer |  | minimum number of night stay for the listing (calendar rules may be different) |
| number_of_reviews | integer |  | The number of reviews the listing has |
| last_review | date |  | The date of the last/newest review |
| calculated_host_listings_count | integer | Yes | The number of listings the host has in the current scrape, in the city/region geography. |
| availability_365 | integer | Yes | avaliability_x. The availability of the listing x days in the future as determined by the calendar. Note a listing may be available because it has been booked by a guest or blocked by the host. |
| number_of_reviews_ltm | integer | Yes | The number of reviews the listing has (in the last 12 months) |
| license | string |  |  |

## Methodology
The analysis of the dataset was conducted using PostgreSQL. The queries were written to explore the dataset and answer questions related to the pricing, hosts, and locations of Airbnb properties in Singapore. The analysis focused on the following questions:

- What are the most expensive neighborhoods in Singapore?
- How are prices distributed across the listings and the percentage differences in prices?
- How does the feature of review count affect pricing of Airbnbs?
- How many Airbnbs are operating against Singapore regulations and how are they priced? How popular are they?

For aggregating data, while averages are usually used, medians were used for this case study due to the presence of extreme outliers in the data in terms of pricing.

## Basic Data Exploration
### 1. Identifying Columns with NULLs

 ``` sql
 SELECT
  COUNT(CASE WHEN id IS NULL THEN 1 END) AS id,
  COUNT(CASE WHEN name IS NULL THEN 1 END) AS name,
  COUNT(CASE WHEN host_id IS NULL THEN 1 END) AS host_id,
  COUNT(CASE WHEN host_name IS NULL THEN 1 END) AS host_name,
  COUNT(CASE WHEN neighbourhood_group IS NULL THEN 1 END) AS neighbourhood_group,
  COUNT(CASE WHEN neighbourhood IS NULL THEN 1 END) AS neighbourhood,
  COUNT(CASE WHEN latitude IS NULL THEN 1 END) AS latitude,
  COUNT(CASE WHEN longitude IS NULL THEN 1 END) AS longitude,
  COUNT(CASE WHEN room_type IS NULL THEN 1 END) AS room_type,
  COUNT(CASE WHEN price IS NULL THEN 1 END) AS price,
  COUNT(CASE WHEN minimum_nights IS NULL THEN 1 END) AS minimum_nights,
  COUNT(CASE WHEN number_of_reviews IS NULL THEN 1 END) AS number_of_reviews,
  COUNT(CASE WHEN last_review IS NULL THEN 1 END) AS last_review,
  COUNT(CASE WHEN reviews_per_month IS NULL THEN 1 END) AS reviews_per_month,
  COUNT(CASE WHEN calculated_host_listings_count IS NULL THEN 1 END) AS calculated_host_listings_count,
  COUNT(CASE WHEN availability_365 IS NULL THEN 1 END) AS availability_365
FROM airbnb;
  ```
 **Table results:**
| id | name | host_id | host_name | neighbourhood_group | neighbourhood | latitude | longitude | room_type | price | minimum_nights | number_of_reviews | last_review | reviews_per_month | calculated_host_listings_count | availability_365 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1291 | 1291 | 0 | 0 |

Only last review and reviews per month have nulls. The null count is fairly significant, both having counts of 1291. Thus, we’re unlikely to use those columns in our analysis.


### 2. Listings Count by Neighbourhood Group

```sql
SELECT neighbourhood_group, COUNT(*) AS num_listings
FROM airbnb
GROUP BY neighbourhood_group;
```
 **Table results:**
 
| neighbourhood_group | num_listings |
| --- | --- |
| West Region | 214 |
| Central Region | 2476 |
| East Region | 163 |
| North Region | 82 |
| North-East Region | 102 |
 
As expected the region with the most number of listings is the central region of Singapore.

 ### 3. Popularity per Neighbourhood
Using number of reviews as a popularity and/or occupancy measure, we will explore which specific neighborhood are the most popular.
 ```sql
 SELECT neighbourhood, neighbourhood_group, SUM(number_of_reviews) AS total_reviews
FROM airbnb
GROUP BY neighbourhood,neighbourhood_group
ORDER BY total_reviews DESC;
```
**Table results:**

| neighbourhood | neighbourhood_group | total_reviews |
| --- | --- | --- |
| Kallang | Central Region | 4472 |
| Rochor | Central Region | 3954 |
| Outram | Central Region | 3921 |
| Downtown Core | Central Region | 3224 |
| Geylang | Central Region | 2086 |
| Bedok | East Region | 1624 |
| River Valley | Central Region | 1431 |
| Bukit Merah | Central Region | 1411 |
| Novena | Central Region | 1154 |

The top 10 neighborhoods are located in the central region however there is one outlier: Bedok. Bedok is in the east region.

### 4. Count of Listings per Neighbourhood
Is Bedok's popularity despite not being positioned in the Central region linked to the number of listings the particular area has.
```sql
SELECT neighbourhood, neighbourhood_group, count(*)
FROM airbnb

GROUP BY neighbourhood, neighbourhood_group
ORDER BY 3 DESC
LIMIT 10
```
**Table results:**
| neighbourhood | neighbourhood_group | count |
| --- | --- | --- |
| Kallang | Central Region | 353 |
| Downtown Core | Central Region | 325 |
| Outram | Central Region | 255 |
| Rochor | Central Region | 214 |
| Novena | Central Region | 182 |
| Queenstown | Central Region | 176 |
| Bukit Merah | Central Region | 163 |
| River Valley | Central Region | 157 |
| Geylang | Central Region | 145 |
| Bedok | East Region | 120 |

This explains Bedok’s popularity as it is in the top 10 of listings count by neighbourhood.

### 5. Median Price per Neighbourhood Group
A query to get Month to Date of total revenue in each product categories of past 3 months (current date 15 April 2022), breakdown by date
```sql
SELECT neighbourhood_group, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price
FROM airbnb
GROUP BY neighbourhood_group
ORDER BY 2 DESC;
```

**Table results:**

| neighbourhood_group | median_price |
| --- | --- |
| Central Region | 175 |
| West Region | 129 |
| East Region | 120 |
| North-East Region | 93 |
| North Region | 88 |

Central has the highest median price and East/West regions have similar median prices.

** Note **: 
- From here onwards, I will no longer display the query results as they tend to be longer and take up too much display space.

### 6. Most Expensive Neighbourhoods with Listings Count

```sql
SELECT neighbourhood, 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price,
  COUNT(*) AS count_listings
FROM airbnb
GROUP BY neighbourhood
ORDER BY median_price DESC;

```
Without looking at the listings count, it shows many neighbourhoods that are not in the central region. This may be because of outliers. Thus, I will output also the number of listings per neighbourhood The most expensive neighbourhood Tuas has only one listing thus the data is an outlier and not a median value. When removing neighbourhoods of only 1 count, you can see that expensive listings are mostly in the central location.


### 7. Percentage Difference in Pricing (Most Expensive Listings)

```sql
WITH ranked_listings AS (
  SELECT
    id, name,
    price,
    LAG(price) OVER (ORDER BY price DESC) AS lower_price
  FROM airbnb
)
SELECT
  id,
  name,
  price,
  ROUND((price - lower_price) / lower_price * 100, 2) AS percentage_difference
FROM ranked_listings
ORDER BY price DESC
LIMIT 10;
```
The top ranked listing, "Penthouse with unblocked views D14 near MRT," is significantly higher in price compared to the rest of the listings, with a price of 135,140. The second ranked listing, "Beary's 3 Bed Private Room," has a price of 109,999 and a percentage difference of -18.6, indicating that it is almost 19% lower in price compared to the lower priced listing. The percentage difference spikes precipitously by the 4th and 5th listing and the differences becomes significantly lower after those listings. This suggests that the top 4 listings are extreme outliers in terms of pricing. 

### 8. Most Common Descriptors in Listing Title

The first query is selecting the neighborhood and neighborhood group columns from the "airbnb" table and counting the number of rows for each unique combination of neighborhood and neighborhood group. The results are then ordered by the count of rows in descending order.

The second query is splitting the "name" column of the "airbnb" table into individual words using the regular expression function "regexp_split_to_table". It then counts the frequency of each unique word and orders the results by the word count in descending order. The subquery is aliased as "words" and is then grouped by the "word" column. Overall, the query is trying to find the most common words used in the Airbnb listings' names.

```sql
SELECT neighbourhood, neighbourhood_group, count(*)
FROM airbnb

GROUP BY neighbourhood, neighbourhood_group
ORDER BY 3 DESCSELECT word, count(*) as word_count
FROM (
  SELECT regexp_split_to_table(name, E'\\s+') as word
  FROM airbnb
) AS words
GROUP BY word
ORDER BY word_count DESC;
```
MRT is the most common descriptor for titles. Promotion of proximity to public transport seems to be a common titling strategy.



### 9. Relationship between Review Count and Pricing

The query categorizes the number of reviews for each Airbnb listing as 'None', 'Few', 'Some', 'Fairly Alot', 'A Lot', or 'Extremely Alot', based on different ranges of number_of_reviews. It then calculates the median price for each category of reviews using the PERCENTILE_CONT function.

```sql
SELECT
CASE
WHEN number_of_reviews = 0 THEN 'None'
WHEN number_of_reviews BETWEEN 0 AND 50 THEN 'Few'
WHEN number_of_reviews BETWEEN 50 AND 100 THEN 'Some'
WHEN number_of_reviews BETWEEN 100 AND 150 THEN 'Fairly Alot'
WHEN number_of_reviews BETWEEN 150 AND 200 THEN 'A Lot'
ELSE 'Extremely Alot'
END AS review_category,
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price
FROM
airbnb
GROUP BY
review_category;
```
The highest median price is for listings with 'Few' reviews (between 0 and 50) and the lowest median price is for listings with 'None' reviews. 
Listings with the most number of reviews have the 3rd lowest median price.
There does not seem to be a direct effect of higher number of reviews dictating higher pricing. As shown before, location and centrality of the neighborhood seems to dictate higher pricing.


## Hosts and their Listings

### 10. Percentage of Listings per Host

We’re going to explore how the hosts are utilizing the app. Airbnb originated as a room sharing app however with this analysis we can see who is actually using Airbnb for commercial rental purposes and other aspects about the hosts on Airbnb.

```sql
select
no_listings_per_host,
count(host_id) as host_count,
count(host_id)/sum(count(host_id)) over () percentage
from
(select host_id, count(distinct id) as no_listings_per_host
from airbnb
group by host_id) as a
group by
no_listings_per_host
order by
no_listings_per_host DESC
```
Using this query, it illustrated the % of hosts that list x number of listings. Majority of hosts (63%) only have 1 listing and 12% of hosts have 2 listings. However, this means that 25% of hosts are likely to use the app for commercial purposes.

### 11. Listings operating against Regulations

All residential properties in Singapore (e.g. condominiums, walk up apartments, flats, bungalows, semi-detached and terrace houses) are intended for long-term residence. Under the law, they are not allowed to be used for short-term accommodation
We will investigate how many percent of listings operate against this regulation.

```sql
SELECT COUNT(*) * 100.0 / (SELECT COUNT(*) FROM airbnb) AS percentage
FROM airbnb
WHERE minimum_nights < 90;
```
Output shows that a whopping 31% of listings go against Singapore rental regulations. Now we will investigate the distribution of hosts in these prohibited listings. Do these listings belong to individual hosts or commercial hosts?

### 12. Count of Listings under 90 days per Host

```sql
SELECT host_id, COUNT(*) as listing_count
FROM (
  SELECT DISTINCT id, host_id
  FROM airbnb
  WHERE minimum_nights < 90
) AS x
GROUP BY host_id
ORDER BY 2 DESC;
```

The output shows that many of these ‘illegal’ listings belong to hosts with multiple listings. One of the hosts operate 86 of these ‘illegal’ listings. Lets investigate on a granular level how many percent of these ‘illegal’ listings each host operates.

### 12. Percentage of Listings (Operating under 90 Days) per Host 

```sql
SELECT host_id, listing_count, (listing_count * 100.0) / x.total_listings AS percentage
FROM (
  SELECT COUNT(*) AS total_listings
  FROM airbnb
  WHERE minimum_nights < 90
) AS x
JOIN (
  SELECT host_id, COUNT(*) as listing_count
  FROM (
    SELECT DISTINCT id, host_id
    FROM airbnb
    WHERE minimum_nights < 90
  ) AS subquery
  GROUP BY host_id
) AS y ON true

ORDER BY percentage DESC;
```
Host 138649185 operates 8% of these listings and Host 1439258 4% of these listings. These two host take quite a significant percentage in operating these listings. Around 20 successive hosts after in the table take 1-3% share of operating these listings. We will check how many percent hosts in this ‘illegal’ listing category have only one listing.

### 13. Percentage of Host Operating 1 Listing (Operating under 90 Days) 

```sql
WITH under_90_days_listings AS (
    SELECT id, host_id
    FROM airbnb
    WHERE minimum_nights < 90
), host_listings AS (
    SELECT host_id, COUNT(*) AS num_listings
    FROM under_90_days_listings
    GROUP BY host_id
)
SELECT COUNT(*) * 100.0 / (SELECT COUNT(*) FROM host_listings WHERE num_listings > 1) AS percentage
FROM host_listings
WHERE num_listings = 1;
```
Results show 44%. That is still a relatively smaller proportion of hosts compared to the more “commercial” hosts that operate ‘illegal’ listings. As investigated above, 63% of hosts in the whole dataset have only one listings. Thus, this distribution in the ‘illegal’ listings is quite disproportionate.



