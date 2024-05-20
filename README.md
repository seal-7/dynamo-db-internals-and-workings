# Dynamo DB a little deeper dive and things to know

## How is data stored on DynamoDB
* AWS stores DynamoDB data on SSD's in their warehouse.
* All the data related to a given partition are store on a single SSD.
* If there are partitions which has very less data, Dynamo automatically merges them to balance the overall partitions.

## What is partition key ?
* Partion key is used as the primary identifier for quering dynamoDB table.
* While querying using parition key, you can only do a exact match. Things like less than or greater than ect does not work.
* Partion key is supposed to be used to group things together.
* You need to make sure that this groups are not very large, else they lead to hot key issues. Where most of the queries are going to a single partition.
* There is a hard limit on DynamoDB where per pertition you can only perform 4000 Reads and 1000 Write per second.
* Rule of thumb to follow is that 1 partition key should not have items exceeding 5% of your overall database.

## What is sort key ?
* Sort key is used in conjunction with partition key.
* For eg: If your partition key is Name of Actors, you can keep sort key as Movies. So if you want to query on Movies for a given action this can be done.
* Sort key allows various operation like less than greater than, startWith contains etc. Hence using sort key smartly can help you tackle a lot of use cases.

## How should you store things in DynamoDB ?
* As dynamo is a Nosql database, you should think more in term of grouping related things together.
* You can use Sort key with delimiters to help you drill down or create a hierarchy
* For eg: If you have a database to store all super markets, the brand name can be your partition key and sort key can have "state#city". So if you want to get all the walmart in a given state, you can search with partition = "D-mart" and sort key begins with "gujarat".
* You can also have items with partition key as userId and sort key like "order#123#12-05-2024" and "address#primary" or "address#secondary" in the same table. If you want to get all the user related data you just query data by partition key which is your userId and all users data should be returned including orders and address. If you just want user's orders than you can query partition = userId and sortKey begins with "order"
  
## What is Global Secondary Index or GSI ?
* GSI is helpful when you want to query your data using different keys than partion key and sort key, for eg: if partion key was director and sort key was movies, but now you want to query using movie to get all directors. You can setup a new GSI where partition key is movie and secondary or sort key is director.
* You can create 20 GSI max per table.
* Everytime new row is added, the data has to be added on primary index as well as each of the GSI. Hence it consumes more write capacity.
* When querying data, it only consumed read capacity of the index being used.

## Pricing methods
* Pay per request, where you pay for each individual request based on RCU and WCU consumed.
* Provisioned, where you pre provision your Read and Write capacity and you pay on hourly basis.
* Dynamo also has autoscaling, where you can define if your provisioned capapcity need to increase in case the RCU/WCU start getting consumed more.
* Good practice is to keep you autoscaling at 70%, so it scale before your table starts throtlling. Based on the traffic pattern you can decrease this number as well. Like for Identity services, during any event there is a sudden initial spike as a lot of the users get on the platform together. There you can consider keeping it 50%.
* Scaling up dynamoDB takes time hence always keep that in mind while setting your autoscaling thresholds.
* DynamoScales up if for 2 consicutive minutes, the consumption was about the threshold you configured.
* Dynamo scales down when for 15 consicutive minutes, the consumption was 20% less than the configured threshold.
* If you have continues traffic or cyclic traffic(increase in day and goes down at night) you should probably use provisioned capacity.
* For cases like a schedule job or when most of the time your database remains ideal, you can consider pay per request. Pay per request is more costier compared to provisioned.
* Number of request does not reach the threshold in 10 mins. So if threshold is 70%, make sure in every 10 mins block, the traffic is not increase by more than 30%. Than DynamoDB should be able to autoscale without any throttling. 

## Reference
* https://www.youtube.com/@CompleteCoding/videos