Twitter Sentiment Analysis
==============

Overview
-------

This project consists on different type of resources


    1. twitter messages: datasets of real twitters associated to a sentiment
    
    2. lexical resources: 
       1. list of words associated to a sentiment
       2. list of words or symbols that are related to the sentence


For each word found in the Twitter messages we have three purposes:

1. Check if the word is listed in all related resource.
2. Count the number of occurrences of each word in the Twitter messages for each emotion.
3. Draw a word cloud associated to the most frequent words in each emotion.

Set up MongoDB
-------

Inside mongoConf directory, type:

    docker-compose up -d

Wait for about 15-20 seconds and then:

    # Init the replica sets (use the MONGOS host)
    #comando 1
    docker exec -it mongos1 bash -c "echo 'rs.initiate({_id: \"mongors1conf\",configsvr: true, members: [{ _id : 0, host : \"mongocfg1:27019\", priority: 2 },{ _id : 1, host : \"mongocfg2:27019\" }, { _id : 2, host : \"mongocfg3:27019\" }]})' | mongo --host mongocfg1:27019"
    #comando 2
    docker exec -it mongos1 bash -c "echo 'rs.initiate({_id : \"mongors1\", members: [{ _id : 0, host : \"mongors1n1:27018\", priority: 2 },{ _id : 1, host : \"mongors1n2:27018\" },{ _id : 2, host : \"mongors1n3:27018\" }]})' | mongo --host mongors1n1:27018"
    #comando 3
    docker exec -it mongos1 bash -c "echo 'rs.initiate({_id : \"mongors2\", members: [{ _id : 0, host : \"mongors2n1:27018\", priority: 2 },{ _id : 1, host : \"mongors2n2:27018\" },{ _id : 2, host : \"mongors2n3:27018\" }]})' | mongo --host mongors2n1:27018"

Now wait about 15 seconds and then:

    # ADD TWO SHARDS (mongors1, and mongors2)
    #comando 1
    docker exec -it mongos1 bash -c "echo 'sh.addShard(\"mongors1/mongors1n1:27018,mongors1n2:27018,mongors1n2:27018\")' | mongo"
    #comando 2
    docker exec -it mongos1 bash -c "echo 'sh.addShard(\"mongors2/mongors2n1:27018,mongors2n2:27018,mongors2n3:27018\")' | mongo"

Connect to the shell:

    mongo --host "localhost:27017,localhost:27016"

Create the indexes for the sharding process (from shell):

    use mydatabase
    db.words.createIndex({feeling: 1})

Run the sharding process:

    sh.enableSharding("maadbProject_id")
    sh.shardCollection("maadbProject_id.words", { "feeling": 1})

Check the shards status:

    sh.status()

Check the chunks balancing for each shard:

    db.getSiblingDB("maadbProject_id").words.getShardDistribution()

Every time you kill or reboot the pc you need to kill the local MongoDB instance:

    ps aux | grep mongo
