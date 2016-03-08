---
layout: post
title:  "Real time monitoring of .NET web applications with logstash, elasticsearch and Kibana"
date:   2015-07-10 15:38:00
categories:
tags:
- logstash
- elasticsearch
- kibana
- windows
- elk
- iis
excerpt: Configure logstash, elasticsearch and Kibana (the ELK stack) on Windows to create a real time monitoring solution for your web application.
---

## Intro
This is a quick run-through of configuring logstash, elasticsearch and Kibana (the ELK stack) on Windows to create a real time monitoring solution for your web application. It took me around 2 hours to get this setup the first time while following <a href="https://www.ulyaoth.net/resources/tutorial-install-logstash-and-kibana-on-a-windows-server.34/">this excellent blog</a>.

I won't go into the detail which this blog has done, so expect some short, sweet bullet points. Then we'll create some visualizations (pretty pictures) from the basic IIS logs, which could help identify problems with your web application.


## Summary of tools

* logstash - used to process unstructured data (e.g. logs) from various sources into structured, readable keys and values which will be pushed to elasticsearch where they can later be queried. Simply tell logstash where your logs are, how to transform the unstructured data into something structured and where your elasticsearch instance is running. The structured output will be forwarded to elasticsearch.

* elasticsearch - store and search large amount of structured, unstructured and time-series data.

* Kibana - visualize your data from elasticsearch.

