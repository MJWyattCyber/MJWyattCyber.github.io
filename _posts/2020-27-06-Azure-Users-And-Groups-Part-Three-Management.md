---
header:
    teaser:
title: "Az-104 Identities and Governance Parth Three: Resource Management"
date: 2020-06-27 08:07
layout: single
categories: [Azure]
toc: true
toc_title: Table of Contents
---

If you're coming into an existing organisation or your organisation hasn't had any governance then getting your head around all the resources can be a big headache and pretty daunting. Luckily, Azure has a few ways that you can ease the burden and help to keep things under control, some of these have already been touched on in the first two parts in this series. 

# Resource Management

## Management Groups

If your organisation has multiple subscriptions then you'll want to be utilising management groups, these groups sit above the subscription level and allow you to group subscriptions together and subsequently apply access permissions, policies and compliance across all subscriptions in that group.

### Creating Management Groups

Creating management groups is pretty straight forward, simply navigate to the service in the portal and select '**+ Add Management Group**'. This will bring up a new blade in which you are asked to provide an ID and display name, with the ID having to consist of lowercase characters only. You can provision nested management groups to allow for further granularity.

![mgmtgroup](/assets/images/Az104UsersAndGroups/portalmgmtgroup.png)

You can also create these in the CLI:

    az account management-group create --name <groupname> --display-name <displayname> --parent <parent management group (leave blank for the root)>

As well as within PowerShell:

    New-AzManagementGroup -GroupName "Groupname" -DisplayName "displayname" 

### Editing Management Groups

Assigning access or policies at a management group level is pretty straight forward, simply navigate to the management group in the portal, selecting it to open a new blade. From here selecting the 'details' option which is present next to the group name.

![details](/assets/images/Az104UsersAndGroups/detailsoption.png)

Within the details page there are further options available, you can get an overview from which you are able to view the items under the management group, You can also perform the following tasks:

- Rename
- Delete
- Move
- Add another management group
- Add a subscription

From the left hand menu you can select different blades, this is where you are able to view the Activity log for the management group, Assign Access via RBAC, look at policies assigned to the management group, and finally review cost analysis and budgets to get a detailed breakdown per group.

## Subscriptions 

Subscriptions sit below management groups and can be considered as a billing and security boundary. You can perform all the access/polciy assignments as at the management group level. You can view the billing information, payment methods etc. If you're the owner you will also have the option to Transfer billing ownership, change the (AAD) directory or cancel the subscription.

## Resource Groups

These are logical containers of resources that you chose to group together in order to manage them together. Most resources created on Azure will need to be stored in a resource group, one resource can only be a member of one resource group, but you can move most resources between resource groups if required. It's recommended that within the organisation you come up with a consistent naming convention for resource groups and resources within them in order to aid with identification of resources. 

### Creating Resource Groups

Resource groups can be created easily from with the portal, simply search for the 'Resource Group' service and select '+ Add' to add a new resource group, you'll need to provide a Subscription, Name and a location for the group. 

Using the CLI:

    az group create --name "resourcegroupname" --Location "location"

Using PowerShell:

    New-AzResourceGroup -Name "Name" -Location "Location

### Moving Resources

If you realise that you need to change things up, or maybe a you need to move resources to be under a different group, this can be achieved for most resources. There are some limiations, resources that can't be moved include:

- Azure Active Directory Domain Services
- Azure Backup Vaults
- Azure App Service Gateways

You will also need to be careful when moving Virtual Machines, you'll need to make sure that all of a VMs dependancies; it's network card, storage etc. move with it for it to function.

The easiest way to move a resource is via the portal, simply navigate into the resource group where the resource is located, select it by clicking on the checkbox and select the 'Move' option from the top ribbon. You'll be asked if you want to move the resource to another subscription or resource group. At this point you can chose an existing location or create a new one. When using the portal resource validation occurs automatically to ensure that the move is valid. 

Using the CLI

    az resource move --destination-group "destination group name" --ids "resource id to move"

Using PowerShell

    Move-AzResource -DestinationResourceGroupName "desitnation group name" -ResourceID "ID of resource to move"

Once a move is initiated the Source and Target resource groups are locked meaning that you can't make any changes until the move is complete. 

### Removing Resource Groups

If you decide that you no longer need a resource group and the resources within it then removing the resource group will remove all it's resources as well. The methods for removing resource groups is similar to creating them.

In the portal simply go into the chosen resource group and select **'Delete resource group'**

¬[delresgroup](/assets/images/Az104UsersAndGroups/deleteresgroups.png)

Once selected you will have to enter the name of the resource group to contine, the portal will also display the resources that you are removing.

Using the CLI:

    az group delete --name "resource group name"

There are a few other switches here but these are optional.

Using PowerShell:

    Remove-AzResourceGroup -Name "resourcegroupname"

## Using Locks

Locks can be placed on a subscription, resource group or an indivdual resource in order to stop any edits to it or to stop it from being deleted. There are two types of locks that can be applied; 'Read-Only' or 'Delete'. A lock is applied in the portal by navigating to the resource group, then selecting the locks blade from the sidebar menu, finally select '+ Add' selecting the type of lock and a name for the lock.

