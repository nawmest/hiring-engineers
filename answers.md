
# Nawfel Mestoui - Solutions Engineer Exercise

## Prerequisites - Setup the environment
I am using my current OS Ubuntu 17.10 to complete this exercise.
<img src="screens/os-version.png"></img>
### Installing Datadog agent
After sign in to Datadog, I <a href="https://app.datadoghq.com/account/settings#agent/ubuntu">installed Datadog agent</a> for Ubuntu:
<img src="screens/install-agent.png"></img>

## [](https://github.com/DataDog/hiring-engineers/tree/solutions-engineer#collecting-metrics)Collecting Metrics:

### Add tags in the Agent config file
After updating the configuration file `/etc/datadog-agent/datadog.yaml`, we can see the host and the new tags from Host Map page in Datadog:
<img src="screens/agent-tags.png"></img>

### Installing Datadog integration for MongoDB
In my current machine I have already MongoDB Community Edition v3.6.6 installed, <a href="https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/"> here are instructions to install it on Ubuntu</a>

To check if the database is active:
`service mongod status`
<img src="screens/mongod-status.png"></img>

In order to install Datadog integration for MongoDB, I followed the instuctions from this <a href="https://app.datadoghq.com/account/settings#integrations/mongodb">page</a>:

After adding a read-only user administrator for datadog (to collect complete server statistics),
and after running verification commands from the instructions page, I get this result:
<img src="screens/mongod-verification.png"></img>

The second part of the integration is to configure the Datadog Agent to connect to my MongoDB instances, for that I need to update the configuration file `/etc/datadog-agent/conf.d/mongo.yaml`

After restarting Datadog service, I can check that MongoDB is well Integrated:

By typing "mongoDb" in the <a href="https://app.datadoghq.com/metric/summary"> metrics summary page</a>, we can see all the MongoDb metrics.
<img src="screens/mongo-metrics.png"></img>

And also if we check the <a href="https://app.datadoghq.com/dashboard/lists">dashboard list page <a/> we can see that MongoDb is present in the list:
<img src="screens/mongo-list.png"></img>

### Creating a custom Agent check
To have a custom Agent check, I create 2 files: `my_check.yaml` in `/etc/datadog-agent/conf.d/` and the pyhton script `my_check.py` in `/etc/datadog-agent/checks.d/`

#### my_check.yaml
```
init_config:
	min_collection_interval: 45
instances:
    [{}]
 ```   
min_collection_interval can be added to the init_config section to help define how often the check should be run globally, in this case it's set to 45.

##### my_check.py
```
from checks import AgentCheck

import random


class HelloCheck(AgentCheck):

    def check(self, instance):
        self.gauge('my_metric',random.randint(0, 1000)
```
The check class inherits from AgentCheck and send a gauge of a random number for the metric 'my_metric' on each call.

#### Bonus Question: the collection interval is set in the yaml file of our Agent check


## Visualizing Data:
The first step was to get an Application key from the <a href="https://app.datadoghq.com/account/settings#api">APIs page</a>
<img src="screens/apikey.png"></img>

The new timeboard will contain:

the new metric: my_metric

'my_metric' with the rollup function applied to sum up all the points for the past hour into one bucket

MongoDb Total number of connections created
	
Find more about Timeboard creation <a href="https://docs.datadoghq.com/api/?lang=python#create-a-timeboard">here</a>

