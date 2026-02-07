---
title: "Evolution of a high-performance system: from synchronous to seamless scalability"
date: 2023-11-20 09:00:00 +0100
categories: [ tech, engineering ]
tags: [ software-design, architecture, refactoring, message-driven, rabbitmq, redis ]
image:
  path: /assets/img/posts/2023-11-19-evolution-of-a-high-performance-system-from-synchronous-to-seamless-scalability/featuredImage.webp
---

This article discusses the transformation of a synchronous process in the context of lastminute.com's customer journey
for travel planning. The existing system was complex and resource-intensive due to its all-in-one nature, causing
scaling and deployment challenges. The article presents the evolved system that separates responsibilities into
microservices using message-driven communication.

## A journey to scalability

How many of you have found yourselves having to meet business requirements behind a synchronous process? I believe many,
just think, for instance, to the customer journey on the lastminute.com. The customers use it to plan unforgettable
vacations by selecting and reserving transportation and accommodations. Once they've viewed the homepage, they can enter
where they want to depart from, where they want to go, and consequently, the travel dates. After waiting for a few
seconds, they will be redirected to a new page, called **results page**, where there will be travel solution proposals
based on the requested itinerary. From this selection, our customer can choose one of the presented solutions and
proceed with the purchase.

But the main question is: **what happens during that ”waiting for a few seconds“?**

Clearly, we are dealing with synchronous communication between two systems. The first one is the results page requesting
travel solutions for a specific itinerary, and the second one is a system that provides them.

Let's focus for a moment on the second system, called **transportation supplier aggregator** which we will refer to with
**TA** throughout this article. Its main goal, as mentioned earlier, is to provide travel solutions for a requested
itinerary. But how does it accomplish this? It does so by aggregating travel solutions from various sources. These
sources are flight and train providers with which lastminute.com has commercial agreements like Global Distribution
Systems (GDS) and low-cost carriers providers.

The two systems, the results page and the TA, communicate via HTTP using the POST method and following the REST
protocol. If, due to business requirements, this type of process, method and communication cannot be changed, how would
you model your TA? The goal of this article is to showcase the kind of system we inherited and how we evolved it.

## The inherited system: an all-in-one system

![Inherited System Architecture](/assets/img/posts/2023-11-19-evolution-of-a-high-performance-system-from-synchronous-to-seamless-scalability/image1.webp){:
.shadow .rounded .mb-4 }
_Figure 1: The inherited system architecture_

The figure above depicts the system inherited by our team, the numbers in the circles indicate the various phases as
time progresses. The results page communicated with the TA (phases 1 and 7), requesting travel solutions with a POST
HTTP method. The TA (a JAVA application), in turn, used to connect directly to external suppliers (phases 2 and 3). Upon
receiving solutions from them, the TA gathered all the responses (phase 4), applied some business logic (phase 5), and
prepared the results for the results page (phase 6).

The phases I want to emphasise are number 2 and 3. When the system received a request for a specific route, it had to
distribute the requests among our N suppliers. As a result, a **main thread** existed that spawned **N collaborator
threads**, each responsible for contacting an individual supplier. Each collaborator thread was responsible for
generating a request for its designated supplier, utilising network protocols to establish communication, collecting the
result, and adapting it to the domain objects of the TA. Once all the collaborator threads had completed their tasks,
the main thread continued with its own responsibilities:

* **Phase 4:** waiting and collecting results from all collaborator threads.
* **Phase 5:** applying business logic to these results.
* **Phase 6:** adapting the results into Data Transfer Objects that represent the travel solutions.
* **Phase 7:** sending them to the results page.

As you can imagine, the inherited TA was a system that suffered from various issues, such as:

* **Multiple Responsibilities:** The system essentially owned everything, taking on various responsibilities.
* **Resource Intensive:** It required a significant amount of resources in terms of the number of instances, as well as
  high-performance components. This system had to handle approximately **50 million requests per day** (Yes, you read
  that correctly), which means around 600 requests per second (calculations were made on days with no particular
  traffic; however, in certain circumstances, the numbers can even double). Notably, every individual request led to the
  generation of N+1 threads to fulfil the system's goals. Each of these threads was responsible for handling complete
  responses from the suppliers, which, in certain instances, consisted of travel solutions encapsulated within an HTTP
  response of 25 megabytes. In summary, it was highly resource-intensive in terms of both threads and memory usage.
* **Scaling:** The system suffered from scaling and coupling issues. If a certain supplier integration was utilised more
  frequently than the others, scaling a single supplier integration wasn't feasible due to the interconnected nature of
  the system. Consequently, scaling required the entire system to be scaled up.
* **Deployment Complexity:** Making even a minor change, such as one related to a single supplier integration, was
  challenging to deploy. The entire system had to be redeployed for such changes.

These factors collectively made the inherited TA a complex and challenging system to manage and scale effectively. We
were only able to scale it horizontally.

## The evolved system

As a team, we had a clear goal of evolving the system while preserving the synchronous process between the TA and all of
its clients. Therefore, our focus was on determining how we could modify the communication process between the TA and
the suppliers. This area is where we had the potential to make changes. How would you have modified it? The following
image describes how we did it.

