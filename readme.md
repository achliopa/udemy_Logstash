# Udemy Course: Processing Events with Logstash

* [Course](https://www.udemy.com/processing-events-with-logstash/learn/v4/overview)
* [Github Repo](https://github.com/codingexplained/processing-events-with-logstash)

## Section 1 - Getting Started

### Lecture 2 - Introduction to Logstash

* Logstash is an open source event processing engine
* with logstash we configure a pipeline responsible for receiving, processing and shipping events
* we can send date to logstash from many different sources and logstash can ship them to many destinations
* it packs functionality to manipulate data (cleaning and enriching them)
* very flexible and configurable
* a logstash pipeline is an orchestration of plugins
* a plugin can receive, manipulate or ship data
* 100s of built in plugins + external
* pipeline: input+(filter)+output (3phrases, each one uses plugins)
* * input and output phrases can use codecs to change data representation of events
* the destination where processed events are sent are called Stashes (e.g Elasticsearch, Kafka)
* the tool was originaly built to handle logs but its not limited to that it can handle html,csv, json etc
* Strong synsergy with ElasticSearch, Kibana + Beats (ELK stack)
* Logstash is horizontaly scalable
* Motivation to use Logstash: why not send events directly to elasticsearch, or eemails from app to elasticsearch
	* Decoupled architecture, centralized event processing @ logstash

### Lecture 3 - Installing Logstash on Mac/Linux

* everything we need to run logstash is packet in an archive (writen in ruby runs on jruby runs ruby in jvm)
* no need to install anything. just run the script
* we need to have java 8 on our machine (have it) jre
* we [download](https://www.elastic.co/downloads/logstash) logstash. we chose the tar.gz file
* we extract it to the destination folder `tar -zxvf`
* in our destination fiolder we have a config dir. here we can configure the number of pipelines we want to have, logging options, mem settings for jvm
* we have a data dir, with date logstash uses internally like
* logs dir hosts logfiles for debugging
* bin directory has the executables also hosts ruby 
* we run logstash from inside bin dir with a command like `./logstash -e "input { stdin { } } output { stdout { } }"`
* it stops and we have a simple pipeline running
* it stops with ctr+c

### Lecture 4 - Intassling Logstash on Windows

* unzip the zip file
* open up cmd
* go to the destination dir
run with `bin\logstash -e "input { stdin { } } output { stdout { } }"`

## Section 2 - Basics of Logstash

### Lecture 5 - Processing our First Event

* 