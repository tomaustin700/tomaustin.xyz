---
id: 3
title: Unit Testing with DateTime.Now
date: 2018-03-20T11:04:35+00:00
author: tom
layout: post
guid: https://192.168.1.10/?p=3
permalink: /2018/03/20/unit-testing-with-datetime-now/
categories:
  - 'c#'
  - unit testing
---
I recently had to write some unit tests for a method which had various outcomes depending on the current Date/Time, this created a problem as the method used DateTime.Now and there is no way to [moq](https://github.com/moq/moq) it. I tried a few different ways of doing this and did a bit of Googling and came up with the following solution. I can&#8217;t take credit for this as it came from [this stackoverflow post.](https://stackoverflow.com/questions/2425721/unit-testing-datetime-now)



The way this works is that you use SystemTime within your methods instead of DateTime and then change the value depending on the test using SetDateTime.