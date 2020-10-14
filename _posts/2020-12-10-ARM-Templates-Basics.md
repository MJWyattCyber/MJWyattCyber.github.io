---
title: "ARM Templates: Getting Started"
layout: single
date: 2020-10-12 19:57
categories: [Azure]
---

If you've ever worked with Azure then chances are you're familiar with Azure Resource Manager for those that are new to both Azure and automation, ARM Templates is code that defines the infrastructure you wish to deploy. This code will reliably deploy the specified infrastructure in a pre-defined state, allowing administrators and operations teams to deploy the same enviornments and solutions on demand. ARM templates allow you to not only deploy infrastructure but also deploy and tune various other components such as Web Apps and service plans. 

# Pre-Requisites

There are two main ways to create ARM templates, the first is rather simple and involves provisioning the resource, Azure will automatically generate a template and offer the option to export a template. This can then be used to re-provision the resources.

Secondly, and the way that I will cover in this post, is by writing them in a text editor suich as Visual Studio or Visual Studio Code. Both options are free (With visual studio community edition) and both have ways to make writing ARM templates easier. I'll focus on Visual Studio Code as it's what I used personally. To set-up VS: Code to assist there are a few key extensions that will aid in writing these templates, the key one being Azure Resource Manager (ARM) Tools.

For extra features another extension that I can recommend is ARM Template Viewer, this will allow you to physically view your resources within code. This can also act as a sanity check when writing templates to ensure the right resources are being deployed. 

![armviewer](/assets/images/ARM/armviewer.png)

# Formatting 

ARM Templates are written in JavaScript Object Notation (JSON) and consist of 5 core 'components'; Schema, variables, parameters, resources and output.

## Schema

This tells the ARM template what schema to use for the template file and details the properties that are within a template. Commonly the date within the schema will be a bit misleading and not recent, this is fine - it simply means that there have been no major changes to the schema. Schema is required in all ARM templates and is the first thing that should be populated. 

![SchemaScreenshot](/assets/images/ARM/schemascreenshot.png)

## Parameters

Parameters can be used to help make an ARM template reusable. Parameters can be used in place of hard coded resource names, to specify SKUs of a resource and more. When defining a parameter in the ARM Template commonly, you aren't defining the value of the parameter itself but instead setting the rules for the parameter, for example you have to specify a type field but you can also set the min/max lengths or a range of allowed values as an example. If you opt not to use a paramter file to specify the value you will have to specify the values of the parameters when you deploy the template. 

![paramexample](/assets/images/ARM/paramex.png)

## Resources

Resources are the meat of a template and define what you are deploying, this can range from a simple storage account all the way up to full enviornments encompassing many different Azure services. Don't panic! ARM templates are meant to be modular and you can break down sections for clarity and readability, you can still deploy everything at once using linked templates but more on that in a future post. 

Sticking with the simple storage account this will look like the following in an ARM template

![storageaccexample](/assets/images/ARM/storageaccexample.png)

## Variables

Variables can help to simplify templates by enabling you to write an expression such as a concat function with a uniquestring and reuse it over and over throughout without typing it multiple times. Variables really come into their own when the object being deployed needs to be globally unique, this is handy when dealing with resources such as storage accounts or web app names. A variable expression may be seen as below:

![uniquestorage](/assets/images/ARM/uniquestorage.png)

## Outputs

Finally, outputs. Outputs are sometimes often overlooked and aren't essential in an ARM Template they can however, be very useful. An output simply displays an output at the end of the deployment once it's completed, if this was initiated from a command line such as powershell then returning certain items such as Storage URIs or endpoints can be really useful as you'd never need to leave the shell to gather this information.

![outputs](/assets/images/ARM/output.png)

This will return the endpoints as listed above, whilst it looks complex you can if you wish just return a string of "Success".

# Summary

That breifly covers the basics of what makes up an ARM template, I didn't want to dive into too much detail here but more provide a quick glance at ARM templates. I'm still learning new things about them but the best way to learn is to make some, edit some and get stuck in. A really useful github repo is Microsofts own ARM quickstart templates, this will give you a good starting point to see how templates are constructed. The repo can be found [here.](https://github.com/Azure/azure-quickstart-templates)