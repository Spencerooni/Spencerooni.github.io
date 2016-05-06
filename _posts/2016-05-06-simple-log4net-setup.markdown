---
layout: post
title:  "Simple log4net setup in a console application"
date:   2016-05-06 19:00:00
categories:
tags:
- log4net
- c#
- .NET
excerpt: Setup colourful logging in your console application using log4net
---

There's not much to this post, other than a very minimal setup for a console application using log4net for logging, with some nice basics, such as:

- red highlighting of errors
- yellow highlighting of warnings
- logs are written to both console and file

First, get <a href="https://www.nuget.org/packages/log4net/">log4net from Nuget</a> and add the following line to <code>AssemblyInfo.cs</code>.

{% highlight c# %}
[assembly: log4net.Config.XmlConfigurator(Watch = true)]
{% endhighlight c# %}

Here's a sample <code>App.config</code>.

{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
  </configSections>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.6" />
  </startup>
  <log4net>
    <appender name="ConsoleAppender" type="log4net.Appender.ColoredConsoleAppender">
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%date %level [%thread] %logger{1} %username - %message%newline" />
      </layout>
      <mapping>
        <level value="WARN" />
        <foreColor value="Yellow, HighIntensity" />
      </mapping>
      <mapping>
        <level value="ERROR" />
        <foreColor value="Red, HighIntensity" />
      </mapping>
    </appender>
    <appender name="RollingFile" type="log4net.Appender.RollingFileAppender">
      <file value="./logs/log.log" />
      <rollingStyle value="Date" />
      <appendToFile value="true" />
      <lockingModel type="log4net.Appender.FileAppender+MinimalLock" />
      <datePattern value="yyyyMMdd" />
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%date %level [%thread] %logger{1} - %message%newline" />
      </layout>
    </appender>
    <root>
      <level value="INFO" />
      <appender-ref ref="ConsoleAppender" />
      <appender-ref ref="RollingFile" />
    </root>
  </log4net>
</configuration>

{% endhighlight xml %}


And finally a simple <code>Program.cs</code>!

{% highlight c# %}
using System;
using System.Security.Principal;
using log4net;

namespace MyApp
{
    internal class Program
    {
        private static readonly ILog Logger = LogManager.GetLogger(System.Reflection.MethodBase.GetCurrentMethod().DeclaringType);

        private static void Main(string[] args)
        {
            Logger.InfoFormat("Running as {0}", WindowsIdentity.GetCurrent().Name);
        }
    }
}
{% endhighlight c# %}