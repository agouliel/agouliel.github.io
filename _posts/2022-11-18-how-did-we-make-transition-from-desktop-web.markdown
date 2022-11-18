---
layout: post
title:  "How did we make the transition from desktop to web"
date:   2022-11-18 15:21:59 +0200
categories: sharepoint
---
At Ionia Management, we try to be cutting-edge with our IT stack. This isn’t always straightforward in the shipping sector. So how did we go about it?

The first step was making our office web-based. Ionia has always been a Microsoft shop, so the choice was obvious: move everything to SharePoint.

We use SharePoint for file storage (OneDrive), online collaboration (Office 365), approvals, workflows, and simple data storage (lists and document libraries). But most importantly, we use SharePoint as a placeholder and starting point for all our custom software. This presupposes that all this software is web-based.

So how did we achieve that? Well, for reporting needs we already had Power BI (see this excellent [post] by a former colleague). This means that reporting was almost ready. Our SQL Server databases are still on premises (we are not so bold yet to move them to the cloud), so we employed the Gateway mechanism to retrieve necessary data for our online Power BI reports. We also doubled down on our hiring of Power BI developers, using mostly UpWork to reach worldwide talent.

For simple applications, we use either Microsoft’s own solutions (for example, Planner) or we build our own Power Apps for custom needs. Power Apps are low-code, and they integrate seamlessly with SharePoint lists for data storage.

And now getting to the meat of things. We had a series of legacy desktop apps that needed to be ported to web. Not easy!

The first step was to move our codebase to the cloud. The solution is obvious: GitHub. We created an [organisational account][organisational-account], plus various repositories for our projects, some private and others public. We educated and pushed in-house and external developers to use git.

Second step was to choose our new development stack. Web apps need a backend and a frontend, so we had to make a careful choice for both.

For the former, the most obvious choice was .NET which would make the porting of our current desktop apps easier. And indeed we did that successfully. But we didn’t stay there. Having used Python for various scripting needs across our systems, we also decided to give Django a try. It has been [said] that Django is like a superpower for solo developers or small teams, and this is true. It enables you to build a backend easily, gives you the ability to offer an API quickly if needed, and even provides a professional GUI out of the box, in the form of the Admin Panel.

For the frontend, we initially went with Razor Pages thinking that we will enjoy easier integration with our .NET backends. While this is true, we later found that the React framework is more powerful and much more popular with developers. So we are a React shop now and our apps look great!

The results of all the above have been very nice. When our users log into their workstations every morning, they are presented with our SharePoint intranet page and they have everything at their fingertips: files, data, reports and applications.

Things haven’t been easy and there is still much to do. But we are always learning and evolving, and we fully intend to advance even further. Oh, and if any of the above sounds interesting, we are always hiring!

[post]: https://www.linkedin.com/pulse/how-bi-tools-helped-us-reduce-forwarding-costs-nicolas-otheitis/
[organisational-account]: https://github.com/ioniaman
[said]: https://panelbear.com/blog/tech-stack/