![newlock](/assets/images/Az104UsersAndGroups/newlock.png)

### Read-Only

A read-only lock will place the selected resources in read-only mode so will stop any modification or delete to the resource(s), this also covers new resources which will be blocked.

### Delete

A delete lock will allow all operations other than the ability to delete the resources.

## Tags

Tags are a great way to help keep organised, they are a key/value pair that can be applied to resources and resource groups. They allow for custom attributes to be assigned to resources, you may choose to apply a tag to identify the internal Business Unit or Project that it belongs to etc. This can be useful when it comes to billing as you can see spend for resources with certain tags.

A resource can have 50 tags, with the name limited to 512 characters for all types of resources except storage which is 128 characters. The value is limited to 256 for all resources. 

Tags can be enforced by policy, policy can also be used to apply tags automatically (more on that later) or simply just added manually to resources, commonly tags can be applied at resource creation. However, they can also be applied via the portal, whilst in a resource or resource group select the 'Tags' panel, from here you can add custom tag values.

![newtag](/assets/images/Az104UsersAndGroups/newtag.png)

Tags can be assigned in bulk, from within a Resrouce Group tick the tick box for the chosen resources and then select 'Assign tags' from the top ribbon, the similar optinos appear of selecting a Name and a Value.

## Policies

At a high level overview, an Azure Policy allows you to enforce rules that your resources need to follow. They can be enforced when the resource is created or enforced later on to audit the existing resources, in turn giving visibility of your resources compliance. An example policy would be to limit the sizes of Virtual Machines that can be created, or audit machines that are not using Managed Storage. 

### Viewing current definitions

You can view the built-in definitions via the portal, simply open the **policy** service and then select the **Definitions** panel under Authoring. This view shows all definitions as well as some built-in initiatives (grouping of definitions). selecting a definition will provide the json behind it, as well as giving you the option to assign it to resources.

### Assigning a Policy

To assign a policy open the **policy** service and select the **Assignments** panel under Authoring. On the new blade you can set the scope, this can be a Management Group, subscription, resource group or resource. You can set any exclusions to tell the policy to ignore these resources. 

The example I will use is to Append a tag and it's value from the resource group, this will apply the tag and it's value to any resource under the group selected.

After selecting the scope and any exclusions click on the three dots next to policy definition, a new window will pop up, select the desired definition (note you can make custom definitions) 

Fill out the required parameters for the chosen definition and press 'Review + Create', you can now view the policy within the Policies service and it's compliance.

# Cost Management

With Azure Cost Management you can really get a handle on your costs allowing you to plan, analyse and reduce unneeded spending in order to maximise the investment. Ultimately it all comes down to cost, so it's important to be able to get the right reports and save costs where appropriate. 

The **cost analysis** service provides a nice overview with graphical representation of your costs and predicted costs, there are numerous views that can be displayed all of which are slightly different. The default view of 'Accumulated cost' will show the current billing period and is useful for situation such as "Will I stay within budget?"

You can also display views such as Accumulated costs which will show costs accumulated for a specific date range. Costs by resource will display the cost by resource so you can see where the money is going. Daily costs will break down the costs per day to help identify if there were any obscurities with your usage on a particular day.

## Saving and Sharing Views

It's important to be able to see this information, but it's even better to be able to share it. You could create a view for certain departements and then provide these views and graphs via the **Share** option, you can share a link to your customised view to allow others to pin it or you can share a copy of it to allow them to fine tune your view even further.

## Exporting data

Furthering from sharing views you can export data out to certain indviduals, this could be useful when it comes to exporting information out to finances or directors to show the daily spending etc. By default you can select to export a Daily export of month-to-date costs, Weekly exports of costs for the last 7 days or a custom range.

## Budgets and Alerts

Budgets can be used in order to cap your spending so that you don't get met with an unexpected bill at the end of the month. This can be helpful to point out if you've left an expensive machine running or forgot to scale down your services. Budgets can be created on a range of scopes; management groups, subscriptions, resource groups etc. 

To create a budget simply navigate to **Budgets** from within **Cost Management** and select '+ Add'. From here select the scope that you want to create the budget for, provide a name and a period for the budget to apply for, e.g. monthly. Finally, selecting an amount to configure the budget for.

![budgetfirst](/assets/images/Az104UsersAndGroups/budgetfirst.png)

On the next screen you can create alerts to automatically trigger when a certain % of the budget is hit. You can chose to use a custom action group if you have that pre-configured or simply just provide email recipients to receive a notification when the budget is triggered. 

![budgetsecond](/assets/images/Az104UsersAndGroups/budgetsecond.png)

# Summary

This concludes a really high level overview of the various management tasks that can be carried out on your Azure resources in order to gain some organisation and control over resources and items within your subscription(s). I have definietly not covered the complete ins and outs of the above mentioned services and offerings and would recommend that you review Microsofts docs for the related services. 

Thanks for reading, and please if you spot any mistakes then please let me know!