# DistributedTracing-Zipkin-ELK


In a microservice architecture, tracing is extremely challenging because requests will span multiple services, each executing one or more processes, across multiple servers.


The Zipkin UI provides us with some basic options to analyze traced requests — we can use a dependency diagram to view the execution flow, we can filter or sort the traces collected by Zipkin per application, length of trace and timestamp.

Using ELK though, takes trace analysis up a notch. Elasticsearch can be used for long-term retention of the trace data and Kibana will allow you gain much deeper insight into the data. This article provides you with the steps to integrate the two tools.


## Installing Zipkin

### Through Docker
docker run -d -p 9411:9411 openzipkin/zipkin

### By Downloading

url -sSL https://zipkin.io/quickstart.sh | bash -s

java -DSTORAGE_TYPE=elasticsearch -DES_HOSTS=http://127.0.0.1:9200 -jar zipkin.jar

## Running our Dockerized ELK

git clone https://github.com/deviantony/docker-elk.git
cd /docker-elk
docker-compose up -d

### Verifying the installation

docker ps

You’ll notice that ports on my localhost have been mapped to the default ports used by Elasticsearch (9200/9300), Kibana (5601) and Logstash (5000/5044).

Everything is already pre-configured with a privileged username and password:

  user: elastic
  password: changeme

And finally, access Kibana by entering: http://localhost:5601 in your browser.


## Setting up demo services

For the sake of demonstrating how to hook up Zipkin with the ELK Stack, I’m going to be using an instrumentation library for Java applications called Brave. Specifically, I will be setting up two Java servlet services that communicate via http. Brave supports a whole lot more but this example will suffice for our purposes.

### First, download the source code

git clone https://github.com/openzipkin/brave-webmvc-example.git


In this example, there are two services that send tracing info to Zipkin — a backend service and a frontend service. 


### The frontend service:

cd /brave-webmvc-example/webmvc4
mvn jetty:run -Pfrontend

### The backend service:

cd /brave-webmvc-example/webmvc4
mvn jetty:run -Pbackend


Both services should report the same success message in the run output:

[INFO] Started Jetty Server

The frontend service can be accessed via http://localhost:8081 and calls the backend service and displays a timestamp.




Because we defined Elasticsearch as the storage type when running Zipkin, the trace data should be indexed in Elasticsearch. To verify, run:

curl -XGET 'localhost:9200/_cat/indices?v&pretty'


Now open Kibana and define the new index pattern. Open Kibana at http://localhost:5601, and commence with defining a ‘zipkin*’ index pattern:

