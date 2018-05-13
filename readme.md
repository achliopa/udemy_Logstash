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
* filters is the second step in the pipeline. they process events before handling them over to the pipeline. 
* we will see the mutate filter and use it to convert datatype to the amount string -> int.
* to sse it in action we sent a JSON file with  the order as string in postman. it is passed to the output as string
```
"@timestamp" => 2018-05-12T18:38:53.358Z,
         "order" => "2346",
          "host" => "127.0.0.1",
        "amount" => 3400.89,
      "@version" => "1"
}
```
* we use the mutate filter in our pipeline config file to convert the order field to an integer. the syntax is 
```
filter {
	mutate {
		convert => { "order" => "integer" }
	}
}
```
* we resend the request ro postma with order as string and we see it in output as integer. but both in file and in terminal it is in the end of the event object alone (processing delay?>)
* the mutate makes sense if e.g we send the events to elasticsearch so they must match mappings
* logstash is optimistic on the data it receives. so if ewe semd a string literal the integer it convert to will be 0
* mutation conver show not be used as validation but as conversion and protection against erroneous systems

### Lecture 10 - Common Filter Options

* logstash offers nummerous mutate options and numberous filter plugins. 
* also it offers common filter options which are generic and not specific to a certain plugin
* *add_filed* option adds one or more fields to the event
* *remove_field* removes one or more fields from the event
* *add_tag* adds one or more tags to the event
* *remove_tag* removes one or more tags from the event
* its useful to tag events and perform conditional processing depending on what tags they contain
* we will showcase common fitler options by removing the generated host firld from the event
```
filter {
	mutate {
		convert => { "order" => "integer" }
		remove_field => ["host"]
	}
}
```
* we see that host field is missing from the output

### Lecture 11 - Understanding the Logstash execution model

* we have a number of inputs listenign for events, each input runs in its own thread to avoid blocking each other, if we have 2 events at the same time, they will be handled concurently
* inputs can apply codecs
* each input sends the event to a work queue
* from the work queue loggstash uses pipeline workers to do the rest of the work
* each worker includes filters outputs and any output codecs.
* each pipeline worker runs in its own thread (multiple events can be processed simultatneously)
* a pipeline worker consumes a range of events from the queue (batches)
* this is a way to optimize event processing to maximize the throughput of the pipeline
* the batch size is configured with 2 options. the maximum batch size and batch delay
* batch delay is the time to wait before processing an undersized batch of events. say we set max size to 50 and delay to 100ms. then a batch will be processed if the size is 50 or if 100ms has passed
* some pipelines output batches of events (using output plugins e.g elasticsearch output plugin) to make us of the bulk api exposed by elastic search so the whole pipeline is multievent start to finish
* the amount of threads started by logstash is determined t startup by evaluating the available hardware on the machine (cpu cores)
* it is possible to configure a fixed number of workers
* logstash uses queues within each pipeline worker (e.g between filters and outputs) this is the buffer of events completeing one stage and waiting for the other. these are in-memory queues. 
* if logstash crashes. events in these queues will be lost
* we can configure pipeline to use disk space for the queues. we scarifice performance bu t data are not lost in such event

## Section 3 - Project Apache

### Lecture 13 - Introduction to the section

* in this section we will build a project to handle logs from the apache server
* we will begin with access logs, read them from files and parse them in  a way that useful fields will be extracted, 
* we will send the processed events to elastic search and use kibana to visualize the data
* we will handle error logs also

### Lecture 14 - Automatic config reload & file input

