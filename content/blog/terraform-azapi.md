---
title: "Filling the gaps in your code with the Terraform azapi provider for Azure"
date: 2022-07-04T16:28:43+02:00
tags: ["terraform", "azure", "bicep", "arm"]
---

> This post originally appeared on [the dataroots blog](https://dataroots.io/research/contributions/terraform-with-azure/).

![](https://dataroots.io/static/a4f1a6518431eb20f2403e36e59a7f57/48c06/terraform-azure.webp)

The cloud is just someone else's computer and to manage that we prefer to use Infrastructure as Code (IaC). dataroots believes that IaC can benefit any team working with cloud resources and most often [Terraform](https://www.terraform.io/) is [our tool of choice](https://registry.terraform.io/namespaces/datarootsio) there. As a data & cloud engineer focusing on Microsoft Azure, that is true for me as well. However, there have been a couple of hick-ups along the road.

We have to talk about the providers
-----------------------------------

Terraform works with a concept of providers. You can think of providers as plugins that add support for a certain platform to which you can deploy infrastructure (this is not entirely correct, but it holds up for 95% of the providers). Terraform and its providers are open source software, but the providers for the 2 most popular clouds - AWS and Microsoft Azure - are almost fully maintained by HashiCorp, the company behind Terraform.

Why is that not always a good thing? The provider for the Google Cloud Platform has seen a great amount of contributions from Googlers. The advantage there is that those Googlers have direct access to their coworkers building the cloud platform and can get dedicated time from Google to add support for new cloud components to the Terraform provider for GCP.

This is not the case for [the provider for Microsoft Azure](https://github.com/hashicorp/terraform-provider-azurerm). I've done some contributions myself, but they are nothing compared to what Tom Harvey, Kat Byte and their coworkers at HashiCorp are doing. The provider is written in Go and uses the [Azure SDK for Go](https://github.com/Azure/azure-sdk-for-go) to alleviate some of the work to talk with the Azure APIs. The provider is probably the biggest user of the SDK and luckily Microsoft chimes in there by being active in the maintenance of that SDK.

Today, Azure offers more than 200 cloud services and a lot of these services are in active development. It seems like new features on Azure are released about daily. Having to update the provider to support all of these new features is a never-ending and always-expanding job. That means that it can take a while before the new Azure service you dearly want to use, becomes available in a new release of the provider.

ARrrrrrrM
---------

![](https://dataroots.ghost.io/content/images/2022/06/image.png)

Azure users have been able to benefit from infrastructure as code for a while now. Azure Resource Management (ARM) templates allow users to code their infrastructure in JSON files and over the years it has become easier to write and maintain these templates. You can export your existing infrastructure into ARM templates and Visual Studio Code has some great extensions that make writing ARM templates easier.

The main advantage of ARM? It supports new services and features on Azure from day 1. When something new becomes available on Azure, ARM is the first place it is added to, often even before it becomes available in the Azure Portal.

So - I hear you asking - why wouldn't we stick with ARM for everything? As with everything in IT, it's not that simple. ARM templates are often convoluted and writing them can still give you headaches. Terraform uses a language called HCL (HashiCorp Configuration Language) which makes it a lot easier to split up everything in components and apply a [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) approach to your IaC code.

Next to the benefit of HCL, the concept of providers makes it easier to work with multiple aspects of a cloud solution. With ARM you can easily deploy a Databricks workspace or a Kubernetes cluster. But what if you'd like to deploy something *inside* those services? That's not possible with ARM and it is with Terraform if you just add the complementary providers. Even if you stick with "plain" Azure, often you'll have to interact with Azure AD. There is [a Terraform provider](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs) for that! And no ARM support...

If this sparked your curiosity, [here](https://devblogs.microsoft.com/devops/arm-templates-or-hashicorp-terraform-what-should-i-use/) is another interesting comparison between Terraform and ARM on the Microsoft Dev Blog.

Bicep, the new kid on the block
-------------------------------

![](https://dataroots.ghost.io/content/images/2022/06/image-1.png)

It seems that Microsoft has heard our complaints. A few years ago, they started working on a new tool called [bicep](https://github.com/Azure/bicep). Like Terraform, it uses a declarative language and provides an extra layer on top of ARM. This is one of the goals of the project:

> Build the best possible language for describing, validating, and deploying infrastructure to Azure.

I must admit, they are nearing that goal. My personal preference still tends to lean more to HCL, but bicep is - again, personal opinion - lightyears ahead of ARM.

Some clear advantages of using bicep over ARM:

-   you can easily split up your code into modules
-   the language reads more fluently so that you can understand what the code is doing a lot faster
-   less boilerplate code

Since bicep compiles down to ARM templates, they weren't able to solve the other complaint: the one of supporting more kinds of resources than the ones supported by ARM. This is also stated in one of the clearly defined non-goals of the project:

> Provide a first-class provider model for non-Azure related tasks. While we will likely introduce an extensibility model at some point, any extension points are intended to be focused on Azure infra or application deployment related tasks.

State
-----

We cannot discuss IaC tools like ARM, bicep and Terraform and not discuss state. [A lot](https://www.bejarano.io/terraform-stateless/) has been written about this topic. There are advantages to having the state, as well as disadvantages. There could exist other implementations to achieve what Terraform state files are trying to achieve, but this is what we have right now.

What does it do? Terraform stores the current state of the infrastructure it knows about in a state file. TL;DR the state makes it possible to compare the current state with the desired state and then derive the needed actions to get to the desired state.

ARM/bicep don't have this concept. It makes it easier to manage your IaC project, but ARM/bicep are also not very good at housekeeping. Whenever you delete a resource from your code, it keeps on existing in the cloud. The only difference being that your code does not know about it anymore and does not update it anymore. That is true if you use the default settings. You can also deploy your resources in the "complete" mode with ARM/bicep. In that scenario, Azure will delete everything that is not in your ARM template. But be aware, when I write *everything*, I really mean everything. Sometimes resources can be automatically created by other resources (looking at you, Azure ML) or as part of interacting with your Azure resources. It might be a good practice to enforce users to really define everything in their ARM/bicep templates, but personally I'm not really fond of this approach.

Once again, Microsoft has listened. Deployment Stacks are (almost) here to the rescue. This is a future Azure service which was [being discussed](https://www.youtube.com/watch?v=-4E5DsC-RcU&feature=youtu.be&t=1657&ref=dataroots.ghost.io) a couple of years ago but has not been formally announced yet. Eventually it would probably become part of [Azure Blueprints](https://docs.microsoft.com/en-us/azure/governance/blueprints/overview). So, how would they resolve this? Deployment Stacks would basically keep track of all resources deployed through an ARM/bicep deployment and would allow you to manage specifically this subset of resources as a whole. It could make it possible for ARM/bicep to adopt a concept similar to Terraform's state, but in a more managed way. Music to my ears, but we're not there yet.

A cloud engineer embarking on a quest
-------------------------------------

"There must be a way to get the best of both worlds" is what I kept wondering. I launched an initiative at [dataroots research](https://dataroots.io/research/) to look into this and went on a quest.

The goals I've set out for my initiative:

-   I want to write bicep code for all aspects of Azure that it supports. This would greatly enhance my experience and make it possible to adopt new features on day 1 of the release.
-   I want to use Terraform providers (except for the one for Azure services) to fill the gaps.
-   The 2 tools have to interact with each other as seamlessly as possible.
-   I don't want to be generating bicep or Terraform code using the other of the 2.

### Terraform and ARM

It has been possible for a while to [deploy ARM templates using Terraform](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/template_deployment). This was often the workaround we've used when we stumbled upon a limitation of the Terraform Azure provider. So this is where I started. It would certainly be possible. This is the main idea I started with:

1.  Write some bicep code
2.  Let bicep generate the ARM templates for it
3.  Let Terraform deploy these

*Sigh*, yeah, that would still be cumbersome. ARM and Terraform are different worlds, letting Terraform handle ARM templates does not always work well. Terraform does not know about the resources that have been deployed in ARM. Unless... you create Terraform data resources for them. But that is missing the point since in the case that there is a data resource available, the regular resource would also have been available.

I just want to be able to deploy bicep templates and then use the deployed services in Terraform. So basically: skipping the second and third steps of having Terraform deploy the ARM resources and just use native tooling (Azure CLI) to deploy the bicep templates directly.

What if you could transfer some information from your deployed bicep/ARM templates into your Terraform code? That is technically feasible. The ingredients for this marvellous recipe (for disaster?):

-   bicep templates have to define [outputs](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/outputs?tabs=azure-powershell&ref=dataroots.ghost.io)
-   we deploy the bicep templates in the regular way
-   *Terraform finds our deployment on Azure: not possible yet*
-   Terraform reads the values of these outputs and uses these where needed in other resources

For the third step we'd need a Terraform data resource that is able to read this information from Azure. Azure stores all past ARM/bicep deployments forever and ever, so the information must be available.

I've created [a pull request](https://github.com/hashicorp/terraform-provider-azurerm/pull/14524) in the Terraform provider for Azure. This would make this information available in our Terraform project and add the missing piece of our road to success. You can see the current status of that PR in the link above.

Meanwhile... in Redmond
-----------------------

When one of them wins, they all win. Microsoft truly [embraces](https://www.hashicorp.com/blog/hashicorp-and-microsoft-extend-multi-year-collaboration-agreement) Terraform as they just want everyone using Azure to have the most optimal experience. Microsoft helping out Terraform users just makes it easier to choose for Azure the next time you're choosing between cloud providers.

> Microsoft's newly released Terraform provider fills the missing gaps in your Terraform configuration

Microsoft has built [its own Terraform provider](https://registry.terraform.io/providers/azure/azapi/latest/docs) for Azure solving exactly the problem outlined in this blogpost. This provider offers users a way to talk directly to the ARM layer in Azure, while still treating those resources as native Terraform resources.

It comes with a single resource: `azapi_resource`

This allowed us to use the recently announced [Azure Private DNS Resolver](https://azure.microsoft.com/en-us/blog/announcing-azure-dns-private-resolver-now-in-preview/) right away after the announcement:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "example-rg"
  location = "westeurope"
}

resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

...
}

resource "azapi_resource" "private_dns_resolver" {
  type      = "Microsoft.Network/dnsResolvers@2020-04-01-preview"
  name      = "pdns-resolver-example"
  parent_id = azurerm_resource_group.example.id
  location  = azurerm_resource_group.example.location

  body = jsonencode({
    properties = {
      virtualNetwork = {
        id = azurerm_virtual_network.vnet.id
      }
    }
  })

  response_export_values = ["*"]
}

```

From day one. We could not believe it. It just worked. Once Terraform adds support for this resource, we'd be able to replace it, or we could just leave it as-is, which is perfectly fine as well.

Now, why is this so much better than the regular [`azurerm_template_deployment`](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/template_deployment)? The main difference lies in how Terraform stores information about this resource in its state. The `azurerm_template_deployment` deploys a *deployment*. The deployment can consist of one or multiple resources, but Terraform has no clue what this is. If you delete this resource, than it would just delete the reference to the deployment and not the resources inside. Same goes for updating: update your template file and Terraform wouldn't even know without [tainting](https://www.terraform.io/cli/commands/taint) the resource.

![](https://dataroots.ghost.io/content/images/2022/06/image-3.png)

Deployments in Azure

This changes with `azapi`. All properties of the resource become part of the Terraform state and Terraform starts tracking changes to it, knows when it has to update the resource in Azure and can properly delete it as well. It becomes a native Terraform resource.

> With azapi, the resources become native Terraform resources, benefiting from all the features Terraform brings to the table.

There's more
------------

Now that you've found out about this great addition to our toolbox, you want to start using Terraform, but what about your existing cloud resources? Well, until recently you had to embark on your own quest and start writing Terraform code for all of them and then running `terraform import` to import these into the Terraform state. I don't think anyone would be looking forward to this.

Again, Microsoft has heard us. The Azure team released another tool called [Azure Terrafy](https://github.com/Azure/aztfy). It does exactly what I described above. It does not cover 100% of the use cases, but for most environments it will get you 99% of the way.

Le nouveau Terraform for Azure est arrivé
-----------------------------------------

My quest came to an end sooner than expected and I am very happy with what I ended up with. The pull request might be no longer necessary, but at least it helped me sharpen my Go skills and it might still benefit other users of the provider. The [recent announcement](https://techcommunity.microsoft.com/t5/azure-tools-blog/announcing-azure-terrafy-and-azapi-terraform-provider-previews/ba-p/3270937) of these 2 new tools made us very happy. Microsoft has clearly listened to the voice of the Terraform users and we're closely watching this space to see what the future still holds for the fans of Infrastructure as Code.