![Evolved System Architecture](/assets/img/posts/2023-11-19-evolution-of-a-high-performance-system-from-synchronous-to-seamless-scalability/image2.webp){:
.shadow .rounded .mb-4 }
_Figure 2: The evolved system architecture_

Also in figure 2 the numbers in the circles indicate the various phases as time progresses. First of all, we decided to
extract the integrations with our providers into dedicated and separate systems. For each provider, we created a *
*search driver microservice** with the following responsibilities:

* Receiving a request (phase 3).
* Translating it into a request for the specific provider (phase 4).
* Converting the provider's response into our domain (essentially creating solutions for our customers), and storing
  them in key-value storage called **driver solutions storage**, which is based on **Redis** (phase 5).

Once we had these satellite microservices in place, we needed to figure out how to manage communication with the TA.
This is where we considered introducing a message-driven system, and that's when queues clusters, based on **RabbitMQ**,
came into play. With RabbitMQ, the TA started to send and receive messages (phases 2 and 6) to/from the search driver
microservices on dedicated queues (we will see it in detail in a moment). After these microservices completed their
tasks, the TA could retrieve travel solutions (that is, the result of the search microservices calls to our providers)
from the storage (phases 7 and 8) and then perform business logic and adapt the results for the lastminute.com results
page, which awaited the user's request for solutions.

The important phases to explore are those related to the queues cluster and the corresponding messages exchanged between
the TA and the search drivers. Below, you will find figure 3 highlighting these phases.

![Search Driver Component](/assets/img/posts/2023-11-19-evolution-of-a-high-performance-system-from-synchronous-to-seamless-scalability/image3.webp){:
.shadow .rounded .mb-4 }
_Figure 3: Search driver interaction detail_

For simplicity, figure 3 illustrates only a single search driver (since the communication phases are the same for all
drivers). For the TA, there is a sequence of steps that can be divided into two main categories: a **scheduling phase
sequence** and a **collection phase sequence**.

The main thread, which has received the user's request, first enters the scheduling phase, where it needs to create a
search task for a specific search driver (referred to as the search driver component in figure 3). Of course, this task
is in a pending state, so the main thread needs to keep track of the count of these tasks created but not yet
completed (phase 1). This search task must be sent to the specific driver for execution. Now, let's delve into phase 2:
this task is dispatched via RabbitMQ to a designated driver, and the thread enters a waiting state (phase 3).

Let's now examine phases 4, 5, and 6. This search task is transmitted as a RabbitMQ message through a queue specific to
a search driver, known as the **search driver request queue** in the figure. This is accomplished using the **Remote
Procedure Call (RPC) pattern**, which is well-detailed on the RabbitMQ website, that is based on the more generic
request-reply pattern described here. Using this pattern implied the usage of two queues. The first one is the search
driver request queue (essentially the one that the RabbitMQ site calls `rpc_queue`). The second one is called the *
*search driver notification queue** in the figure (basically the one that the RabbitMQ site calls `callback_queue`).

As we mentioned earlier, the first queue will contain messages related to the search tasks for the workers (in our case,
the search drivers), along with two additional parameters: the `reply_to` queue (the search notification queue) and the
`correlation_id`. On the other hand, the search notification queue will contain messages containing completion
notifications for tasks and the `correlation_id` of the request. This arrangement allows the TA to identify which search
task corresponds to a given completion notification.

The reason for employing this pattern is clear: we need to invoke a function on a separate system, in our case, the
search driver, and react promptly to the result of this function on the TA side. The function that the search driver
must execute includes (as we just said): extracting the necessary information from the search task message to create a
request to an external supplier, calling it via an API, adapting the external supplier's results into a well-known
domain, storing them in the driver solution storage (phase 5), and sending a completion notification (phase 6). With the
TA receiving this notification, it can update its internal register counter for pending tasks (phases 7 and 8), thanks
to the `correlation_id`, and proceed to the collection phase sequence. Here, the main thread can exit the waiting state
immediately based on the absence of pending tasks and essentially retrieve solutions from the storage (phases 3 and 9).
Once phase 9 is completed, the TA can continue its flow by applying business logic to all the results from the various
search drivers and sending them all to the results page, making our customers happy!

## The accomplished goals of the evolved system

The goal of this new system is essentially to mitigate the drawbacks of the old one, so we can say that we have
accomplished the following:

* **Single responsibilities and reduced resource intensity:** In the new system, responsibilities have been distributed
  among more microservices. The search drivers now have a singular responsibility, which is communicating with external
  suppliers. The TA has become lighter; its primary responsibility now involves orchestrating with the satellite
  microservices and implementing business logic. This change has also led to improved resource efficiency compared to
  before.
* **Increased scalability and reduced deployment complexity:** The microservices are now less tightly coupled compared
  to the old system. As shown in figure 2, we can deploy each search driver and the TA as independent systems. This
  flexibility allows for easy independent scaling if a specific search driver requires more resources due to higher
  usage than others.
* **No violation of the original contract:** The various systems that utilise the TA, such as the results page, do not
  need to make any changes. The transition is transparent, and the original contract remains intact.
