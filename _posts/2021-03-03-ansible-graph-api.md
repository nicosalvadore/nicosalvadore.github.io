---
layout: post
title:  "Use MS Graph APIs with Ansible"
date:   2021-03-03 15:00:00 +0100
tags: [ansible, systems, automation]
---
# Use MS Graph APIs with Ansible

## What was the use-case ?
I had this need at work to query some data from Azure/Microsoft365 and pass it to another internal department to be used in some kind of billing scenario.

There are many ways to get data from the Cloud (with a capital C) in general. You can of course use the web interface for ponctual stuff... But who wants to login to a web portal, find the right menu and copy/paste the wanted information ? Not me for sure !

When you really put your work in production, developing some kind of automation is king.

For whatever reason, Ansible might be the right choice for you, as it was for me. So follow along :)

## Overview of what has to be done
Ansible will query Azure with the Graph API, and you need to be authenticated to query your 365/Azure tenant. So we will create first an `App registration` into Azure Active Directory.
Then we will create an Ansible playbook that will use the `uri` module to make HTTP requests to Azure and give back the data.

## Azure Active Directory configuration
First thing to do, sign in to the AAD admin center and open the menu `App registration`.

![AAD app registration](/assets/images/20210303_app-registrations-menu.png)

Here you can create a `New registration` with the following parameters :
- `Name` : define a label.
- `Supported account types` : Accounts in this organizational directory only - Single tenant.

Then go into this new app configuration and open the menu `API permissions`. You can here add specific read/write rights to many objects in azure.

To add a right for the Graph API we will use later, follow these steps :
1. Add a permission
1. Microsoft Graph
1. Application persmissions
1. Select the object and the access you want your playbook to use. For exemple : `User.Read.All`.
![Graph API permission](/assets/images/20210303_app-registrations-permissions.png)

Because of the kind of permissions your selected, it might be required to `Grant admin consent for [Company Name]` through the button right next to `Add a permission`.

We need now to configure `Client secret`. For this, go into the `Certificates & secrets` menu and add a new `Client secret`.
You can select the expiry date of your secret according to your security policy. An ID and a value will be then displayed. Store the secret value in your password manager, as you won't be able to gather it again from AAD.

In addition, you should also take note of the following values, they should be displayed on the `Overview` menu page :
- `Application (client) ID`
- `Directory (tenant) ID`

## Ansible playbook
To communicate with Azure via the MS Graph API, we first need to get a token from the MS OAuth endpoint. And then with this token we will be able to request our data.

I'm not an expercienced user of this kind of authentication process, so maybe there is a more efficient method, let me know !
I used [the Graph API official documentation](https://docs.microsoft.com/en-us/graph/auth-v2-service#4-get-an-access-token) to get started.

Below is the first part of the playbook :

    ---
    - name: Using Graph API to request users' data.
      hosts: localhost
      gather_facts: no
      vars_files:
      - vars/main.yml
      - vars/vault.yml

      tasks:

- We will execute the play directly on the machine running Ansible, it could be your workstation, a server running Ansible, or AWX/Tower. That's why the `hosts` is set to localhost.
- I'm skipping facts gathering because we don't want any information from the local machine.
- Then I'm including two files:
    - `main.yml` contains default variables like the Azure tenant ID and the Application ID.
    - `vault.yml` is an encrypted file that stores the secret. I'm using **Ansible Vault** for this. Check it out if you don't know how to store sensitive information in your plays. It's really easy and works great !


Now the first task that is used to get the OAuth token :

    - name: Authenticating to Graph API.
      uri:
        url: "https://login.microsoftonline.com/{{ graphapi_tenantid }}/oauth2/v2.0/token"
        method: POST
        body:
          client_id: "{{ graphapi_clientid }}"
          scope: https://graph.microsoft.com/.default
          grant_type: client_credentials
          client_secret: "{{ graphapi_secret }}"
        return_content: yes
        body_format: form-urlencoded
        headers:
          Content-Type: application/x-www-form-urlencoded
      register: auth


We send a POST request to the OAuth HTTP endpoint including :
- The AAD tenant ID in the URL path
- A body that contains some data that Microsoft expects, including our client ID and the secret.
- We store the response with the `register` keyword

Then comes the task used to gather the initially needed data, in this example we simply ask for the list of users in this Azure/M365 tenant.

    - name: Gathering users from AAD.
      uri:
        url: https://graph.microsoft.com/v1.0/users?$select=userPrincipalName,assignedLicenses
        body_format: form-urlencoded
        headers:
          Authorization: "Bearer {{ auth.json.access_token }}"
      register: response

In this second task we use the `uri` to make an HTTP GET request to MS Graph API. And as you can see in the path, we're asking for the list of users.

You'll find the official documentation for the Azure user resource [here](https://docs.microsoft.com/en-us/graph/api/user-list?view=graph-rest-1.0&tabs=http).

We can select which properties we want to get back with the `$select` keyword in the URL's arguments. In my example we want the UPN and a list of all assigned licenses for each user entry returned.

Do not forget to include the token we got back in the first task in the `Authorization` header.

As a last optional step, let's display the json array returned with the help of the `debug` module :

    - name: Displaying list of AAD users.
      debug:
        msg: "{{ response.json }}"

And here is the array we got as a response :

    {
        "msg": {
            "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(userPrincipalName,assignedLicenses)",
            "value": [
                {
                    "assignedLicenses": [
                        {
                            "disabledPlans": [],
                            "skuId": "e43b5b99-8dfb-405f-9987-dc307f34bcbd"
                        }
                    ],
                    "userPrincipalName": "john.doe@example.com"
                },
                {
                    "assignedLicenses": [
                        {
                            "disabledPlans": [],
                            "skuId": "6fd2c87f-b296-42f0-b197-1e91e994b900"
                        }
                    ],
                    "userPrincipalName": "jane.doe@example.com"
                }
            ]
        }
    }

You could now loop on a json-sub array in a following task to make some modifications on all user entries returned, or whatever you need. Good luck and have fun automating !

Full Ansible play :

    ---
    - name: Using Graph API to request users' data.
      hosts: localhost
      gather_facts: no
      vars_files:
      - vars/main.yml
      - vars/vault.yml

      tasks:

      - name: Authenticating to Graph API.
        uri:
          url: "https://login.microsoftonline.com/{{ graphapi_tenantid }}/oauth2/v2.0/token"
          method: POST
          body:
            client_id: "{{ graphapi_clientid }}"
            scope: https://graph.microsoft.com/.default
            grant_type: client_credentials
            client_secret: "{{ graphapi_secret }}"
          return_content: yes
          body_format: form-urlencoded
          headers:
            Content-Type: application/x-www-form-urlencoded
        register: auth

      - name: Gathering users from AAD.
        uri:
          url: https://graph.microsoft.com/v1.0/users?$select=userPrincipalName,assignedLicenses
          body_format: form-urlencoded
          headers:
            Authorization: "Bearer {{ auth.json.access_token }}"
        register: response

      - name: Displaying list of AAD users.
        debug:
          msg: "{{ response.json }}"
