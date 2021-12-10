---
id: 115
title: Unit Testing with DateTime.Now
date: 2018-03-20T11:48:52+00:00
author: tom
layout: revision
guid: http://3.8.252.104/2018/03/20/3-revision-v1/
permalink: /?p=115
---
I recently had to write some unit tests for a method which had various outcomes depending on the current Date/Time, this created a problem as the method used DateTime.Now and there is no way to [moq](https://github.com/moq/moq) it. I tried a few different ways of doing this and did a bit of Googling and came up with the following solution. I can&#8217;t take credit for this as it came from [this stackoverflow post.](https://stackoverflow.com/questions/2425721/unit-testing-datetime-now)



The way this works is that you use SystemTime within your methods instead of DateTime and then change the value depending on the test using SetDateTime.