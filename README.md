# 🌍 Tailored Travel Destination and Activity Recommendation System

---

## 🎯 Project Overview

This project focuses on building a **personalised travel recommendation system** that suggests UK travel destinations and activities based on user preferences, travel constraints, and activity themes. The goal is to create a scalable, efficient, and user-friendly system that employs **SQL**, **data analysis**, and survey inputs to deliver tailored recommendations.

---

## 🛤️ Objectives

1. **Personalisation**: Recommend destinations and activities based on user-selected preferences and constraints.
2. **Efficiency**: Design an optimised database and query structure to handle user data effectively.
3. **Scalability**: Ensure the system can support a growing dataset of destinations, activities, and user inputs.
4. **Usability**: Develop a user-friendly survey interface for data collection and feedback.

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
    - Travel times from a central location (Staines) to each destination.

---

### **2. Filtering System**
#### **1st Level Filtering**
- Captures basic user inputs through the survey:
  - **Travel Mode**: Users select car, train, or flight.
  - **Travel Time Constraints**: Time limits by each travel mode.
- SQL queries implemented to:
  - Filter destinations by travel mode and time constraints.
  - Return destinations matching user-specified criteria.

#### **2nd Level Filtering**
- User selects:
  - Preferred nations (e.g., England, Scotland, Wales).
  - Preferred destination types (e.g., City, Coast).
- SQL query enhancements include:
  - Parsing JSON-like responses into individual elements.
  - Filtering destinations based on the combination of nation and destination type.
- Created reusable CTEs for:
  - Splitting multi-valued fields (e.g., comma-separated nations or types).
  - Efficiently joining user preferences with destination data.

---

### **3. Query Highlights**
#### Destination Filtering by User Preferences
Filters destinations by combining travel mode, time, nation, and destination type constraints.

```sql
WITH split_nation AS (
    SELECT 
        user_id, 
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(nation, ',', '","'), '"]'), '$[0]'))) AS nation1,
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(nation, ',', '","'), '"]'), '$[1]'))) AS nation2
    FROM user_responses
    WHERE user_id = 'user_1'
),
expanded_nation AS (
    SELECT user_id, nation1 AS nation FROM split_nation
    UNION ALL
    SELECT user_id, nation2 AS nation FROM split_nation
    WHERE nation IS NOT NULL
),
split_type AS (
    SELECT 
        user_id, 
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(destination_type, ',', '","'), '"]'), '$[0]'))) AS type1,
        TRIM(JSON_UNQUOTE(JSON_EXTRACT(CONCAT('["', REPLACE(destination_type, ',', '","'), '"]'), '$[1]'))) AS type2
    FROM user_responses
    WHERE user_id = 'user_1'
),
expanded_type AS (
    SELECT user_id, type1 AS type FROM split_type
    UNION ALL
    SELECT user_id, type2 AS type FROM split_type
    WHERE type IS NOT NULL
)
SELECT 
    fn.user_id, 
    d.destination_id, 
    fn.nation, 
    d.region, 
    d.destination_type
FROM expanded_nation fn
JOIN destinations d
  ON fn.nation = d.country
JOIN expanded_type ft
  ON d.destination_type = ft.type
WHERE fn.user_id = 'user_1';
```

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
