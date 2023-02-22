---
title: The art of Import
date: 2023-02-22 19:00:00 +0700
categories: [Solutioning]
tags: [dataflow]
---

In previous blog post titled [The art of Export]({% post_url 2023-02-20-the-art-of-export %}), I covered various aspects of the exporting data. In this post, we will discuss about the oposite workflow: importing data. The solution chosen for importing will be effected by a critical component - the storage - because without storage you would have to recieve data chunks directly from clients. This would blocks your APIs from handling other requests and degrade your system's performance, especially if you don't use data streaming in your programming language. That why haveing a storage storage system that can handle uploading robustly is a requirement to get deep into my solution. If you are developing your system on cloud, you can easily find an appropriate storage solution from your provider, such as S3 on AWS or Cloud Storage on GCP.

## Implementation

There is a diagram that describes the workflow of my solution, which consists of three parts. First, the client makes a request for an upload URL and an Import ID which they will use to upload their heavy file and check the task status later. Second, after the file is successfully uploaded, a trigger message is fired, and a worker will listen to that event. The worker has the responsibility to do every optimization to make the import as fast as posible and update the task's status into the database once the import is completed. Finally, the client repeatedly checks the task's status until it is completed (either succesfully or unsuccessfully).

![import-workflow](/assets/img/2023-02-22-import-workflow.png)
