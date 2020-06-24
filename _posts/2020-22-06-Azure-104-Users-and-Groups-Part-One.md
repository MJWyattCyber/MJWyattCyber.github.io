---
header:
    teaser: /assets/images/MsSurface.jpg
title: "Az-104: Identities and Governance Part 1: Users and Groups"
date: 2020-06-22 19:12
layout: single
toc: true
toc_label: Table of Contents
categories: [Azure]
---

After setting a date to take the Az104 it was time to prepare and study for the exam, I decided that I wanted to continue writing blog posts and providing content whilst also studying, and afterall if you can explain a concept to someone else then you truly know the subject. A result of this is that I aim to write up a series of posts covering the topics outlined in the [syllabus](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4pCWy) or at least some of them. I want to point out that this will by no means prepare you for the exam and is just the main points that I pulled out as a portion of my indivdual studies. 


# Azure Active Directory

First things first it's essential to start at the beginning with the bread and butter of users and groups. When I mention users and groups I primarily mean users and groups within Azure Active Directory or AAD. AAD is the cloud counterpart to Active Directory which is a directory (phone book of sorts) service found in most organisations and historically hosted on-prem. AAD is *not* a cloud replacement, they are designed to go hand in hand and AAD is designed to be an extension to the existing Active Directoy. 

It's important to know that within Azure a subscription is associated to a **single Azure AD Directory**, whilst multiple subscriptions can trust the same directory, a subscription can only trust one directory. The same user identity can spread across multiple directories providing access to multiple subscriptions, for example if you had an indivdual subscription for each Dev/Test/Prod envrionment and as such 3 directories. The same user identity could be used across all 3. Likewise you could have a single directory where the users reside and each subscription trust the same directory. 

# Managing Users

The user accounts that reside within AAD are granted a set of default permissions, this is determined by the type of the user, the users role assignments and whether or not they own indivdual objects. There are three main types of user accounts in AAD with each type having a different level of access to align with the scope of the work that will be carried out. Now of course this can all be fine tuned and **should** be fine tuned to suit.

### Administrators

Commonly administrators are there to control who is allowed to access certain resources and who can do what. Typically speaking there will be only a handful of admins within an AAD instance, it's also recommended to make the admin accounts different from the day to day user account. For example, if Bill was an Administrator he'd have his standard user account and an admin account, with the admin account only used to carry out administrative tasks. 

An example of an adminstrative role that could be assigned out to a user would be the role 'User Administrator', this is a built in role which allows users the ability to admin the users within a directory, you'd be able create, edit or delete users, assign roles to thers, reset passwords etc. These tasks would be possible in the portal, via the Azure CLI or PowerShell. I'll cover some of the commands used later.

### Member Users

Member users are your bog standard user, these users are considered internal users to the organisation and will normally have a user account created for them when they join the organisation. Commonly these users will be synced across from Active Directory via Azure AD connect, however this is outside the scope of this post.

Member users have default permissions to enable them to edit their own profile... and that's about it. These users shouldn't be able to manage any other user as standard. These users will commonly end with the same email address, typically this will be the organisation/work addresses. E.g. Joe.Bloggs@contoso.com You can use custom domains here but ownership of the domain will have to be verified by an administrator typically in the form of adding a TXT record with a specified value. Custom domains can be configured from the option **Custom Domain Names** from within the **Manage** section of Azure Active Directory. 

### Guest Users

Guest users will typically be external users and are commonly referred to as 'invited users' or their accounts are known as B2B'd (Business to Business) accounts. These accounts are used when you wish to collaborate with an external party or a customer to share resources or to publish an application to them. These accounts have restricted access and will use their own work, school or personal account to sign in with. You will send an invitation link to the user's email address from which they will be able to logon and gain access to the resources that are shared with them. 

Be cautious; by default Member users can invite guest users, this can be disabled by an administrator.


## Creating users

In order to create users you will need to be assigned either the **User Administrator** or **Global Administrator** role, with this assigned you will be able to create new users as well as inviting guest users into Azure Active Directory. 

### Using the portal

Using the Azure portal this is a fairly straight forward process;

- Navigate to the service 'Azure Active Directory
- Within the left hand 'Manage' section select 'Users'
- Then you'll be able to select the '+New User' or '+New Guest User' options. 

![newusers](/assets/images/Az104UsersAndGroups/Az104newusers.png)

For member users you will have to provide a few options such as a username (in the form of an email), full name, first and last names. you can also specify the users groups, usage and job title. Job titles can be useful to set for management as you can later define dynamic groups to add all users who meet certain rules to the same group, more on this later. 

![newmemberuser](/assets/images/Az104UsersAndGroups/az104newinternaluser.png)


### Using the CLI

New users can be made using the Azure CLI command, the base command is

    az ad user create

However, there is some key information which needs to be provided when issuing this command:

![CLIUser](/assets/images/Az104UsersAndGroups/azclienewuser.png)

This isn't the full list of what needs to be provided, for example I could've also specified information such as the users full name, job title, location etc. 

