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

![Screenshot (18)](https://user-images.githubusercontent.com/90657593/172741645-f3dab7e0-9ea6-4beb-9453-6f0318b56899.png)

PART 5.1 - PROGRAMMING ASSIGNMENT
Import the Movielens dataset into MongoDB. Refer to README about file contents and headings.
https://grouplens.org/datasets/movielens/1m/ (Links to an external site.) (Links to an external site.)   [you may replace :: in the dateset with comma or tab to import]


1. Importing all csv to collections
2. replaced :: with ,
3. added headerline
4. executed below commands
```
mongoimport --db=homework2 --collection=users --type=csv --file="D:/info7250/hw2/ml-1m/ml-1m/users.csv" --headerline
mongoimport --db=homework2 --collection=ratings --type=csv --file="D:/info7250/hw2/ml-1m/ml-1m/ratings.csv" --headerline
mongoimport --db=homework2 --collection=movies --type=csv --file="D:/info7250/hw2/ml-1m/ml-1m/movies.csv" --headerline


```
![Screenshot (19)](https://user-images.githubusercontent.com/90657593/172745107-e7b90702-6bc2-4efb-8899-4a7157961885.png)


1. Find the number Females and Males from the users collection using MapReduce. Do the same thing using count() to compare the results.

```
#Using count method

> db.users.find({"Gender": "F"}).count();
1709
> db.users.find({"Gender": "M"}).count();
4331
>
# Using map reduce
var mapper_gender = function () {
	emit(this.Gender, this.UserID);
 };
 
 var reducer_count_per_gender = function(gender, userId_arr) {
   return userId_arr.length;
};

db.users.mapReduce(mapper_gender, reducer_count_per_gender, {
out: "user_count_per_gender"
});
```
Result matches with count:
![Screenshot (20)](https://user-images.githubusercontent.com/90657593/172746638-a20eb649-6e27-477a-b06a-2a3118089e07.png)

2. Find the number of Movies per year using MapReduce
```
var mapper_movies_per_year = function () {
	var titleStr = this.title;
	var titleStrLen = titleStr.length;
	var year = this.title.substring(titleStrLen-5, titleStrLen-1);
	emit(this.title, this.movieId);
 };
 
 var reducer_movies_per_year = function(year, movieId_arr) {
   return movieId_arr.length;
};

db.movies.mapReduce(mapper_movies_per_year, reducer_movies_per_year, {
out: "movie_count_per_year"
});



```
3. Find the number of Movies per rating using MapReduce
```
var mapper_movies_per_rating = function () {
	
	emit(this.Rating, this.MovieID);
 };
 
 var reducer_movies_per_rating = function(year, movieId_arr) {
   return movieId_arr.length;
};

db.ratings.mapReduce(mapper_movies_per_rating, reducer_movies_per_rating, {
out: "movie_count_per_rating"
});



```

Result of the collection
![Screenshot (21)](https://user-images.githubusercontent.com/90657593/172750332-e8c75a9f-5138-4624-82c7-6cb4c8da1b5b.png)

```
Verifying the result with count
> db.movie_count_per_rating.find()
{ "_id" : 1, "value" : 60796 }
{ "_id" : 4, "value" : 379508 }
{ "_id" : 3, "value" : 283748 }
{ "_id" : 5, "value" : 247312 }
{ "_id" : 2, "value" : 116845 }
> db.ratings.find({"Rating": 1}).count();
60796
```

PART 5.2 - PROGRAMMING ASSIGNMENT
  Repeat 5.1 using Aggregation Pipeline.
  
  Part 5.2.1 Find the number Females and Males from the users collection
  ```
  	db.users.aggregate([
	{ $group: { _id: "$Gender", countbyGender: {$count: {}} } }	
	]);
  ```

Result:
![Screenshot (22)](https://user-images.githubusercontent.com/90657593/172751817-d95fc08f-571a-4da8-878b-32082d8bfbc4.png)

Part 5.2.2 Find the number of Movies per year
  ```
  db.movies.aggregate([
  {
    $group: {
      _id: {
         { $split: [ "$title", " " ] },
      },
      value: { $sum: 1 }
    }
  }
])
  
  
  ```
  
  Part 5.3.3 Find the number of Movies per rating 
  ```
  db.ratings.aggregate(
  	{ $group: { _id: "$Rating", countbyRatings: {$count: {}} } }	
  );
  
  ```
  ![Screenshot (23)](https://user-images.githubusercontent.com/90657593/172753832-b87b50ba-9fd0-4415-8eac-98d6a52db388.png)

PART 6 - PROGRAMMING ASSIGNMENT 

Java Code to Import data into Mongo db
```
import java.io.File;
import java.io.FileNotFoundException;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

import javax.print.Doc;


public class MongoConnectMain {


    public static List<Document> readLogFileAndDocument() throws FileNotFoundException {
        List<Document> documentList = new ArrayList<>();
        File file = new File("D:\\Companies\\MongoConnect\\src\\main\\resources\\access.log");

        Scanner sc = new Scanner(file);
        sc.useDelimiter("\n");
        System.out.println(sc.next());
        while(sc.hasNext()){
            String srcString = sc.next();
            String[] srcArr = srcString.split(" ");
            String ipAddr = srcArr[0];

            String timeStamp = srcArr[3].replaceAll("\\[", "").replaceAll("\\]", "");
            String website = srcArr[6];
            System.out.println(ipAddr);
            System.out.println(timeStamp);
            System.out.println(website);

            Document newDoc = new Document();
            newDoc.append("ipAddress", ipAddr);
            newDoc.append("timeStamp", timeStamp);
            newDoc.append("website", website);
            System.out.println(newDoc);

            documentList.add(newDoc);

        }
        // closing the scanner stream
        sc.close();
        return documentList;
    }

    
    public static void main(String[] args) throws FileNotFoundException, UnknownHostException {
        MongoClient mongo = MongoClients.create();
        MongoDatabase db = mongo.getDatabase("homework2");
        MongoCollection<Document> accessCollection = db.getCollection("access");

        // it ll add data to access collection only if the count is 0
        if (accessCollection.countDocuments() == 0) {
            List<Document> listOfDoc = readLogFileAndDocument();
            accessCollection.insertMany(listOfDoc);
            System.out.println("Document counts " + accessCollection.countDocuments());

        }

    }
}

```

- Number of times any webpage was visited by the same IP address.
```
db.access.aggregate(
  	{ $group: { _id: "$website"}},
	{$group: {_id: "$ipAddress", countbyIp: {$count: {}}}}
	
  );
  




```


- Number of times any webpage was visited each month.








