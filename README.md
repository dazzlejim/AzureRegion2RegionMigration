# Azure Region-to-Region Migration
![banner]

<br/>

## Overview
This guide is intended to provide several considerations when relocating services between Azure regions and/or geographies. The tips outlined in this document are based on customer experiences and the challenges observed as a Cloud Solution Architect when assisting customers with their Azure-to-Azure migration projects.

> **Note:** This is not an exhaustive list of recommendations and by no means intends to address every customers scenario. It should be used purely as a guide and to prompt the folks responsible for moving Azure services between regions on the things they should take into consideration. For more detailed information please refer to the links provided in this guide. Like any good open-source project, please feel free to contribute your thoughts, ideas and experiences.

<br/>

## Scenario
Enterprise grade customers are constantly looking for ways to reduce latency, ensure compliance with the latest data governance and sovereignty regulations, and achieve business continuity on Azure. The scenario described in this guide is an existing Azure enterprise customer that is considering relocating their Azure hosted applications, data, and services from one region/geo to another. This scenario could come about for instance when a new Azure region opens closer to your location of business and you would like to bring your Azure services that may be hosted in another region or geography closer to your customers. For reference see the [Azure region network latency statistics][RegionLatency] when considering relocating.

In this case you'll be looking for ways to relocate Azure services with maximum ease, and minimal effort and risk. Unfortunately, there is currently no magic button that you can push to relocate all of your resources at once - the best case scenario is if all of your Azure resources have been deployed as code and they can simply be re-deployed to another Azure region. This however is rarely ever the case so careful consideration and planning is required to ensure a smooth transition.

> **Note:** The intention could be to relocate all or only some of your current applications and services to a new target Azure regions. The guidance outlined here applies regardless of your scenario and really only differs with regard to the decommissioning activities required.

<br/>

