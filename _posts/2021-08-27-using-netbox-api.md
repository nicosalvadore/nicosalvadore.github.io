---
layout: post
title:  "First steps with NetBox Python API"
date:   2021-08-27 22:00:00 +0100
tags: [automation, python, networking]
excerpt_separator: <!--more-->
---

I often browse the NetBox github repo looking for next features and whatnot, just because I'm naturally interested about this project. And I found myself reading this [discussion post](https://github.com/netbox-community/netbox/discussions/7023).

Basically, this person is storing dates in a field in the device object, and would like to be notified when we're approaching this deadline.
You never know when you'll need that support contract, but usually it's just after you've let it expired...

I'd never used the NetBox API until now, it was then a great excuse to start poking around it.
<!--more-->

##  What is NetBox ?
[NetBox](https://netbox.readthedocs.io/) is a web application to document your network infrastructure, so basically an IPAM + DCIM.
In addition, it's heavily focused on providing all the needed tools to begin your journey into network automation. It has strong community backing behind it, and really is a great product to use. You'll find everything about it in the docs linked above and on the [github repo](https://github.com/netbox-community/netbox/).

## Python and NetBox
As you'll see in the discussion, I've suggested to use a python script and NetBox API to retrieve the devices, then check the date stored in the object. Which could then give us the possibility to notify whatever endpoint you'd need (email, http, sms, whatever you need).

[Here is the first version of my script](https://gist.github.com/nicosalvadore/2b87fc74ea0b17ec6d642ca405048000).
As you'll see, it really is just an example of how simple automation can be at first. Are you tired of checking the expiration dates of all of your licensed devices every month, AUTOMATE IT !

I didn't implement to notification part because it can be very specific to your environment. You could send notification via email using pure SMTP, or via a 3rd party library like Sendgrid's, but also via a webhook to MS Teams or Slack, the possibilities are endless.

As always, my devs skills are what they are, please do not hesitate to suggest improvements should you find any.

## Used packages
I've used a few python packages in my script, here there are :
- *pynetbox* : a wrapper around NetBox REST API, it really speeds up the development process.
- *requests* : the standard library for sending HTTP requests, not mandatory in this case. You'll need if you're using a self-signed certificate on your NetBox instance.
- *datetime* & *dateutil* : two library to work with dates and time.

As these libraries are not packaged in the Python3 default binary, you'll need to download them on your system. Use [pip3](https://pip.pypa.io/) to handle your dependencies.