---
layout: post
title: "Tickwork powered by AWS"
date: 2016-05-20 16:12:46 -0400
comments: true
categories: ruby, coding, aws
---

[My previous post](http://softwaregravy.com/blog/2016/05/20/introducing-tickwork/) introduced [Tickwork](https://github.com/softwaregravy/tickwork) which performs scheduling in Ruby, but doesn't contain a clock.
The intentionally missing component of the scheduling library means that something must regularly call the manager to drive the scheduling of events.

AWS recently introduced [Cloudwatch Events](https://aws.amazon.com/blogs/aws/new-cloudwatch-events-track-and-respond-to-changes-to-your-aws-resources/) -- essentially a cron in the cloud that does nothing on it's own.
We can use this to power Tickwork. 

The way we can do this is by having Cloudwatch Events push to [SNS](https://aws.amazon.com/sns/), which we'll then call out to our Rails app with an HTTP post.
How much does this cost? $0. Cloudwatch Events are free (as far as I can tell), and we are comfortably in the free tier for SNS. (100,000 HTTP calls per month are free, then just $0.60/1 million after that. [source](https://aws.amazon.com/sns/))

To make this easy, I've created [AWS Tickwork](https://github.com/softwaregravy/aws_tickwork), an engine for Rails. 
The main part of AWS Tickwork is an SNS controller which makes receiving posts very easy. 
This controller simply calls Tickwork.  The engine also contains an optional migration to use ActiveRecord as the datastore required by Tickwork. 

A full app example using AWS Tickwork can be found in the [spec directory](https://github.com/softwaregravy/aws_tickwork/tree/master/spec/dummy). 
I'm also using this engine successfully in another project ... maybe a future post.