## Relocation strategy
In this section we look at the overall approach and how to formulate a plan of attack. At a high level consider the following phases when planning for the move:
1. [**What** - Discovering the resources that are to be relocated](#What---Discovery)
2. [**How** - Method used to move resources](#How---Methods)
3. [**When** - Planning the migration stages](#When---Staging)
4. [**Where** - Understanding the new target region(s)](#Where---Target-Azure-regions)
5. [**Who** - Assemble the team!](#Who---Assemble-the-team)

### What - Discovery:
Before migrating any resources, one of the obvious first steps to take is to understand what actually needs to be relocated and to ask - *"should we relocate all, some or no resources to a new Azure region"*.

To get a view of all Azure resources within your Azure subscription consider creating an inventory dashboard to help you identify service types and to get an idea of the quantity as it'll help to have a view of the scale of the migration tasks that lay ahead - see this blog from [Billy York][InventoryDashboard] on using [Azure Monitor Workbooks][WorkBooks] to build out an inventory dashboard.

A common approach used by customers when initially migrating to Azure from on-premises is to use a migration discovery and assessment tool such as [Azure Migrate][Azuremigrate]. These tools are used to help organizations learn more about their environment including the inventory, capacity and utilization of their Virtual Machines. They can also be useful when planning to move from one Azure region to another. In addition consider also using [Azure Service Map][ServiceMap] to discover dependencies and identify connections between Virtual Machines. The information collected by these tools will help your organization group applications and services based on their relationship with one another and their dependencies - these groupings of applications and services allow you to break the potentially daunting task of relocating large amounts of services down into smaller, more manageable groups to migrate. To complement this information also consider using [Network Watcher][NetworkWatcher] which will enable you to visualize your Azure virtual network topology.

Another key aspect is to ensure application owners are identified and assigned to all Azure resources. This will be useful when you start planning the migration phasing and will help ensure the relevant application owners have input into the timing (think potential outage windows), testing and any necessary change control procedures. The recommended approach here is to use [Tags][Tags] to indicate application owners for each resource.

Once you have good visibility of your Azure resources it's important to also understand how customers, both internal and external, interact with your services and applications. Thought needs to be put into how clients will be re-directed after the relocation of a service or application and how/if the move will impact them.

This also includes understanding integration points with other internal or external services. For example there are scenarios where Internet facing Firewalls are used and IP addresses may be white-listed in a connecting system somewhere. This is important to understand as Azure assigns [public IP addresses][PublicIps] in blocks based on the region, so the public IP addresses used in the new target Azure region will be different. Any whitelisted addresses will need to be updated by the relevant parties.

In addition, the URLs of your PaaS based services are important to understand as they may need to change during the relocation to the new target Azure region. Because the URLs are globally unique, any new and old PaaS based service cannot share the same name, at the same time.

Also understand what [business continuity and disaster recovery][BCDR] solutions may be in place for the applications that are intended to be moved to the new Azure region. Depending on the application and method used to provide disaster recovery, these same methods can be used to shift the application to the new target Azure region and to redirect customers. As you'll see below, the techniques and tools used to provide disaster recovery capabilities can also be used for migration. Depending on the criticality of the application, a disaster recovery solution may, or may not be in-place - identifying this, along with establishing the application and service groupings will provide key input when planning the [staging phase](#When---Staging) of the relocation.

Beyond the technical view also be sure to understand the skills that are likely required and the availability of the people your organization is likely to need to complete the migration to the new target Azure region. You may discover that there is either a skill and/or availability gap and you may need to bring in external people to assist. Alternatively consider scheduling the migration to align with resource availability.

### How - Methods:
There is no "silver bullet" when it comes to moving cloud services between regions and it will likely require a combination of approaches to tackle the various types of services which range from [IaaS to PaaS][IaaS], applications and data. Once the discovery is complete you can then consider how each of the Azure service types can be relocated, it is likely that they'll need to be divided into groups based on either the method of migration or the applications dependencies. Below are some of the more common methods to consider, some of which are also highlighted in this [Disaster Recovery][DisasterRecovery] architecture. As with any IT system change it's important to determine how each change activity can be tested and the outcomes validated - if in doubt, test, test, and test again!

* **Redeploy/Recreate** - Many customers by now are investing in [Infrastructure-as-code][Iac] to provision their Azure resources in an automated and predictable fashion. By leveraging your organizations Infrastructure-as-code assets and skills, they can be used to re-provision Azure services into your new target region which can make testing easier prior to cutting over and helps to ensure consistency in service configuration. Where practical, this would be the preferred approach as when the services are relocated to the new region, they would remain under the control of your organizations CI/CD pipeline tooling.

    Alternatively if the existing Azure resources are not already defined as code then consider using methods such as [exporting the configuration][ExportArm] of your Azure resources to an ARM template - although this process typically will not produce a ready to use ARM template it will at least give you a good head start and also provides a configuration "as-built" for the resource. Manual redeployment also could be an option if the Infrastructure-as-code method is not suitable for whatever reason. Another factor to consider is if custom virtual machines images are used within the organization and if these can be used to provision new virtual machine instances in the new target Azure region.

* **Specialized Moving Tools** - Utilities such as [Azure Resource Mover][ResourceMover] can be used to relocate some Azure resources and include the ability to test the move before committing to actually moving the resources. At the time of writing the utility can be used to move the following Azure resources:
    * Virtual machine
    * SQL Database 
    * SQL Elastic Pool
    * Virtual Network
    * Load balancer
    * Public IP
    * Resource group
    * Network Security Group
    * Network Interface
    * Availability Set 
    * Along with SQL Server, Key Vault & Disk Encryption Set with documented manual steps

* **Replication** - Many services have native replication capabilities built in which can be leveraged when relocating a service to an alternative location. Replication can be beneficial as the approach can allow for multiple instances of the service to exist simultaneously across many Azure regions. This effectively provides a distributed working solution while transitioning from the current Azure region to the new Azure region. 

    It can also add flexibility when scheduling the relocation activities and can help reduce the risks associated with "big-bang" cut-overs which are typically ideal to avoid. Many methods used to replicate services are utilized in multi-region [Disaster Recovery][DisasterRecovery] application architectures. As an example, consider the following services and their ability to replicate natively:
    * Active Directory Domain Controllers *(including DNS)*: A prime example of an application that has built-in replication capabilities is a Domain Controller. The Active Directory Domain Controller services are hosted on Windows virtual machines, the ability to enable replication requires the virtual machines hosting the services to be able to communicate with each other over a private network. Most applications that are hosted on virtual machines and have native replication capabilities will also require the ability to communicate between the current Azure network and the new target Azure network - for more details refer to the [Network Integration](#Network-integration) section in this guide.
    * Azure Image Gallery: Many customers will create custom virtual machine Operating System images to provision Azure virtual machines from, and although not always critical to the day-to-day operation of an application they still need to be available in the new target Azure region to enable the provisioning of new virtual machines.
    * SQL: There are three SQL service offering types on Azure, the more traditional do-it-yourself SQL product on a virtual machine, SQL Managed Instance and Azure SQL which is a fully managed PaaS offering. For the self-managed traditional SQL product consider configuring replication between the current Azure region and the new target Azure region using [SQL AlwaysOn groups][SqlAlwaysOn] - like many other virtual machine hosted applications, this method does require network connectivity between the current Azure network and the new target Azure network. For the managed versions (SQL Managed Instance and Azure SQL) review the [native replication capabilities][MovingSql] to enable the move between Azure regions. Both of these services utilize their public endpoints for connectivity.

* **Cloning** - This method utilizes services such as [Azure Site Recovery][SiteRecovery] to effectively clone a virtual machine and it's attached disks from one Azure region to another. Again this is a technique that is commonly used as a [Disaster Recovery][DisasterRecovery] solution. The benefits of this method is that when a virtual machine is powered on in the new target Azure region it will essentially be the same virtual machine and will have the same identity if it were domain joined to Active Directory. There are options to either keep the same IP address or to assign a new IP address from the new target Azure virtual network. For more details on this including the pros and cons refer to the [Network Integration](#Network-integration) section in this guide.

* **Redirection** - This is not so much about moving Azure resources rather it focuses on moving (or redirecting) customers to the services in the new Azure region. Once the resources have been relocated from the old Azure region to the new target region the next step is to redirect your organizations internal and external clients to the newly relocated services. It's important to distinguish between internal clients that may be connecting to your Azure services over internal private networks and those considered to be external because they connect to your Azure services over the public Internet. This is not to be confused with clients that are actual employees of your organization rather it's related to how they connect to your services.

    In addition to people communicating with your Azure services there will be other IT systems that will also need to communicate. In some instances, the communication methods will be similar.

    * **Internal clients:** For internal clients, whether they be actual people or other IT systems, they will typically communicate over a private network that connects the organizations on-premises network to their Azure virtual network(s) - these services are likely to include either an [Express Route][ExpressRoute] and/or a [VPN][VPN] connection. These networking services are regionally dependant meaning new instances will likely need to be established within the new target Azure region and would almost definitely be required if your organization is considering decommissioning the old Azure region once all resources have been relocated. In addition to re-establishing connectivity with the new Azure region virtual network(s), it's possible that the IP addresses of the newly relocated Azure services would have also changed. Note that when relocating virtual machines for instance, the options are to either retain the original IP address or assign a different IP address from a new range.
    * **External clients:** External clients typically take a different approach to connectivity as your organizations Azure services are likely to be published using public facing Internet IP addresses and would (hopefully) be secured behind some form of network security system. The public facing Internet IP addresses that represent these external services are regionally dependant meaning that once the Azure services have been relocated to the new target region, their public facing Internet IP addresses will change to a new address from that Azure regions address range. As you go live with the services in the new Azure region be sure to update the external DNS records to reflect the new IP addresses in use. Thankfully there are services available that prevent the need to use these IP addresses directly, Azure services such as [Traffic Manager][TrafficManager] and [Front Door][FrontDoor] provide an abstraction and a more friendly DNS name to use . They also provide the ability to redirect clients and failover between Azure regions making them ideal solutions when needing to redirect clients to your new target Azure region. Again these techniques are commonly used as a [Disaster Recovery][DisasterRecovery] solution and your organization may already have similar solutions in place that could help facilitate the relocation.
<br/>

### When - Stages:
This is where you determine the staging of the migration of services from the current Azure region to the new target Azure region. At a high-level consider the following stages when doing your planning:
1. **Now, later or never:** First up consider the timing of the relocation in terms of application life cycle and whether your organisation should prioritize the relocation of all applications and services to the new target Azure region with some degree of urgency, or should the relocation adopt a more pragmatic and phased approach where the natural life cycle of an application is taken into consideration. The answer depends on several factors including the perceived benefits of relocating vs. any risks associated with the move along with the potential costs and complexities of operating across multiple Azure regions/geographies for a period. Additionally, some applications and services may have a limited lifespan and it may make sense to let these applications live out their life in the old Azure region rather than invest the time and effort in relocating them.
2. **Preparing the new target Azure region:** You may already be familiar with the [Microsoft Cloud Adoption Framework for Azure][CAF] (a.k.a CAF) and the concept of [Landing Zones][LandingZone] which provide a pattern for establishing an architectural approach and a reference guide for enterprises. If not then I recommend taking the time to review this guidance as it may help when defining the new Azure region. Essentially, whether you've followed none, some or all of the guidance it's likely that you'll be needing to establish the core services within the new target Azure region - these are likely to include shared services such as firewalls and Domain Controllers, monitoring, management and connectivity capabilities among others. These are essential to the daily operations of your cloud applications so careful consideration is needed prior to relocating any  services from the current Azure region. If you're considering relocating all services into the new target Azure region then you'll need to re-establish the core services that your organization currently rely on as not to lose these capabilities. 
    
    Also consider if this is the right time to change the design of your Landing Zone in the new target Azure region and what the potential risks and impacts may be - for some organizations this can be a prime opportunity to address any shortcomings with their current Landing Zone design, for others it may be a case of acknowledging the Landing Zone guidance but they may consider adopting changes as a continuous improvement program post relocation. A key area to address is the network design for the new target Azure region and how/if it integrates with the current Azure region - for more details refer to the [Network Integration](#Network-integration) section in this guide.
3. **Proof of concept:** This is where you get the opportunity to evaluate tools, techniques, and processes. It's important to gain a good understanding of how tools may work, what their limitations are and to assess the time and effort required to perform the service migration. This is also useful to determine potential outage windows for those services that may need to be taken offline for a period and also to determine a rollback strategy if something goes wrong during the migration. This phase is really intended to prove that the approach has been tested in order to reduce risk and gain business confidence - it also presents an opportunity to determine the effectiveness of the [Landing Zone][LandingZone] within the new target Azure region and whether any changes are required.
4. **Non Production environments - Development, Test and UAT:** Like any other significant changes to your IT environment, it's good practice to test the waters first with non-critical systems that are not directly used for production purposes. For some customers, their non-production environments are very similar to their actual production environments so it can make sense to get the migration procedures down to a fine art before embarking on making changes to any live production systems. Ideally during this stage your organization will also be able to validate the procedures required to redirect "test" clients to the services located in the new target Azure region as well as validating the roll-back procedure if something goes wrong. 

    As always, this does not completely remove all risk to the relocation of actual production services, but it should at least reduce the risk levels providing the pre-production systems are similar enough to those in production. If for whatever reason your organizations IT systems are significantly different from the production systems then consider what additional testing and validating may be required to reach an acceptable level of comfort prior to relocating any production IT systems. Ideally you want to reduce the "unknowns" as much as possible/practical before proceeding so there are no unwelcomed surprises.

    Now is also a good time to test the support process with your organizations support providers - this may be directly with Microsoft, managed service providers and/or 3rd party application support teams. If your organization has a Microsoft support agreement, then you may have access to a Customer Success Account Manager who could provide advice and guidance. Also ask your support providers if they advise proactively creating support tickets for the migration activities that include critical systems, often it can help if you're able to provide information in advance to reduce the effort and time in the advent of an issue when performing the migrating.
5. **Production (and Disaster Recovery environments):** Ideally by this stage, most if not all risks should be well understood and mitigation techniques have been identified to address them. Also, most if not all unknowns should ideally be addressed and you should have confidence in what to expect regarding any relocation activities - any guess work and assumptions should have been turned into facts during the previous stages.

    It is common for customers to host production services in more than one Azure region, typically the design pattern can include "live" production services in one region and "stand-by" production services in another - this is a common disaster recovery solution pattern described in the [Disaster Recovery][DisasterRecovery] design guidance. In the case of a live/stand-by solution design pattern, it's typical that both ["preferred" and "alternative" regions][Regions] are utilized - for more details on this topic refer to the [Where - Target Azure regions](#Where---Target-Azure-regions) section in this guide. Alternatively, customers may distribute their applications across multiple Azure regions in a live/live design pattern, while others may only utilize just a single Azure region.

    What is important at this stage is to effectively failover between the old Azure region and the new target Azure region with as little downtime and loss of data as possible. As outlined in the [Methods](#How---Methods) section of this guide you would have by now determined the best method(s) to use and can begin to action them to relocate your organizations Azure services.

    Depending on the size and complexity of the applications and services in your organizations current Azure region, the relocation activities may be completed over a period of days, weeks or even months. Not all applications exist in isolation, even when you've grouped applications and services together, it's likely there will still be dependencies on other applications and/or services. As groups of applications and services are relocated to the new target Azure region, it's likely they'll still need to continue interacting with applications and services still hosted in the old Azure region that are yet to be relocated. For details on options for configuring your network in preparation for the migration refer to the [Network Integration](#Network-integration) section in this guide.
6. **Decommissioning:** This stage doesn't actually come at the very end, in reality decommissioning activities are more likely to occur incrementally on a migration group-by-group basis. The actual final stage of decommissioning may be when the networking integration with the old Azure region is removed. As with any type of migration, be certain that the old applications and services are no longer being used in any way prior to removing them.

<br/>

### Where - Target Azure regions:
When reviewing the new target Azure regions be sure to have a good understanding of the differences between ["preferred" and "alternative" regions][Regions], these are designed to work together as regional pairs within a geography and a number of Azure services utilize this model for their geo-redundant capabilities. An example of an Azure service that uses this model are [Azure Storage Accounts][StorageAccountRedundancy]. When you create a storage account, if enabling geo-redundancy you select the primary region for the account - the paired secondary region is determined based on the primary region and can't be changed. This model may change over time however it's worth having an understanding of the geo-redundant capabilities of the Azure services you intend relocating to help inform your choice of new target Azure regions. Ideally the new target Azure regions should match the existing Azure regions regarding preferred vs alternative so you'll be able to retain the same solution design model for your applications and services once relocated.

Also take note of the [products that are available in each Azure region][ProductsByRegion] as they do vary region by region based on whether they are [categorized as foundational, mainstream, or specialized services][ServiceCategories]. In addition note that certain capabilities such as [Availability Zones][AZs] are typically only available in preferred regions. You don't want to get part way into a region migration only to discover that the new target region you've selected does not actually have the Azure service or particular capabilities you need.

> **Note:** If network latency is important in your organization's decision making process you can refer to the [Azure region network latency statistics][RegionLatency] which shows the network round-trip latency statistics between Azure regions.

<br/>

### Who - Assemble the team:
Relocating applications and services from one Azure region to another is very likely going to involve many people across many teams. Consider the skills and resources available within your organization as well as the time commitment required by these teams to focus on the migration. In no particular order, here are the likely roles that would be involved in a enterprise level migration:

* **Application owners / Key stakeholders** - The people responsible for your organizations applications clearly have a vested interest in ensuring the applications and services that they are responsible for will continue to serve their customers during the relocation to the new target Azure region. These people are also likely to have an understanding of the applications peak times of usage which is going to help inform the migration schedule - performing migration activities during off-peak is obviously going to present less risk to the business.
* **Scrum Masters / Project Managers** - Good coordination and planning is key to ensuring a successful migration. Typically for enterprise customers, there will need to be someone leading the migration and coordination activities across the various people involved.
* **Internal Operations team** - As applications and services are migrated to the new Azure target region there will remain the need to provide monitoring and alerting during the relocation. As new resources are provisioned within the new target Azure region they'll need to be registered with your organizations monitoring tools, and as services are decommissioned from the old Azure region they will need to be removed from monitoring tools.
* **Solution Architects** - Having a good understanding of the overall architectures for both the core [landing zone][LandingZone] environment and that of individual applications is critical in ensuring a smooth relocation to a new target Azure region. Solution Architects should ideally have a good high-level overview of your organizations IT landscape and should provide the guidance needed when designing the landing zone in the new target Azure region and how it integrates with the systems in the old Azure region. In addition, Solution Architects would work across the other engineers and developers when planning application and services relocation activities.
* **Network Engineers** - One of the key aspects that is likely to be required for a successful relocation is the network integration - specifically how the old and the new Azure regions communicate with each other and how clients access the services once relocated to the new target Azure region. The more [IaaS][IaaS] type services your organization has, the more you'll likely be relying on a Network Engineer.
* **Security Engineers** - Securing an organizations data and applications should always be top of mind and the Security Engineer has an important part to play when relocating to a new target Azure region. Working alongside the other architects and engineers, the Security Engineer would be focusing on ensuring the appropriate security guardrails are in place within the new target Azure region using the guidance outlined in [Cloud Adoption Framework - Landing Zones][LandingZone] and also that once relocated, the applications and services are integrated into the organization's security operations center (SOC) tooling.
* **Application developers** - Depending on how the applications and services have been developed, there could be scenarios where configuration in the code is referencing another system or service - consider a web application that includes a connection string to a backend database. The application developers are likely to be in the best position to understand if changes may be required to the code configuration in the advent of reference changes for a system or service as a result of the relocation.
* **Your local Microsoft team** - If your organization has a Microsoft support agreement then you may have access to technical resources that can provide advise and guidance both leading up to and during the migration to the new target Azure region.

> **Note:** Your organization may include a combination of internal staff, contractors and/or outsourced managed service partners - regardless of the mix, the important point here is that it's crucial to have a trusted team of people with the appropriate knowledge and skills involved from the initial planning stage through to the execution.

<br/>

## Network integration
One of the more challenging aspects of relocating to e new Azure region is the network integration - this is more so the case if your organization has a large number of [IaaS][IaaS] based applications that are dependent upon virtual networking. Here are some things to consider with regard to networking when relocating to a new Azure region:

1. **Understanding what's in place today:** You'll need to have a good understanding of the existing network solutions your organization has in place today that provide services to the applications and services in the current Azure region. If there are [IaaS][IaaS] Azure resources within your current Azure region then it's very likely that there are an existing [Express Route][ExpressRoute] and/or [VPN][VPN] solutions in place. In more complex environments there may be multiple instances of these networking services - if your organization has additional instances of these networking services that are provisioned within non-production environments then be sure to factor these into the migration pilot planning.
  
    In addition to the connectivity also determine what public IP addresses are provisioned in the current Azure region - it's important to understand this as Azure allocates public IP addresses from a range unique to each region. The public IP address is region-specific and won't be retained in the target region after the move.

    Another important point to understand is whether static private IP addresses are used and if there are "hard-coded" IP addresses present in any configuration or application - the hope is that this is not the case and DNS is used instead. This is important to understand as it may limit your options when determining which relocation method to employ for a particular application or service and it also may limit the ability to communicate effectively between the old Azure region and the new target Azure region.
2. **Establishing the new connectivity:** New instances of networking services will need to be provisioned in order to continue providing access to the applications and services once they are relocated to the new target Azure region. If you have an Express Route circuit then check with the [provider][ExpressRouteProviders] to determine if they operate in the new target Azure region that you're intending relocating to - it is possible that you may need to choose a new Express Route provider for the new region. For the duration of the relocation there may be service and cost duplication until the migration is complete and the old networking services can be decommissioned. 

    Depending on the duration and order of your migration plan, consider if full network capacity is required immediately or if you're able to start small then scale up as needed - this could either mean starting with just a VPN initially then provisioning the Express Route circuit at a later date or starting with a smaller Express Route circuit initially then [increase the bandwidth][ExpressRouteModify] to support the capacity requirements as needed.
3. **Ensuring coexistence during the relocation:** For any lengthy relocation program it's reasonable to expect that there will need to be some form of connectivity between the applications in the older Azure region and those that have been relocated to the new target Azure region. This coexistence period may be days, weeks or months depending on the scale and complexity of your Azure environment. For less complex environments consider enabling [network peering][NetworkPeering] to establish private network connectivity between the virtual networks in the old and new Azure regions. For more complex scenarios where there may be multiple virtual networks in multiple locations consider [Azure Virtual WAN][VirtualWan] as a means to establish private network routing between the old Azure region and the new.

     In some scenarios it may not be possible to enable private network routing between the old Azure region and the new target Azure region. This scenario can come about if the virtual network address ranges in the new target Azure region need to be the same as those in the old Azure region - in order to be able to route between networks the [IP address ranges need to be non-overlapping][OverLappingAddresses]. This is typically a result of "hard-coded" IP addresses that cannot be changed. The best-case scenario is not to have dependencies on "hard-coded" IP addresses and to use DNS where possible. This however may not be an issue for your organization and does depend on each applications requirements and the order of the relocation activities - this highlights the importance of the discovery and planning stages.
4. **Cutting over to the new Azure region:** The critical part has come where you now redirect clients to the services that have been relocated to the new Azure region. Refer to the ['How - Methods'](#How---Methods) section of this guide for considerations on redirecting both internal and external clients.

<br/>

## What could break during the relocation process
As with any migration project there is the risk that something will break during the relocation process either by accident or due to the nature of the service being relocated, ideally these can be identified during the [discovery stage](#What---Discovery) and a plan can be put in place to mitigate the risk.
Here are some points to consider:
* **Automation, Scripts and Infrastructure-as-code:** Your organization may have invested in automation techniques and these can certainly assist with the relocation process however be sure to identify and change any references to Azure regions, service names and service ULRs in the code to reflect the new Azure region. Ideally values such as these that possibly may change during the lifecycle of an application would be configured as either 'parameters' or 'variables' within the code making the changes less onerous.
* **Public IP addresses:** Azure allocates [public IP addresses][PublicIps] from a range unique to each Azure region, this means that when a new public IP address is provisioned in the new target Azure region, the addresses will have changed from what it is currently. As you go live with the services in the new Azure region be sure to update the external DNS records to reflect the new IP addresses in use. If external parties have whitelisted your organizations public IP addresses, then they will need to be supplied with the new IP addressed of the relocated services.
* **PaaS service URLs:** Many Azure PaaS based services are configured with globally unique DNS names such as Azure Storage _(https://<unique_name>.blob.core.windows.net)_ and Azure App Service _(https://<unique_name>.azurewebsites.net)_. If you're provisioning new instances of these services within the new target Azure region while the current service is still provisioned in the old Azure region then there will be a naming conflict as they cannot exist at the same time, with the same name. It is likely that the old PaaS service and the new PassS service that has been provisioned in the new target Azure region will need to run in parallel for a period during the migration process - this means the new PaaS service will likely need a name that is different to the current name. Again DNS comes into play here and can provide a layer of abstraction using [CNAME records][DnsCname] making any service name change easier to manage.
* **Private Link:** [Azure Private Link][PrivateLink] enables you to access Azure PaaS Services over a [Private Endpoint][PrivateEndpoint] in your virtual network. This service uses the PaaS service unique URL as described above and which may change during the relocation to the new target Azure region.
* **Backup Vault (Recovery Points):** When virtual machines are relocated to the new target Azure region they will need to be first de-registered with the current [Azure Backup][AzureBackup] service and then re-registered with the new instance of Azure Backup located in the new region. This will cause the access to existing recovery points for that virtual machine to be lost as the recovery points cannot be relocated to the new Azure Backup service instance in the new target Azure region.
* **Log Analytics Workspaces:** Log Analytics Workspaces should ideally be per region (ref to [Best practices for Analytics Workspace][AnalyticsWorkspaceBestPractices]), this means new instances of Log Analytics Workspace(s) would ideally be provisioned within the new target Azure region. Unfortunately at the time of writing it is [not possible to move a Log Analytics Workspace to another region][AnalyticsWorkspaceMove]. If keeping the historic Log Analytics data is important to your organization then consider either keeping the current Workspace until you're confident you no long require the data or use one of the few methods available to [export the data][AnalyticsWorkspaceExport] to a Storage Account in the new target Azure region.
 
In addition, consider if the relocation of applications and services to the new target Azure region will be "like-for-like" with changes being kept to a minimum or if your organization will take this opportunity to remediate or redesign any aspect of the Azure environment as part of the relocation efforts. This really is a case of risk vs. reward and careful consideration is needed as it can be tempting to change as you relocate however be clear on the risks and implications.

<br/>

## Cost considerations
* **Reserved Instances (and Reserved Data Capacity):** Reserved Virtual Machine Instances enable you to get prioritized compute capacity in Azure regions at a reduced price and can automatically apply to other Virtual Machine sizes within the same group and region. To move a Reserved Virtual Machine Instance to another Azure region requires you to [exchange][RIexchange] it - an exchange allows you to receive a prorated refund based on the unused amount, which applies fully to the purchase price of a new Azure Reserved Virtual Machine Instance. There's no penalty or annual limits for exchanges however the lifecycle of any Reserved Instance should be taken into consideration.
* **Regional cost variations:** For some Azure services the [price][AzurePricing] will vary region by region so be sure to review the pricing before commencing with the relocation.
* **Service duplication:** It is likely that at some point during the relocation there will be multiple instances of a particular service for a period of time until the old service has been decommissioned so be sure to understand this and factor it into your budget. It can be beneficial to run services in parallel during the relocation as it can provide for an easy roll-back if required however it may result in increased Azure costs for the duration. 

<br/>

## Reference Links

* [Service Map][ServiceMap]
* [Inventory Dashboard][InventoryDashboard]
* [Azure Monitor WorkBooks][WorkBooks]
* [Azure products by region][ProductsByRegion]
* [Azure regions][Regions]
* [Azure Site Recovery][SiteRecovery]
* [Disaster Recovery][DisasterRecovery]
* [Azure Migrate][AzureMigrate]
* [IaaS vs PaaS][IaaS]
* [Resource Tags][Tags]
* [Business Continuity / Disaster Recovery][BCDR]
* [Resource Mover][ResourceMover]
* [Infrastructure-as-Code][IaC]
* [How to export ARM Templates][ExportArm]
* [Shared Image Gallery][SharedImageGallery]
* [Traffic Manager][TrafficManager]
* [Azure Front Door][FrontDoor]
* [SQL Always-On][SqlAlwaysOn]
* [Moving SQL][MovingSql]
* [Azure Region Latency][RegionLatency]
* [Express Route][ExpressRoute]
* [Express Route Providers][ExpressRouteProviders]
* [Modifying Express Routes][ExpressRouteProviders]
* [VPN (Virtual Private Network)][VPN]
* [Cloud Adoption Framework][CAF]
* [Landing Zones][LandingZone]
* [Storage Account Redundancy][StorageAccountRedundancy]
* [Service Categories][ServiceCategories]
* [Availability Zone's][AZs]
* [Virtual Network Peering][NetworkPeering]
* [Azure Virtual WAN][VirtualWan]
* [Overlapping IP Addresses][OverLappingAddresses]
* [Public IP Addresses][PublicIps]
* [Azure Private Link][PrivateLink]
* [Azure Private Endpoint][PrivateEndpoint]
* [DNS CNAMEs][DnsCname]
* [Azure Backup][AzureBackup]
* [Analytics Workspace Best Practices][AnalyticsWorkspaceBestPractices]
* [Moving an Analytics Workspace][AnalyticsWorkspaceMove]
* [Exporting Analytics Workspace][AnalyticsWorkspaceExport]
* [Exchanging Reserved Instances][RIexchange]
* [Azure Pricing][AzurePricing]
* [Network Watcher][NetworkWatcher]

<br/><br/>

<!-- Local -->
[Banner]: images/banner.png

<!-- External -->
[ServiceMap]: https://docs.microsoft.com/en-us/azure/azure-monitor/vm/service-map/
[InventoryDashboard]: https://www.cloudsma.com/2020/10/ultimate-azure-inventory-dashboard/
[WorkBooks]: https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview/
[ProductsByRegion]: https://azure.microsoft.com/en-us/global-infrastructure/services/
[Regions]: https://docs.microsoft.com/en-us/azure/availability-zones/az-overview#region-and-service-categories
[SiteRecovery]: https://docs.microsoft.com/en-us/azure/site-recovery/azure-to-azure-architecture
[DisasterRecovery]: https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/disaster-recovery-enterprise-scale-dr
[AzureMigrate]: https://docs.microsoft.com/en-us/azure/migrate/
[IaaS]: https://azure.microsoft.com/en-us/overview/what-is-iaas/
[Tags]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/tag-resources
[BCDR]: https://docs.microsoft.com/en-us/azure/best-practices-availability-paired-regions
[ResourceMover]: https://docs.microsoft.com/en-us/azure/resource-mover/
[IaC]: https://docs.microsoft.com/en-us/azure/devops/learn/what-is-infrastructure-as-code
[ExportArm]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/export-template-portal
[SharedImageGallery]: https://docs.microsoft.com/en-us/azure/virtual-machines/shared-image-galleries
[TrafficManager]: https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview
[FrontDoor]: https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview
[SqlAlwaysOn]: https://docs.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-manually-configure-multiple-regions
[MovingSql]: https://docs.microsoft.com/en-us/azure/azure-sql/database/move-resources-across-regions
[RegionLatency]: https://docs.microsoft.com/en-us/azure/networking/azure-network-latency
[ExpressRoute]: https://docs.microsoft.com/en-us/azure/expressroute/
[ExpressRouteProviders]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-locations
[ExpressRouteModify]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-howto-circuit-portal-resource-manager#modify
[VPN]: https://docs.microsoft.com/en-us/azure/vpn-gateway/
[CAF]: https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/
[LandingZone]: https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/enterprise-scale/architecture
[StorageAccountRedundancy]: https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy?toc=/azure/storage/blobs/toc.json
[ServiceCategories]: https://docs.microsoft.com/en-us/azure/availability-zones/az-overview#comparing-region-types
[AZs]: https://docs.microsoft.com/en-us/azure/availability-zones/az-overview#availability-zones
[NetworkPeering]: https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview
[VirtualWan]: https://docs.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about
[OverLappingAddresses]: https://docs.microsoft.com/en-us/azure/virtual-network/concepts-and-best-practices
[PublicIps]: https://docs.microsoft.com/en-us/azure/virtual-network/public-ip-addresses
[PrivateLink]: https://docs.microsoft.com/en-us/azure/private-link/
[PrivateEndpoint]: https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview
[DnsCname]: https://en.wikipedia.org/wiki/CNAME_record
[AzureBackup]: https://docs.microsoft.com/en-us/azure/backup/
[AnalyticsWorkspaceBestPractices]: https://techcommunity.microsoft.com/t5/azure-sentinel/best-practices-for-designing-an-azure-sentinel-or-azure-security/ba-p/832574
[AnalyticsWorkspaceMove]: https://docs.microsoft.com/en-us/azure/azure-monitor/logs/move-workspace
[AnalyticsWorkspaceExport]: https://docs.microsoft.com/en-us/azure/azure-monitor/logs/data-platform-logs
[RIexchange]: https://docs.microsoft.com/en-in/azure/cost-management-billing/reservations/exchange-and-refund-azure-reservations
[AzurePricing]: https://azure.microsoft.com/en-us/pricing/
[NetworkWatcher]: https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-monitoring-overview