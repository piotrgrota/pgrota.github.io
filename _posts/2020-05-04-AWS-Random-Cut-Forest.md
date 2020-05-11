---
layout: post
title: How to detect Fraud on Real Time Data ?
---


![_config.yml]({{ site.baseurl }}/images/post4/3-Figure2-1.png)

### Problem:

How we can detect Financial Fraud on real-time data ? Let's dive in and see how we can implement this. 
We use simple balance data generation for clients to simulate it.

<b>Use case: </b> 

In finance, online anomaly detection algorithms are used to alert customers to potentially fraudulent credit card transactions

### Algorithm:

Anomaly detection is one of the cornerstone problems in
data mining. Even though the problem has been well studied over the last few decades, the emerging explosion of
data from the internet of things and sensors leads us to reconsider the problem .

As the name suggest we will construct (Random), cuts them to the same number of points and creates trees (Cut). It then looks at all of the trees together (Forest) to determine whether a particular data point is an anomaly: Random Cut Forest.

Let's see by example :

#### Random Cut Forest.

* Below is example of data points - let's say you have transaction balance for particular client

![_config.yml]({{ site.baseurl }}/images/post4/chart1.png)

How we can say if balance marked as white point is anomaly or not 
<br />

* We insert line at random location through the data points. 
Each side of the subdivision represents a node in the tree.

![_config.yml]({{ site.baseurl }}/images/post4/chart2.png)

You can see node <b>A</b> represent a root node (which always exists and that is the projection of all data points) and <b>B </b> , <b>C</b> represent both left and  right side of division.
Color is determining if we have any nodes on that side apart  tested one. 

<br />

* Put another line to divide points

![_config.yml]({{ site.baseurl }}/images/post4/chart3.png)

You can see our tree is expanding .
Node <b>B</b> now has been expanded into two more <b>D</b> and <b>C</b>.  Still we have more nodes on both sides of division so we need to continue.


<br />

* Put another line to divide points


![_config.yml]({{ site.baseurl }}/images/post4/chart4.png)

You can see  we divided this time right side but finally we get to our tested point.
<b> F</b> is white - because we have only tested on this side
on right side we node that is not our tested node. 


The final step performed by the Random Cut Forest algorithm is to combine the trees into a forest. If lots of the samples have small trees then the target data point is likely to be an anomaly. If only a few of the samples have small trees then it’s unlikely to be an anomaly.


#### Implementation on real time data :

We will use [AWS](https://aws.amazon.com/) and [Kinesis](https://aws.amazon.com/kinesis/) since it can be used for real time data. 


Below is sample producer that will produce Transaction balance for our clients: 

<script src="https://gist.github.com/piotrgrota/a6fb62591861b0070a70246bd00353f5.js"></script>


Now let's use SQL and use Random Cut Forest Algorithm:

Stream were created using this [Cloud Formation Template](https://raw.githubusercontent.com/piotrgrota/aws_playground/master/aws_streaming_app/template.yml)

<br />
SQL require following parameters and return JSON object

Parameters: 

* numberOfTrees - Using this parameter, you specify the number of random cut trees in the forest. (in our example it is 100)
* subSampleSize - Using this parameter, you can specify the size of the random sample that you want the algorithm to use when constructing each tree. (in our example it is 256)
* timeDecay - You can use the timeDecay parameter to specify how much of the recent past to consider when computing an anomaly score. (in our example it is 100000)
* shingleSize - a shingleSize of 10 at time t corresponds to a vector of the last 10 records received up to and including time t. The algorithm treats this sequence as a vector over the last shingleSize number of records. 
* withDirectionality - A Boolean parameter that defaults to false. When set to true, it tells you the direction in which each individual dimension makes a contribution to the anomaly score. It also provides the strength of the recommendation for that directionality.


Returns:

* Attribution score: A nonnegative number that indicates how much this column has contributed to the anomaly score of the record. In other words, it indicates how different the value of this column is from what’s expected based on the recently observed trend. The sum of the attribution scores of all columns for the record is equal to the anomaly score.

* Strength: A nonnegative number representing the strength of the directional recommendation. A high value for strength indicates a high confidence in the directionality that is returned by the function. During the learning phase, the strength is 0.

* Directionality: This is either HIGH if the value of the column is above the recently observed trend or LOW if it’s below the trend. During the learning phase, this defaults to LOW.


<script src="https://gist.github.com/piotrgrota/593f40b1490b71ab6749b4f12234bd98.js"></script>


With help of AWS we can calculate Anomaly Score with explanation:

Sample Data produced by this query

<br />

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:7px 3px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;background-color:yellow;}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}

