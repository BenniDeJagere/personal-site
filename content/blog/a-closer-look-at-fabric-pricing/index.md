---
title: "A closer look at Microsoft Fabric pricing, billing, and autoscaling"
date: 2023-07-21T12:17:06+02:00
draft: true
images: ["/2023/a-closer-look-at-microsoft-fabric-pricing-billing-and-autoscaling/feature.png"]
tags: ["Microsoft Fabric", "OneLake", "smoothing", "Power BI", "billing"]
---

If you're considering using Microsoft Fabric, you're probably thinking "How much is this going to cost me?" Continue reading to learn how Microsoft might have just created the most compelling data platform offering available today.

<!--more-->

It's still a bit early to get a complete and thorough overview of how Microsoft is planning on billing for Fabric. But if we look at some public statements and documentation, we can piece it all together to get a pretty good idea of what to expect.

## Fabric is free?

That is, for now. If you're enabling your Microsoft Fabric trial, you won't be billed for it. This is a temporary measure while Fabric is still in preview. This trial allows you to try out all features of Fabric and get a good idea of the performance you'd get and how this would consume the allocated capacity. I highly recommend enabling the trial and trying out some workloads to determine the capacity you'll need.

## Capacity?

I knew I wasn't going to be able to write a few paragraphs without coming to the keyword **capacity**. Fabric is all about capacities. Let's compare this to other cloud services:

| Service                       | Model     | Billing factors                  |
| ----------------------------- | --------- | -------------------------------- |
| Azure Storage                 | IaaS      | storage, bandwidth, transactions |
| Azure VM                      | IaaS      | CPU, memory, disk                |
| Azure SQL                     | PaaS      | vCore, storage                   |
| Azure App Service             | PaaS      | vCore, storage, features         |
| Databricks                    | PaaS-SaaS | CPU & memory + DBU               |
| Snowflake                     | SaaS      | Snowflake credit                 |
| Microsoft 365                 | SaaS      | user                             |
| Power BI Premium Per User     | SaaS      | user                             |
| Power BI Premium Per Capacity | SaaS      | capacity / vCPU                  |
| Microsoft Fabric              | SaaS      | capacity / CU                    |

