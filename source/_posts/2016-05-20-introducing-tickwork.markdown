---
layout: post
title: "Introducing Tickwork"
date: 2016-05-20 07:02:42 -0400
comments: true
categories: ruby, coding
---

Tickwork is a Ruby library which supports scheduling of jobs. 
It is essentially a fork of clockwork, but has been significantly simplified. 
The simplification comes mainly from the removal of the self-driving time engine. 
That is to say, tickwork requires external calls to move through time.

Why would you possibly want a scheduling library which can't execute it's own schedule?
The typical way to run an application relying on clockwork is to have a process dedicated to running the schedule.
This process basically just spins in a loop checking for work that needs to be done. 
There are a couple of negatives to this:

1. It requires a separate process. On platforms like Heroku, that involves running an extra dyno. 
2. It is vulnerable to missing jobs during restarts, deploys, or other failures. We had this problem at my old company, Thinknear. 
We scheduled critical jobs to run in the top half of the hour and would deploy in the bottom half of the hour.
If a job is missed, there's no easy or built in way to catch up.

Tickwork, on the other hand, does not require a separate process (though you may still choose to use one). 
It also doesn't miss jobs. 
Tickwork only moves through time (scheduling jobs) when told to do so. 
It is not vulnerable to missing work due to restarts, long running jobs, or other externalities.
It uses an "at least once" approach vs. clockwork's "at most once". 
If a tickwork run is interrupted, say, due to a deployment, it never records finishing and so will re-run that period of time on its next invocation.
Furthermore, if tickwork is run twice, and the second run is initiated before the first run finishes, then it will repeat the same run.

This robustness does come at a cost.
Unlike clockwork, tickwork requires a datastore. 
It requires only a small omount of data, just one master timestamp and one timestamp per recurring job.
Tickwork is compatible with ActiveSupport::Cache::Store, the only caveat being that it must be a shared cache across all application hosts. 
Creating a Database table would also be trivial (coming in future blog post).

As a last note, clockwork supported dynamic scheduling of jobs via the database.
This feature no longer exists in tickwork.
