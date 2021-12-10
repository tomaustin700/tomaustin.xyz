---
id: 66
title: Tracking cryptocurrency prices using Binance API
date: 2018-01-17T08:26:44+00:00
author: tom
layout: post
guid: http://3.8.252.104/?p=66
permalink: /2018/01/17/tracking-cryptocurrency-prices-using-binance-api/
categories:
  - api
  - asp.net
tags:
  - api
  - asp.net
  - binance
  - cryptocurrency
---
After recently investing in various cryptocurrencies through [Binance](http://www.binance.com) I found that their website didn&#8217;t show enough data regarding total profit/loss, they did however have an api which could be used to gather that data. The solution was to develop a simple (I am no asp.net developer!) site which would show current portfolio price and total profit/loss as an amount and as a percentage. You can see which endpoints Binance offer [here](https://www.binance.com/restapipub.html) . My approach was to find the total investment amount using the depositHistory enpoint, convert that to USD at the time of investment using the [cryptocompare api](https://www.cryptocompare.com/api/) (this api can give historical crypto prices so we can determine the cost at time of investment) and then find the price of my current investments by finding their ethereum price and converting that to usd. I am sure there is probably a more elegant way of doing this however I wanted something quick an easy. I called the cryptocompare api myself using HttpClient and passed in the symbol I wanted the price for and the time I wanted it (cryptocompare want the time as a timestamp).

<pre class="brush: csharp; title: ; notranslate" title="">HttpClient client = new HttpClient();
CryptoHistory usdPrice = null;
HttpResponseMessage usdResponse = await client.GetAsync("https://min-api.cryptocompare.com/data/pricehistorical?fsym=ETH&tsyms=USD&ts=" +
 deposit.InsertTime.ToString().Remove(deposit.InsertTime.ToString().Length - 3));
if (usdResponse.IsSuccessStatusCode)
{
...
...
...
}

</pre>

The Binance api was called using [this](https://github.com/morpheums/Binance.API.Csharp.Client) C# client made by [Jose Mejia](https://github.com/morpheums) and works perfectly so there was no point in calling the endpoints myself when someone else had wrapped it up. Anyway you can find the completed wesbite on my github [here](https://github.com/tomaustin700/Binance-Crypto-Tracker), I will warn you that is is very crude and I am no asp.net developer so it could all have probably been done a lot more elegantly using ajax and partialviews. Feel free to send a pull request with improvements. The finished site can be found [here](https://crypto.tomaustin.xyz/)