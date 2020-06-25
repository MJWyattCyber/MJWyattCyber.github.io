---
header:
    teaser: /assets/images/MsSurface.jpg
title: "Az-104 Identities and Goverence Part 2: RBAC"
date: 2020-06-25 22:10
categories: [Azure]
layout: single
toc: true
toc_title: Table of Contents
---

This is the second part of Users and Groups within Identity and Management for Azure, I am loosley following along with the Az104 syllabus whilst creating these articles as well as making reference to the Microsoft Learn documentation. Disclaimer: This will not prepare you for the Az104 Exam! This is just a snippet intended to share the knowledge as well as aid myself in preparing for the exam.

Azure RBAC (Role Based Access Control) ultimately allows administrators to control who has access to certain Azure resources, and what level of access is provided. These roles can be applied to groups of users of outlined in part 1 of this series, it can also be applied to applications or service principals. This will be applied at a particular scope such as a resource group, subscription or whole tenant. This fine-grained approach allows administrators to appropriately set the right level of permissions to the right resources. 

## Built-in roles

RBAC has hundreds of built-in roles that are constantly being updated and growing, it’s strongly recommended that where possible these roles are used, however, there is the option to create your own which I will cover later. The three **main** roles that are important to learn are:

-	**Owner:** Has full access to all resources, and able to elevate others permissions to match
-	**Contributor:** Can create and manage the resources but is NOT able to grant others the same access
-	**Reader:** Can view the resources

These permissions are present for every service, for example you would have the built-in roles of ‘Virtual Machine Contributor which you could apply to a certain subscription within the organisation to give a user or set of users access to create and manage virtual machines in the chosen scope. 

## Getting the right Scope

RBAC roles can be assigned to one of 4 scopes; Management Group, Subscription, Resource Group or Resources. 

### Management Groups

Management groups will help with organisation and managing multiple subscriptions, management groups should be used if there are multiple subscriptions across different areas of the business, this way you can effectively manage access, policies and compliance across all subscriptions

### Subscriptions

Subscriptions act as a security and billing boundary, each subscription can have a different billing setup, so it’s easy to set up subscriptions for each business unit to easily report on how much they are spending.

### Resource Groups

These are containers that hold Azure related resources, every resource in the group will be managed together and have full control on what resources are stored in the same groups.

### Resources

This relates to the actual resources, this can range from a managed disk, to a virtual network scoping here is the least privilege but is sometimes suggested depended on the goal.
 
# Differences between Azure AD Roles and RBAC Roles

Whilst Azure AD does have its own roles, these are typically only related to users, passwords and domains. Azure AD roles consist of:

-	**Global Admin:** Manage admin access in AAD and able to grant other users’ permissions, by default is the account used to sign up for the directory. 
-	**User Admin:** Able to manage users and groups, raise support tickets, monitor health etc. 
-	**Billing Admin:** Able to manage subscriptions, make purchases and monitor service health.

The big takeaway is that Azure RBAC roles are applied to Azure based resources, whilst Azure AD roles are limited to users, groups etc. However, there is one key overlap to be aware of and that is that a Global Admin in Azure AD has the ability to elevate themselves to **User Access Administrator**, this role can then provide access to Azure resources and should be used in the use case that a subscription is lost or someone who managed an azure resource has left and no one else has access. 
## Assigning admin access to a subscription
In order to be able to assign other users admin access to a subscription you must have the following permissions in the ‘actions’ area of the role for the subscription scope.
-	Microsoft.Authorisation/rolesAssignments/write
-	Microsoft.Authorisation/rolesAssignments/delete
By default, the Subscription Owner and User Access Admin roles have these actions. (This will be clarified in the creating an RBAC role later) If using the User Access Admin Role it’s important to revoke access once you are finished. This is performed in the portal by selecting Yes for the ‘Access Management for Azure resources’ option.

### Using PowerShell

