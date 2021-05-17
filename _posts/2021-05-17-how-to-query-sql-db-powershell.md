---
layout: post
title:  "How to query your SQL databases from powershell"
date:   2021-05-17 18:00:00 +0100
tags: [systems, automation, powershell]
excerpt_separator: <!--more-->
---

##  A quick and easy way !

You need to read, insert or update rows in a database, and it has to be done in Powershell ?
I have encountered this need too recently. And I was scratching my head to produce a clean and simple script.

I found the perfect solution, a Powershell module that offers an abstraction to directly query your databases : **SimplySql**. <!--more-->
- [PowershellGallery page](https://www.powershellgallery.com/packages/SimplySql)
- [GitHub repo](https://github.com/mithrandyr/SimplySql)

This modules supports several DB engines, such as MSSQL, Oracle, MySql, SQLite, PostgreSQL. And can even run on a linux distro if you've installed powershell :)

## Show me how !

Here is how to do a SELECT with this module. We first have to open the connection, then make our request. Don't forget to close the connection at the end.

    #Requires -Modules SimplySql

    $user = "myUsername"
    $password = ConvertTo-SecureString "mySecret"  -AsPlainText -Force

    $creds = New-Object System.Management.Automation.PSCredential($user, $password)

    Open-MySqlConnection -Server db-01.example.net -Database mydatabase -Credential $creds
    $planets = Invoke-SqlQuery -Query "select * from planets where type = 'terrestrial'"

    $planets | % {$_.name}

    Close-SqlConnection

The output would be :

    Mercury
    Venus
    Earth
    Mars

You can of course use other SQL statements, such as `INSERT` and `UPDATE`.
You'll find everything you need in the [docs](https://github.com/mithrandyr/SimplySql/wiki) available on the github repo.
