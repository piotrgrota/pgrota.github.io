---
layout: post
title: How to Analyse Real Time Data in AWS ?
---


![_config.yml]({{ site.baseurl }}/images/post3/Product-Page-Diagram_Kinesis-Data-Analytics-How-it-Works.png)

## Problem:

How we can Analyse Real Time Data with AWS ? 

<b>Use case: </b> 

These days our software producing a lot of information. Real-time analytics allows businesses to get insights and act on data immediately or soon after the data enters their system.

Real time app analytics answer queries within seconds. They handle large amounts of data with high velocity and low response times. For example, real-time big data analytics uses data in financial databases to inform trading decisions.

Analytics can be on-demand or continuous. On-demand delivers results when the user requests it. Continuous updates users as events happen and can be programmed to respond automatically to certain events. For example, real-time web analytics might update an administrator if page load performance goes out of preset parameters.

#### What will be needed :

1. AWS Account
2. We gonna use Cloud Formation as our Infrastructure as a Code
3. We gonna use Python - will generate data
4. We gonna use SQL - use for transformation and aggregation


#### Architecture:

![_config.yml]({{ site.baseurl }}/images/post3/RealTimeDatainAWS.png)

[More interactive...](https://app.cloudcraft.co/view/e32ceb51-4f48-4230-abcf-a07114807e1b?key=FYRDbrAXE2NiJI3l1XFgqQ)

First we need to understand few terms :

1. Kinesis Data Stream:

    Amazon Kinesis Data Streams (KDS) is a massively scalable and durable real-time data streaming service. KDS can continuously capture gigabytes of data per second from hundreds of thousands of sources such as website clickstreams, database event streams, financial transactions, social media feeds, IT logs, and location-tracking events. The data collected is available in milliseconds to enable real-time analytics use cases such as real-time dashboards, real-time anomaly detection, dynamic pricing, and more.

    [More on Kinesis](https://docs.aws.amazon.com/streams/latest/dev/introduction.html)

2. Amazon Kinesis Data Analytics
    
    Amazon Kinesis Data Analytics is the easiest way to analyze streaming data, gain actionable insights, and respond to your business and customer needs in real time. Amazon Kinesis Data Analytics reduces the complexity of building, managing, and integrating streaming applications with other AWS services. SQL users can easily query streaming data or build entire streaming applications using templates and an interactive SQL editor. Java developers can quickly build sophisticated streaming applications using open source Java libraries and AWS integrations to transform and analyze data in real-time.

    Amazon Kinesis Data Analytics takes care of everything required to run your real-time applications continuously and scales automatically to match the volume and throughput of your incoming data. With Amazon Kinesis Data Analytics, you only pay for the resources your streaming applications consume. There is no minimum fee or setup cost.

    [More on Data Analytics](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/what-is.html)

3.  Cloud Formation:

    AWS CloudFormation provides a common language for you to model and provision AWS and third party application resources in your cloud environment. AWS CloudFormation allows you to use programming languages or a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This gives you a single source of truth for your AWS and third party resources.

    [More info on Cloud Formation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)



### Let's start putting this together:

* First - Create user in AWS so you can have programmatic access via Access Keys (You can use AWS CLI: 
[Cli configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html))

Then:

#### Cloud Formation - Infrastructure

<script src="https://gist.github.com/piotrgrota/823450a426a0ca6909f667d26c4c0774.js"></script>

[Create Stack](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/)

After creation of Stack please try to generate data using sample Python script

#### Producer Code - our application that simulate data

<script src="https://gist.github.com/piotrgrota/88d060266fb3eea0d8a5f5bb42746dea.js"></script>


#### Apply SQL and see results

<script src="https://gist.github.com/piotrgrota/765cdf8eeb6fe0d11ccdb6be69728504.js"></script>

You can use we used few aggregation functions and we setup our window to be 3 minutes . You can read about Tumbling Window later...


After some time you should be able to see similar results:

![_config.yml]({{ site.baseurl }}/images/post3/RealTimeDataSQL.gif)

You should see data are showing up "Analysis" in Real Time based on data. Was that hard I don't think so..

Let's try to break it down:

#### Concept used in solution:

* SQL is based on concept Tumbling Window:

In a tumbling window, tuples are grouped in a single window based on time or count. A tuple belongs to only one window.

For example, consider a time-based tumbling window with a length of five seconds. The first window (w1) contains events that arrived between the zeroth and fifth seconds. The second window (w2) contains events that arrived between the fifth and tenth seconds, and the third window (w3) contains events that arrived between tenth and fifteenth seconds. The tumbling window is evaluated every five seconds, and none of the windows overlap; each segment represents a distinct time segment.

![_config.yml]({{ site.baseurl }}/images/post3/tumbling-window.png)


[More Info about Window Queries](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/windowed-sql.html)


* Shard for Kinesis Input Stream

    A shard is a uniquely identified sequence of data records in a stream. A stream is composed of one or more shards, each of which provides a fixed unit of capacity. Each shard can support up to 5 transactions per second for reads, up to a maximum total data read rate of 2 MB per second and up to 1,000 records per second for writes, up to a maximum total data write rate of 1 MB per second (including partition keys). The data capacity of your stream is a function of the number of shards that you specify for the stream. The total capacity of the stream is the sum of 
    the capacities of its shards.

    Concept is similar to Kafka Partition. I created only 1 so you won't pay too much

* Partition key
    A partition key is used to group data by shard within a stream. Kinesis Data Streams segregates the data records belonging to a stream into multiple shards. 
    Concept is similar to Kafka Partition Key


You may wonder why there is no Kinesis Output Stream code according to architecture it should be (I wanted to be an exercise to reader)

Please delete you stack created via Cloud Formation so you won't pay more.


## Conclusion:

You could see how easy it is to to create real time analysis application without setup of virtual machine.
I was only focusing on code itself. Please keep in mind that in Production environment I would require to calculate proper
shards for our Kinesis stream, monitoring and maybe use [Apache Spark](https://spark.apache.org/) in order to provide more Complex Analysis.


Code for whole application here: [GitHub Code](https://github.com/piotrgrota/aws_playground/tree/master/aws_streaming_app)


