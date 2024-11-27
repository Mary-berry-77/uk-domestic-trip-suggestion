# 🌍 Tailored Travel Destination and Activity Recommendation System

---

## 🎯 Project Overview

This project focuses on building a **personalised travel recommendation system** that suggests UK travel destinations and activities based on user preferences, travel constraints, and activity themes. The goal is to create a scalable, efficient, and user-friendly system that employs **SQL**, **data analysis**, and survey inputs to deliver tailored recommendations.

---

## 🛤️ Objectives

1. **Personalisation**: Recommend destinations and activities based on user-selected preferences and constraints.
2. **Efficiency**: Optimise database and query structures to ensure rapid filtering and scalability.
3. **Scalability**: Design for seamless integration of additional destinations, activities, and themes.
4. **Usability**: Build a simple yet powerful survey interface, feeding directly into the recommendation engine.

---

### Data Flow
1. **Input**: User survey responses (travel modes, time preferences, countries, destination types) collected via Google Forms.
2. **Processing**:
   - **1st Level Filtering**: Filters destinations by travel mode and time constraints.
   - **2nd Level Filtering**: Further narrows down by countries and destination types.
   - **SQL Techniques**: JSON Parsing, CTEs, UNPIVOT for multi-valued inputs.
3. **Output**: Final list of recommended destinations tailored to user preferences.
   
---

### Tools and Technologies
- **Database**: MySQL (Relational schema design and SQL query implementation)
- **Query Testing**: DBeaver
- **Programming**: Python (future API integration for survey inputs and automation)
- **Data Sources**: Public datasets (VisitBritain)

--- 


## 🚀 Progress (As of Today)

### **1. Database Development**
- **Core Database Structure Created**:
  - Designed a relational database with the following key tables:
    - `destinations`: Stores UK travel destinations and their attributes (city, coast, countryside).
    - `activities`: Contains activities available at each destination.
    - `themes`: Defines 20 activity themes with keywords and descriptions.
    - `travel_mode`: Manages travel modes (car, train, flight) and associated travel times.
    - `user_responses`: Captures user survey data, including preferences and constraints.
  - Established relationships between tables for efficient querying.

- **Sample Data Inserted**:
  - Populated tables with test data, including:
    - 86 UK destinations.
    - Activities tagged with themes, seasons, and types.
    - Travel times from a central location to each destination.

---

### **2. Filtering System**
#### **1st Level Filtering**
- The first level of filtering captures basic user inputs provided through a survey. It includes:
  - **Travel Mode**: Users can select one or more options among car, train, and flight.
  - **Travel Time Constraints**: Maximum time limits set.
- The filtering logic dynamically processes user inputs using the following steps:
  1. **Unpivot User Inputs**: JSON-formatted responses are unpivoted to transform multiple travel modes into a structured format.
  2. **Filter by Travel Mode and Time Constraints**: Destinations are filtered based on user preferences for travel mode and travel time.

#### SQL Query for 1st Level Filtering 

```sql
WITH split_mode AS (
    SELECT
        user_id,
        JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(travel_mode, ',', '","'), '"]'), '$[0]')) AS mode1,
        JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(travel_mode, ',', '","'), '"]'), '$[1]')) AS mode2,
        JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(travel_mode, ',', '","'), '"]'), '$[2]')) AS mode3
    FROM survey_responses
),
unpivot AS (
    SELECT user_id, travel_mode
    FROM (
        SELECT user_id, mode1 AS travel_mode FROM split_mode
        UNION ALL
        SELECT user_id, mode2 AS travel_mode FROM split_mode
        UNION ALL
        SELECT user_id, mode3 AS travel_mode FROM split_mode
    ) AS combined
    WHERE travel_mode IS NOT NULL
)
SELECT tm.destination_id, tm.travel_mode, tm.travel_time
FROM unpivot un
JOIN travel_mode tm ON tm.travel_mode = un.travel_mode
WHERE tm.travel_time <= un.travel_time_preference
  AND un.user_id = 'user_1';
```

