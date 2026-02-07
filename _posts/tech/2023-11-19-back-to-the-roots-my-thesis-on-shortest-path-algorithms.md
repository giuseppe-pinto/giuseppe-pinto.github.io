---
title: "Back to the Roots: My Thesis on Shortest Path Algorithms"
date: 2023-11-19 12:00:00 +0100 
categories: [ tech, education, computer-science ]
tags: [ graph-theory, a-star, algorithm, waynaut, thesis ]
image:
  path: /assets/img/posts/2023-11-19-back-to-the-roots-my-thesis-on-shortest-path-algorithms/logo-uni.png
  alt: A* Algorithm Graph Exploration
---

<style>
  .preview-img, img.preview-img, figure.preview-img { display: none !important; }
</style>

![A* Algorithm Graph](/assets/img/posts/2023-11-19-back-to-the-roots-my-thesis-on-shortest-path-algorithms/a-star.png){: .w-75 .shadow .rounded .mb-4 }

Every senior engineer has a starting point. For me, the bridge between academic theory and real-world engineering was
built during my Bachelor's degree at the **Università degli Studi della Campania "Luigi Vanvitelli"**.

I am sharing my Engineering Thesis titled: **"Shortest path algorithms with a multimodal case study"**.

## The Context: Waynaut & Multimodality

This work wasn't just theoretical. It was deeply connected to my first professional experience at **Waynaut** (later
acquired by lastminute.com). The challenge was fascinating: how do you calculate the optimal route between two points
when the "graph" isn't just roads, but a complex mix of **trains, flights, carpooling, and buses**?

## What's inside?

In this paper, I explore the mathematical foundations that power modern travel search engines:

* **Graph Theory Fundamentals:** How to model real-world locations as nodes and transport connections as weighted edges.
* **The Algorithms:** A comparative analysis of **Dijkstra**, **Bellman-Ford**, and **A* (A-Star)**.
* **The "Agony" Metric:** How we weighted edges not just by price or duration, but by "agony" (a combination of wait
  times, number of transfers, and comfort).
* **Real-world Application:** How the **A* Algorithm** was tailored with specific heuristics (Manhattan, Euclidean,
  Diagonal distances) to power the **Wayfinder** engine.

If you are interested in Graph Theory, backend algorithms, or just want to see how a "young me" tackled these problems,
feel free to download the full document below.

---

### 📄 Download the Thesis

<a href="/assets/posts/2023-11-19-back-to-the-roots-my-thesis-on-shortest-path-algorithms/shortest-path-algorithms-with-a-multimodal-case-study.pdf" class="btn btn-outline-primary btn-lg">
  <i class="fas fa-file-pdf"></i> Download Full PDF (Italian)
</a>

_Note: The thesis is written in Italian, but the algorithms and math speak the universal language of code._