## Prerequisites
* Java. It's required for logash. It also must be on your path.
  * [Grab it from Oracle's website](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) if you need it.
  * Follow instructions on [this Stack Overflow answer](http://stackoverflow.com/a/6521412/609176) if you're not sure how to add Java to your path. You'll want to add:
    * Variable: <code>JAVA_HOME</code>
    * Value: <code>C:\Program Files\Java\jdk1.8.0_45</code>

## Setup

Head over to <a href="https://www.elastic.co/downloads" target="_blank">https://www.elastic.co/downloads</a>.

Download:

* <a href="https://www.elastic.co/downloads/logstash" target="_blank">logstash</a>
* <a href="https://www.elastic.co/downloads/elasticsearch" target="_blank">elasticsearch</a>
* <a href="https://www.elastic.co/downloads/kibana" target="_blank">Kibana</a>

Extract each zip to a common folder (I've called mine "monitoring"). You should end up with something like:

* <code>C:\monitoring\logstash</code>
* <code>C:\monitoring\elasticsearch</code>
* <code>C:\monitoring\kibana</code>

### logstash

First, some explanation. We want to take a standard log line from a web application running on IIS, which looks like...

<code>2015-07-09 09:21:32 ::1 POST /WebApplication/Claims/1 - 80 - ::1 Mozilla/5.0+(Windows+NT+6.1;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/43.0.2357.132+Safari/537.36 200 0 0 84
</code>

...and push it to elasticsearch in a structured manner. There's a few ways to extract this information. In this example, it's being done by <b>matching the order which the terms appear</b>. It's <b>important to specify the types</b> here to have full searching power later on. You don't want everything being a <code>string</code>!

To strip out the detail and specify types, we'll need to tell logstash how to interpret it. Introducing [grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html), which works by combining text patterns into something that matches your logs. There are [currently over 120 patterns](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns) to match against.

So, here is our grok filter, which is added to the logstash config.

<code>
match => ["message", "%{TIMESTAMP_ISO8601:log_timestamp} %{IPORHOST:site} %{WORD:http_method} %{URIPATH:page} %{NOTSPACE:query_string} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:client_host} %{NOTSPACE:useragent} %{NUMBER:http_response} %{NUMBER:sub_response} %{NUMBER:sc_status} %{NUMBER:time_taken}"]
</code>

You'll note the first part of the filter is <code>{TIMESTAMP_ISO8601:log_timestamp}</code> which is simply stating the type followed by a term to identity the matched value by. When you look back at the example log line, you'll see the first value is <code>2015-07-09 09:21:32</code> which is a timestamp. Simples!

Below is a full config file which you can use for the standard IIS log format. It will extract the values as explained above and push them to elasticsearch. Copy the config (and amend it to your needs) to a new file and name it <code>logstash.conf</code>. Save it to your logstash bin folder <code>C:\monitoring\logstash\bin</code>.

<pre>
<code>

input {
	file {
		type => "IISLog"
		path => "C:/inetpub/logs/LogFiles/W3SVC*/*.log"
		start_position => "beginning"
	}
}

filter {

	# ignore log comments
	if [message] =~ "^#" {
		drop {}
	}
 
 	# check that fields match your IIS log settings
	grok {
		match => ["message", "%{TIMESTAMP_ISO8601:log_timestamp} %{IPORHOST:site} %{WORD:http_method} %{URIPATH:page} %{NOTSPACE:query_string} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:client_host} %{NOTSPACE:useragent} %{NUMBER:http_response} %{NUMBER:sub_response} %{NUMBER:sc_status} %{NUMBER:time_taken}"]
	}
  
	# set the event timestamp from the log
	# https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
	date {
		match => [ "log_timestamp", "YYYY-MM-dd HH:mm:ss" ]
		timezone => "Etc/UCT"
	}
	
	# matches the big, long nasty useragent string to the actual browser name, version, etc
	# https://www.elastic.co/guide/en/logstash/current/plugins-filters-useragent.html
	useragent {
		source=> "useragent"
		prefix=> "browser_"
	}
	
	mutate {
		remove_field => [ "log_timestamp"]
	}
}

# output logs to console and to elasticsearch
output {
	stdout {}
	elasticsearch {
		host => "localhost"
		protocol => "http"
		port => 9200
	}
}
</code>
</pre>

Now all we need to do is start the logstash process and it will monitor any location(s) specified in the _input_ section of the config.

<pre>
<code>
cd C:\monitoring\logstash\bin
logstash.bat agent -f logstash.conf
</code>
</pre>

Now, once elasticsearch is running, any new log lines will now be pushed there in a nice format!

### elasticsearch

Open a command prompt and navigate to the bin directory for elasticsearch.

<code>
cd C:\monitoring\elasticsearch\bin
</code>

As a one off, you'll need to run the install command.

<code>
service install
</code>

To start the elasticsearch process, simply execute the following:

<code>
service start
</code>

That's it! elasticsearch should now be running at <code>http://localhost:9200</code>. Hit that and you should get a nice json response to let you know that the service is running.

If you need to stop the process, simply execute:

<code>
service stop
</code>

If you need any more information you can [check out the official docs on the elastic website](https://www.elastic.co/guide/en/elasticsearch/reference/1.6/setup-service-win.html).


### Kibana

The simplest one!

<pre>
<code>
cd C:\monitoring\kibana\bin
kibana.bat
</code>
</pre>

That's it! There is a config file in the bin directory, but the defaults should suffice for now. Kibana should availble at <code>http://localhost:5601</code>.

You'll need to have a few logs in elasticsearch to complete the Kibana setup. When you first open Kibana you'll be taken to a settings page titled "_Configure an index pattern_". Check the checkbox "_Use event times to create index names_". As we're using elasticsearch, the defaults should be fine and you should be able to click "Create".

### Create some useful graphs

The basic IIS logs contain some useful data, like http response code, response time and the requested URI. This should give enough information to identity some problems in our web application. We could easily tell if response times are more than a second or we're getting lots of 404s, 500s, etc.

Some examples below. You can see the filter criteria I have used in the left pane.

#### http response codes
<image src="/images/kibana-http-resonse-codes.png" alt="kibana-http-response-codes" />

<br>

#### browser breakdown
<image src="/images/kibana-browser-breakdown.png" alt="kibana-browser-breakdown" />

<br>

<b>Can't find your terms?</b> They're probably cached by Kibana.

* Click "Settings"
* Select your index
* Click "Reload field list" (the yellow button)

<img src="/images/kibana-settings-reload-indicies.png" alt="kibana-settings-reload-indicies"/>
