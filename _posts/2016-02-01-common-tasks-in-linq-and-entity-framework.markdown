---
layout: post
title:  "Common tasks in LINQ and entity framework"
date:   2016-02-01 09:00:00
categories:
tags:
- LINQ
- .NET
- entity-framework
- c#
excerpt: Simple examples of common LINQ and entity framework tasks which I kept having to Google, so this is my own personal reference!
---

<p>This is a personal reference for me but if it helps someone, all the better!</p>

<h2>Group items by multiple keys</h2>
e.g. Group activities by the year and month of their start date ("StartDate" is of type <code>DateTime</code>).

{% highlight c# %}
var activitesGroupedByMonthAndYear =
	from a in activities
	group a by new { Year = a.StartDate.Year, Month = a.StartDate.Month }
	into activitiesGrouped
	select new
	{
	    Key = new { Month = activitiesGrouped.Key.Month, Year = activitiesGrouped.Key.Year },
	    Activities = activitiesGrouped
	};
{% endhighlight c# %}

<h2>Case insensitive string contains search</h2>

Out of the box, the entity framework provider will do a case insensitive search when just using <code>Contains</code>. It will be converted to a SQL "like" where clause (<code>where value like %searchValue%</code>).

{% highlight c# %}
var result = people.Where(p => p.Name.Contains("Dave"));
{% endhighlight c# %}

The above would generate the following SQL:

{% highlight SQL %}
select * from people where Name like %Dave%
{% endhighlight c# %}