The following command can be used to assign the **Owner** role to another user. 

    New-AzRoleAssignment `
        -SignInName “talkie.toaster@exampleaddress.com” `
        -RoleDefinition “Owner” `
        -Scope “/subscriptions/<target subscription ID>”

### Using the CLI

This can also be done using the CLI:

	Az role assignment create \
		--assignee talkie.toaster@exampleaddress.com \
		--role “Owner”
		--subscription <Subscription name or ID>

# Securing resources with RBAC

RBAC is great to increase security, and it does this by allowing a fine granularity of control over what users or groups of users have access to what resources, when you add the ability to do this on a per resource basis you can really lock down the items within the subscription(s). This ensure that you follow the least privilege principle and everyone has just enough access to complete their role. It's important to also keep in mind that this is also used when providing applications permissions to access certain resources in a resource group.

## RBAC In the portal

The easiest way to apply RBAC roles is via the portal and this is done via the 'Access Control (IAM)' option

![accesscontrol](/assets/images/Az104UsersAndGroups/accesscontrol.png)

From this blade you can check access, set role assignments and set deny assignments.

## How does RBAC work?

RBAC three main steps, those being the 'Who', who is getting the access?, 'What' access are they getting? and finally 'where'?

### Who

This is referred to as a securtiy principal, but essentially it just means, which user, group or application

### What

This is the role definition of what can be done, utlimately the role will consist of a list in jSon format containing 'actions' and 'not actions' outling this.

### Where

Finally, the scope. As discussed at the start of this article, the scope decides on where the access is applied. 

# Custom roles

In the event that the built-in roles don't suit your needs then you may have to create a custom role, Microsoft recommend that where possible you utilise the built-in roles as there are a few that cover most tasks. However, if a custom role is the way to go then it's recommended you copy the built-in role that is closest to achieving your intended outcome. As with all things Azure the roles are defined in jSon and made of 'Actions' and 'NotActions' essentially Actions say what you can't do and NotActions say what you can not. So for example, the Contributor role has * in it's 'Actions' settings stating it can do anything, then it has 'NotActions' denoting that it can't:

 - Delete Roles
 - Create Roles
 - Grant User Access Admin
 - Create/Update Blueprints
 - Delete Blueprints

 These combine to provide the permissions, they are then assigned by placing the ID of the chosen scope in the 'AssignableScopes' section.

## Listing exiting roles

 In order to create a custom role it's recommended you review and modify an existing one, this can be done via the CLI or PowerShell, for the CLI the command is:

    az role definition list --name "role name" --output json

In PoerShell this would look like:

    Get-AzRoleDefinition -Name "Role name" | ConvertTo-Json

## Finding the resource provider

In order to add the permissions required you'd need to know the resource provider, let's take virtual machines as an exmaple. To find all the resource providers related to virtual machines we could enter the PowerShell

    Get-AzProviderOperation */virtualMachines/*

You can then pick the required permission and copy and paste that into your 'Actions' or 'NotActions' segment.

## Creating Custom Roles

There are a few steps to creating custom roles, the first and foremost is to make sure that you have defined the permissions and assignable scope for the role in a json file and saved that with the name of the role. Next you're going to want to run the create command which can be done via the CLI or PowerShell, For the CLI:

    az role definition create --role-definition custom-role-name.json

For PowerShell:

    New-AzRoleDefintion -InputFile "custom-role-name.json"

## Updating roles

There may be a time where you need to tweak a custom role, or perhaps you didn't get the permission right first time round... no one blames you... what? we don't... *honestly!*

Using the CLI:

    az role definition update --role-definition "<<path to-json-file>>"

Using PowerShell:

    Set-AzRoleDefinition -InputFile "<<Path-to-json-file>>"

## Listing where the role is assigned

If you wish to track where the role is assigned then you can run the following commands.

CLI:

    az role assignment list --role "role name"

PowerShell: 

    Get-AzRoleAssignment -RoleDefinitionName "Role name"

**Note:** Look how similar this is to listing the role and it's actions itself. It's important to not get these muddled. Definition = Permissions. Assignment = Where it's applied.

You can also track the activity of RBAC via the 'Activity log' service within Azure, you'll have to filter this down to just RBAC if you're working in a busy subscription. Activity log is great.

## Delete Custom Roles

Look, okay. It's not like we didn't warn you that using the built in rules was a good idea... but, you should totally use the built-in roles. If you were stubborn and wanted to have a go yourself, or maybe you no longer have a "need" for a role anymore, then do not fear! You can remove custom roles using Powershell/CLI.

In the CLI:

    az role assignment delete --role "role name"

PowerShell: 

    Remove-AzRoleAssignment -ObjectID <object_id> -RoleDefinitionName "Role name" -Scope /subscriptions/<sub ID>

Oh PowerShell, it's not your fault you're overly complex and awkward, making us find the object ID... Bless.

It's important to point out at this point that the role deletion can also be done via the portal, but first you have to manually remove all the role assignments.

# Summary

That's it really to this one! RBAC is a wonderful thing that really gives admins fine tuning potential to get the level of access for the users just right. Whilst using the built-in rules are recommended sometimes you just have to go crazy and edit your own. Thanks for taking the time to read, if you spot anything out of place or incorrect please let me know!