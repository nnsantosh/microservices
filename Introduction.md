# Micro service architecture

Microservice architecture is based on services and their intercommunication. <br/>
Services are fine grained which means they perform specific task and are light weight. <br/>
A service performs specific business activity with a specified outcome and is self contained and is loosely coupled. <br/>
A service is like black box for its consumers and it may consume other services under the hood. <br/>

## Advantages: 
Low coupling between components <br/>
Improves modularity <br/>
Promotes parallel development <br/>
Promotes scalability <br/>

## Drawbacks:
Infrastructure costs are usually higher <br/>
Integration testing complexity <br/>
Service management and deployment <br/>
Nanoservice anti pattern- Its an anti pattern where a service is too fine grained. In this case the overhead like interprocess communication, serialization etc outweigh its utility. <br/>

Another thing to note is that GUI applications may not be suitable candidates for micro service architecture especially if these require lot of data exchange between services. <br/>

## Why do many microservice project fail?
Due to lack of : <br/>
Planning <br/>
knowledge <br/>
Skills <br/>
Time <br/>

## How to prevent projects from failing?
Determine applicability <br/>
Prioritize automation- using continous integration, deployment and microservice template <br/>
Have a clear Planning- realistic and manageable schedule <br/>
Avoid common pitfalls <br/>








