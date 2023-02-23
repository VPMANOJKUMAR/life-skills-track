# Event-Sourcing
## What is Event Sourcing?
Event Sourcing is a pattern for storing data as events in an append-only log. This simple definition misses the fact that by storing the events, you also keep the context of the events; you know an invoice was sent and for what reason from the same piece of information. In other storage patterns, the business operation context is usually lost, or sometimes stored elsewhere.

An event, within the context of Event Sourcing, represents a fact that took place within your business. 

Every change made is represented as an event, and appended to the event log. An entity’s current state can be created by replaying all the events in order of occurrence. The system information is sourced from the events. As the event has already happened, it’s always referred to in the past tense (with a past-participle verb, such as InvoiceSent). 

Many assume Event Sourcing is primarily for auditing, but this is a limited view of the pattern. An append-only log is great for auditing, but Event Sourcing is so much more than that. An audit log is a chronological record of what changed without any context. As the context is stored within the events, the ‘why’ and ‘when’ of the event are implicitly stored within the data for the event. Beyond preparing an audit log, an event-sourced system has a wealth of information and context stored within that can be incredibly valuable to the business.

Event Sourcing has a wide variety of applications across many sectors, including finance, logistics, healthcare,  retail, government, transport, video game development and many more.

## How does Event Sourcing work?
Consider a simple banking example of a current account. In such a system there is a single entity, representing a bank account. For simplicity, we assume there is a single account, so it does not need to be explicitly identifiable via an account number or otherwise. The account contains the current account balance.

Two commands are available – instructions to deposit and withdraw money, and these need to specify the amount of money to either deposit or withdraw. A business rule ensures a withdrawal command can only be processed if the amount requested is equal to or less than the current account balance.

