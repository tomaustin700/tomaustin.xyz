---
id: 136
title: 'Monitoring C# applications using Bosun'
date: 2018-12-11T21:35:48+00:00
author: tom
layout: post
guid: http://3.8.252.104/?p=136
permalink: /2018/12/11/monitoring-c-applications-using-bosun/
categories:
  - bosun
  - 'c#'
  - devops
---
I recently stumbled across [this interesting write-up](https://nickcraver.com/blog/2018/11/29/stack-overflow-how-we-do-monitoring/) by Nick Craver about how [Stack Overflow](http://www.stackoverflow.com) do their monitoring. If you have any interest in DevOps or infrastructure in general I highly recommend giving it a read. One section caught my eye&#8230; [Bosun](https://bosun.org/).

  


Bosun is a data collection tool for metrics and meta data, if you&#8217;ve read any of my previous posts you will have seen my post about the metric gathering package I wrote called [Metricity](http://tomaustin.xyz/2018/05/21/metriticity-a-net-metrics-gathering-nuget-package/). I started writing Metricity because I couldn&#8217;t find anything which could collect metrics of my choosing and store them somewhere on an internal network. Metricity worked but you had to spin up your own sql database to store the data and there was no UI for analysing and viewing the data (I did start writing a website using [Vue](https://vuejs.org/) which could show the data but that&#8217;s for another post). Bosun does everything Metricity can and more!

  


By default Bosun can record metrics from servers (cpu load, clock speed, hdd capactity, etc) which is useful but the feature which really caught my eye was its C# Nuget package called [Bosun Reporter](https://github.com/StackExchange/BosunReporter). Bosun Reporter allows any data to be sent to Bosun from within your application, I decided I had to test this out so the first thing to do was spin up a Bosun server.

  


The Bosun homepage has an excellent [quick start guide](https://github.com/bosun-monitor/bosun/releases) which covers how to install the server using Docker, something to be aware of is that running Bosun from a Docker container is not currently supported in production environments. This is fine for me but they do provide versions for pretty much every operating system [here](https://github.com/bosun-monitor/bosun/releases). I have been meaning to get into Docker for a while so this seemed like a great time to spin up a new Ubuntu server, install Docker and pull the latest Bosun version from the Docker Registry.

  


I followed the quick start guide and quickly had Docker installed and Bosun running &#8211; that was easy! If you want to do this yourself without following the guide simply select Docker when prompted during the Ubuntu installation (this became an option on 18.10). Once installation has finished run:

<pre class="wp-block-preformatted">docker run -d -p 4242:4242 -p 8070:8070 stackexchange/bosun:0.6.0-pre</pre>

This differs ever so slightly from the quick start guide as the guide uses version 0.5.0 which I did try at first but various features were missing. After a quick chat in the [Bosun slack](https://bosun.slack.com) I was told to try the 0.6.0 pre release which had everything I needed. The instance should start and you should be able to access your server using the IP of your Docker server and the port number (in our case 8070). You should be greeted with the following:<figure class="wp-block-image">

<img loading="lazy" width="1205" height="365" src="https://i2.wp.com/www.tomaustin.xyz/wp-content/uploads/2018/12/image-8.png?fit=640%2C194&ssl=1" alt="" class="wp-image-137" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-8.png 1205w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-8-300x91.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-8-768x233.png 768w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-8-1024x310.png 1024w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-8-720x218.png 720w" sizes="(max-width: 1205px) 100vw, 1205px" /> </figure>  


At this point I do suggest going through the quick start guide as it explains in excellent detail all the features and workflow. From this point I will assume your fairly familiar with the UI (there isn&#8217;t much to it!).

  


Because I wanted to see how easy it was to push data from Bosun Reporter I made a new C# console application and installed Bosun Reporter. If you want to skip ahead and see the finished code then it&#8217;s on my GitHub [here](https://github.com/tomaustin700/BosunExample) but I will go into the details a bit here.

The first thing to do is create a Metrics Collector and setup the connection and setting.

It is recommended that the collector is made static, you would then initilise it when your application loads up. Now that we have collector setup we can start collecting some metrics! I am only going to go into recording metrics using EventGauge however Bosun Reporter supports several different ways of collecting data, these are explained in detail within the [documentation](https://github.com/StackExchange/BosunReporter).

  


For this example I have created a simple console application that calls an API to get the weather for London, we will then measure the response time and log it in Bosun. We first need to create the metric we want to use for recording, this is as simple as calling CreateMetric.

We have to specify a few parameters to give Bosun some context to what we are collecting, in our case GetWeather corresponds to the metric name, time taken is the unit of measurement and the last parameter is a short description of what&#8217;s being collected. Adding data to the GetWeather metric is as simple as calling .Record and passing in the data you want to record. To make things a bit more reusable I wrapped up the recording in a Time method which accepts a Func<Task> as a parameter along with the EventGuage we made earlier. This method is called in the following way:

In order to record the length of time the Func<Task> takes to execute we first create a stopwatch and start it, then we Invoke our method, stop the stopwatch and finally record our data in our metric.

In my application I have setup a simple ElaspedEventHandler which then calls the Time method every 5 seconds which has the GetWeather method passed into it. This is just to get some data into Bosun so we can have a look at what we get. We can now run our application and start generating some data. Something to be aware of is by default Bosun Reporter will only submit data every 30 seconds (this is a nice touch as it stops us spamming Bosun), this can be tweaked by changing the SnapshotInterval parameter of the MetricsCollector.

  


Now if we open Bosun and have a look hopefully we should start seeing some data rolling in. The first thing we should notice is that under Items and then Hosts we should see our device name.<figure class="wp-block-image">

<img loading="lazy" width="452" height="152" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-9.png" alt="" class="wp-image-150" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-9.png 452w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-9-300x101.png 300w" sizes="(max-width: 452px) 100vw, 452px" /> </figure> 

So far so good! 10cbc024198e is the name of the Bosun server and tom-pc is what we are running our application on. If we then click that device name and select Available Metrics we should see our data.<figure class="wp-block-image">

<img loading="lazy" width="425" height="281" src="http://tomaustin.xyz/wp-content/uploads/2018/12/image-10.png" alt="" class="wp-image-151" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-10.png 425w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-10-300x198.png 300w" sizes="(max-width: 425px) 100vw, 425px" /> </figure> 

TestApp.GetWeather is the data we have pushed! As you can see I tried a few of the other data gathering features of Bosun Reporter but lets dive deeper into GetWeather.<figure class="wp-block-image">

<img loading="lazy" width="1166" height="580" src="https://i2.wp.com/www.tomaustin.xyz/wp-content/uploads/2018/12/image-11.png?fit=640%2C318&ssl=1" alt="" class="wp-image-152" srcset="https://tomaustin.xyz/wp-content/uploads/2018/12/image-11.png 1166w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-11-300x149.png 300w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-11-768x382.png 768w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-11-1024x509.png 1024w, https://tomaustin.xyz/wp-content/uploads/2018/12/image-11-720x358.png 720w" sizes="(max-width: 1166px) 100vw, 1166px" /> </figure> 

We are presented with a nice graph showing our response time in ms and the interval of our API requests. From this we could then create an expression and use that to create an alert, this could then email us if a response took longer than our average response time. That&#8217;s just one example but Bosun supports all sorts of operators and aggregation techniques. I suggest checking out the [documentation](https://bosun.org/expressions) for a deeper dive into Bosun&#8217;s expression language. All of the source code for the example I created for this can be found [here](https://github.com/tomaustin700/BosunExample) and I suggest you give Bosun and Bosun Reporter a try for youself!