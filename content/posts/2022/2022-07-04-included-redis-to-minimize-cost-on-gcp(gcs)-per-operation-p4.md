---
title: "Included Redis to minimize cost on GCP(GCS) per operation"
series: ["Creating an open source Python project from scratch"]
date: 2022-07-04T00:00:00+00:00
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

Cloud computing is a blessing. But we need to use it wisely. In this post we would try to minimize out cost on GCS GCP using Redis.

# Continuous Integration

SO, Google Cloud Service provide Block Storage service by the name of Google Cloud Storage (GCS). It is the equivalent of Amazon’s S3 and Azure’s Blob Storage. At least we are taking them as comparable.

The basic concept of Blob storage is pretty much same. They allow you to create some bucket. And put your objects on them. On a note, the bucket name must be **globally unique**. [Here is why?](https://stackoverflow.com/questions/24112647/why-are-s3-and-google-storage-bucket-names-a-global-namespace)

We are trying to connect GCS through a Golang service. And after that we may connect a Redis DB to reduce the hit on GCS. This will bring down our cost in GCS.

*I did the work, or my work was inspired from [this site.](https://adityarama1210.medium.com/simple-golang-api-uploader-using-google-cloud-storage-3d5e45df74a5)*

Making of the environment is almost same like mentioned in above. I will talk about the problems I had faced.

Creating a Bucket and straight forward. Log into https://console.cloud.google.com. Make a project. Choose Cloud Storage and Start creating your desired buckets. Google will help you there to make clear the terms and take decision.

The problem I had faced is, when I have to make a auth key so that any services with that specific key may make any changes in my GCS bucket.

The above mentioned blog cleared the topic but somehow that didn’t worked for me. So I had to find my own way.

To make that auth key, I went on ["https://console.cloud.google.com/iam-admin/serviceaccounts"](https://console.cloud.google.com/iam-admin/serviceaccounts) there. I had to choose my project. And then click on the **“+ CREATE SERVICE ACCOUNT”** button. Type your ***Service account name.*** Then GCP will generate a ***Service Account ID*** of his own for you

The full client side library support you may need, will be found [in this URL](https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-go). I will focus on the Redis part mainly.

So, My use case was, to make an image repository to store all the data provided by our end users and our admin users. There might be so many categories. I am taking only Users as example. And they can upload as many image as they want.

So I made each bucket as per the category and made directory inside as per their primary id key. So it became easy for me to search the items next time by their category/bucket and their ids.

A full list of all useful action you need, [is available on here.](https://cloud.google.com/storage/docs/samples/storage-upload-file) I am not going to repeat them here.

***So, till now, making a GCS project on GCP, creating bucket, making an JSON auth key, uploading object, getting them, modifying them and deleting is done.***

*I hope you had found out the way. If not, please go through the links I provided.*

&nbsp;<br>
&nbsp;<br>
&nbsp;<br>
&nbsp;<br>

Here comes another part.

I had made a project on GCP GCS. It is up and running. Serving like hell :)

**BUT, I am using a service provided by Google. It costs us money, as they serving. In every GET, POST, PUT, DELETE. In every HIT. And if the count of your users go up, it might happen that, Google may eat up your bank account.**

*So, what to do?*

In a common sense/case, your image needs to read more frequently then writing. Also from updating or deleting them.

We can assume the case that, a user uploads a photo of his NID. So then, the image might be blurry, or wrong. He needs to DELETE it and RE-UPLOAD. Fine. This action can take place maximum 4/5 times in the lifetime of an image. For rest of the time, this user needs to READ this image again and again and again.

And every time your user READ this image, Google will charge you.

    To reduce that costing, we are putting a cache here. The idea is like,

    1. User POSTED an image. Upload it to GCS. Make the V4 URL to access the image from anywhere. Save this URL in Redis with respective dir path like in the GCS(to make it abstract).

    2. User DELETED an image, make the change in GCS. Make an updated V4 URL. Update it to the Redis.

    3. User UPDATED this image. After doing the update, update it to the Redis too.

    4. And Most importantly, In every GET action, Our Redis has the most updated V4 URL. Serve the user from the Redis. We don’t have to ask GCS every time to make an V4 URL. And thus we save our money.

There is an example for that:

<script src="https://gist.github.com/Masum-Osman/a1bafec1a9d7e8b70fe0c35f61990b24.js"></script>

I had initiated a Redis DB connection. Then I just have to have more three function. For **Get, Set** and **Delete** actions.

<script src="https://gist.github.com/Masum-Osman/380f2a6fba8d323bd7e600e87846d0b5.js"></script>

I am setting my data here. I would like to add some information here. You may see that, I am using SET as the data structure here. I didn’t used the default STRING type here. Because I had said, a user may upload as many picture as he want. So saving them in STRING is not a good idea because someone who may use my V4 URLs, might not parse them according to the image type. So I’ve decided to use SET so that any kind of end user can split the URLs by index or any other way…

And the other thing is, I am setting the Redis Expiration time near to 7 days.

The reason for that is, GCP gives us a V4 URL that has maximum lifetime of 7 days. After that, the URL will not response with the object you need in return. So to Minimize the GCS call along with ensuring the Maximum availability, I have chosen the time less then few minute so that the URL may not fail to response with expected data, and the hit on GCS is minimized.

<script src="https://gist.github.com/Masum-Osman/12b653bfa2cfc99ea8fbd77f5438b7cf.js"></script>

<script src="https://gist.github.com/Masum-Osman/83a02891b7ad6d5f712d73b78fe2f992.js"></script>

So, how to decide between?

<script src="https://gist.github.com/Masum-Osman/240713e4d3f03e9277d1e33370fde86a.js"></script>