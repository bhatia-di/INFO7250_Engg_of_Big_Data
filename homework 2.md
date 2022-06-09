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
out: "average_stock_price_high_mapreduce"
});

```
Result of map reduce looks like:
![image](https://user-images.githubusercontent.com/90657593/172735816-baef8eea-a44b-4208-98de-4b763e65ec6d.png)

```
# Validating the average output per stock symbol:
> show collections
average_stock_price_high_mapreduce
nsye_stock_data
> db.average_stock_price_high_mapreduce.find()
{ "_id" : NaN, "value" : 14.35082247860069 }
{ "_id" : "AA", "value" : 52.459682054669976 }
{ "_id" : "AAI", "value" : 10.518446478515186 }
{ "_id" : "AAN", "value" : 19.847593646277858 }
{ "_id" : "AAP", "value" : 44.72131195335273 }
{ "_id" : "AAR", "value" : 19.208936170212784 }
{ "_id" : "AAV", "value" : 12.498480836236933 }
{ "_id" : "AB", "value" : 30.64627297543215 }
{ "_id" : "ABA", "value" : 25.994470198675543 }
{ "_id" : "ABB", "value" : 12.583610986042316 }
{ "_id" : "ABC", "value" : 47.789574069113186 }
{ "_id" : "ABD", "value" : 15.721916592724059 }
{ "_id" : "ABG", "value" : 15.429047379032303 }
{ "_id" : "ABK", "value" : 51.317208457924 }
{ "_id" : "ABM", "value" : 24.52846106112286 }
{ "_id" : "ABR", "value" : 18.430806896551722 }
{ "_id" : "ABT", "value" : 48.1880094506796 }
{ "_id" : "ABV", "value" : 31.986431181485983 }
{ "_id" : "ABVT", "value" : 49.196723684210546 }
{ "_id" : "ABX", "value" : 22.683009677931192 }
Type "it" for more

```

PART 3.2. Part 3.1 result will not be correct as AVERAGE is a commutative operation but not associative. Use a FINALIZER to find the correct average.
(Hint: pass sum and count from the reducer)
(https://docs.mongodb.com/manual/reference/method/db.collection.mapReduce/index.html)

```
var mapper_average_stock_price_high_w_finalizer = function () {
	emit(this.stock_symbol, {stock_price_sum: this.stock_price_high, count: 1});
 }
 
 var reducer_average_stock_price_high_w_finalizer = function(stock_symbol, stock_price_high_arr) {
 var result = {stock_price_sum: 0, count: 0};
 
 for (var i =0; i<stock_price_high_arr.length; i++) 
 { 
 result.count += stock_price_high_arr[i].count;
 result.stock_price_sum += stock_price_high_arr[i].stock_price_sum; 
 }
 return result;
 
};
var final_average_stock_price_high = function(stock_symbol, result) {
	result.avg_stock_high = result.stock_price_sum / result.count;
	return result;
}

db.nsye_stock_data.mapReduce(mapper_average_stock_price_high_w_finalizer, reducer_average_stock_price_high_w_finalizer, {
out: "average_stock_price_high_w_finalizer_1",
finalize: final_average_stock_price_high
});




```

Ouput of the following code:
![Screenshot (17)](https://user-images.githubusercontent.com/90657593/172740415-33f3cca6-47fa-4ae2-833a-0634f9f3e6f7.png)

PART 4. Calculate the average stock price of each price of all stocks using $avg aggregation.
```
	db.nsye_stock_data.aggregate([
		{ $group: { _id: "$stock_symbol", averageStockPrice: { $avg: "$stock_price_high" } } }
	]);
```







