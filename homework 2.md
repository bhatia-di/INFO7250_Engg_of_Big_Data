PART 2 - PROGRAMMING ASSIGNMENT
Write a .bat/.sh to import the entire NYSE dataset (stocks A to Z) into MongoDB.
NYSE Dataset Link: http://newton.neu.edu/nyse/nyse.zip

Solution: 

1. Changed directory to =>  C:\Program Files\MongoDB\Server\5.0\bin
2. Open Git Bash
3. executed command => bash D:/info7250/hw2/importNSYEData.sh  
4. ** close any working mongo sessions 

```
  #!/bin/bash
  
  for dataFileSuffix in {A..Z};
  do
	 FILELOC="D:/info7250/hw2/nsye/nyse/NYSE/NYSE_daily_prices_$dataFileSuffix.csv"
	 echo $FILELOC
	 mongoimport --db=homework2 --collection=nsye_stock_data --type=csv --file=$FILELOC --headerline

  done  

  
  
```
![image](https://user-images.githubusercontent.com/90657593/172733153-4167c178-3e31-4959-841c-2f1b9fe584db.png)

5. Checking count of uploaded documents
```
 > use homework2;
switched to db homework2
> show collections
nsye_stock_data
> db.nsye_stock_data.count()
9211031

```


PART 3.1 - PROGRAMMING ASSIGNMENT
Use the NYSE database to find the average price of stock_price_high values for each stock using MapReduce.

```
 var mapper_average_stock_price_high = function () {
	emit(this.stock_symbol, this.stock_price_high);
 }
 
 var reducer_average_stock_price_high = function(stock_symbol, stock_price_high_arr) {
   return Array.avg(stock_price_high_arr);
};

db.nsye_stock_data.mapReduce(mapper_average_stock_price_high, reducer_average_stock_price_high, {
out: average_stock_price_high_mapreduce
});

```