* we start by building a simple pipeline configuration 
* we will use a testfile with a handful of http requests
* in our project dir we create a folder event-data with a testfile named apache-access.log
* we also create a pipelines folder with a pipeline.conf file impelemnting a simple pipeline
```
input {
	stdin {}
}

output {
	stdout {
		codec => rubydebug
	}
}
```
* this is more of a pipeline boilerplate tht listens for events in command line and outputs the m in terminal
* we want to enable auto reload of our config file so that we dont have to restart logstash every time we change it. we use the `--config.reload.automatic` option when running logstash
* we start logstash using the script and the autoreload option `/logstash-6.2.4/bin/logstash -f pipelines/pipeline.conf --config.reload.automatic`
* our pipeline is simple and useless. we mod it to read from the test file
```
  file {
    path => "/home/achliopa/workspace/udemy/logstash/myfolder/project/event-data/apache_access.log"
  }
```
* Autoreload does not work because we were using the stdin plugin. autoreload config has issues with stdin. so we restart logstash
* no events are processed from teh file. this is because the file plugin processes new lines added ot he file not the existing ones
* we can change that by adding a config option `start_position => "beginning"` to start processing from the beginning of the file. 
* this happens only the first time because logstash to save memory keeps track of the lines it has processes and does not process them again
* logstash saves status in a sinceDB file in /logstash/data/plugins/inputs/file
* it keeps one such file per input file.
* the last number in this file is the byte offset. so that logstash knows how many bytes of the file have been processed. the file is persisten among logstash restarts. we delete it to process form start
* we restart logstash and our file is indeed processes and a stream, of events outputed to console
```
    "@timestamp" => 2018-05-12T21:28:22.427Z,
          "path" => "/home/achliopa/workspace/udemy/logstash/myfolder/project/event-data/apache_access.log",
          "host" => "achliopa-ThinkPad-T530",
       "message" => "78.133.44.127 - - [20/Sep/2017:18:42:01 +0200] \"GET /css/admin.css\" 200 12964 \"https://codingexplained.com/admin/products\" \"Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36\"",
      "@version" => "1"
}
```
* we add a line to the file to see that is gets processed. we save and indeed its processed as an event
* instead of modifying our test log file we will add http input in the pipeline to test it with http requests
* to add comments in config files use hash # in front of the line
```
# file {
#   path => "/home/achliopa/workspace/udemy/logstash/myfolder/project/event-data/apache_access.log"
#   start_position => "beginning"
# }

  http {

  }
}
```
* autoreload takes action and our pipeline is reconfigured. autoreload checks every 3 seconds for changes, we can set that as an option
* what happens when we reload.
  * logstash periodically reads the pipeline config file from disk (every 3secs by default)
  * it checks if the contents have changed compared to the currently used configuration. 
  * if its the same it sleeps
  * if it has changed it verifys the new pipeline configuration, if invalid it sleeps
  * if it is valid. it stops existing pipeline
  * lastly it starts the new pipeline
* we test the new config with an http req of raw text body to localhost:8080 (PUT)
* it works our whole bocy message is processes as a string
```
{
    "@timestamp" => 2018-05-12T21:41:42.885Z,
          "host" => "0:0:0:0:0:0:0:1",
       "message" => "173.163.21.50 - - [20/Sep/2017:19:01:42 +0200] \"GET /products/view/124\" 404 4160 \"-\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36\"",
      "@version" => "1",
       "headers" => {
              "content_length" => "208",
                "request_path" => "/",

```

### Lecture 15 - Parsing Requests with Grok

