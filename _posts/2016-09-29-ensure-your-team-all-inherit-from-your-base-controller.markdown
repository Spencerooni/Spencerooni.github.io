---
layout: post
title:  "Ensure your team all use the same Controller base class"
date:   2016-09-29 15:00:00
categories:
tags:
- c#
- .NET
- asp.net-mvc
excerpt: Write a simple unit test to enforce that everyone inherits from your custom base Controller
---

I've worked on a few asp.net MVC projects where someone has created a class which inherits from <code>System.Web.Mvc.Controller</code>, added some helpful code which all controllers can use and hoping that others will make use of them. Unfortunately, this doesn't always happen and that can be sad, particularly if you have overwritten methods in the base class which take part in the application lifecycle. Luckily there's a very easy way to make people aware of it. Write a unit test which ensures that exactly 1 class inherits from <code>System.Web.Mvc.Controller</code> (your custom base controller).

{% highlight c# %}

namespace MyProject.Tests.Controllers
{
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Reflection;
    using System.Web.Mvc;
    using MyProject.Controllers;
    using NUnit.Framework;
    
    [TestFixture]
    public class BaseControllerTests
    {
        [Test]
        public void AssertThat_OnlyCustomBaseController_DirectlyInheritsFrom_Controller()
        {
            // Get all the class types whose base type is System.Web.MVC.Controller
            Type systemWebMvcControllerType = typeof(Controller);
            Assembly webAssembly = Assembly.GetAssembly(typeof(CustomBaseController));
            Type[] types = webAssembly.GetTypes().Where(type => type.BaseType == systemWebMvcControllerType).ToArray();

            // Assert
            Assert.AreEqual(1, types.Length);
            Assert.AreEqual("CustomBaseController", types[0].Name);
        }
    }
}

{% endhighlight c# %}

