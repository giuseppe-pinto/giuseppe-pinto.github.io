---
title: "Engineering Multimodal Travel: My Journey at Waynaut"
date: 2023-08-16 09:00:00 +0200
categories: [tech, career]
tags: [java, algorithms, graph-theory, backend, startup]
image:
  path: /assets/img/posts/2023-08-16-engineering-multimodal-travel-waynaut/waynaut.png
---


<style>
  .preview-img, img.preview-img, figure.preview-img { display: none !important; }
</style>

![Waynaut Platform Architecture](/assets/img/posts/2023-08-16-engineering-multimodal-travel-waynaut/waynaut-1.jpg){: .w-75 .shadow .rounded .mb-4 }

Hello everyone! I am writing this to share the story of my first professional experience at **Waynaut**. I started my journey with this company in 2014 when I moved to Milan. At the time, I was still a student and new to the world of technical interviews and professional development. I am deeply grateful to the team for believing in my potential and giving me the chance to grow.

## What was Waynaut?

Waynaut was a startup with a bold mission: to create a platform capable of selling **multimodal travel solutions**.

But what does "multimodal" actually mean? It refers to combining several transport options—flights, trains, buses, and public transport—into a single, seamless journey. The goal was to provide customers with a complete set of solutions based on their criteria, mixing different types of transportation in one package.

### A Concrete Example
Imagine you need to travel from **Milan to Salerno**.
* In Milan, you have several airports.
* In Salerno, however, there is no airport.

So, a typical user would need to:
1. Buy a flight from Milan to Naples.
2. Take a taxi or bus to Naples Central Station.
3. Take a train from Naples to Salerno.

A **multimodal platform** solves this headache by finding this combination automatically and offering it as a single result (and potentially a single ticket), avoiding the need to visit three different websites.

> **Fun Fact:** Waynaut’s vision was supported by research from the European Commission and aligned with the multimodal vision of Amadeus.

---

## My Role & The Tech Stack

When I joined Waynaut, my primary responsibility was to contribute to the development of this platform from the ground up. We had to make several architectural decisions to handle this complexity.

### The Foundation: Java & Geonames
We opted to use **Java** as our main programming language and **MySQL** as our database system.

Our first major challenge was creating a comprehensive database of "access points"—airports, train stations, bus stops, and city centers. After careful analysis, we established our foundation using **Geonames**, which allowed us to map thousands of locations accurately.

### Integrating the Providers ( The REST vs SOAP Struggle)
Simultaneously, our commercial department secured partnerships with major transportation providers, including:
* **Kiwi.com**
* **Trenitalia**
* **BusBud**
* **Volagratis**
* **Traghetti Lines**

My job was to integrate their APIs into our system. This was challenging because every provider used a different technology. Some offered straightforward **REST APIs**, while others relied on protocols like **SOAP**.
We had to build an abstraction layer to normalize all this data into our internal domain model.

---

## The Core Challenge: The Graph & The Algorithm

Once we gathered data from these multiple sources, the real engineering challenge began: **How do we combine them to generate the best route?**

It became evident that we were dealing with a **Graph Structure**:
* **Nodes:** represented the access points (stations, airports).
* **Edges:** represented the connections (flights, train rides).

The question was: *How could we provide optimal solutions in milliseconds?*

### The A* (A-Star) Algorithm
After researching various shortest-path algorithms, we settled on implementing the **A* Algorithm**. We enhanced it using a **reactive programming approach** to handle the asynchronous nature of calling multiple external APIs simultaneously.

> **Academic Milestone:** This implementation became the central theme of my **Engineering Thesis** at the *Università degli studi della Campania Luigi Vanvitelli* titled *"Shortest path algorithms with a multimodal case study"*.

---

## Infrastructure & Evolution

Beyond the core algorithm, I had the opportunity to work with:
* **MongoDB:** For handling unstructured data.
* **AWS (Amazon Web Services):** Where our entire infrastructure was deployed and scaled.

After our first successful launch, Waynaut caught the attention of big players in the industry. Eventually, the company was **acquired by the lastminute.com group**.

This journey was truly remarkable. Building a novel, highly challenging system from scratch gave me the perfect foundation to specialize in the technological domain of travel.