* [grok patterns](https://github.com/logstash-plugins/logstash-patterns-core)
* [grok filter plugin documentation](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)
* now that are processing the access log events, we need to parse them to usful info extracting their fiedls with useful info.
* we will use the grok filter plugin which is used to make sense of unstructured data
* this is done by configuring text patterns with regex. we can add our one regex but grok included a lot of ready made
* there is a github repo with all the available grok patterns. in patterns dir
* [grok-patterns](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns) file contains general purpose patterns (regex) for datatypes
* we can use these patterns identifiers in the plugin and not mess with regex
* the syntax of adding patterns to grok is %{SYNTAX:SEMANTIC}. syntax is the regex or patterns identifier
* semantic  is a semantic name for the data extracted with the syntax. like a field name
* suppose we have a string of text "John Dow john@dow.com 32" containg info about a person. if we want to parse this unstructured text to STRUCTURED fields we can use %{WORD:firs_name} %{WORD:last_name} %{EMAILADDRESS:email} %{INT:age:int}
* note that for numbers we add a second trasformation after SEMANTIC. this is becouse parsing produces strings and we want the value to be an integer
* we add a grok filter to our pipeline
```
filter {
  grok {
    match => { "message" => "%{IP:ip_address} %{USER:identity} %{USER:auth} \[%{HTTPDATE:req_ts}\] \"%{WORD:http_verb} %{URIPATHPARAM:req_path}\" %{INT:http_status:int} %{INT:num_bytes:int}" }
  }
}
```
* tyhis filter will be a grok match filter on the "message" field of the event (the body). `173.163.21.50 - - [20/Sep/2017:19:01:42 +0200] "GET /products/view/124" 404 4160 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"`
* we see that it aplies SYNTAX:SEMANTIC pairs on one by one the areas of the string wrapping the match patterns on characters that we want excluded. we use specialized syntax patterns from the large list avaialble
* we sent an http req to localhost:8080 from postman and see the fields extracted
```
...
           "host" => "0:0:0:0:0:0:0:1",
     "ip_address" => "173.163.21.50",
       "@version" => "1",
           "auth" => "-",
     "@timestamp" => 2018-05-13T15:12:58.666Z,
      "http_verb" => "GET",
      "num_bytes" => 4160,
    "http_status" => 404,

          "req_ts" => "20/Sep/2017:19:01:42 +0200",
       "req_path" => "/products/view/124"

```

### Lecture 16 - Finishing the Grok Pattern

* In [patterns](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns) there are ready made grok match patterns to extract info from common logs like httpd logs. we will use the *HTTPD_COMBINEDLOG*
* the way to use it is simple. we replace our match pattern with `match => { "message" => "%{HTTPD_COMBINEDLOG}"}`. we test it and it works. fields are extracted
* we see aproblem agent and referrer fileds start and end with a ". we see the definition of the pattern in github and see that both these patterns do not use quotation marks around them to exclutde them. but use instead the QS sysntax. this means that quotation marks are captured by the patterns
* to remove them we add the mutate filter plugin after grok in the pipeline
```
  mutate {
    gsub => [
      "agent", '"', "",
      "referrer", '"', ""
    ]
  }
```
* this plugin takes an array of the fields it will operate on. we spec the char we want to transform "\"" a quotation mark and replace it with empty string "".
* the order of plugins is important. also the fields are the product of the first grok plugin
* we test it and it works. quotation marks are removed
* there is another approach to solving this problem. we delete the mutate plugin and dont use the pattern just by its name in grok but by its representation whi we will modify `%{HTTPD_COMMONLOG} %{QS:referrer} %{QS:agent}`
* instead of using the QS pattern we use GREEDYDATA. greedydata matches anything inside of quotation marks. result is ok!
* we still got a problem integer values are strings . we cannot solve it without digging deeper in the HTTPD_COMMONLOG pattenr. so we will solve it with mutate
```
  mutate {
    convert => {
      "response" => "integer"
      "bytes" => "integer"
    }
  }
```

### Lecture 17 - Accessing Field Values

* we will learn how to reference field values. this is useful on applying conditional statements
* we can refernce a filend name of an event with its name e.g *request* or the anme in square brackets *[request]*. we can reference nested fields by cascading field names in square brackets e.g `[headers][req_path]`
* we will use field refernence to create aoutput event log from the pipeline with a made up name relevant to the pipeline input log type.
* we add a field in our pipe line from input, its named type and we add it on both input plugins
```
  http {
    type => "access"
  }
```
* this field is static and passes from the pipeline down to the output
* we will use it to make a dynamic name of our output file. the way to interpolate field names in strings is `%{fieldname}`
```
  file {
    path => "%{type}_temp.log"
  }
```
* we test by sending a postman req. we see the new field beeing added to the event and our filename corectly created and our file poppulated

### Lecture 18 - Formatting Dates

* [joda date format](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html)
* we will learn how to format dates because we want date to be part of our filename. the way to interpolate date in a string is `%{+DATE_FORMAT}`. the date format must be a joda date format e.g `%{+yyyy-MM-dd}`
* we modify our output file plugin to include the date in its name 
```
  file {
    path => "%{type}_%{+yyyy-MM-dd}.log"
  }
```
* but what date is passed in the name? the events date. so it sends a request to the pipeline to check if the event has a timstamp field. so the result is to have a file per each date of the events. the event timestamp has nothing to do with any date contained in the input data rather than the timestamp the event was processed in logstash
* we can use this technique to pass dates on the index field that will be used by elasticsearch when indexing docs 

###  Lecture 19 - Setting the time of the event

* an event contains a timestamp of when logstash received the event / when the input received the event.
* in some use cases we dont want that. we would like the event o carry a timestamp of when the event was created in the original system. e.g the apache server timestamp. (e.g as it appears in the lod file we input)
*  this is good for backtracking or when processing results on differenceies in timestamps
* the way to do it is by adding another filter plugin called date
```
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
  }
```
* what this plugin does is that it uses one of the fields that were generated by GROk or by other input named *timestamp* as source of the *@timestamp* event property by applying a JODA date format (the JODA format should express the way date is formated in the input field).  the result is that timestamp and @timestamp are the same and this is reflected in the output file name
```
     "timestamp" => "20/Sep/2017:18:42:01 +0200",
          "type" => "access",
       "request" => "/js/admin.js",
    "@timestamp" => 2017-09-20T16:42:01.000Z,
```
* if the parse fails the logstash will pass as timestamp value ~dateparsefailure
* with both timestamps the same we can remove the GROK generated timestamp ( we have to do it in the date plugin, after the copy)
```
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
    remove_field => ["timestamp"]
  }
```

### Lecture 20 - Introduction to conditional statements

* conditional statements in logstash follow conventional syntax.
```
if EXPR {
  
} else if EXPR {
  
} else {
  
}
```
* expressions should be valid logstah syntax e.g `[type] == "access"`
* operators supported:
  * equality: ==, !=, <, >, <=, >= e.g `if [headers][content_length] >= 1000 {}`
  * regexp: =~, !~ e.g `[some_field] =~ /[0-9]+/ { # some_filed only contains digits }`
  * inclusion: in,not in e.g `if [some_field] in ["one","two"."three"] {}`
  * boolean: and, or 
  * unary: !   !([a] == [b]) equivalent to [a] != [b]

### Lecture 21 - Working with conditional statements

* [glob pattern](https://www.elastic.co/guide/en/logstash/current/glob-support.html)
* we will prepare the pipeline to handle both acces and error logs
* we will set type to access or error depending the type of event we are procesing
* conditional statements can be introduced only at the root leve of the three stages of the pipeline
* we remove the type from both input file plugin and http plugin
* we add condtional statemnt in the filter part to determin if the request uri contains the field error
```
  if ([headers][request_uri] =~ "error") {
    mutate {
      replace => { type => "error"}
    }
  } else {
    mutate {
      replace => { type => "access"}
    }
  }
```
* we move the grok parsing th emutate and date plugin we used for access logs in the else statement 
* we send an http req and see type access is added
* we append error to the URI path and see type set to error
* we want to be able to input files from a dir
* we start by replacing the file name adding a widcard `path => "/home/achliopa/workspace/udemy/logstash/myfolder/project/event-data/apache_*.log"`
* we can use glob patterns for file and directory patterns e.g `/path/to/*.log` `/path/to/**/*.log` `/path/to/{nginx,apache}/*.log`
* when we input from file, the path of the file is registered as event field so we can apply conditional logic to it `i f [path] =~ "errors" `
* if the grok pattern does not match we get "_grokparsefailure" in tags field. we use it to the conditiona statemtns to handle the case of error in grok parsing
```
    if "_grokparsefailure" in [tags] {
      drop { }
    }
```
* what we do is drop the event so it dont appear in output