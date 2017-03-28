Requirment:
From the source, we are getting data in the form of sr.no, name,city,role.
Now, the source could be real time streaming or it could be any other source. 
In the example, I have used netcat service as a source that listens on a given port and turns each line of text into an event. 
Requirement is to save the data into HDFS in the text format and before saving data to HDFS, it has to filtered on the basis of role.
Only managers data has to be saved into HDFS and data with other roles have to be ignored.
For example, following data is allowed : 

1,alok,mumbai,manager

2,jatin,chennai,manager

following data is not allowed :

3,yogesh,kolkata,developer

5,jyotsana,pune,developer

How to achieve this requirement?
1. Using regex_filter interceptor. 
This interceptor filters events selectively by interpreting the event body as text and matching the text against a configured regular expression. 
The supplied regular expression can be used to include events or exclude events.

a1.sources.r1.interceptors = i1

a1.sources.r1.interceptors.i1.type = regex_filter

a1.sources.r1.interceptors.i1.regex = developer

a1.sources.r1.interceptors.i1.excludeEvents = true


2. Using HDFS sink
HDFS sink allows data to be stored in HDFS using text/sequence files and also it can store data in compressed format.

a1.channels = c1

a1.sinks = k1

a1.sinks.k1.type = hdfs

a1.sinks.k1.channel = c1

a1.sinks.k1.hdfs.path = hdfs://quickstart.cloudera:8020/user/hive/warehouse/managers ##assumption is that Hadoop is CDH

a1.sinks.k1.hdfs.fileType= DataStream

a1.sinks.k1.hdfs.writeFormat = Text

How to run the example?
First of all, you would require Hadoop to run the example as sink is hdfs.
If you dont have Hadoop cluster then change sink to log and then just setup flume.
Store flume.conf in some directory and run the agent using following command -

flume-ng agent --conf conf --conf-file flume_conf.conf --name a1 -Dflume.root.logger=INFO,console

Note that agent name is a1.

I have used netcat as a source.
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

So, once flume agent starts, run following command on other terminal to start netcat 
telnet localhost 44444

Its almost done now,just provide the input text 

1,alok,mumbai,manager

2,jatin,chennai,manager

3,yogesh,kolkata,developer

4,ragini,delhi,manager

5,jyotsana,pune,developer

6,valmiki,banglore,manager

Navigate to console in which you have started flume agent. You would observe that a file would have got created in HDFS under 
hdfs://quickstart.cloudera:8020/user/hive/warehouse/managers with filtered data containing only manager records.
