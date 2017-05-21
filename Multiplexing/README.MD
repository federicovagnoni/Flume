### Requirement: 

From the source, we are getting data in the form of employee_role, employee_id, employee_name, employee_city. 
Now, the source could be real time streaming or it could be any other source.
In the example, I have used netcat service as a source that listens on a given port and turns each line of text into an event. 
We have to store data HDFS in the text format, and data has to be stored separately based on role. Manager’s data is to be stored at flume_multiplexing_data/manager and developer’s data at flume_multiplexing_data/developer. Here, we have two sinks pertaining to HDFS and corresponding two channels. 

For example, following data should be stored under flume_multiplexing_data/manager (employee_role = 1):
Role,ID,Name,City
1, E1,Viren,mumbai
1, E3,Ofer,kolkata

following data should get stored under flume_multiplexing_data/developer (employee_role = 2) :

2, E4,Sanjeev,delhi
2, E6,Amruta,banglore

#### How to achieve this requirement?

Using regex_extractor interceptor and multiplexing. 
Regext_extractor interceptor extracts regex match groups using a specified regular expression and appends the match groups as headers on the event. It also supports pluggable serializers for formatting the match groups before adding them as event headers.

# Describe regex_extractor to extract different patterns
# Capture event attribute and add it as event header with attribure name as "role"
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = regex_extractor
a1.sources.r1.interceptors.i1.regex = ^(\\d)
a1.sources.r1.interceptors.i1.serializers = t
a1.sources.r1.interceptors.i1.serializers.t.name = role


By multiplexing, flume allows to multiplex event flow to one or more destinations. 
It can replicate the event or route the event to selective channels.  
Managers data to be routed to channel c1 and developers data to be routed to channel c2.

# Define channel selector and define mapping
a1.sources.r1.selector.type = multiplexing
a1.sources.r1.selector.header = type
a1.sources.r1.selector.mapping.1 = c1
a1.sources.r1.selector.mapping.2 = c2

Using HDFS sink HDFS sink allows data to be stored in HDFS using text/sequence files and also it can store data in compressed format.

managers data - routed to channel c1 and stored under flume_data_multiplexing/manager

# Describe first HDFS sink k1 to store manager's data only, its associated with channel c1
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = flume_data_multiplexing/manager
a1.sinks.k1.hdfs.fileType=DataStream
a1.sinks.k1.hdfs.writeFormat=Text

Developer’s data - routed to channel c2 and stored under flume_multiplexing_data/developer
# Describe HDFS sink k2 to store developer’s data only, its associated with channel c2
a1.sinks.k2.channel = c2
a1.sinks.k2.type = hdfs
a1.sinks.k2.hdfs.path = flume_data_multiplexing/developer/
a1.sinks.k2.hdfs.fileType=DataStream
a1.sinks.k2.hdfs.writeFormat=Text

#### How to run the example?

First of all, you would require Hadoop to run the example as sink is hdfs. If you dont have Hadoop cluster then change sink to log and then just setup flume. Store flume.conf in some directory and run the agent using following command -

flume-ng agent --conf conf --conf-file flume_conf.conf --name a1 -Dflume.root.logger=INFO,console

Note that agent name is a1.

I have used netcat as a source. a1.sources.r1.type = netcat a1.sources.r1.bind = localhost a1.sources.r1.port = 44444

So, once flume agent starts, run following command to send events to Flume.
telnet localhost 44444

Its almost done now,just provide the input text

1, E1,Viren,mumbai
2, E2,John,chennai
1, E3,Ofer,kolkata
2, E4,Sanjeev,delhi
1, E5,Ramesh,pune
2, E6,Amruta,banglore