### Using Powershell

If using the command line is your thing this can also be accomplished via the use of PowerShell with the Az module installed.

    New-AzureADUser

This method requires a little more in terms of input as you need to define a 'Password profile'.

![psuser](/assets/images/Az104UsersAndGroups/psnewusers.png)

It's recommended that if the command line method is more your thing then you set up key variables and pass these through, there is also bulk tasks that can be completed which I will cover shortly.


## Removing Users

For each constructive action there is an equal destructive one, that comes in the form of removing users, removing users can also be performed by either, the portal, PowerShell or the CLI.

### Using the portal

In the portal, this is as simple as selecting the tick box to the left of the users name and then selecting the 'delete user' option from the action bar at the top.

![portaldelete](/assets/images/Az104UsersAndGroups/deluserportal.png)


### Using the CLI

The CLI command for removing users is very similar to the command for adding them:

    az ad user delete --id <UPN or ObjectID>

![clidelete](/assets/images/Az104UsersAndGroups/deluserclie.png)


### Using PowerShell

Likewise the command to remove users using PowerShell is also very similar to the command used to create, simply changing the verb:

    Remove-AzureADUser

![psdelete](/assets/images/Az104UsersAndGroups/deluserps.png)


### Recovering Users

If you accidently deleted the wrong user or you created a script that deleted a few too many users then do not fear! All deleted users remain suspended for 30 days, during which the account can be restored.

They can be found in the 'deleted users' section from the left hand side.

## Bulk Tasks

Whilst the methods above are fine when it comes to one user, or even a few users there will be times where you have to perform these tasks at scale. In order to do this you can utilise 'Bulk activities'. This allows you to import a CSV file which will be created by yourself containing the list of all the users you wish to affect. When using bulk activities to invite new users a few things to keep in mind would be:

- Naming Conventions: Ensuring that you keep to a naming convention for the accounts, such as firstname.lastname@emailaddress.com
- Passwords: Keep to a passwod convention for the initial password provided to the user. 


# Managing Groups

Groups within Azure AD make it easy to manage permissions to certain resources, by utilising groups you can assign the security permissions to the group itself rather than going one by one and assigning each user access rights. This is simplified further by the introduction of Dynamic Groups which allow you to set rules for group membership such as Job title or Departement.

## Group Types

Within Azure AD there are two main types of groups, Security groups and Office groups.

### Security Groups

These are the most common form of group and will allow members to be given access to a shared resources, for example you could have a group that provided Virtual Machine contributor access and apply the access to all users in the group. Creation of these groups requires the Azure AD Administrator.

### Office 365 Groups

These are groups that allow you to provide access to shared mailboxes, calendars, files or a SharePoint site etc. This option also lets you give those outside your organisation access to the group. By default, this can be carried out by Standard Users and Admins.


## Creating Groups


### Using the Portal

As with most Azure Resources the same options of using the portal, CLI or PowerShell are avilable. With the Portal being the most straight forward and visual, it's simply a case of selecting the '+New Group' option once in **Azure Active Directory** > **Groups** from the Manage section on the left hand side. 

![Newgroupportal](/assets/images/Az104UsersAndGroups/newgroupportal.png)

As you can see there are the common options here such as 'Group type' which is discussed above, and then Group Name, Description and Membership Type which will be covered later.
You can also assign owners or members at this stage, this isn't required you can create an empty group and assign the users later.

### Using the CLI

The command used to create a group in the CLI is:

    az ad group create --display-name "Group Name" --mail-nick "Mail nickname"

![newgroupcli](/assets/images/Az104UsersAndGroups/newgroupcli.png)

As with before there are other operators that can be used here such as adding a description, setting it to be mail enabled etc. 

### Using PowerShell

The command used to create a group via PowerShell is:

    New-AzureADGroup -DisplayName "Group name" -MailEnabled $false/true -SecurityEnabled $true/false -MailNickName "MailNickname"

![newgroupps](/assets/images/Az104UsersAndGroups/newgroupps.png)

There's a little bit more configuration involved when using the command line tools when it comes to creating groups, making it much simpler to use the portal when it comes to a smaller number of groups.

## Assigning members to groups

When it comes to group membership there are two ways in which this can be handled and the manner in which the group is setup will determine how the users are added. The membership type will either be set to 'Assigned' or 'Dynamic User/Device'

### Assigned Membership

Assigned membership is the most common type of membership, and simply put is when an administrator or group owner adds users directly to the group. This is typically done at group creation and via the portal by selecting the '+ Add Members' option from within the group management page. Note, owners can also be added or removed through the same manner.

Doing this via the command line is also possible with the **Azure CLI** syntax being:

    az ad group member add --group "Group Display Name" --member-id "UserPrincipleName"

Likewise, performing this by using the **PowerShell** syntax:

    Add-AzADGroupMember -MemberUserPrincipalName "UPN" -TargetGroupDisplayName "Group Display Name"

It's worth pointing out that there are many different combinations that can be used here, such as using Object ID's instead of displaynames etc. I'd recomend checking out the overview guides to both the CLI and Azure PowerShell which I'll link at the bottom of the post.