Given this design, two events can be identified – Account Credited and Account Debited. These events include the amount of money that was deposited or withdrawn. This could even be simplified to a single event with either a positive or negative amount, but for the sake of this example they are kept separate.
![this is image](https://media.sitepen.com/blog-images/2020/03/image2-1.png)
Notice that the events are ‘past tense’ – they specify what happened in the system at the time they were recorded, and are only recorded if processing a command was successful. With designs like this, care needs to be taken so that commands are not confused with events, especially if they effectively mirror each other.

Given the following command sequence:

1. deposit { amount: 100 }
2. withdraw { amount: 80 }
3. withdraw { amount: 50 }

The most basic event sourcing implementation requires an event log, which is just a sequence of events. The system processing these commands would end up with an event log of:

![this is image](https://media.sitepen.com/blog-images/2020/03/image3-1.png)
The third command could not be processed as the requested amount exceeded the available balance.

To derive the current account balance, the system needs to process or ‘source’ the events in sequential order. Given the two events, this derivation processing output would look like:

bank account { current balance: 0 } (starting state)
bank account { current balance: 100 } (processed: Account Credited, +100)
bank account { current balance: 20 } (processed: Account Debited, -80)
The current balance gets determined by processing all events up to the current time, but as each event has an implicit timestamp of when it was recorded, the state of the bank account’s balance can be determined at any point in time by only processing events up to that time.

This is a complete (if trivial) event sourcing design. In a real system, this example would likely require a few more pieces.

Implementers may want to record the sequence of commands to be able to identify how an event came to be, as well as have a separate ‘error event’ log that can record command requests that failed to process, so that accurate error handling can take place and to maintain complete history of the entire system – successes and failures.
Over time as the number of commands increases, the system may also want a way to record a running tally of the current account balance, so that when a withdraw command gets received, the business logic doesn’t need to reprocess the full list of events everytime to determine if the command can get processed (i.e. that the account has sufficient balance available to allow the withdrawal). This is an example of a derived state store, and is effectively the same as what an entity store would be for the system.

The following illustrates what the entity store for this example would look like, once all commands have been processed:

![this is image](https://media.sitepen.com/blog-images/2020/03/image1-1.png)
It’s clear this is a lot simpler than an event store implementation – which is a big reason why many designers choose to only use entity stores. The current account balance is immediately available to read without having to process all historic events.

It is not however an either/or question between event sourcing and maintaining entity stores. It is often the case that entity stores are also present within event sourcing designs.


## Why would one use Event Sourcing?
Event Sourcing is by no means the silver bullet for software systems. Yet it can be an incredible tool if you know all the whens and the hows.

Event Sourcing provides a complete audit trail of business transactions. You can't delete a transaction or change it once it has happened. You can use those stored transactions to determine the system's state at any previous point in time by querying the events up to that point in time. It enables you to answer historical questions from the business about the system.

Troubleshooting is another fantastic Event Sourcing advantage. You can troubleshoot problems in a production system by copying the production event store and replaying it in a test environment. By troubleshooting, you might discover source code errors. Rather than performing risky manual data adjustments, you can fix the code and replay the event stream letting the system recalculate values correctly based on the new code version.

Event Sourcing often leads to better performance and scalability than complex relational storage models since you only use append-only operations for small immutable event objects. Having append-only operations allows you to reap the benefits of databases such as Cassandra. Moreover, Event Sourcing is usually combined with CQRS, which could be a crazy improvement compared to the "classic" relational storage models.
### Represents how we think
In the real world, people think in events. When somebody asks you how your day was, you will most likely tell them the interesting events that occurred. And you probably won’t describe the exact state of the world at time X.

Similarly, a domain expert will probably talk about a series of events when describing a process. When you use event sourcing, it becomes easier to model this in your system.

### Generating reports becomes easy
Want to know how many times a user changed their email address? With event sourcing, you already store all this data.

Want to know how many times an item is removed from a user's cart? Simply count the EventRemovedFromCart events.

With event sourcing, you have way more insights into your data, and you can generate reports retroactively.

### You have a reliable audit log
You can generate an audit log that shows exactly how a system got into a state.

Think of your bank account, for example. Event sourcing generates a log of transaction events that makes it crystal clear why I’m broke at the end of every month.
![this is image](https://www.eventstore.com/hubfs/ES-Pillar-10.svg)

## Domain-Driven Design
Domain Driven Design is important because it requires meaningful and continued collaboration between the architect and the product owner. It takes the developer away from the purely technical and theoretical world, and imposes a reality on their development skills. It also forces the product owner (or customer) to think through what they require from the system and recognise the capabilities of it. By making all the stakeholders cooperate, a shared understanding is created, and progress is more efficient.

The creation of a ubiquitous language involves Knowledge Crunching, the process of taking unrelated terms from business and development and creating something from the scattered terms. It’s collaborative, it can be messy, but can also be the beginning of a beautiful friendship.

Using Domain Driven Design with Event Sourcing is not mandatory. However, the main concepts such as speaking the same language as the business and proper business process modelling are also good foundations for building an Event Sourcing system. The better understanding we have of the business processed, the more precise the business information will be in our events.
## CQRS
CQRS is a development of CQS, which is an architectural pattern, and the acronym stands for Command Query Separation. CQS is the core concept that defines two types of operations handled in a system: a command that executes a task, a query that returns information, and there should never be one function to do both of these jobs. The term was created by Bertrand Meyer in his book ‘Object-oriented Software Construction’ (1988, Prentice Hall). 

CQRS is another architectural pattern acronym, standing for Command Query Responsibility Segregation. It divides a system’s actions into commands and queries. The two are similar, but not the same. Oskar Dudycz describes the difference as “CQRS can be interpreted at a higher level, more general than CQS; at an architectural level. CQS defines the general principle of behaviour. CQRS speaks more specifically about implementation”.
![this is image](https://www.eventstore.com/hubfs/ES-Pillar-14.svg)

## pros and cons
### Pros

- Great for fail-safety. If the downstream source fails, its data can be reconstituted from the event store.
- Extremely flexible. Any type of message can be stored. Any consumer can access the event store as long as appropriate access rights are granted.
- Excellent for real-time data reporting, especially when used with an event-driven message broker such as Kafka.  
### Cons
- Requires an extremely efficient network infrastructure given the inherent time sensitivity of the pattern and the susceptibility to potential latency issues.
- Requires a reliable way to control message formats, for example, using a schema registry.
- Different events will contain different payloads. There needs to be a single source of truth for defining and determining message formats for a particular event.
## Event Sourcing Conclusion
Event sourcing is a powerful pattern offering several valuable benefits. Another benefit is simplified future expansion, given the event log also serves as a long-timeframe pub/sub mechanism. New, unforeseen processing components or integrations can easily get added at any point, where they can then process the event log to bring themselves up to current state.

However as with any large architectural design decision, great care needs to be taken to ensure it is appropriate for a particular use case. Constraints around domain model complexity, data consistency/availability requirements, data growth rates and system lifespan/long-term scalability all need to get considered (by no means an exhaustive list!). Equally importantly, consideration also needs to be given for the teams that will be developing and supporting such a system over its lifespan.

## Reference
- (https://www.sitepen.com/blog/architecture-spotlight-event-sourcing)
- (https://www.redhat.com/architect/pros-and-cons-event-sourcing-architecture-pattern)
