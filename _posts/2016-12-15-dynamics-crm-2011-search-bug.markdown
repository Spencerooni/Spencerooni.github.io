---
layout: post
title:  "The 'David' bug"
date:   2016-12-15 13:00:00
categories:
tags:
- Dynamics CRM 2011
- bug
excerpt: Ridiculous bug
---


Back in September 2011, my first project at Kainos was customising a Dynamics CRM solution for a customer. It wasn't the most exciting. Not much code. Quite a lot of manual testing. But to be honest, to get into the swing of things in the working world, it wasn't too bad.

So, the project is in user acceptance testing and a few bugs are being raised. Then we got this one:


<img src="/images/dynamics-crm-2011-bug-description.png" alt="dynamics crm 2011 bug description"/>

<small>"Client Overviews" is just their list of client records</small>

Right... that's a bit weird. Sounds ridiculous. Probably something silly. User error. In saying that, I didn't have much reason to doubt them. They'd been pretty bang on with any issues raised up until then.

So I opened up one of our instances of Dynamics CRM to test and navigated to the Client Overview list, typed "David", hit enter, waited for spinner to disappear... and voila! Results! Of course it works! Look, everyone is called Dav... Wait a second... Alison, Andrew, Ben, Bernadette, Claire, David... I cleared the search box (so no search criteria) and hit enter. Same result. Everyone. Page 1 of loads. Not filtered. Typed "david" again, lower case this time. Same result. Everyone. Page 1 of loads. What if I click the search button instead of hitting enter, maybe it's calling something different? Nope, same result.

So, the obvious questions to ask.

Does search work with other names?

Tried <b>"Ben"</b>.

Yep, works.

Lowercase, <b>"ben"</b>.

Yep.

What about <b>"be"</b>.

Yep, still get Ben in my filtered list, plus Bernadette.

What about <b>"bbbbb"</b>.

No results, looks good.

Tried another name just to be sure, <b>"Claire"</b>.

All good.

My full name? <b>"David Spence"</b>?

Yep, all good too.

Literally, the only word I tried which didn't filter anything... was the word "David". Where do you even start with that? My team started to blame me! They were half joking, but they still thought I'd done something. The "David" bug. I've obviously hard coded my name in the code somewhere. But this is out of the box searching! What the hell have I done? <code>CTRL + F</code> in the entire source control folder, not a mention of my name.

I think I spent the next day staring at a screen in disbelief. Maybe if I hit enter slightly faster...

<blockquote>"David, have you made any progress on that David bug yet?"</blockquote>

Nope... What the heck could I check? Type more random names and words until another one magically doesn't work and then try and find what's common between the two?

I spun up a brand new environment, without any of our customisations. Literally a blank canvas. None of our custom C# or JavaScript. I added some names for testing and went to the Client Overview list. I wasn't sure if I wanted the search to work or not. If it still didn't work here, it can't be my fault. This is base functionality. It'd be Microsoft's fault. But then I can't fix it... And if it does work, well... then it's back to blaming me.

Ok so back to "David". What about just <b>"D"</b>?

Success! Filtered correctly. What about <b>"Da"</b>?

Success again! We've lost Derek, but still got Davids and Daniels. <b>"Dav"</b>?

Yep fine. <b>"Davi"</b>? Yep grand. Now the real test... <b>"David"</b>...

All results. Page 1 of loads.

Seriously? The EXACT word David and no other combination? Tried <b>"David Spence"</b> again. Grand. Exact match. 1 result. Arse.

<blockquote>"David, have you made any progress on that David bug yet?"</blockquote>

NO! Can we just tell them to search for Davids with "Davi"? Cause that works! Anyway, it took another two days of digging an infinite hole to randomly type another word which returned all results! The mythical word <b>"One"</b>! Don't tell me how I thought of such a magical word, but I managed it.

<blockquote>"Richard, was there ever someone in Kainos called 'One' and did they work on this project..? Nope, ok. Damn."</blockquote>

What do "david" and "one" have in common? Let's go back to "David" a second because there was something I didn't try.

<b>"avid"</b>

All results. Page 1 of loads.

Aha! So it's not David that it's broken for. Turns out just typing the last two words of my name "id" returned everyone. In fact, turns out typing "id" with any letters before, after on inbetween other letters returned every record. So "xid" and "idx" and "xidx" returned everyone. So "id" is a programmer-ish thing, right? Maybe a developer on the Dynamics CRM team has made a mistake in the query which gets run in the very specific situation when the search term begins or ends with id... But that doesn't explain why the term "one" returns everyone. "id" != "one".

Any ideas yet?

Well, every entity in Dynamics has a set of build in fields as standard. An account (or with our terminology, "Client Overview") comes with standard fields like:

<ul>
<li>Id</li>
<li>Address Line 1</li>
<li>Address Line 2</li>
<li>Telephone</li>
<li>Mobile</li>
</ul>

We'd customised it and added a few of our own, too. But even looking at these, do you notice anything? At this stage I was thinking VERY abstract. Stupidly abstract. Trying to make anything fit. This is the only notion in the system I could think of where the term "id" has any mention in relation with accounts. I felt like when I was on placement, taking snapshot images of remote machines (playing around in those scary terminal menus before your OS boots up) and every time I touched the letter G of the sequence of instructions I had to type, the machine restarted. My manager thought I was nuts. "I swear it's the letter G! It's doing something!". Turned out that was a timing thing and I was so consistent that about five times in a row, I'd typed the letter "G" after the same exact number of seconds, and the machine restarted. This was one of those moments. But...

See how <b>"Telephone"</b> ends in with the word <b>"one"</b>? That was one my magic "I found another word that returns everything" words.

Navigate to people, search term:

<b>"Telephone"</b>

All results. Page 1 of loads.

YES!!

Let's confirm, another field is for mobile.

<b>"xxxMobile"</b>

All results. Page 1 of loads.

Winner. Report to Microsoft! No idea what weird check or error was in their code, but it wasn't my fault.

Here was my comment in the bug, 5 years ago!

<img src="/images/dynamics-crm-2011-bug-report.png" alt="dynamics crm 2011 bug report"/>




