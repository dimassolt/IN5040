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

### Query: 
````
select 
first(timestamp) as Week_start, 
last(timestamp) as Week_end, 
sum(precipitation) as Prec 
from san_francisco #ext_timed_batch(timestamp, 7 days)
having sum(precipitation) >= 3
````
### Output: 
````
1790267917299 {Week_start=1418166000000, Week_end=1418684400000, Prec=6.160000000000002}
1790267921983 {Week_start=1483484400000, Week_end=1484002800000, Prec=4.7}
1790267922069 {Week_start=1484694000000, Week_end=1485212400000, Prec=3.42}
1790267922241 {Week_start=1487113200000, Week_end=1487631600000, Prec=3.960000000000001}
1790267924229 {Week_start=1514934000000, Week_end=1515452400000, Prec=4.0600000000000005}
````

## Question 4:

### Query:

````
select 
first(timestamp) as Start_date, 
last(timestamp) as End_date, 
avg(averageWindSpeed) as Wind,
sum(precipitation) as Prec 
from san_francisco #length(3)
having sum(precipitation) > 0 and avg(averageWindSpeed) > 19
````


### Output: 
````
1790270490722 {Wind=19.23666666666668, Start_date=1429826400000, End_date=1429999200000, Prec=0.4099999999999997}
1790270494594 {Wind=19.536666666666687, Start_date=1483830000000, End_date=1484002800000, Prec=3.369999999999999}
1790270497723 {Wind=20.20333333333337, Start_date=1527544800000, End_date=1527717600000, Prec=1.1102230246251565E-16}
1790270497759 {Wind=19.686666666666707, Start_date=1528063200000, End_date=1528236000000, Prec=1.1102230246251565E-16}
1790270497772 {Wind=19.08666666666671, Start_date=1528236000000, End_date=1528408800000, Prec=1.1102230246251565E-16}
1790270497778 {Wind=19.760000000000037, Start_date=1528322400000, End_date=1528495200000, Prec=1.1102230246251565E-16}
1790270497785 {Wind=19.760000000000037, Start_date=1528408800000, End_date=1528581600000, Prec=1.1102230246251565E-16}
````

## Qestion 5: 

Let us consider the following stream sequence:
A1 C1 A2 B1 D1 A3 B2 C2 B3 C3 C4 A4 B4 List the events which match on the following patterns:
• EveryA->B
• A->EveryB
• Every A -> Every B
• Every(A -> B)

### EveryA->B
A1 A2 B1 A3 B2 B3 A4 B4 

(A1 B1)

(A2 B1)

(A3 B2)

(A4 B4) 


###  A->EveryB
A1 A2 B1 A3 B2 B3 A4 B4 

(A1 B1)

(A1 B2)

(A1 B3)

(A1 B4)

### Every A -> Every B

A1 A2 B1 A3 B2 B3 A4 B4 

(A1 B1) (A1 B2) (A1 B3) (A1 B4)  

(A2 B1) (A2 B2) (A2 B3) (A2 B4)

(A3 B2) (A3 B3) (A3 B4)

(A4 B4)

### Every(A -> B)

A1 A2 B1 A3 B2 B3 A4 B4 

(A1 B1) 

(A3 B2) 

(A4 B4)


## Question 6: 

