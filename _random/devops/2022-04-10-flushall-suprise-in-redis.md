---
layout: post
title: Flushall suprise in Redis
subtitle: A suprise I found when I tesed a Redis cluster configuration
nav_order: 1
parent: devops
---

## Problem

While testing a application that has been using Redis Cluster as a cache engine, I found a interesting behavior that lead to a unexpected result. The result is Primary Node didn't sync with Replica Node after I ran a command `flushall`. No matter how many time I restart the cluster or run the sync command, Primary Node has no data but Replica Node contains every key that should be removed.

## Story

Our application has been using Redis Cluster to cache heavy read rows. So to get the new state of changes in the database, I have to remove all related keys. But, the word `but` lead to many weird things, I am too lazy to delete each key. That why I chose `flushall` as my rescue command to remove all keys in the Primary Node.

Sooner, I realised my application does not work as I expected. They still used the cache value that I removed by `flushall`. Double check between the Primary and Replica Node, I found that the data was not synced. With 100% confident, I had restarted the cluster and hoped it back to work. Unfortunately, it was not.

## Solution

After some painful testing, I found that I could replace the key in Replica Node by set new value in Primary Node. After that, if I removed the key by `del` command, Replica Node will be synced and the key was gone.
