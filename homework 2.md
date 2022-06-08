PART 2 - PROGRAMMING ASSIGNMENT
Write a .bat/.sh to import the entire NYSE dataset (stocks A to Z) into MongoDB.
NYSE Dataset Link: http://newton.neu.edu/nyse/nyse.zip

```
  #!/bin/bash
  for dataFile in 'D:/Engg of Big data/hw2/nsye/nyse/NYSE/NYSE_daily_prices_*.csv';
  do
     echo $dataFile
  done  
  
  
```
