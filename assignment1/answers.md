## Question 1: 
### Query: 

````
select averageTemperature, minimumTemperature, maximumTemperature from jfk
where averageTemperature > 85
````

### Output: 
````
1790201914466 {minimumTemperature=77.0, averageTemperature=87.0, maximumTemperature=98.0}
1790201914606 {minimumTemperature=82.0, averageTemperature=87.0, maximumTemperature=95.0}
1790201916706 {minimumTemperature=79.0, averageTemperature=86.0, maximumTemperature=95.0}
````

## Question 2: 

### Reflections: 
Sliding window produce few more outputs with a heigher result than tumbling (83.57 vs 83.14). It can be the case since the sliding window can try/remember more results than the tumbling which may skip results from the previous period.

### Query 1:
````
select 
first(timestamp) as Start_date, 
last(timestamp) as End_date, 
avg(averageTemperature) as Temp 
from jfk #length(7)
having avg(averageTemperature) > 82
````

### Output 1:
````
1790265620492 {Temp=82.57142857142857, Start_date=1470693600000, End_date=1471212000000}
1790265620499 {Temp=83.14285714285714, Start_date=1470780000000, End_date=1471298400000}
1790265620505 {Temp=83.57142857142857, Start_date=1470866400000, End_date=1471384800000}
1790265620512 {Temp=83.14285714285714, Start_date=1470952800000, End_date=1471471200000}
1790265620518 {Temp=82.71428571428571, Start_date=1471039200000, End_date=1471557600000}
````

### Query 2:

````
select 
first(timestamp) as Start_date, 
last(timestamp) as End_date, 
avg(averageTemperature) as Temp 
from jfk #length_batch(7)
having avg(averageTemperature) > 82
````

### Output 2:
````
1790265756441 {Temp=83.14285714285714, Start_date=1470780000000, End_date=1471298400000}
````

## Question 3: 

### Query 3: 
````
select 
first(timestamp) as Week_start, 
last(timestamp) as Week_end, 
sum(precipitation) as Prec 
from san_francisco #ext_timed_batch(timestamp, 7 days)
having sum(precipitation) >= 3
````
### Output 3: 
````
1790267917299 {Week_start=1418166000000, Week_end=1418684400000, Prec=6.160000000000002}
1790267921983 {Week_start=1483484400000, Week_end=1484002800000, Prec=4.7}
1790267922069 {Week_start=1484694000000, Week_end=1485212400000, Prec=3.42}
1790267922241 {Week_start=1487113200000, Week_end=1487631600000, Prec=3.960000000000001}
1790267924229 {Week_start=1514934000000, Week_end=1515452400000, Prec=4.0600000000000005}
````