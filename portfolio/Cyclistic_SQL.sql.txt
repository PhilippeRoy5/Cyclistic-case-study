/****** Creating year table with the 12 months csv files ******/

Select *
	INTO [cycle_DB].[dbo].[Year_Table]
FROM 
(
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202105']
UNION
Select ride_id, rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202106']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202107']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202108']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202109']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202110']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202111']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202112']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202201']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202202']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202203']
UNION
Select ride_id,rideable_type, start_station_name,end_station_name,
member_casual,day_of_week,month
FROM [cycle_DB].[dbo].['202204']
) a


-- ----------------------Cleaning out the "NULL" from the table------------------------------------------


DELETE 
FROM [cycle_DB].[dbo].[Year_Table]
WHERE start_station_name LIKE '%NULL%'
AND end_station_name LIKE '%NULL%'



--------------------------------------------Adding the Days names into a new column------------------------------------------


ALTER TABLE [cycle_DB].[dbo].[Year_Table]
add week_day Nvarchar(50);

UPDATE [cycle_DB].[dbo].[Year_Table]
SET week_day = CASE
	WHEN day_of_week = 1 THEN 'Sunday'
	WHEN day_of_week = 2 THEN 'Monday'
	WHEN day_of_week = 3 THEN 'Tuesday'
	WHEN day_of_week = 4 THEN 'Wednesday'
	WHEN day_of_week = 5 THEN 'Thursday'
	WHEN day_of_week = 6 THEN 'Friday'
	ELSE 'Saturday'
	END


-- ------------------------------------------Adding the Month names into a new column ------------------------------------------


ALTER TABLE [cycle_DB].[dbo].[Year_Table]
add month_year Nvarchar(50);

UPDATE [cycle_DB].[dbo].[Year_Table]
SET month_year = CASE
	WHEN month = 1 THEN 'January'
	WHEN month = 2 THEN 'February'
	WHEN month = 3 THEN 'March'
	WHEN month = 4 THEN 'April'
	WHEN month = 5 THEN 'May'
	WHEN month = 6 THEN 'June'
	WHEN month = 7 THEN 'July'
	WHEN month = 8 THEN 'August'
	WHEN month = 9 THEN 'September'
	WHEN month = 10 THEN 'October'
	WHEN month = 11 THEN 'November'
	ELSE 'December'
	END


--------------------------------------Removing original month and day of week columns--------------------------------------


ALTER TABLE [cycle_DB].[dbo].[Year_Table]
DROP COLUMN day_of_week, month


-- ------------------------------------------Cleaning ride ID ------------------------------------------


DELETE
FROM [cycle_DB].[dbo].[Year_Table]
WHERE LEN(ride_id) <> 16




-- ---------------------Removing the maintenance Stations from the Start Stations------------------------------------------


DELETE 
FROM [cycle_DB].[dbo].[Year_Table]
WHERE start_station_name LIKE '%TEST%'
AND start_station_name LIKE '%REPAIR%'


-- ----------------------------Removing the maintenance Stations from the End Stations------------------------------------------


DELETE 
FROM [cycle_DB].[dbo].[Year_Table]
WHERE end_station_name LIKE '%TEST%'
AND end_station_name LIKE '%REPAIR%'



--------------------------------------------Cleaning Start Station names------------------------------------------

ALTER TABLE [cycle_DB].[dbo].[Year_Table]
ADD clean_start_station Nvarchar(255);

