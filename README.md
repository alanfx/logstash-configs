# logstash-configs

This is a [Logstash](http://logstash.net/) configuration file that can be used to parse [RadarGun](https://github.com/radargun/radargun/wiki)  logs into Elasticsearch, so that Kibana can be used to browse the results. In order to do this, the logs are parsed, and the day in the log messages is set to (today - 1 day) to show up in the Kibana interface close to the current time.

The attached Logstash config file does the following:
- The input section reads the specified RadarGun log files. Note that you can use wildcard characters to parse multiple files
- The filter section:
  - Parses RadarGun log4j messages into the following fields:
    - @timestamp: Used by Logstash and Elasticsearch to specify the time of the event
    - category: The name of the class where the log message originated
    - host: The name of the node taken from the log file name
    - level: the Log level of the message
    - thread: The name of the thread where the log message originated
    - summary: The message with the timestamp, level, category, and thread removed
    - message: The full text of the log message
  - Add a "RadarGun" tag to the event
  - Parses GC log messages that are interspersed with the RadarGun logs into the following fields:
    - @timestamp: Used by Logstash and Elasticsearch to specify the time of the event
    - host: The name of the node taken from the log file name
    - user: user time for the collection
    - sys: system time for the collection
    - real: real time for the collection
    - summary: The message with the timestamp removed
    - message: The full text of the log message
    - Add a "GCLog" tag to the event, and a "FullGC" tag to log messages that contain the text "Full GC"
  - Creates multiline events that get a "multiline" tag
  - Adds a tag to any log messages that start with the "ISPN|ARJUNA|JDGS|JGRP######: " string
  - Parses messages containing GlobalTransactions into trans_host, trans_id, and loc fields
  - Adds a "MergeView" tag for MergeView messages
  - Parses key IDs into a key_id field and adds a "key_found" tag
  - Adds a "availability" tag for cache availability transitions
  - Any events tagged with "_grokparsefailure" are dropped
- The output section writes the events to Elasticsearch and stdout in JSON format
 
# TODO:
- Improve performance of parsing. There are currently a lot of grok rules.
- Can some rules be dropped? Do rules need to be added?
- Check the performance of using seperate Logstash, Elasticsearch, and Kibana processes
