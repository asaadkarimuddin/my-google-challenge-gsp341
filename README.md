# my-google-challenge-gsp341
ml google challenge gsp341 
Task 1: Create a dataset to store your machine learning models
It can be called any name to pass the test but for the remaining instructions to work use `asaad` as the dataset name.


bq mk asaad


#####################################################################

Task 2: Create a forecasting BigQuery machine learning model.
Create the first ML model using a JOIN between two bike share tables. Again any names will work but keep them as ‘asaad_1’ and ‘asaad_2’ for the remaining instructions to work. 
BigQuery Console Query Editor



CREATE OR REPLACE MODEL asaad.asaad_1

OPTIONS

  (model_type='linear_reg', labels=['duration_minutes']) AS

SELECT

    start_station_name,

    EXTRACT(HOUR FROM start_time) AS start_hour,

    EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,

    duration_minutes,

    address as location

FROM

    `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips

JOIN

    `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS stations

ON

    trips.start_station_name = stations.name

WHERE

    EXTRACT(YEAR FROM start_time) = 2018

    AND duration_minutes > 0



###############################################################################################
Task 3: Create the second machine learning model. 
BigQuery Console Query Editor




CREATE OR REPLACE MODEL asaad.asaad_2

OPTIONS

  (model_type='linear_reg', labels=['duration_minutes']) AS

SELECT

    start_station_name,

    EXTRACT(HOUR FROM start_time) AS start_hour,

    subscriber_type,

    duration_minutes

FROM `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips

WHERE EXTRACT(YEAR FROM start_time) = 2018


########################################################################################################
Task 4: Evaluate the two machine learning models.


BigQuery Console Query Editor
Query 1

-- Evaluation metrics for asaad_1

SELECT

  SQRT(mean_squared_error) AS rmse,

  mean_absolute_error

FROM

  ML.EVALUATE(MODEL asaad.asaad_1, (

  SELECT

    start_station_name,

    EXTRACT(HOUR FROM start_time) AS start_hour,

    EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,

    duration_minutes,

    address as location

  FROM

    `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips

  JOIN

   `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS stations

  ON

    trips.start_station_name = stations.name

  WHERE EXTRACT(YEAR FROM start_time) = 2019)

)



Query 2
-- Evaluation metrics for asaad_2

SELECT

  SQRT(mean_squared_error) AS rmse,

  mean_absolute_error

FROM

  ML.EVALUATE(MODEL asaad.asaad_2, (

  SELECT

    start_station_name,

    EXTRACT(HOUR FROM start_time) AS start_hour,

    subscriber_type,

    duration_minutes

  FROM

    `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips

  WHERE

    EXTRACT(YEAR FROM start_time) = 2019)

)




##########################################################################################################################
Task 5: Use the subscriber type machine learning model to predict average trip durations
The following query will list busiest stations in descending order. The busiest station for 2019 was “21st & Speedway @PCL”.
BigQuery Console Query Editor




SELECT

  start_station_name,

  COUNT(*) AS trips

FROM

  `bigquery-public-data.austin_bikeshare.bikeshare_trips`

WHERE

  EXTRACT(YEAR FROM start_time) = 2019

GROUP BY

  start_station_name

ORDER BY

  trips DESC



///////////////////////////////////////////////////////////////////////
Then predict trip length.
BigQuery Console Query Editor




SELECT AVG(predicted_duration_minutes) AS average_predicted_trip_length

FROM ML.predict(MODEL asaad.asaad_2, (

SELECT

    start_station_name,

    EXTRACT(HOUR FROM start_time) AS start_hour,

    subscriber_type,

    duration_minutes

FROM

  `bigquery-public-data.austin_bikeshare.bikeshare_trips`

WHERE 

  EXTRACT(YEAR FROM start_time) = 2019

  AND subscriber_type = 'Single Trip'

  AND start_station_name = '21st & Speedway @PCL'))
