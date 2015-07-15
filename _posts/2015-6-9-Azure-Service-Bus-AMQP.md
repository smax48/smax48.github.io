---
layout: post
title: Azure Service Bus & AMQP
---

When you have a requirement to integrate .NET & Java applications, one very reliable option is to use a service bus.
In my situation, because the main part is written in .NET for hosting in Microsoft Azure, the obvious choice is Azure Service Bus.

Azure Service Bus supports a number of different APIs, e.g.:

* Native .NET client
* REST API
* AMQP 1.0

If you want a reliable handling of messages in your java app, the last option might seem very attractive, because in this case (in theory) you can use standard JMS-compatible libraries and even deploy a full-featured J2E application server.

There are some articles on the Azure documentation website which describe how to use AMQP from a Java code:

1. [Overview](https://azure.microsoft.com/en-gb/documentation/articles/service-bus-amqp-overview/)
2. [Code example](https://azure.microsoft.com/en-us/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/)

At first sight, everything is fine. Just start from the sample code and go ahead :-).  However, after more investigation things get more complicated:

1. Only two Java frameworks are known to work with Azure Service Bus AMQP implementation (Apache QPID, SwiftMQ). It is a bit weird.
2. It seems that you MUST use a very specific version of Apache QPID (link is provided in the second article above).  
With the latest build from the Maven Central Repo you will get auth errors (because of Azure SAS keys I think).
This basically means that you will have to live with all potential bugs that are inevitably present in that specific version that works (which is not very recent).
3. I couldn't find any code sample for the second mentioned library (SwiftMQ)
4. In the discussion on the second article people complained about forced disconnection after a certain period of time of doing nothing (i.e. just listening to new messages).
So you need to handle this situation as well (not sure if any JMS implementation can help you with this. I am not a J2E expert.)
5. It is very interesting what would happen if Azure SB itself will be restarted :)
So despite the fact that AMQP is fast and supports two-way communications, at this moment it is very risky to use it with Azure SB in a real project.

Hence your are left with the only choice - REST. Fortunately, Microsoft provides a wrapper library for use in Java, and it supports timeouts for GET requests.
So, if the source queue is empty, a read request will be blocked until there is a new message or timeout triggers.

Because of the stateless nature of REST services, it is possible to implement a simple endless loop for listening without any bad consequences (of course you will need some sort of cancellation capabilities). Besides this, all you need to do is to monitor your Java app and restart it if necessary.
In my case I am going to deploy the listener java app as a daemon in Linux and use a combination of CRON tasks & JMX to monitor the current application state.
