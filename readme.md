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

* to process our first event we need to configure the pipeline. a pipeline consists of 3 phrases. inputs filters and outputs. 
* we will use only inputs and outputs for now to keep it simple
* to set it up we will write a config file
* logstash uses proprietary syntax
* apart from using config file we can set apipeline as argument when we lauch logstash
* to pass a pipeline config as an argument we use the flag `-e`
* our simple pipeline will input from stdin and output to stdout
* the syntax fr the pipeline configuration as argument is a string literal with `input  { } and output { }` (object/hash)to define our plugins. we use stdin plugin and stdout without further config so it is `"input { stdin { } } output { stdout { } }"`
* we add the --debug flag for more verbose output
* we star logstash. when we type input in terminal we get it back in terminal. we see that the output is timestammped and carries a uuid
```
{
      "@version" => "1",
    "@timestamp" => 2018-05-12T13:12:24.951Z,
          "host" => "achliopa-ThinkPad-T530",
       "message" => "hello from logstash"
}
```
* we will set our pipeline config in a file which is advisable
* in the cofig directory we make a folder names *pipelines* and add a file named *pipeline.conf". the file is plain text file and can have any filetype we want(logstash conv is .conf). we write our simple pipeline conf in it
```
input {
	stdin {

	}
}

output {
	stdout {
		
	}
}
```
* to launch logstash with a pipeline set in a file we use the -f flag and a relative or absolute path to the file `bin/logstash -f config/pipelines/pipeline.conf `
* even though he have not set up any filter logstash process the events by adding extra fields before outputing
* we can add a codec named rubydebug with will output the events as json objects
* ruby debug uses a ruby lib called awesome print
```
output {
	stdout {
		codec => rubydebug
	}
}
```
* everytime we modify the pipeline config we need to restart logstash to use it
* it seems rubydebug is standard now as the output does not change
* the fields are: timestamp , host (ip in external hosts, hostname in local): message for the even 

* Lecture 6 - handling JSON Input

* in this lecture we will learn how to handle input JSON, with JSON we can input structured date with multiple fields
* for now we will i nput JSON directly from the terminal , my simple pipeline treats its as a string literal.  `message" => "{ \"amount\": 10, \"quanity\": 2 }",`
* to fix that we will use a codec named json in input `codec => json` of our pipeline
* we run again our input json string in terminal `{ "order": 3232, "amount": 23.5 }
` and get back in output its fields extracted
```
{
         "order" => 3232,
    "@timestamp" => 2018-05-12T14:28:35.406Z,
        "amount" => 23.5,
      "@version" => "1",
          "host" => "achliopa-ThinkPad-T530"
}
```
* if we input an array of json objects logstash will create an event for each object
```
[{"order":12,"amount":23.4},{"order":13, "amount":12.4}]
{
         "order" => 13,
    "@timestamp" => 2018-05-12T14:32:42.307Z,
        "amount" => 12.4,
      "@version" => "1",
          "host" => "achliopa-ThinkPad-T530"
}
{
         "order" => 12,
    "@timestamp" => 2018-05-12T14:32:42.307Z,
        "amount" => 23.4,
      "@version" => "1",
          "host" => "achliopa-ThinkPad-T530"
}

```
* if we pass erroneous json we get back an event with a new field for the error
```
{ error in json}
[2018-05-12T17:34:46,743][WARN ][logstash.codecs.jsonlines] JSON parse error, original data now in message field {:error=>#<LogStash::Json::ParserError: Unexpected character ('e' (code 101)): was expecting double-quote to start field name
 at [Source: (String)"{ error in json}"; line: 1, column: 4]>, :data=>"{ error in json}"}
{
    "@timestamp" => 2018-05-12T14:34:46.745Z,
          "tags" => [
        [0] "_jsonparsefailure"
    ],
      "@version" => "1",
       "message" => "{ error in json}",
          "host" => "achliopa-ThinkPad-T530"
}

```
* logstash does not discard the events even when there is an error
* our pipeline now is able to handle signle lines of inout in terminal. if i write json bojects spanning multiple lines i get parsing error
* multiline events are handled differently

### Lecture 7 - Outputting Events in File

* we can have multiple outputs. we just stack the plugins in the pipeline config file. 
* we add a file pgugin in the output phrase. in its config object we set the poutput file path (relative to the logstash install directory or absolute)
```
	file {
		path => "output.txt"
	}
``` 
* we restart logstash and input a json string to the terminal. our event is recorded in the file as JSON object `{"order":32,"@version":"1","@timestamp":"2018-05-12T14:50:47.497Z","host":"achliopa-ThinkPad-T530","amount":233}
` actually a minified version with one event/line to enable easy parsing to other programs
* processing has near realtime performance

### Lecture 8 - Working with HTTP Input

* we will now see how we send events to our pipeline through http using an input plugin, the plugin is called http. there are other input plugins for tsp or udp
* in its simplest form the http input plugin gets no  config options. this means it will listen to localhost port 8080. we set host and port and restart logstash
```
	http {
		host => "127.0.0.1"
		port => 8080
	}
```
* we will use postman to send the request, curl is also OK
* we use PUT method to localhost:8080, in body we select raw-> json (application/json)
* we add the json in body and send the request
* the event is outputed together withthe http header  as json in the file and as json type in stdout

```
{
       "headers" => {
                "http_version" => "HTTP/1.1",
          "http_cache_control" => "no-cache",
             "http_user_agent" => "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36",
                   "http_host" => "localhost:8080",
                "content_type" => "application/json",
                 "http_origin" => "chrome-extension://fhbjgbiflinjbdggehcddcbncdddomop",
                "request_path" => "/",
                 "request_uri" => "/",
             "http_connection" => "keep-alive",
          "http_postman_token" => "1b23280e-8fb7-5aa9-28f7-eb5240f5f418",
        "http_accept_language" => "en-US,en;q=0.8,el;q=0.6",
              "content_length" => "38",
        "http_accept_encoding" => "gzip, deflate, br",
              "request_method" => "PUT",
                 "http_accept" => "*/*"
    },
    "@timestamp" => 2018-05-12T15:13:54.794Z,
         "order" => 2345,
          "host" => "127.0.0.1",
        "amount" => 3445.89,
      "@version" => "1"
}
```
* we see that logstash treats both objects as a multiline event and puts all in one line. this is because http plugin treat each request as single event regardles how many json objects we pass in the body
* with curl our request is `curl -XPUT -H "Content-Type: application/json" -d '{"order": 4343,"amount": 2312.45}' http://localhost:8080`

### Lecture 9 - Filtering Events

*  [all mutate options](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html)