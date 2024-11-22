# 🌍 Tailored Travel Destination and Activity Recommendation System

## 🎯 Project Overview
This project is designed to build a **personalised travel recommendation system** for community centre members. It leverages **SQL** and **data analysis** to recommend UK destinations and activities based on user preferences, travel constraints, and activity themes.

---

## 🛤️ Objectives
1. **Personalisation**: Suggest destinations and activities tailored to user preferences.
2. **Efficiency**: Use SQL to manage and analyse data effectively.
3. **Scalability**: Create a system that accommodates diverse data and user inputs.
4. **Usability**: Deliver actionable insights via a user-friendly survey system.

---

## 🗺️ Current Progress

### **Database Creation**
The database was **entirely built from scratch** using publicly available information from [VisitBritain](https://www.visitbritain.com). It includes:
- **Destinations**: 86 UK destinations categorised by `city`, `coast`, and `countryside`.
- **Activity Data**: Various activities linked to destinations, each with associated themes, seasons, and types.
- **Themes**: 20 activity themes, each with keywords and descriptions for better user understanding.
- **Travel Modes**: Options for travel by car (hired coach), train, or flight, with time constraints for each mode.
- **Travel Times**: Calculated travel times from a central location (Staines) to each destination by available travel modes.

### **SQL Query Implementation**
The project’s initial query achieves the following:
1. **Assigns Unique IDs**: Uses `ROW_NUMBER()` to generate IDs for activities.
2. **Splits Themes**: Extracts multiple activity themes from a single column.
3. **Normalises Data**: Unpivots themes to create a relationship-friendly table structure.

#### Query Code
```sql
WITH activity_cte AS (
    SELECT 
        destination_id,
        destination,
        activity,
        ROW_NUMBER() OVER (ORDER BY activity) AS activity_id,
        theme,
        season, 
        type 
    FROM activity_uk
),
split_theme AS (
    SELECT 
        destination_id,
        destination,
        activity,
        activity_id,
        SUBSTRING_INDEX(SUBSTRING_INDEX(theme, ',', 1), ',', -1) AS theme_1,
        SUBSTRING_INDEX(SUBSTRING_INDEX(theme, ',', 2), ',', -1) AS theme_2,
        SUBSTRING_INDEX(SUBSTRING_INDEX(theme, ',', 3), ',', -1) AS theme_3,
        SUBSTRING_INDEX(SUBSTRING_INDEX(theme, ',', 4), ',', -1) AS theme_4,
        season,
        type
    FROM activity_cte
),
pivoted_theme AS (
    SELECT 
        destination_id,
        destination,
        activity,
        activity_id,
        theme_1 AS theme,
        season,
        type
    FROM split_theme
    UNION ALL 
    SELECT 
        destination_id,
        destination,
        activity,
        activity_id,
        theme_2 AS theme,
        season,
        type
    FROM split_theme
    UNION ALL 
    SELECT 
        destination_id,
        destination,
        activity,
        activity_id,
        theme_3 AS theme,
        season,
        type
    FROM split_theme
    UNION ALL 
    SELECT 
        destination_id,
        destination,
        activity,
        activity_id,
        theme_4 AS theme,
        season,
        type
    FROM split_theme
)
SELECT 
    destination_id,
    destination,
    activity,
    activity_id,
    theme,
    season,
    type
FROM pivoted_theme;
```
