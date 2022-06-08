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
	 mongoimport --db=hw2 --collection=nsye_stock_data_attempt --type=csv --file=$FILELOC --headerline

  done  
 
  
  
```
![image](https://user-images.githubusercontent.com/90657593/172733153-4167c178-3e31-4959-841c-2f1b9fe584db.png)
