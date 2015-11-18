---
layout: post
title:  "Authenticating and authorizing users from Active Directory in ASP.NET MVC"
date:   2015-07-12 15:10:00
categories:
tags:
- asp.net
- asp.net-mvc
- active-directory
- web.config
excerpt: Learn how to leverage Active Directory for authorization and roles in your ASP.NET web application.
---

So you have your users and roles setup in Active Directory and you want to leverage them for authorization and roles in your ASP.NET web application? Look no further! 

Considering how easy this is, it took me a day to figure this out. There are so many configurations floating around the web when you google for "_active directory_" and "_asp.net_" it's easy to end up doing something super complicated.

The main magic is done by adding the following to <code>web.config</code> under the section <code>system.web</code>:

{% highlight xml %}

<authentication mode="Windows" />
<roleManager enabled="true" defaultProvider="AspNetWindowsTokenRoleProvider" />

{% endhighlight xml %}

As a basic level, it really is that simple. As long your on the domain, you can now authorize against users and roles from your Active Directory setup.

Going a step further, you'll probably want to check for roles. At a high level, you could ensure everyone accessing the application must be in particular role (Active Directory group). You may have 500 people setup in your Active Directory, but you only want to allow half of them to access the application. A solution for this would be to add all of your users to a group in Active Directory and restrict your web application at a high level.

{% highlight xml %}
<authorization>
  <allow roles="domainXXX\GroupXXX" />
  <deny users="*" />
</authorization>
{% endhighlight xml %}

But adding roles restrictions in <code>web.config</code> <b>has a few problems</b>. If you add role restrictions here and you want to whitelist a controller or controller action later on (my case was for a status page to ensure the app was running) then things get a bit tricky. This comes down to mixing the approach of authorizing roles in <code>web.config</code> and attribute based role checking.

What I wanted to do was allow anonymous access to a single controller.

{% highlight c# %}
[AllowAnonymous]
public class BuildStatusController : Controller
{
}
{% endhighlight c# %}

But adding this <code>[AllowAnonymous]</code> attribute didn't work. The role check in <code>web.config</code> seems to override the ability to allow anonymous access to some parts of the application. Basically, <b>an application wide role check in <code>web.config</code> mixed with allowing anonymous access via attributes doesn't work</b>. My solution was to remove all role checks from config and instead add the basic authorisation check in code when the app starts, so it would be applied application wide. In doing this, any usages of <code>[AllowAnonymous]</code> are honoured.

App_Start/FilterConfig.cs

{% highlight c# %}
public static class FilterConfig
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        if (filters == null)
        {
            throw new ArgumentNullException("filters");
        }

        filters.Add(new HandleErrorAttribute());

        var authorizeAttribute = new AuthorizeAttribute
        {
            Roles = "domainXXX\GroupXXX" // Role = group in Active Directory
        };

        filters.Add(authorizeAttribute);
    }
}

filters.Add(authorizeAttribute);
{% endhighlight c# %}

Now I am ensuring that all users of the application are in a particular group. At the same time, I am allowing anonymous access to some pages which don't require authorization.
