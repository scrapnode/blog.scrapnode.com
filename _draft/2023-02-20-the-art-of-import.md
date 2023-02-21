---
title: The art of Import
date: 2023-02-20 21:00:00 +0700
categories: [Solutioning]
tags: [dataflow]
---

Almost every system has a dataflow that involves importing and exporting entities for various purposes. Having worked with these systems for eight years, I have encountered a variety of ways to implement them, but none have made me entirely satisfied. Some are easy to implement, but they don't perform well at larger scales, while others are effective solutions that work well at any data size, but are not straightforward. For example

- A performance-focused approach involves creating an export request and returning a task ID that allows the client to use a real-time connection to check the task status and download URL.
- On the other hand, a feasibility-focused approach entails making some queries, transforming them in memory, then streaming the results to the client. However, the more items that need to be exported, the more memory is required

Today, I want to introduce to everyone a solution that I have been using for a long time, which strikes a balance between performance and feasibility.

## Implementation

![export-workflow](/assets/img/2023-02-20-export-workflow.png)