#### **2nd Level Filtering**
- The second level of filtering enables users to specify additional preferences:
  - **Nations**: Countries such as England, Scotland, or Wales.
  - **Destination Types**: Categories like City, Coast, or Countryside.

- This step processes multi-valued user inputs and refines the destinations further:
  1. **JSON Parsing**: Parses multi-value preferences (e.g., "England, Scotland") into separate elements.
  2. **CTEs for Reusability**: Uses Common Table Expressions (CTEs) to clean and transform the data.
  3. **Destination Matching**: Matches the parsed preferences with destination data to narrow down results.

#### SQL Query for 2nd Level Filtering

```sql
WITH split_nation AS (
    SELECT
        user_id,
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(nation, ',', '","'), '"]'), '$[0]'))) AS nation1,
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(nation, ',', '","'), '"]'), '$[1]'))) AS nation2
    FROM second_filtering
),
expanded_nation AS (
    SELECT user_id, nation1 AS nation FROM split_nation
    UNION ALL
    SELECT user_id, nation2 AS nation FROM split_nation
),
filtered_nation AS (
    SELECT user_id, nation
    FROM expanded_nation
    WHERE nation IS NOT NULL
),
split_type AS (
    SELECT
        user_id,
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(destination_type, ',', '","'), '"]'), '$[0]'))) AS type1,
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(destination_type, ',', '","'), '"]'), '$[1]'))) AS type2
    FROM second_filtering
),
expanded_type AS (
    SELECT user_id, type1 AS type FROM split_type
    UNION ALL
    SELECT user_id, type2 AS type FROM split_type
),
filtered_type AS (
    SELECT user_id, type
    FROM expanded_type
    WHERE type IS NOT NULL
)
SELECT 
    fn.user_id, 
    tue.destination_id, 
    tue.country, 
    tue.region, 
    tue.destination_type
FROM filtered_nation fn
JOIN user_destinations tue ON fn.nation = tue.country
JOIN filtered_type ft ON tue.destination_type = ft.type
WHERE fn.user_id = 'user_1';
```
---

### **4. Achievements**

1. **Core Database Completed**:
   - Relational schema established with sample data for testing.

2. **Survey System Prepared**:
   - Designed questions for 1st and 2nd level filtering.
   - Data capture structure implemented.

3. **SQL Query Implementation**:
   - Filtering logic for both travel constraints and destination attributes.
   - Efficient handling of multi-valued fields through JSON parsing and joins.

4. **Scalability Proof**:
   - Sample data validates that the database can handle complex queries.


### **5. Next Steps**

1. **Recommendation Algorithm**:
   - Implement a point-based system to rank destinations and activities based on user preferences.
   - Integrate dislikes and likes for better personalisation.

2. **Survey Platform Development**:
   - Build a user interface for real-time conditional surveys.
   - Link survey results directly to the database for live updates.

3. **Performance Optimisation**:
   - Test query efficiency on larger datasets.
   - Refactor SQL for minimal execution time.

4. **Data Visualisation**:
   - Create ER diagrams for clear documentation.
   - Add visual tools for exploring filtered recommendations.

5. **Final Testing and Deployment**:
   - Conduct end-to-end testing of the survey and recommendation pipeline.
   - Deploy the system for real-world user feedback.


### **6. Project Timeline**

| **Task**                       | **Status**     | **Deadline** |
|--------------------------------|----------------|--------------|
| Database Schema Design         | ✅ Completed   | N/A          |
| Core Table Population          | ✅ Completed   | N/A          |
| 1st and 2nd Level Filtering    | ✅ Completed   | N/A          |
| Recommendation Algorithm       | 🚧 In Progress | TBD          |
| Survey Platform Development    | 🚧 Pending     | TBD          |
| System Integration & Testing   | 🚧 Pending     | TBD          |

---

### **Tools and Technologies**

- **Database**: MySQL
- **Query Testing**: DBeaver
- **Programming Language**: Python (for future integration)
- **Data Source**: Publicly available datasets ([VisitBritain](https://www.visitbritain.com))
