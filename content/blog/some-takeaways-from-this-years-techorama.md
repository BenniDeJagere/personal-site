---
title: "Some interesting takeaways from this year's Techorama"
date: 2022-06-09T16:37:50+02:00
tags: ["techorama", "azure", "kubernetes", "sql server"]
toc: false
---

> This post originally appeared on [the dataroots blog](https://dataroots.io/research/contributions/some-takeaways-from-this-years-techorama/).

![](https://dataroots.io/static/585458d77377bdef46719884dae51f1b/5803e/IMG_20220524_084801.webp)

Last week was a busy week for fans of the Microsoft technology stack like myself. Microsoft hosted its yearly developer conference, Microsoft Build, announcing [lots of exciting updates](https://aka.ms/build-2022-book-of-news) to new and existing Azure services. In the meantime, the Belgian community of Microsoft technology users gathered in Kinepolis Antwerp for this year's edition of Techorama.

First off, let me start by thanking the incredible [crew](https://techorama.be/team/) and [partners](https://techorama.be/partners/) to make Techorama happen. For known reasons, the conference couldn't take place over the last 2 years, at least not in a physical form. While it's great to see that a lot of companies and communities organised virtual meetups and conferences and we haven't stopped sharing knowledge, sitting behind a screen for 2 consecutive days doesn't give the same experience as an in-person conference. It felt heartwarming to see all the familiar faces in our local developer community again.

Now, the main reason to attend Techorama is not just the awesome community of developers and partners, the delicious food and drinks, but the interesting sessions. As Techorama is one of Belgium's biggest tech conferences, attendees are often faced with tough choices when it comes to building their schedule, since most of the time you have over 10 🤯 concurrent sessions. This was my personalised schedule:

-   *Pain, Grief, Perseverance and Technology (opening keynote)* by Derek Martin
-   *So, what about JSON in my database?* by Thomas Hütter
-   *Azure Synapse Analytics Serverless* by Nico Jacobs
-   *Adopting A DevOps Process for Your Database* by Steve Jones
-   *Take your network security to the next level on Azure PaaS* by Erwin Staal
-   *Offensive Azure Security* by Sergey Chubarov
-   *From 0 to 60 with Azure Kubernetes Service* by Jakob Ehn
-   *Database Worst Practices - Secrets of SQL Server* by Pinal Dave
-   *Azure SQL Database - A true story journey from migration, through synchronization and beyond* by Pieter Vanhove
-   *Kubernetes goes Serverless with Azure Container Apps* by Thorsten Hans
-   *Identify Poorly Performing Queries - Three Tools You Already Own* by\
    Grant Fritchey
-   *The Next Decade of Software Development (closing keynote)* by Richard Campbell

As you might have noticed, as a data & cloud engineer focused on Azure, my main interests lie in cloud best practices, databases and data platforms. Discussing every talk would lead this blogpost too far, so I picked out a few interesting takeaways from the list above.

The keynotes
------------

Attendees often tend to have mixed opinions about keynotes, but I must say I really liked them. They provided a different take on the life of a software professional and gave me an interesting perspective to what the future still holds for us. Both keynote speakers are really good speakers, so I'd really recommend to attend their sessions of they ever speak at a conference you're attending.

Security on Azure
-----------------

When thinking about security of cloud services, you usually have to consider (at least) 3 layers of security:

-   Networking: protecting a resource on the level of networking, making sure its firewall is properly configured and in some scenarios only allowing access from private networks
-   Authentication: providing users or systems with a reliable and properly managed way to authenticate to a service
-   Authorization: when a principal has correctly been authenticated, figuring out what the principal is allowed (not) to do inside the system

### Networking

![](https://dataroots.ghost.io/content/images/2022/06/Screenshot-2022-06-03-at-12.02.07.png)

better think twice before enabling this setting

You might have come across the checkbox above when configuring access Azure resources like a SQL Server. And when configuring network access, you might think the checkbox above is a good idea. Personally, I'd assume this would grant network access to this SQL Server from all other Azure resources I've created.

Unfortunately, that answer is incorrect. It grants access to the entirety of Azure. A malicious user, acting in their own tenant, creating a virtual machine meant to attack Azure users, could also get through. In general, it seems that this checkbox could use a clearer warning and I would now advise against checking it. Thanks to Sergey Chubarov for demonstrating this behaviour.

![](https://dataroots.ghost.io/content/images/2022/06/IMG_20220525_093437.jpg)

Sergey's mitigations for the attacks on cloud infrastructure that he demonstrated

### Authorization

In that same session, Sergey also shared how it's very important to secure any servers or virtual machines hosting the Azure AD Connect controller so that local administrators cannot access it. The server has to be secured in the same way as a domain controller, because underneath it would be able to use the same level of permissions.

The future for Kubernetes on Azure
----------------------------------

If you talk with any cloud engineer, you'll hear them say that Kubernetes is the next big thing in the cloud, if not already the current. But with great power, comes great responsibility. And often, the sheer flexibility you get with Kubernetes, also make it a very complex solution.

> Azure Container Apps is the serverless and managed version of Kubernetes we've all been waiting for

The refresher sessions to reiterate over all aspects of Kubernetes were really helpful, but what was even more promising was the demo of Azure Container Apps. It is the serverless and managed version of Kubernetes we've all been waiting for. The service was announced in preview last year but it has shown an incredible and speedy growth of new features. They might soon replace Azure Container Instances and Azure Web Apps for Containers. Like Azure Container Instances, the service is built on top of Kubernetes, but it only exposes a limited subset of its features, in a managed way. As beautifully demonstrated by Thorsten Hans, deploying a local Docker container to Azure Container Apps only takes 3 minutes. The demo gods have been good to him 😉

Going deeper into this specific service, would lead this blog post too far, but it is certainly something we'll tell you more about in a future blog post or session.

Azure SQL Databases: more than just a database
----------------------------------------------

Without any doubt my favourite session during the whole conference was the one given by Pieter Vanhove. He turned the session into a mini-workshop where attendees were divided into small groups and had to - literally - use pieces of a puzzle to build a solution for a concrete use case. In the use case, a company was planning on migrating their on-premise SQL Server to the cloud and we had to design all aspects of the migration journey.

![](https://dataroots.ghost.io/content/images/2022/06/IMG_20220525_144211.jpg)

the puzzle with the preferred migration design

Azure provides users with options which allows them to really finetune every aspect of the migration and configuration of a SQL database so that you can make it fit every company's needs.

JSON in SQL databases
---------------------

While Thomas Hütter's session showed some more of SQL Server's flexibility and that it can shine in dealing with JSON in a relational SQL database, I am still convinced that this is often a bad idea. The session showed some interesting features of SQL Server and it's worth it to check out [the documentation](https://docs.microsoft.com/en-us/sql/relational-databases/json/json-data-sql-server?view=sql-server-ver16&ref=dataroots.ghost.io) on this topic. What I missed in the session is why you would store JSON in relation databases in the first place. Often this comes down to a mismatch in the technical design of an application. As Thomas pointed out, more often JSON data is stored in NoSQL databases like Azure Cosmos DB.

Conclusion
----------

I could keep on rambling on how you've really missed out if you weren't there at Techorama, but the takeaways above really demonstrate that users of the Microsoft Azure cloud really have a promising future ahead and that there is always more to learn about it. See you in Kinepolis Antwerp in May 2023 for next year's edition of Techorama?
