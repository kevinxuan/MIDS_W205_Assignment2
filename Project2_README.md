# Project 2: Tracking User Activity
### Author: Kevin Xuan

## Project Objective
For this project, we are given a json data which consists of data from different customers. Our objective is to transform these individual event log data to one structured table so that data scientists can use the table to conduct further data analysis and visualization and provide business insights for these customers. 


## Procedure for Tranforming Json to Structured Table
The following will be step by step procedures on to achieve our objective of converting json data to one structured table and then land the table into HDFS so that data scientists can access it. 

To begin, we are given a dataset named "assessment-attempts-20180128-121051-nested.json", a json file containing information about an exam such as the name of the exam and the time in which exam is taken. To get the dataset, run
```
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp`
```
to download the dataset.

### 1. Setup Docker-Compose Environment
The first step we need to do is to setup the docker-compose environment so that we have zookeeper, kafka, cloudera, spark, and mids server ready to use. With zookeeper and kafka container, we are able to create a topic under kafka and allow other containers to access and update the topic. For cloudera, it's a data cloud server which stores large amount of data in the HDFS architecture. For spark, it's a container where we transform messy data or flatten table to a structured table which is ready for data scientists to perform data exploration. The MIDS container helps us direct to our messy data file which is stored under the MIDS instance. 

To spin up all these clusters, run
```
docker-compose up -d
```
You should see all clusters being initiated with green colored "done" on the right.


### 2. Create Kafka Topic
Publish and Consume messages with Kafka