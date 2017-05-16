### Requirement: 

From the source, we are getting employees records.
Now, the source could be real time streaming or it could be any other source.
In the example, I have used netcat service as a source that listens on a given port and turns each line of text into an event. 
Requirement is to replicate this data to two sinks i.e following HDFS directories 

 sink1 - hdfs://<HADOOP_HOME>/flume_data_replicating/employee_records1
 
 sink2 - hdfs://<HADOOP_HOME>/flume_data_replicating/employee_records2 

#### How to achieve this requirement?

Using channel selector as replicating, two sinks associated with two sinks

#### How to run the example?

First of all, you would require Hadoop to run the example as sink is hdfs. 

flume-ng agent --conf conf --conf-file flume_replicating_conf.conf --name a1 -Dflume.root.logger=INFO,console

Note that agent name is a1.

I have used netcat as a source. a1.sources.r1.type = netcat a1.sources.r1.bind = localhost a1.sources.r1.port = 44444

So, once flume agent starts, run the following command to send event to Flume.
telnet localhost 44444

Its almost done now,just provide the input text -

E1,Viren,mumbai,1

E2,John,chennai,2

E3,Ofer,kolkata,1

E4,Sanjeev,delhi,2

E5,Ramesh,pune,1

E6,Amruta,banglore,2

To view results navigate to HDFS directories configured as sinks.