</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-0pky">  ROWTIME </th>
    <th class="tg-0pky">USER</th>
    <th class="tg-0pky">ANOMALY_SCORE</th>
    <th class="tg-0pky">ANOMALY_EXPLANATION</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0pky">2020-05-11 16:58:18</td>
    <td class="tg-0pky">hai</td>
    <td class="tg-0pky">0.85799485</td>
    <td class="tg-0pky">{"balance":{"DIRECTION":"LOW","STRENGTH":"0.0393","ATTRIBUTION_SCORE":"0.8580"}}</td>
  </tr>
  <tr>
    <td class="tg-0pky">2020-05-11 16:58:18</td>
    <td class="tg-0pky">ellis</td>
    <td class="tg-0pky">0.80416286</td>
    <td class="tg-0pky">{"balance":{"DIRECTION":"LOW","STRENGTH":"0.0650","ATTRIBUTION_SCORE":"0.8042"}}</td>
  </tr>
  <tr>
    <td class="tg-0pky">2020-05-11 16:58:18</td>
    <td class="tg-0pky">ellis</td>
    <td class="tg-0pky">0.9453035</td>
    <td class="tg-0pky">{"balance":{"DIRECTION":"LOW","STRENGTH":"0.0799","ATTRIBUTION_SCORE":"0.9453"}}</td>
  </tr>
  <tr>
    <td class="tg-0pky">2020-05-11 16:58:18</td>
    <td class="tg-0pky">ellis</td>
    <td class="tg-0pky">0.84835047</td>
    <td class="tg-0pky">{"balance":{"DIRECTION":"HIGH","STRENGTH":"0.0502","ATTRIBUTION_SCORE":"0.8484"}}</td>
  </tr>
  <tr>
    <td class="tg-0pky">2020-05-11 16:58:18</td>
    <td class="tg-0pky">randi</td>
    <td class="tg-0pky">0.94130754</td>
    <td class="tg-0pky">{"balance":{"DIRECTION":"LOW","STRENGTH":"0.0805","ATTRIBUTION_SCORE":"0.9413"}}</td>
  </tr>
</tbody>
</table>


<br />


<script src="https://gist.github.com/piotrgrota/da8366ee2f8184ebc6095fb57b514e6b.js"></script>

* Little more complex example:
    * First we are grouping data by 1 minute Time Windows and do sum
    * Then we are calculating anomaly for data
    * We used different parameters since they are less data and I wanted to show some results 



<br />

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:7px 3px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;background-color:yellow;}
.tg .tg-4yk9{border-color:inherit;color:rgb(0, 0, 0);text-align:left;vertical-align:bottom}
.tg .tg-0lax{text-align:left;vertical-align:top}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-4yk9">ROWTIME</th>
    <th class="tg-4yk9">User</th>
    <th class="tg-4yk9">SUM_BALANCE</th>
    <th class="tg-4yk9">ANOMALY_SCORE</th>
    <th class="tg-0lax">ANOMALY_EXPLANATION</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-4yk9">2020-05-11 16:18:01</td>
    <td class="tg-4yk9">adam</td>
    <td class="tg-4yk9">6977</td>
    <td class="tg-4yk9">1.039002096</td>
    <td class="tg-0lax">{"SUM_BALANCE":{"DIRECTION":"LOW","STRENGTH":"1442.9215","ATTRIBUTION_SCORE":"1.0390"}}</td>
  </tr>
  <tr>
    <td class="tg-4yk9">2020-05-11 16:18:01</td>
    <td class="tg-4yk9">randi</td>
    <td class="tg-4yk9">8624</td>
    <td class="tg-4yk9">1.018373352</td>
    <td class="tg-0lax">{"SUM_BALANCE":{"DIRECTION":"LOW","STRENGTH":"372.4000","ATTRIBUTION_SCORE":"1.0184"}}</td>
  </tr>
  <tr>
    <td class="tg-4yk9">2020-05-11 16:18:01</td>
    <td class="tg-4yk9">piotr</td>
    <td class="tg-4yk9">0</td>
    <td class="tg-4yk9">0.9440401252</td>
    <td class="tg-0lax">{"SUM_BALANCE":{"DIRECTION":"LOW","STRENGTH":"145.4000","ATTRIBUTION_SCORE":"0.9440"}}</td>
  </tr>
  <tr>
    <td class="tg-4yk9">2020-05-11 16:18:01</td>
    <td class="tg-4yk9">jack</td>
    <td class="tg-4yk9">9784</td>
    <td class="tg-4yk9">0.906500485</td>
    <td class="tg-0lax">{"SUM_BALANCE":{"DIRECTION":"LOW","STRENGTH":"211.7747","ATTRIBUTION_SCORE":"0.9065"}}</td>
  </tr>
  <tr>
    <td class="tg-4yk9">2020-05-11 16:18:01</td>
    <td class="tg-4yk9">david</td>
    <td class="tg-4yk9">11341</td>
    <td class="tg-4yk9">0.8904428845</td>
    <td class="tg-0lax">{"SUM_BALANCE":{"DIRECTION":"HIGH","STRENGTH":"378.5098","ATTRIBUTION_SCORE":"0.8904"}}</td>
  </tr>
</tbody>
</table>




### Conclusion 

Machine Learning algorithms can do great jobs on Real Time data and with AWS you can do easy detection Fraud with knowing SQL only.
More info below: 

[Kinesis Random Cut Forest](https://docs.aws.amazon.com/kinesisanalytics/latest/sqlref/sqlrf-random-cut-forest-with-explanation.html)

