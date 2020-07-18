## Microservice Template

Is a code project that can be used to start of from when developing a new microservice. <br/>
It should basically save us time. <br/>

### Why is it important?

Significant amount of time setting up- Project structure and adding set up related code like security concerns, initial connections to databases and referencing to all required libraries. <br/>
Similar code for each microservice set up - To make new service set up as fast and easy as possible. <br/>

The template should contain: <br/>
1. cross cutting concerns-logging, application metrics, connection setup and configuration to databases and message brokers. <br/>
2. Project structure- include unit test project, domain project if you follow domain driven design. <br/>

The main objective is to help us save time.

Examples to refer: <br/>
https://github.com/Nike-Inc/riposte-microservice-template <br/>

### Code Repository setup:
1. monolithic repository which houses all the projects <br/>
2. discrete smaller repositories based on domain or component <br/>

### Monolithic Repository:
Pros: <br/>
Easier to keep input/output contracts in sync <br/>
Can version the entire repo with a build number <br/>

Cons:
Different teams working on the same repo can break the build, disrupting CI/CD for other teams. <br/>
Easier to create tight coupling <br/>
Long build times and large repo size for download. <br/>

### Discrete Repository:
Pros: <br/>
Different teams can own different repositories. <br/>
Scope of a single repo is more clear. <br/>

Cons: <br/>
Contract versioning becomes more complex. <br/>
Unless managed properly, discrete repos can easily become monoliths. <br/>
More up front cost in setting up repos and CI/CD pipeline. <br/>

### Which code repository setup to use?
For large teams monlithic repository is easier to manage.

## Microservice decomposition
The most commonly used technique to decompose a system into micro services is based on business capability. <br/>
In some cases it may be required to decompose system based on technical capabilities or functional capabilities. <br/>

Refer diagram below:
![serviceDecomposition](https://github.com/nnsantosh/microservices/blob/master/serviceDecompositionDiagram.jpg)



