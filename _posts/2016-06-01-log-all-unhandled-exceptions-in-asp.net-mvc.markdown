---
layout: post
title:  "Log all unhandled exceptions in asp.net MVC"
date:   2016-06-01 14:00:00
categories:
tags:
- log4net
- c#
- .NET
- asp.net-mvc
excerpt: Simple logging of unhandled exceptions in asp.net MVC
---

Create a new filter which implements IExceptionFilter and log however you'd like (I'm using log4net's Logger class).

{% highlight c# %}
public class Log4NetExceptionFilter : IExceptionFilter
{
    private static readonly ILog Logger = LogManager.GetLogger(System.Reflection.MethodBase.GetCurrentMethod().DeclaringType);

    public void OnException(ExceptionContext context)
    {
        Exception ex = context.Exception;

        Logger.Error(ex.Message);
    }
}
{% endhighlight c# %}

Register your filter globally in FilterConfig.cs.

{% highlight c# %}

public static class FilterConfig
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        filters.Add(new Log4NetExceptionFilter());
    }
}

{% endhighlight c# %}

That's literally it! Simples!