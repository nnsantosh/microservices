## Inter-Service Communication
Is the ways in which the microservices in our system can communicate with each other. <br/>
1. Remote Procedure Invocation- Service sends request to another service and waits for response. <br/>
Can be done using technologies such as REST and Apache Thrift. <br/>
Commonly synchronous message communication is used but other techniques are also available though they are complex and less commonly used. <br/>
Refer diagram below:
![synchronousCommunication](https://github.com/nnsantosh/microservices/blob/master/rpiDiagram.jpg)
2. Asynchronous message based communication is used if response is not immediately required. <br/>
Commonly implemented using message bus. So communication is dependent on message broker like RabbitMQ. <br/>
Publish subscribe pattern is used. It leads to more overhead. <br/>
Refer diagram below:
![asynchronousCommunication](https://github.com/nnsantosh/microservices/blob/master/asyncCommunicationDiagram.jpg)
3. In some cases Custom or domain specific protocols are used which can be potentially the only option when communicating with third party systems or legacy systems which would require protocols such as FTP or SMTP. <br/>

## Microservice registry
If one service has to send message to another service it should be aware of the available services and their network location.  <br/>
Ideally the number of instances of a service can be adjusted based on load. But other services that use these services should be able to handle. <br/>
To solve this problem we can introduce service registry that holds in a database data about all the currently available instances of services and their network location. <br/>
Whenever a microservice instance starts up it registers itself with this service registry which will add the instance details into its database. <br/>
Similary on shutdown it sends request to the service registry to remove the instance from database. <br/>
Also to handle scenario where service shuts down unexpectedly and hence does not send request to service registry the service registry uses health check service to monitor available service instances at regular interval. <br/>

A separate registry service can be introduced to handle all this so that individual services need not bother about this. But this introduces another service. <br/>

## Microservice discovery
1. Client Side Discovery where the service that needs to call another service obtains the network location of that service instance by directly querying the service registry database. <br/>
The service registry will also have some load balancing logic to determine which instance to select and return to the calling service. <br/>
2. Server Side Discovery where the service that needs to call another service will have no knowledge of the service registry but it simply sends it request to a load balancer. <br/>
The load balancer it would then query the service registry for an instance of the required service and forward the request there. <br/>
One advantage of this is it is easier for clients since they need not be aware of service registry. <br/>
Disadvantage is there is more network hops involved. <br/>

Consul - https://www.consul.io/
A service mesh solution that provides a strongly consistent data store that can be used not only for service discovery purposes but also for health checking 
(we'll get to this in a later lecture!), as a configuration server and offers multi-datacenter support out of the box. 
Each of these features can be used independently. Consul is a popular choice not only because of its relative simplicity to set up but also because of its reliability, 
scalability and supporting features.

Netflix Eureka - https://github.com/Netflix/eureka
Eureka, that is part of the open souced Netflix stack, can be best described as an AWS Service registry for resilient mid-tier load balancing and failover. 
This is a good alternative if a simple load balancer is needed in a cloud environment, however, for additional features such as monitoring and configuration, 
you will either have to compliment this with additional tools or look at other alternatives.

Apache Zookeeper - https://zookeeper.apache.org/
Often used with Apache Curator (http://curator.apache.org/), Zookeeper was one of the first tools to be used for distributed service coordination. 
As a result, it is very mature, robust and has many features available out of the box. 
In fact, it's used by quite a few big names in the industry such as YouTube and eBay.
However, being one of the older tools out there it's showing its age. 
Compared to its competitors it consumes significantly more resources and is more time-consuming to set up and maintain due to its dependencies and complexity. 
You most likely will end up using barely any of the extra features that are available, so you'll be suffering the hit of these disadvantages without any real benefit. 
As a result, it is a much less common choice nowadays and is usually found in more established companies where alternatives were not available at the time.