```
from datadog import initialize, api

options = {
    'api_key': 'b94670e66009a0332312afdeec0ca939',
    'app_key': 'cbdd2353176dfca0df90a7558e10067c255ccc4a'
}

initialize(**options)

title = "Nawfel Timeboard"
description = "New timeboard to visualise our new metrics"
graphs = [{
	"definition": {
		"viz": "timeseries",
		"requests": [
       		  {
               		"q": "avg:my_metric{host:nawmest-Aspire-E5-575G}",
               		"type": "line",
               		"style": {
                       		"palette": "dog_classic",
                       		"type": "solid",
                       		"width": "normal"
               		},
               		"conditional_formats": [],
               		"aggregator": "avg"
       		  }
        	],
   	},
    	"title": "my_metric random values"
},
{
   	"definition": {
  		"viz": "timeseries",
  		"requests": [
    		  {
      			"q": "avg:my_metric{host:nawmest-Aspire-E5-575G}.rollup(sum, 3600)",
      			"type": "line",
      			"style": {
        			"palette": "dog_classic",
        			"type": "solid",
        			"width": "normal"
      		  	},
      		"conditional_formats": [],
      		"aggregator": "avg"
    		  }
  		],
	},
	"title": "Sum of my_metric over 1h"
},
{
        "definition": {
                "viz": "timeseries",
                "requests": [
                  {
			"q": "anomalies(avg:mongodb.connections.totalcreated{host:nawmest-Aspire-E5-575G}, 'adaptive', 2)",
                        "type": "line",
                        "style": {
                                "palette": "dog_classic",
                                "type": "solid",
                        	"width": "normal"
                        },
                "conditional_formats": [],
                "aggregator": "avg"
                  }
                ],
        },
        "title": "MongoDb Total number of connections created"
}]

read_only = True
api.Timeboard.create(title=title,
                     description=description,
                     graphs=graphs,
                     read_only=read_only)
```
### Taking a snapshot of the graph
I took a snapshot of the graph by clicking on the camera icon, and select my email using @ notation and send it to myself.
<img src="screens/snap.png"></img>

<img src="screens/mail.png"></img>

### Bonus Question: What is the Anomaly graph displaying?
The highlighted area in the graph represent the expected range of the metric based on previous values, so anything outside that range is an anomaly.

## Monitoring Data:

I created a new Metric Monitor that watches the average of the custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

Warning threshold of 500
Alerting threshold of 800
And also ensure that it will notify you if there is No Data for this query over the past 10m.

<img src="screens/monitoring01.png"></img>

Send you an email whenever the monitor triggers.
Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.

<img src="screens/monitoring2.png"></img>

### Bonus Question:
Set up two scheduled downtimes for this monitor:
Scheduled downtimes are set up from this <a href="https://app.datadoghq.com/monitors#/downtime">page</a>

One that silences it from 7pm to 9am daily on M-F
<img src="screens/downtime01.png"></img>

And one that silences it all day on Sat-Sun.
<img src="screens/downtime2.png"></img>

## Collecting APM Data:
Tracing Python Applications using Ddtrace:
The Flask trace middleware will track request timings and templates. It requires the Blinker library, which Flask uses for signalling.

To install the middleware, add:
```
from ddtrace import tracer
from ddtrace.contrib.flask import TraceMiddleware
```
and create a TraceMiddleware object:
```
traced_app = TraceMiddleware(app, tracer, service="my-flask-app", distributed_tracing=False)
```
<a href="http://pypi.datadoghq.com/trace/docs/web_integrations.html#flask">See Web Frameworks documentation </a>

Final Flask App code:
```
from flask import Flask
import logging
import sys

from ddtrace import tracer
from ddtrace.contrib.flask import TraceMiddleware

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

# instrumenting DD APM for Flask
traced_app = TraceMiddleware(app, tracer, service="my-flask-app", distributed_tracing=False)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run()
 ```
 Link of dashboard : https://app.datadoghq.com/apm/service/flask/flask.request?start=1536299777594&end=1536303377594&paused=false&env=prod
 
 <img src="screens/APM.png"></img>
 
### Bonus Question: What is the difference between a Service and a Resource?
A service is a set of processes that work together.
A resource is a software artifact supporting specific data used by a service.

## Final Question:
Datadog has been used in a lot of creative ways in the past. We’ve written some blog posts about using Datadog to monitor the NYC Subway System, Pokemon Go, and even office restroom availability!

Is there anything creative you would use Datadog for?

The first thing that comes to my mind is healthcare, the public are already generating and sharing huge amounts of personal health data through consumer devices such as smart watches and wristbands that monitor sleeping patterns, exercise, heart rate, calorie consumption and more.

### Thank you Datadog for this interesting and fun exercise :)