### Dynamic Membership

Dynamic membership is essentially automatic and rule based, this option allows administrators to configure rules and queries to define group membership. This can be very useful if you want all employees with the job title of "Developer" to be in a developer group that has access to particular development environment's etc.

These rules are made during group creation and work in a query format when you chose a property, operator and a value. You can set up multiple queries to further fine tune group membership.

![dynamicmembership](/assets/images/Az104UsersAndGroups/newgroupdynamic.png)

## Removing Members

Removing group members is fairly straight forward, this can be done in the portal by going into the group and then selecting **members** from the left hand side menu, select the user(s) that you wish to remove and click **remove**. This is primarily for assigned membership type.

### Using the CLI

The command here mirrors the command for adding a user with one key difference, remove instead of add:

    az ad group member remove --group "Group Display Name" --member-id "UserPrincipleName"

### Using PowerShell

Likewise the command mirrors it's counterpart of addition but using the Remove verb instead:

    Remove-AzADGroupMember -MemberUserPrincipleName "UserPrincipleName" -GroupDisplayName "Group Display Name"

That's pretty much the basics of group management, I'd highly recommend checking out the overviews of both the CLI and PowerShell for all available options in the command syntax.

# Extra Tasks

There are a few other misc tasks that didn't fall under user or group management, I'll cover these here.

## Configuring Self-Service Password Reset

Self-Service Password Reset aims to provide users who aren't signed in but have forgetten their password the ability to reset their own password and thus login, this is intended to reduce help-desk volume and automate a common task within most service desks. 

To enable this service your Azure AD account will need to have a Premium P1 or P2 license, with the ability to enable write-back in P2 license versions. Write back is essentially writing the change back to Active Directory.

### How it works

SSPR is initiated by the user by clicking on the **Can't access your account** link on the sign in page, the portal then goes on to check the following:

1. Localisation: ensuring that the page is displayed in the appropraite language
2. Verification: This is a username and a CAPTCHA check to ensure that the user is not a bot.
3. Authentication: The user is then prompted for the data required to specify a reset, this may be a code emailed to them, app notification or answer security questions.
4. Password Reset: If successful the user will be able to reset their password.
5. Notify: A message is sent to the user to notify them of the change. This can be configured.

There are a few ways that you can customise the page to reassure your users that the page is legitmate, these settings might be adding a company logo to the sign in page. This customisation is done via the portal and selecting **Password Reset** and then **Customisation** within the Azure Active directory service.

### Enabling SSPR

SSPR is pretty simply to enable, simply navigate to **Password Reset** from within the Azure Active Directory blade, you will be given the option to enable this for 'None, Selected, All' users.

![SSPREnable](/assets/images/Az104UsersAndGroups/ssprenable.png)

### Authentication Methods

Once enabled you will have to configure the authentication options available to users, this can range from a Mobile app notification through the Authenticator app, a code provided via Email, Text message sent to a registered device or answering security questions. You can also chose the amount of authentication methods required between 1 or 2 methods are supported.

![AuthMethods](/assets/images/Az104UsersAndGroups/ssprauth.png)

### Notifications

You also have some control over what notifications are sent once SSPR has been completed, the options available are to notify the user of password reset and the option to notify all other admins when other admin accounts reset their passwords.

## Device Identity

Not all devices that our users have access to are owned by our organisation, because of this there will be times that users will be using a range of devices to connect to our resources which we will want to have some control over. We sometimes want the option to add devices to our AAD in order to track how our users are accessing certain assets. There are three main 'Device registration' options:

### Azure AD registered

These are devices that are often referred to as BYOD, and are normally privately owned and the user account on the local device is often a local account or a personal Microsoft Account. This is the least restrictive form as it covers all OS types.

### Azure AD joined

These devices are owned by the organisation and user access to the assets is performed via their work accounts. This option is only available to Windows 10 and Windows Server 2019 devices.

### Hybrid Azure AD Joined

These are similar to Azure AD Joined devices in that the users signed into the devices will belong to our Azure AD and the devices are commonly owned by the organisation, however, this option is better suited when a hybrid cloud model is followed and on-premises and cloud access is required. This option supports Windows 7, 8.1, 10 and Windows server 2008 and later.

## Azure AD Join

Azure AD join lets you join devices to AAD without the need to sync them with any on-premises Active Directory instance. This service is best suited to organisations that are more cloud focused but can be utilised in hybrid settings.

There are some key concepts to keep in mind when scoping Azure AD Join, these are:

- **Identity Infrastructure** 
- **Device Management**
- **Application access**
- **Device Provisioning**

For more information I'd recommend checking out the microsoft learn document [here](https://docs.microsoft.com/en-gb/learn/modules/manage-device-identity-ad-join/3-what-is-ad-join) as it covers Azure AD in depth, whereas this post is just covering a high level overview.

# Summary

That's it for this post, I hope to release more of these looking into other aspects of Azure Users and Groups. Thanks for reading.