The cloud is all about the Shared Responsibility Model and you can see, the more responsibility goes to the cloud provider (so the more you're moving towards SaaS), the more you're not thinking anymore about the hardware underneath. And this also translates to the billing model.

## How a capacity is allocated

Microsoft Fabric shares the same platform as Power BI and billing also seems to go in that direction. In Power BI you can activate Power BI Premium for a user to give them access to more features. You can also buy a Power BI Premium capacity which translates to a certain amount of vCores and maximum memory. This capacity can then be shared by multiple users and workspaces. Fabric Capacities work in the same way. They are meant to be **shared across projects, users, and workloads**. It doesn't matter if one user is using the Lakehouse, another user is running notebooks, and a third one is executing SQL in the Warehouse. They can all share the same capacity.

{{< figure src="fabric-capacity-sharing.png" caption="Fabric Capacities can be shared amongst multiple users, projects, and workloads across an organization*" >}}

Capacities are assigned to one or more Workspaces. Everything inside that workspace, be it a Warehouse, Lakehouse, Spark job, or notebook, will be using that same capacity.

**Attentive readers will point out that you can create Elastic Pools for Azure SQL to share amongst multiple databases. I know, this was for demonstration purposes only.* :wink:

## SKUs

Since Fabric builds upon the Power BI platform, you can just reuse the Power BI Premium SKUs. Microsoft also announced F-SKUs which can be paid for using Microsoft Azure. The SKUs each give you a certain amount of Capacity Units (CU). Prices below are from the Azure region West US 2.

[Source 1](https://learn.microsoft.com/en-us/power-bi/enterprise/service-premium-what-is) | [Source 2](https://blog.fabric.microsoft.com/en-us/blog/announcing-microsoft-fabric-capacities-are-available-for-purchase) | [Source 3](https://learn.microsoft.com/en-us/fabric/enterprise/buy-subscription)

| SKU name | Fabric CUs | Power BI Premium vCores | Power BI Premium Memory (GB) | Price (USD/month) |
| -------- | ---------- | ----------------------- | ---------------------------- | ----------------- |
| F2       | 2          | N/A                     | N/A                          | 262.80            |
| F4       | 4          | N/A                     | N/A                          | 525.60            |
| F8       | 8          | N/A                     | N/A                          | 1051.20           |
| F16      | 16         | N/A                     | N/A                          | 2102.40           |
| F32      | 32         | N/A                     | N/A                          | 4204.80           |
| P1       | 64         | 8                       | 25                           | ± 5000            |
| F64      | 64         | N/A                     | N/A                          | 8409.60           |
| F128     | 128        | N/A                     | N/A                          | 16819.20          |
| P2       | 256        | 16                      | 50                           | ± 10000           |
| F256     | 256        | N/A                     | N/A                          | 33638.40          |
| P3       | 512        | 32                      | 100                          | ± 20000           |
| F512     | 512        | N/A                     | N/A                          | 67276.80          |
| P4       | 1024       | 64                      | 200                          | ± 40000           |
| F1024    | 1024       | N/A                     | N/A                          | 134553.60         |
| P5       | 2048       | 128                     | 400                          | ± 80000           |
| F2048    | 2048       | N/A                     | N/A                          | 269107.20         |

Why are the F-SKUs so much more expensive? Simple! For the P-SKUs, you have to commit to yearly billing. For the F-SKUs, you can pay monthly. It is already confirmed that in a few months, you'll also be able to commit to yearly billing for the F-SKUs and get a discount. This is just the same principle as with the yearly or 3-yearly reservations you can buy right now for services like Azure Synapse.

The F-SKUs are priced as Pay-As-You-Go so you only pay for exactly what you consume. Continue reading to learn how you can pay even less than the prices above.

## Capacity Units & monitoring your usage

So, these capacity units, what are they? This is the billing unit used in Microsoft Fabric. You can compare it to Snowflake credits or Databricks DBUs. It's a unit that combines CPU and memory. Everything you do in Fabric consumes CUs. The more CUs you have, the more you can do without being throttled.

As Fabric is all about data analytics and reporting, you can also report on your own CU consumption. Microsoft released a [Fabric Capacity Metrics app](https://appsource.microsoft.com/en-us/product/power-bi/pbi_pcmm.microsoftpremiumfabricpreviewreport?exp=ubp8) that you can install in your tenant. This app will give you an overview of the capacity usage of your Fabric capacities.

{{< figure src="https://store-images.s-microsoft.com/image/apps.28804.01aa8216-bdd9-47d7-a910-97ffae8e2f63.fe89af37-9f72-43fb-acd7-b36d04f3cc06.662ccdcc-b621-4db1-9868-4c76d37a4135" caption="Fabric Capacity Metrics app" >}}

## Smoothing & throttling: the magic of Power BI Premium coming to Fabric

So does that mean that if you have an F2 SKU with 2 CUs, your workloads can use the power of 2 CUs on a constant load for an entire month? Yes. But, also, more. Since Microsoft Fabric is built upon Power BI, users can benefit from a principle called *smoothing*. This is not new and [documentation](https://learn.microsoft.com/en-us/power-bi/enterprise/service-premium-smoothing) already exists. [This page](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app-timepoint-page) in the Microsoft Fabric documentation also mentions the smoothing feature, so we can assume that this is also available in Fabric.

Let's compare the 2 graphs taken from the documentation below:

{{< figure src="https://learn.microsoft.com/en-us/power-bi/enterprise/media/service-premium-smoothing/cpu-usage-no-smoothing.png" caption="Without smoothing" >}}

{{< figure src="https://learn.microsoft.com/en-us/power-bi/enterprise/media/service-premium-smoothing/cpu-usage-with-smoothing.png" caption="With smoothing" >}}

When smoothing is not applied, you can see that the CPU usage is very spiky. It also tends to go over the maximum capacity. This means that workloads start to get throttled or might even fail. When smoothing is applied, the CPU usage is much more constant and the maximum capacity is never exceeded. This means that you can run more workloads in the same capacity.

So what kind of sorcery is this? The total amount of CUs every workload consumes is still the same. The difference is that it is spread out over time. We can also see a difference between interactive workloads and background workloads (e.g. a scheduled job). Let's see what the documentation says about this:

>Interactive operations average your capacity usage over a short time frame, such as five minute intervals. Background operations on the other hand, average your capacity usage over a much larger 24 hour time frame. The benefit of this method is that operations that require many resources, such as refreshes, get smoothed because they're averaged over a long period of time.

>During each timepoint, Power BI adds up the average CPU usage from both the interactive and background operations. If the CPU usage for a specific timepoint exceeds the SKU limit, autoscale kicks in if enabled. If autoscale isn't enabled, or if the CPU usage is higher than what autoscale can handle, throttling is applied.

Looks great, right? A lot of data engineers used to run their intensive workloads during off-time to avoid interfering with the interactive workloads during the day. With smoothing applied, those background operations would be spread out over the next 24 hours. It is to say, only their consumption of CUs would be spread out. The operations would still complete within the same timeframe as without smoothing.

{{< figure src="smoothing.png" caption="Smoothing applied on a background workload" >}}

So even with a smaller SKU like F2, you can get amazing performance because short-lived workloads can use much more CUs than the 2 CUs that the SKU provides. You are billed based on the average performance you need instead of the maximum/peak performance you need.

When I read the documentation on smoothing, it all clicked for me. During the [keynotes at Microsoft Build](https://build.microsoft.com/en-US/sessions/852ccf38-b07d-4ddc-a9fe-2e57bdaeb613?source=sessions) and [conference talks]({{< relref "speaking/data-platform-next-step-dbt-fabric.md" >}}) Microsoft employees often mentioned that processing engines in Fabric can allocate more capacity whenever they need within fractions of seconds. This could only be possible with features like smoothing on the billing side.

## Autoscaling & manual scaling

Today, the P-SKUs already offer autoscaling. This was also [announced](https://blog.fabric.microsoft.com/en-us/blog/announcing-microsoft-fabric-capacities-are-available-for-purchase) to be coming to F-SKUs. So if you notice that you still get throttled, even with the smoothing, you can enable autoscaling.

{{< figure src="autoscale.png" caption="Autoscaling configuration" >}}

Once your CU consumption reaches the maximum, more capacity can automatically be provisioned through a Microsoft Azure subscription. The extra provisioned CUs automatically go away again once your workloads consume less than what your SKU provides. This is a great way to avoid throttling and to avoid paying for more capacity than you need.

If you're on an F-SKU today you can apply manual scaling. F-SKUs are billed Pay-As-You-Go and managed as an Azure resource. So if you provision an F2 for just 1 hour, you only 36 cents. At any point in time, you can pause, resume, or scale up/down your F-SKU. Are you planning an intensive workload that might throttle your capacity? Just temporarily bump your capacity and scale it down again when you're done.

## How to pay even less

The F-SKUs purchased through Microsoft Azure can be paused when you don't need them. If your company isn't doing anything with data or analytics during the weekend, you can just pause the capacity and not save ± 25%. I can't confirm yet how this would work with the smoothing, but it's only reasonable that pausing also means that the smoothing is paused (e.g. if your workload is spread out over 24h and you pause after 12h, the remaining 12h will be consumed when you resume again).

## Why the region matters

When you read up on Fabric pricing, you'll notice a lot of different numbers going around. Microsoft Fabric pricing is determined by the region you choose. This is exactly the same principle as in Microsoft Azure. Experienced users tend to look at the East US 2 region quite often as it often tends to have lower prices compared to others. Could you pick whatever region you want?

**The region you choose determines where your data will reside.** We haven't talked about OneLake yet, all pricing above was related to processing power.

## Pricing for storage and bandwidth

Microsoft's announcements about Fabric link to the [regular bandwidth pricing page](https://azure.microsoft.com/en-us/pricing/details/bandwidth/#pricing). This was already a cheap offering, so it's great to see this extended to Microsoft Fabric as well.

OneLake storage pricing follows the same pricing as Azure Data Lake Storage gen2. The price used seems to be the one for Hot ZRS storage. E.g. $0.023 per GB in US West 2.

## Which SKU should you pick?

Today: start with the trial. Why would you pay, if you can get it for free, amirite? :laughing: Jokes aside, when your trial is over, I'd recommend starting with the F2 SKU and scaling up as needed. Once you reach the F32 SKU, you could switch over to the P1 SKU and enable autoscaling. The SKUs will probably change in the future as some changes are already announced (e.g. Reserved Instances and autoscaling for F-SKUs).

## Conclusion

The following features make Microsoft Fabric a really compelling offer:

* **Smoothing**: you only pay for the average performance instead of the peak performance
* **Autoscaling**: if you get throttled a couple of times per month, you can automatically provision more capacity to avoid paying for more expensive SKUs
* **Manual scaling**: if you know in advance you'll temporarily need more performance, manually provision more capacity
* **Pay-As-You-Go**: you can dynamically pause and resume your capacity to save some costs
* **Shared capacity**: you can share your capacity across multiple users, projects, and workloads
* **Free trial**: you can try out all features of Fabric for free
* **Incredible performance for a low price**: early users have reported impressive performance on the F2 SKU, which is only $0.36/hour

These unique set of billing features might cause a shift in how data teams use a data platform. Why would you wait for off-time during the night if thanks to smoothing & autoscaling you can continuously run intensive data processing jobs during the day?

I've personally never seen such extensive and impressive billing features in a data platform. It's really great to see that Microsoft is taking this seriously and is trying to make it as easy as possible for customers to get the most out of their investment. We finally have a true data platform built (and billed) for the cloud era.
