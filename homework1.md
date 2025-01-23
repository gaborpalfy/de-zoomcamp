Question 1:
  Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.
  What's the version of pip in the image?

Answer 1:
  - From a MINGW64 shell run the following:
  -   docker run -it --entrypoint bash python:3.12.8
  -   pip --version is 24.3.1

Question 2:

From the docker-compose.yaml file, what is the hostname and port that pgadmin should use to connect to the postgres database?

  services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

Answer 2:
  postgres:5433

Question 3:

  Use the green_taxi_trips_2019-10.csv for this question. During the period of October 1st 2019
  (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:
  
  Up to 1 mile
  In between 1 (exclusive) and 3 miles (inclusive),
  In between 3 (exclusive) and 7 miles (inclusive),
  In between 7 (exclusive) and 10 miles (inclusive),
  Over 10 miles

Answer 3:
  - With the pgdatabase and pgadmin container configuration, run docker-compose up
  - In MINGW64, run:
    URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"
  python ingest_data.py \
    --user=postgres \
    --password=postgres \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url=${URL}
  In pgadmin, run the following SQL query:
    SELECT 
    SUM(CASE WHEN trip_distance <= 1 THEN 1 ELSE 0 END) AS trips_1_or_less,
    SUM(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 ELSE 0 END) AS trips_1_to_3,
    SUM(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 ELSE 0 END) AS trips_3_to_7,
    SUM(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 ELSE 0 END) AS trips_7_to_10,
    SUM(CASE WHEN trip_distance > 10 THEN 1 ELSE 0 END) AS trips_more_than_10
    FROM green_taxi_trips
    WHERE lpep_pickup_datetime >= '2019-10-01'
      AND lpep_dropoff_datetime < '2019-11-01';

    The result is: 104802	198924	109603	27678	35189

Question 4: Longest trip for each day 
  Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.
  
  Tip: For every day, we only care about one single trip with the longest distance.
  
  2019-10-11
  2019-10-24
  2019-10-26
  2019-10-31

Answer 4:
  Run the query:
  SELECT DATE(lpep_pickup_datetime) AS pickup_date, MAX(trip_distance) AS longest_trip_distance
  FROM green_taxi_trips
  WHERE DATE(lpep_pickup_datetime) IN ('2019-10-11', '2019-10-24', '2019-10-26', '2019-10-31')
  GROUP BY DATE(lpep_pickup_datetime);
  
  Result:
  "2019-10-26"	91.56
  "2019-10-31"	515.89
  "2019-10-24"	90.75
  "2019-10-11"	95.78

Question 5:

  Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18?

  Consider only lpep_pickup_datetime when filtering by date.
  
    East Harlem North, East Harlem South, Morningside Heights
    East Harlem North, Morningside Heights
    Morningside Heights, Astoria Park, East Harlem South
    Bedford, East Harlem North, Astoria Park
  

Asnwer 5:

  Run the query:
    SELECT z."Zone", gt."PULocationID", SUM(gt."total_amount") AS total_amount_sum
    FROM green_taxi_trips gt
    JOIN zones z ON gt."PULocationID" = z."LocationID"
    WHERE DATE(gt."lpep_pickup_datetime") = '2019-10-18'
    GROUP BY z."Zone", gt."PULocationID"
    HAVING SUM(gt."total_amount") > 13000;

  Result:
    Zone, PULocationID, total_amount_sum
    "East Harlem North"	74	18686.680000000088
    "East Harlem South"	75	16797.260000000075
    "Morningside Heights"	166	13029.79000000004

Question 6:

  For the passengers picked up in October 2019 in the zone named "East Harlem North" which was the drop off zone that had the largest tip? Note: it's tip , not trip
  
  We need the name of the zone, not the ID.
  
  Yorkville West
  JFK Airport
  East Harlem North
  East Harlem South

Answer 6:
  Query: 
    SELECT z."Zone", MAX(gt."tip_amount") AS largest_tip
    FROM green_taxi_trips gt
    JOIN zones z ON gt."DOLocationID" = z."LocationID"
    WHERE DATE(gt."lpep_pickup_datetime") BETWEEN '2019-10-01' AND '2019-10-31'
    AND gt."PULocationID" = (
        SELECT "LocationID"
        FROM zones
        WHERE "Zone" = 'East Harlem North'
    )
    GROUP BY z."Zone"
    ORDER BY largest_tip DESC
    LIMIT 1;

  Result:
    Zone, largest_tip
    "JFK Airport"	87.3

Question 7:

Which of the following sequences, respectively, describes the workflow for:

Downloading the provider plugins and setting up backend,
Generating proposed changes and auto-executing the plan
Remove all resources managed by terraform

Answer 7:
  terraform init, terraform apply -auto-approve, terraform destroy
