# Project 2: Tracking User Activity
#### Author: Kevin Xuan

## Project Objective
For this project, we are given a json data which consists of data from different customers. Our objective is to transform these individual event log data to one structured table so that data scientists can use the table to conduct further data analysis and visualization and provide business insights for these customers. 


## Procedure for Tranforming Json to Structured Table
The following will be step by step procedures on to achieve our objective of converting json data to one structured table and then land the table into HDFS so that data scientists can access it. 

To begin, we are given a dataset named "assessment-attempts-20180128-121051-nested.json", a json file containing information about an exam such as the name of the exam and the time in which exam is taken. To get the dataset, run
```
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp`
```
to download the dataset.

### Setup Docker-Compose Environment
The first step we need to do is to setup the docker-compose environment so that we have zookeeper, kafka, cloudera, spark, and mids server ready to use. With zookeeper and kafka container, we are able to create a topic under kafka and allow other containers to access and update the topic. For cloudera, it's a data cloud server which stores gigantic amount of data and follows HDFS architecture. For spark, it's a container where we transform messy data or flatten table to a structured table which is ready for data scientists to perform data exploration. The MIDS container helps us publish message/data into the created kafka topic. 

To spin up all these clusters, run
```
docker-compose up -d
```
You should see all clusters being initiated with green colored "done" on the right.


### Check out Hadoop
Let's check out the cloudera container to see if there is any data stored in the server. This is to prevent to create a table name that already exists in the database. Run
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
You should see something like the following:
```
Found 2 items
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2021-07-03 06:37 /tmp/hive
```
Here, we see that we have not saved any data into hadoop. 

### Create Kafka Topic
The next step we do is to create a kafka topic so that we can publish messages/data with Kafka. Here we can create a topic with any name. Since our dataset revolves around assessments, we create the topic named "student-assessment". To do so, run 
```
docker-compose exec kafka kafka-topics --create --topic student-assessment --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
Here we enter the kafka container, creating a topic named "student-assessment" and opening a zookeeper port so that we can access the topic from other containers. After running this code, we should see the following:
```
Created topic student-assessment.
```
This means we have successfully create a kafka topic.

### Publish Message with Kafka
After creating the topic, we then publish the dataset (the json file in which we downloaded earlier) with kafkacat into the kafka topic `student-assessment`. To do so, run
```
docker-compose exec mids bash -c "cat /w205/project-2-kevinxuan/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t student-assessment"
```
To better understand the code, we enter the mids container, and then under the container we find and display the json file which is located under the `/w205/project-2-kevinxuan/` directory. Since the json file is formatted to display one row if we , we apply `jq '[]' -c` to pretty print the json file so that we are able to view important important within the dataset and get more details about it. After performing the pretty print, we use kafkacat to publish the messages inside the json file inside the kafka topic `student-assessment`. We have `-b kafka:29092` to allow us to publish message from other containers, and we have `-t student-assessment` to tell system that we want to store data inside the kafak topic named `student-assessment`.

Here we will not see any outputs from the above command. Now we have successfully publish the message.

### Spin up Spark Cluster
Next, we will spin up the Spark cluster. Since I have already linked jupyter with Spark cluster, we just need to spin up the `spark` container and then write commands under the jupyter file to conduct transformation on the dataset. To start the cluster, run
```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 7000 --ip 0.0.0.0 --allow-root --notebook-dir=/w205/' pyspark
```
We should see something like:
```
[I 06:36:56.072 NotebookApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
[I 06:36:56.427 NotebookApp] Serving notebooks from local directory: /w205
[I 06:36:56.428 NotebookApp] 0 active kernels 
[I 06:36:56.430 NotebookApp] The Jupyter Notebook is running at: http://0.0.0.0:7000/?token=52ad2b41c9b8a18eaccd0771bdb8c93fe4fdb55d2b7b3e53
[I 06:36:56.430 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 06:36:56.431 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://0.0.0.0:7000/?token=52ad2b41c9b8a18eaccd0771bdb8c93fe4fdb55d2b7b3e53
```
The output indicates that we have successfully started the cluster. For our instance, our static external IP address is `34.81.138.56` with port number `7000`, so to run pyspark with jupyter we open a new web page and access URL `34.81.138.56:7000`. 
Note: we maybe asked for the token number to access our jupyter instance, and we just copy and paste the token from the output on the last line above to access. We will then be directed to the jupyter platform, and now we open `Kevin_pyspark_Jupyter.ipynb` to start the process of 'consuming message with spark'.

### Check out Hadoop again
After consuming message with Spark, transforming the data to a structured table, and saving the table to HDFS, we run
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
and should see the following:
```
Found 3 items
drwxr-xr-x   - root   supergroup          0 2021-07-03 08:30 /tmp/exam
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2021-07-03 06:37 /tmp/hive
```
Here, we see that compared to before we have an extra file named `/tmp/exam`. This indicates that we have successfully land our transformed table in which we have conducted transformation in Pyspark into HDFS. If we want to conduct data analysis in a later stage, we can access the data in HDFS by calling `read_parquet` in Pyspark. 

## Conclusion
Now we have completed the stage of setting up clusters, creating kafka topic, publishing message with kafkacat into topic, consuming message and transforming data to structured table with Pyspark, saving structured table into HDFS, and answering the business questions in which we are curious. Once we are done with this project, we need to shut off all these clusters to stop using all these hardware resources. To turn off all clusters, run
```
docker-compose down
```
#### Note: Please DON'T run the command above if you have not finished the project. Once all the clusters are shut down, all the data in the containers such as the Kafka topic "student-assessment" and the table "/tmp/exam" in HDFS will be removed. 