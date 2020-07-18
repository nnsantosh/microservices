# Managing Data in micro-service

## Shared Database Pattern

Where the services would be able to access each other's data since they share common Database
Database transactions are used to guarantee data consistency  and integrity.
Schema changes can affect multiple services and doing such changes become challenging since more micro-services reference the common database.
It may also lead to performance issues since one service which is trying to access data may be blocked by another service which is modifying the data.
Tracking such issues also becomes difficult.
As a result this shared database pattern is considered as anti pattern for larger more complex systems.

Refer Diagram below:
![SharedDatabasePattern](https://github.com/nnsantosh/microservices/blob/master/Shared_Database_Pattern_Diagram.jpg)

## Database per service

Where there is complete separate database for each service.
Different services can use different database technologies depending on what best suits their requirement.
Disadvantages include implementing queries that require data across different databases. However this can be achieved through different ways such as API composition and event sourcing.

Refer Diagram below:
![DatabasePerServicePattern](https://github.com/nnsantosh/microservices/blob/master/Database_per_Service_Pattern_Diagram.jpg)


## API Composition

In API Composition a service referred to as API Composer queries data from multiple services and then performs an  in-memory join of the data obtained to produce final result.

Refer Diagram below for an example:
![APIComposer](https://github.com/nnsantosh/microservices/blob/master/APIComposerDiagram.jpg)

Disadvantage is that some queries may require transferring of huge amount of data to the composer service making it inefficient. An alternative to this approach is event sourcing.

## Event Sourcing

This pattern helps services keep track of state changes to objects in a reliable way.
The main difference in event sourcing when compared to normal database is that we use event store and persist events to it for inserting, deleting and updating records.
Services are able to subscribe to different types of events handled by the event store. In this way the event store acts as a form of message broker.

Refer diagram below for an example:
![EventStorePublish](https://github.com/nnsantosh/microservices/blob/master/EventPublishDiagram.jpg)

In this example when the Product service updates the price of Product A in its Product Database it also publishes an event to the event store with these details.
The Product Search service and the Product recommendation service have subscribed to the event store to receive the price updated events.
Therefore the event store when it receives the price updated from the Product service it publishes these events to the Product Search service and the Product recommendation service.

Refer diagram below to get a snapshot of how all events related to Product A may appear in the event store:
![EventStoreSnapshot](https://github.com/nnsantosh/microservices/blob/master/EventStoreSnapshot.jpg)

The event store is able to construct the state of an object at any particular time by replaying events affecting that object up to that time.
For Example:
If we wanted to know what the state of Product A was on 11th May 2018 at 11 pm we process the events that occurred up to this time without including the events that occurred after this time.

For optimization purposes to reduce loading time the event store can periodically store snapshots of objects. In this case to get the object's current state the most recent snapshot is loaded and then only the events after the snapshot are processed to construct the latest state.
The frequency at which the snapshots are taken is decided by the development teams and depends on many factors like resources available, the number of objects that would require a snapshot and the frequency of events for each object.

The benefits of using event sourcing pattern are that we have an audit log of all events and we can determine the state of an object at any particular time by replaying events up to that period.

Disadvantages:
1. Steep learning curve for the development team
2. Results in additional complexity in querying data.

Refer diagram below to see how an event sourcing pattern can be used to keep Product recommendation service updated with data that it requires from Product service and Order Service by publishing events. In this way data need not be sent for large datasets across services.
![EventSourcingFullExample](https://github.com/nnsantosh/microservices/blob/master/EventSourcing_Full_Example_Diagram.jpg)

Some of the frameworks in use:

AxonIQ (Java) - https://axoniq.io/

Axon is an open-source Java framework and server that supports event-driven microservices. It provides implementations of the most important building blocks when developing event-driven micro-services following domain-driven design, such as aggregates, command and event buses, and repositories. It is a robust and reliable framework that has a free version as well as an enterprise edition with some extra features, however, the free edition already offers more than you'll need to get started. Specifically for event-sourcing, here's an official article explaining how the Axon Server fits in as an event store when using the Axon framework to implement event sourcing: https://axoniq.io/resources/event-sourcing.

Event Store (All programming languages) - https://eventstore.com/

If you prefer not to use a full framework, or if you just want to build most of the stuff yourself to get a better understanding of what's going on - a good option is to download the Event Store database that was built specifically with event sourcing in mind. It 's free and open-source, with clients for different languages as well as access to functionality via an HTTP web API, making it a viable option to use with just about any programming language.

## Two Phase Commit
Problem:
Distributed transaction involves altering data on multiple databases.
As a result processing of distributed transaction is more complicated. Because the database must coordinate the committing or rolling back of transaction as a self contained unit.
The two phase commit mechanism can be used to ensure integrity of data in a distributed transaction.
Refer example below:
![TwoPhaseCommit](https://github.com/nnsantosh/microservices/blob/master/TwoPhaseCommitCoordination.jpg)

We would need a separate service known as Coordinator that will act as a coordinator for the two phase commit.
There are two phases in a two phase commit pattern:
1. In the first phase which is the commit request phase the coordinator sends a query to commit message to each of the service involved in distributed transaction.
The services execute the transaction but do not commit and they reply with a yes/no depending on if they were successful or not.
2. In the second phase which is the commit phase if all services have replied yes then the coordinator sends commit message to each of the service.
And these will then commit the transaction and reply with an acknowledgement. However if one service has replied with No then the coordinator sends rollback message and the services rollback the transaction.

If there is a scenario where all the services have replied "Yes" in the first phase but one of the service failed to send second acknowledgement in the second phase then the coordinator sends message to all the services to revert the previous transactions actions.

Main disadvantage:
1. Development complexity
2. Blocking Protocol meaning services will wait indefinitely for the coordinator to send instructions on how they have to proceed. This means if coordinator goes down services could hang indefinitely.

## Saga Pattern
This is an alternative to two phase commit to manage distributed transactions.
A saga can be considered as a sequence of local transactions in different microservices.
Each local transaction updates the DB of that microservice and then publishes message/event to trigger next local transaction in the saga.
If one local transaction fails the saga executes a series of compensating transactions  that rollback changes of local transactions forming part of the distributed transaction that have already been executed.
There are two main different types of saga implementations:

1. Choreography based sagas:

Refer to the diagram here for an example:
![ChoreographyBasedSaga](https://github.com/nnsantosh/microservices/blob/master/ChoreographyBasedSagaDiagram.jpg)

Each local transaction publishes domain events that will trigger local transactions in other services until the saga is completed.
So in the above example payment service obtains payment from a customer, then the order is created and the then the products are reserved for the customer, the order service first sends payment request event to the message broker to start the saga. The payment service which has subscribed to the payment requests event receives this message and attempts to obtain payment from the customer. It then sends a message indicating if the payment has been received successfully which will be picked up the order service. If the payment was not received successfully the saga ends and no further actions are needed. However if the payment was received successfully, the order service attempts to create the order and sends an event indicating if this was successful which is picked up by both product service and payment service. If the order creation was not successful then the payment service must execute the compensating transactions to rollback the actions taken to obtain payment from the customer which would most likely involve a refund and sending an email to the customer or notifying them on the screen. If the order was created successfully, the product service which subscribes to this event will attempt to reserve the products requested for the order. It will then send an event indicating whether the products are reserved successfully. IF so then the saga ends as no further actions are required. However if the products were not reserved successfully both the order service and payment service will rollback the actions they have performed in the saga through compensating transactions. As you can see, the way distributed transactions are handled is different from two phase commit where transactions would not be committed to the database before these have been approved by all participating services.

2. Orchestrator based sagas:

Refer to the diagram here for an example:
![OrchestratorBasedSaga](https://github.com/nnsantosh/microservices/blob/master/OrchestratorBasedSagaDiagram.jpg)

In Orchestrator based sagas, an orchestrator which is usually spawned specifically for each saga instructs the microservices involved in the saga what local transactions to execute, so basically it coordinates the whole saga. To use the same example that was taken for the choreography based saga and implement it using an orchestrator.
The order service creates an order saga object which doesn't need to be a separate service it can just be an object residing inside the order service.
The order creation saga will instruct the payment service to request payment and will wait for a message indicating if payment was received successfully.
If not then the order creation saga terminates immediately as no more actions are required. However if payment was received successfully, the creation saga sends a message to the order service to create an order. The order service will then reply to the order creation saga on whether the order was created successfully. If the order wasn't created successfully the order creation saga instructs the payment service to rollback its previous actions taken to request payment and then this saga terminates. On the other hand, if the order was created successfully the order creation saga sends a request to the product service to reserve product. The product service will reply with a message indicating if the products were reserved successfully. If so then the saga can terminate as no further actions are required in the saga.

Advantage of Saga pattern:
The overall advantages of the Saga pattern are that it enables an application to maintain data consistency across multiple services.

Disadvantage:
The development process becomes more complex and also takes longer as compensating actions need to be developed for each transaction to be able to roll back.

Also ideally the microservice should publish the event and update the database automatically to prevent race conditions and potential data inconsistencies.