UPDATE [cycle_DB].[dbo].[Year_Table]
SET clean_start_station = TRIM( 
	REPLACE( REPLACE(start_station_name, '*',''), '(TEMP)','')



--------------------------------------------Cleaning End Station names------------------------------------------


ALTER TABLE [cycle_DB].[dbo].[Year_Table]
ADD clean_end_station Nvarchar(255);

UPDATE [cycle_DB].[dbo].[Year_Table]
SET clean_end_station = TRIM (
	REPLACE(REPLACE(end_station_name, '*',''), '(TEMP)','')



-----------------------------Removing  original start and end station names from table------------------------------------------



ALTER TABLE [cycle_DB].[dbo].[Year_Table]
DROP COLUMN start_station_name, end_station_name


--------------------------------------------Creating the Final Table for exploration------------------------------------------

SELECT *
INTO [cycle_DB].[dbo].[Final_table]
	FROM (
	SELECT ride_id, rideable_type, member_casual, week_day,
	month_year,clean_start_station,clean_end_station, CAST(crid.total_time as time) as ride_length
	FROM [cycle_DB].[dbo].[Year_Table]

--------------------------------------------Removing ride length under 1 minute------------------------------------------

DELETE
FROM [cycle_DB].[dbo].[Final_table]
WHERE ride_length < '00:01:00'




-------------------------/****** Creating the Geo locations table  ******/---------------------------------------------

		
/** Departing station table **/


----------Fixing coordinates for casual rider Start_station by creating a casual_dep table---------------------------


WITH casual_dep AS (

 SELECT distinct(clean_start_station), ROUND(AVG(cast(start_lat as float)),4) as dep_lat,
 ROUND(AVG(cast(start_lng as float)),4) as dep_lng, count(ride_id) as casual
 FROM [cycle_DB].[dbo].[Year_Table]
 WHERE member_casual = 'casual'
 GROUP BY clean_start_station
 ),

 -------------Fixing coordinates for member Start_station by creating a member_dep table---------------

 member_dep AS(

  SELECT distinct(clean_start_station), ROUND(AVG(cast(start_lat as float)),4) as dep_lat,
 ROUND(AVG(cast(start_lng as float)),4) as dep_lng, count(ride_id) as member
 FROM [cycle_DB].[dbo].[Year_Table]
 WHERE member_casual = 'member'
 GROUP BY clean_start_station
 ),

 --------Joining Depart station for member and casual together and exporting to CSV file-------------

Depart_station_viz AS (

 Select cd.clean_start_station, cd.dep_lat, cd.dep_lng,cd.casual, md.member
 From [cycle_DB].[dbo].[casual_dep] cd
 JOIN [cycle_DB].[dbo].[member_dep] md
 ON md.clean_start_station = cd.clean_start_station
 ORDER BY md.member DESC )


 /****** Arriving station table  ******/

--------------Fixing coordinates for member End_station by creating a member_arr table-------------------

WITH member_arr AS(

SELECT distinct(clean_end_station), ROUND(AVG(cast(end_lat as float)),4) as arr_lat,
 ROUND(AVG(cast(end_lng as float)),4) as arr_lng, count(ride_id) as member
 FROM [cycle_DB].[dbo].[Year_Table]
 WHERE member_casual = 'member'
 GROUP BY clean_end_station
 ),

 -- Fixing coordinates for casual rider End_station by creating a casual_arr table

casual_arr AS(
SELECT distinct(clean_end_station), ROUND(AVG(cast(end_lat as float)),4) as arr_lat,
 ROUND(AVG(cast(end_lng as float)),4) as arr_lng, count(ride_id) as casual
 FROM [cycle_DB].[dbo].[Year_Table]
 WHERE member_casual = 'casual'
 GROUP BY clean_end_station
 ),
 arrive_station_viz AS(

 Select ca.clean_end_station, ca.arr_lat, ca.arr_lng,ca.casual, ma.member
 From [cycle_DB].[dbo].[casual_arr] ca
 JOIN [cycle_DB].[dbo].[member_arr] ma
 ON ma.clean_end_station = ca.clean_end_station
 ORDER BY ma.member DESC
 )


---------------------------/******DATA EXPLORATION & ANALYSIS ******/---------------------------------

------------------------------------Calculation the total rides --------------------------------------

SELECT count(ride_id) as Total_rides
FROM [cycle_DB].[dbo].[Final_table]

SELECT member_casual, count(ride_id) as rides
FROM [cycle_DB].[dbo].[Final_table]
GROUP BY member_casual


------------------------Calculation the amount of rides on different months per rider type --------------------

SELECT month_year, count(ride_id) as rides
FROM [cycle_DB].[dbo].[Final_table]
WHERE member_casual = 'member'
GROUP BY month_year

SELECT month_year, count(ride_id) as rides
FROM [cycle_DB].[dbo].[Final_table]
WHERE member_casual = 'casual'
GROUP BY month_year


------------------------Calculation the amount of rides on different days per rider type --------------------------------------


SELECT week_day, member_casual, count(ride_id) as rides
FROM [cycle_DB].[dbo].[Final_table]
WHERE member_casual = 'member'
GROUP BY week_day, member_casual

SELECT week_day, member_casual, count(ride_id) as rides
FROM [cycle_DB].[dbo].[Final_table]
WHERE member_casual = 'Casual'
GROUP BY week_day, member_casual


--------------------- Calculating the amount of rides by Bike type for each rider type-------------------------


SELECT rideable_type, member_casual, count(ride_id) 
FROM [cycle_DB].[dbo].[Final_table]
WHERE rideable_type <> 'docked_bike'
GROUP BY rideable_type, member_casual


---------------------- Calculating the average ride length per rider type ------------------------------------------

SELECT member_casual, AVG(ride_length) as AVG_length
FROM [cycle_DB].[dbo].[Final_table]
WHERE member_casual = 'member'

SELECT member_casual, AVG(ride_length) as AVG_length
FROM [cycle_DB].[dbo].[Final_table]
WHERE member_casual = 'casual'




