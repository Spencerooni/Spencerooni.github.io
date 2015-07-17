---
layout: post
title:  "Visualizing GPS co-ordinates and elevation in 3D using JavaScript"
date:   2015-07-16 15:30:00
categories:
tags:
- javascript
- vis.js
- highcharts.js
- 3D
- graph
- strava
excerpt: Investigating options for visualizing GPS co-ordinates and elevation in interactive 3D charts using JavaScript.
---

I recently saw a feature on <a href="http://blog.veloviewer.com/interactive3d-elevation-profiles" target="_blank">Veloviewer</a>, which I thought was really nice and wanted to see how difficult it would be to replicate. The feature being _interactive 3D elevation profiles_. Instead of displaying a basic graph of elevation over time, they display the elevation at each GPS co-ordinate in a lovely 3D graph.

Here's what they have:

<iframe class="segEmbed" style="width:100%;height:350px;" src="http://veloviewer.com/segment/4397805/embed?" frameborder="0" scrolling="no"></iframe>

<br>
You can play with a few more of them live on the Veloviewer website:

- <a href="http://veloviewer.com/climbs" target="_blank">http://veloviewer.com/climbs</a>

It's a nice way to visualize the ups, downs and direction changes of your activity, especially given that it's interactive, so you can twist, drag, rotate, zoom, etc. It's a nice feature.

Why am I interested? Well, I do a bit of cycling and I like my technology. These interests came together and I created <a href="https://www.microsoft.com/en-gb/store/apps/straza-mate/9nblggh0cn1r" target="_blank">a Windows Phone app called Straza Mate</a> which integrates with <a href="https://www.strava.com">Strava</a> (Strava being one of the most popular sites for athletes to upload and share their activities). So I'm interested in what I can do with my data (using <a href="http://strava.github.io/api/" target="_blank">Strava's extremely good API</a>) and nice ways to relay information.

If you want to visualize a ride on Strava like I'm trying to do, you can retrieve the raw activity stream via their <a href="http://strava.github.io/api/v3/streams/" target="_blank">Stream API</a> (you'll need to have a Strava account and create an application at [strava.com/developers](http://strava.com/developers) to obtain an <code>access_token</code>).

<pre>
<code>
https://www.strava.com/api/v3/activities/{activity id}/streams/latlng,altitude?resolution=medium&access_token={access token}
</code>
</pre>

The same can be done for <a href="http://strava.github.io/api/v3/streams/#effort" target="_blank">efforts</a> and <a href="http://strava.github.io/api/v3/streams/#segment" target="_blank">segments</a>.

<h2>SVG-VML-3D</h2>
<a href="http://www.lutanho.net/svgvml3d">http://www.lutanho.net/svgvml3d</a>

Judging by their JavaScript imports, this looks to be the library of choice for Veloviewer. I played with it for a few hours and found it tricky. There <a href="http://www.lutanho.net/svgvml3d/svgvml3d_doc.html" target="_blank">isn't much documentation</a> (it's not much more than a list of method names), the examples are quite complex and I couldn't find many articles about it. Apart from localizations, there doesn't seem to have been any update to the library since 2006! However, it's obviously very powerful, so I'll probably come back and have another go.


<h2>Highcharts</h2>

<a href="http://www.highcharts.com/demo/3d-scatter-draggable">http://www.highcharts.com/demo/3d-scatter-draggable</a>

After much googling and skimming over many "50 best JavaScript graph plugins" websites, I experimented with the Highcharts 3D scattergraph (which seemed to pop up a lot). Unfortunately, they don't have an equivalent 3D bar chart, so it's just a bunch of dots. It's effective though. I quite liked it at first, but when you start rotating and playing with the graph, it becomes hard to read as some points look as if they cross over when they're in the same line of view. There's probably not much which Highcharts can do about it, I think the scattergraph simply doesn't lend itself well to the usage I have. Unless there is an opacity setting which I haven't found... There is however a nice option to reverse points on an axis which I needed to do, given the latitude and longitude don't start from the same corner.

{% highlight javascript %}
xAxis: {
    reversed: true // thank you!
},
{% endhighlight javascript %}


Other than that, I've pretty much taken the <a href="http://jsfiddle.net/gh/get/jquery/1.9.1/highslide-software/highcharts.com/tree/master/samples/highcharts/demo/3d-scatter-draggable/" target="_blank">base example</a>, replaced the data with my data points and amended some of the basic configuration like height, margin, depth, line width, etc. It's very simple and documentation is good. There is also a little hamburger menu in the top right where you have the option to download images of the current view, which is nice. Zooming isn't the best. Zooming is disabled below, but when enabled you have to select a square area and the points within your selection are brought closer.

If you check out the JavaScript in the example and think "woah!" don't be put off! 90% of the code is the raw data points from Strava for an activity! Skip past all of the numbers and you'll see the basic initialization of the chart.

<b>Note</b>: You can only have 1000 data points for Highcharts 3D scatter graphs

<iframe width="100%" height="400" src="//jsfiddle.net/spencerooni/wgqemr8v/11/embedded/result,js,resources,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

<br>

<h2>Vis.js</h2>
<a href="http://visjs.org">http://visjs.org</a>

This library is fantastic. To get this result was incredibly simple. The documentation is good, it's easy to customize and it's free (Vis.js is dual licensed under both Apache 2.0 and MIT). However, unlike Highcharts, I don't see a way to reverse an axis, so I might have to dig into their code to get it 100% correct (currently, left turns on the graph are actually right turns and vice versa). This 3D chart also zooms really nicely.

They have a <a href="http://visjs.org/examples/graph3d/playground/index.html" target="_blank">playground</a> where you can paste your data and change all the settings and see the result. I played about with it, got the settings to my liking (bar width, aspect ratio, etc), took their basic example and configured the settings as I had done in their playground.

<iframe width="100%" height="450" src="//jsfiddle.net/spencerooni/tgaLjjdq/2/embedded/result,js,resources,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

