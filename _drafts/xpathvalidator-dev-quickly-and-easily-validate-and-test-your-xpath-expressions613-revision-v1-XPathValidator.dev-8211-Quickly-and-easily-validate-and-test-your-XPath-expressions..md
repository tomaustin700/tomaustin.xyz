---
id: 622
title: 'XPathValidator.dev &#8211; Quickly and easily validate and test your XPath expressions.'
date: 2020-04-11T15:00:31+01:00
author: tom
layout: revision
guid: http://3.10.198.211/2020/04/11/613-revision-v1/
permalink: /?p=622
---
If you&#8217;ve never heard of [XPath](https://developer.mozilla.org/en-US/docs/Web/XPath) you can think of it as a quick and easy way to query an XML document, it uses a path notation for navigating through the hierarchical structure of XML similar to that of a URL. I use XPath&#8217;s on a daily basis and use it to return elements from UI visual trees (you can find more about this from the [presentation I did for Digital Lincoln](https://www.crowdcast.io/e/automated-ui-testing/)) but I&#8217;ve often found it hard to test expressions as they are being written and thought a xpath validator would be really useful. To address this I started by writing a basic console application which would take an XPath as an input and check if it was valid &#8211; this worked well but I thought a website would be easier so I built [XPathValidator.dev](https://XPathValidator.dev). I am aware there are a few other XPath testing sites out there but I wanted something simple and clean that would just get the job done.

There are two ways the site can be used for; the first is to just validate XPath expressions and the second is to run those expressions and return the result by also providing XML. Let&#8217;s quickly try out both these features.

## XPath Validation

This is the main feature I built the site for but I image most people will be more interested in evaluating their XPath expressions however I&#8217;ll quickly go through it. Simply provide an expression and hit validate. If you&#8217;re XPath if syntaxly correct you will get a nice green tick.

<pre class="wp-block-code"><code>//*</code></pre>

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="539" height="351" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image.png" alt="" class="wp-image-614" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image.png 539w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-300x195.png 300w" sizes="(max-width: 539px) 100vw, 539px" /></figure>
</div>

And if not you&#8217;ll get a cross with a error message.

<pre class="wp-block-code"><code>///</code></pre>

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="524" height="345" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image-1.png" alt="" class="wp-image-615" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image-1.png 524w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-1-300x198.png 300w" sizes="(max-width: 524px) 100vw, 524px" /></figure>
</div>

## Expression Evaluation

This is what I image most people will use the site for so let&#8217;s see what it can do! Let&#8217;s start basic and return a node. I&#8217;m going to using the [XML Example Document provided by W3Schools](https://www.w3schools.com/xml/xpath_examples.asp).

<pre class="wp-block-code"><code>/bookstore/book&#91;1]/title</code></pre>

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="527" height="346" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image-2.png" alt="" class="wp-image-617" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image-2.png 527w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-2-300x197.png 300w" sizes="(max-width: 527px) 100vw, 527px" /></figure>
</div>

Great! How about selecting all the book prices?

<pre class="wp-block-code"><code>/bookstore/book/price&#91;text()]</code></pre>

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="534" height="418" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image-3.png" alt="" class="wp-image-618" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image-3.png 534w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-3-300x235.png 300w" sizes="(max-width: 534px) 100vw, 534px" /></figure>
</div>

Or even get the title nodes where their price > 35.

<pre class="wp-block-code"><code>/bookstore/book&#91;price>35]/title</code></pre>

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="533" height="369" src="http://tomaustin.xyz/wp-content/uploads/2020/04/image-4.png" alt="" class="wp-image-619" srcset="https://tomaustin.xyz/wp-content/uploads/2020/04/image-4.png 533w, https://tomaustin.xyz/wp-content/uploads/2020/04/image-4-300x208.png 300w" sizes="(max-width: 533px) 100vw, 533px" /></figure>
</div>

There are a lot more XPath operators and functions out there and I suggest you check out the documentation over on [W3Schools](https://www.w3schools.com/xml/xpath_intro.asp) if you&#8217;re interested in learning more. The site was built in .Net Core 3.1 and is [open source on GitHub](https://github.com/tomaustin700/XPathValidator), I&#8217;m happy to accept pull requests with additional functionality and if you have any problems then just [open an issue](https://github.com/tomaustin700/XPathValidator/issues/new). Enjoy!