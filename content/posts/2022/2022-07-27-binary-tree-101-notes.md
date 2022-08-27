---
title: "Binary Tree Notes 101"
series: ["Binary Tree Notes 101"]
date: 2022-07-27T00:00:00+00:00
draft: false
author: "Masum Osman Khan"
canonical: https://blog.devops.dev/included-redis-to-minimize-huge-cost-on-gcp-gcs-per-hit-e64112fe5f75
categories:
  - blog
tags:
  - GCP
  - Golang
  - Redis
thumbnail: golang
---


### Array representation of a Binary Tree:

1. **To store a tree in array,** there is a formula. 

The Index a data may get, depends on the following formula-

***Left : 2*i + 1***

***Right: 2*i + 2***

1. **Get to know the L/R position of a specific node:**

If the index is **odd**, It is a Left Node. (Because, ***2*i + 1***)

If the index is **even**, It is a Right Node. (Because, ***2*i + 2***)

1. **To Get the parent of a child node is:**

For the left node: Parent = **i/2**

For the right node: Parent = **i/2 - 1**

1. Length of Array:
    
    **2^x -1**
    

Here x is the level of a tree.