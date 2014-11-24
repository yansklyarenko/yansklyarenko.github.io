---
layout: post
title: "Build queue starvation in CC.NET"
date: 2014-11-24 22:54
comments: true
categories: 
---

Recently I've come across an interesting behavior in CruiseControl.NET in regards to the build queues and priorities. 

If there are many projects on one server, and the server it not quite powerful, and more than one build queue is configured, and (that's the last one) these build queues have different priorities, you might end up in a situation when CC.NET checks for modifications the same set of projects all over again, and never starts an actual build. If you add the projects from that server to the CCTray, you can observe the number of projects queued for the build has reached a certain number and never decreases.

This phenomenon is call "build queue starvation". It was [described and explained by Damir Arh in his blog](http://www.damirscorner.com/AvoidingQueueStarvationInCruiseControlNET.aspx).

Let me summarize the main idea.

When one build queue has a higher priority than another queue, CC.NET favors the projects from the first queue when scheduling for modifications check. Now imagine that a trigger of the projects from the higher priority queue is quite small and the number of such projects is big enough. This leads to the situation when the first project in high priority queue is scheduled for build the second round before the last project in that queue has built its first time. 

As a result, the lower priority queue is "starving" - none of its projects ever gets a chance to be built. The fix suggested in the link above suits my needs - the trigger interval has just been increased. 

I should say it's not easy to google that if you're not familiar with the term "build queue starvation". Besides, CC.NET doesn't feel bad in this situation, and hence doesn't help with any warnings - it just does its job iterating the queue and following the instructions.


> Written with [StackEdit](https://stackedit.io/).