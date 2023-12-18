# spark-data-analysis
Initially lat and lng coordinates were casted to float (they are stored as string).

After that Null values were found. There is only one entity with null lat and/or lng:
```
+-----------+------------+--------------+-----------------------+-------+------+----+----+
|         id|franchise_id|franchise_name|restaurant_franchise_id|country|  city| lat| lng|
+-----------+------------+--------------+-----------------------+-------+------+----+----+
|85899345920|           1|       Savoria|                  18952|     US|Dillon|NULL|NULL|
+-----------+------------+--------------+-----------------------+-------+------+----+----+
```

This issue was resolved using opencage. To make sure that opencage correctly identifies lat and lng based on country and city name, some unit tests were used:
```
assert abs(fill_null_lat_lng('Kazakhstan', 'Pavlodar') - 52.287303) <= 1 # arbitrary known address
assert abs(fill_null_lat_lng('Kazakhstan', 'Pavlodar', 'lng') - 76.967402) <= 1  # arbitrary known address
assert abs(fill_null_lat_lng('Us', 'Rapid City', 'lat') - 44.080) <= 1  # address from dataset
assert abs(fill_null_lat_lng('Us', 'Rapid City', 'lng') - -103.250) <= 1  # address from dataset
```

After that geohashes were computer for restaurants. Unit tests for them were as follows:
```
assert generate_geohash(52.287303, 76.967402) == 'v9q9' # an arbitrary example
assert generate_geohash(0, 0) == 's000' # 0 case
assert generate_geohash(-90, 90) == 'n000' # edge case
assert generate_geohash(34.578, -87.021) == 'dn4h' # an arbitrary case from data, checked manually
```

Next weather data was copied and joined to original dataset. To avoid data multiplication, restaurants data was left-joined to weather records, but first of all, temperature was averaged per date and geohash (so that there are no multiple rows for a sinlge geohash).
Result was saved, parttitioned by wthr_date.
