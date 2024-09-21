# blablacar

Used mermaid for visualization
![image](https://github.com/user-attachments/assets/9b351008-277f-415c-b412-95738949d3fa)

For the DIMs, I didn't list out all the explicit possible columns. Since it's a dim, it would generally be information related to that given key field.
For Facts, I limited the number of possible information to include to the ride logistics piece. There is the option to delve into pricing and enable pricing analysis. I didn't in order to avoid over complicating the model for the case study. In the real world, it would be part of the full extended data model set. 

**Core Entities:**

How to read table info:
**table_name**: description
  - column data type | column name | key id | description

    
**Table descriptions:**
**DIM_USER**: both drivers and passengers infos.

**DIM_LOCATION**: : Location sepcific info (origins, destinations, and stops).

**DIM_DATE**: date related information (day, day of weeks etc).

**FACT_TRIP_OFFER**: trip from the driver.
  - int trip_offer_id PK : unique table ID
  - int driver_id FK : join key for dim_users: driver information
  - int origin_id FK : join key for  dim_location: start location 
  - int destination_id FK : join key for dim_location: last destination
  - datetime departure_date FK : join key for dim_date: datetime set to leave
  - int available_seats :  number of seats available
  - decimal price_per_seat: pricing
  - string status: trip status

**FACT_TRIP_BOOKING**: bookings made by a passenger.

**FACT_TRIP_STOP**: each stop in a trip including intermediate stops .


**Optimization:** 
**Dimensions Table:** 
1. Clustering, possible to do geo-local partition by assigning numerical ID groups.
    e.g Dim_Location (country/region), at a more complex level of clustering (creating frequency bins, how often this location is driven to, offer provided etc, group it into least low, medium, high) 

**Fact tables:**
1. partition tables All tables on event date.
2. Clustering by. most likely some form of location first, pending usage from analyst.
3. Indexing + Sharding when we're working with extremely large tables. (100gb + tables) index turns off at 10gb. Sharding a bit of overhead, but in terms of organization and quick queries, definitely helps. Particularly more important when it isnt date partitioned or hitting parition limits. 
  

**Sample Usage**
Question 1:
How many trip offers have been published last month?

SELECT COUNT(*) AS trip_offers_count
FROM FACT_TRIP_OFFER AS fo
JOIN DIM_DATE d ON fo.departure_date = d.date_id
WHERE d.month = MONTH(CURRENT_DATE - INTERVAL 1 MONTH);


Question 2: 
What country had the highest number of publications last month?

SELECT l.country, COUNT(*) AS trip_offers_count
FROM FACT_TRIP_OFFER fo
JOIN DIM_DATE d ON fo.departure_date = d.date_id
JOIN DIM_LOCATION l ON fo.origin_id = l.location_id
WHERE d.month = MONTH(CURRENT_DATE - INTERVAL 1 MONTH)
GROUP BY l.country
ORDER BY trip_offers_count DESC;


  
