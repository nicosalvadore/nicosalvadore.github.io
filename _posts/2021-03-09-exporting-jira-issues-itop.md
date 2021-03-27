---
layout: post
title:  "Exporting Jira issues to iTop ITSM"
date:   2021-03-19 18:00:00 +0100
tags: [python, automation]
excerpt_separator: <!--more-->
---
# Exporting Jira issues to iTop ITSM

## The goal
What if you have a backlog / history of hundreds or thousands of issues stored in your Jira server (Cloud SaaS or on-prem) and you'd like to export them ? You don't have a lot of choices by default : Word or PDF export, or a CSV.

Only the CSV can easily be processed with code, and even then you don't have access to comments that were added in the issues, nor attachments (images, text files, pcap, etc...).

That was my dilemma, I needed to export our 700+ issues from a [SaaS Atlassian Jira](https://www.atlassian.com/fr/software/jira) instance to import them back to a new on-prem ITSM : [iTop](https://www.itophub.io/page/about-itop).

*Thank you Python and REST APIs !*
<!--more-->

The code is available in [this github repo](https://github.com/nicosalvadore/jira2itop-tickets) for those who don't like to read. :)

## The plan
So I've decided to write a short python script that :
1. Connect to Jira and loop through my *Projects* to gather all issues and the needed attribues :
    - title
    - description
    - all comments
    - all attachments
    - creation & closing date
2. Connect to iTop to create new tickets with the data gathered on Jira.

## The research
### Jira
Mandatory RTFM first: [docs here.](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/)

We have to create a token to be able to call the API, so open your Atlassian account > Security > API tokens, [direct link.](https://id.atlassian.com/manage-profile/security/api-tokens)
Store the key in your password manager as you won't be able to get it another time.

So the API is pretty easy to use, and well documented. But what if you'd prefer not to mess with the `requests` python3 module. Do not fear, as a `jira` python3 module is available and works flawlessly too ! Here are the [docs.](https://jira.readthedocs.io/en/master/examples.html)

Well, just import the `jira` module at the top of your script, create a new object with your server properties, and off you go ! Don't forget install the module first with `pip3 install jira`.

```python
from jira import JIRA


user = 'jira-admin@example.com'
apikey = 'jira-api-secret'
server = 'https://customername.atlassian.net'

options = {
 'server': server
}

jira = JIRA(options, basic_auth=(user, apikey))
my_issue = jira.issue("DEVEL-42")

print(my_issue.fields.description)
```

The nice thing about this `jira` module is that you are given an object back. So there's no need to parse a json array and use brackets to get whatever value you need : `json.loads(response)["identifier"]` -> `reponse.identifier`. Pretty cool !

### iTop
What about iTop ? Docs [here.](https://www.itophub.io/wiki/page?id=2_7_0%3Aadvancedtopics%3Arest_json)

It's a less "modern" API, but powerful enough for our needs. Each request is waiting for the object/resource impacted, and a json array with the attributes we want to add/update/delete.
You'll find an example of how to use this API [here.](https://www.itophub.io/wiki/page?id=2_7_0%3Aadvancedtopics%3Acreate_ticket#using_python)

As with Jira, we need to add a `REST Services` profile on our iTop user to allow these kind of connections. Simply edit a user and add this profile.

This time, we need the python3 `requests` module to call the API. I've written 4 shorts functions to ease this process. These 4 functions are in a separated file that is imported in `main.py`.
- `create()` -> adds a UserRequest resource.
- `update()` -> updates a UserRequest resource.
- `attachment()` -> adds an attachment to a UserRequest resource.
- `request()` -> sends the HTTP call to iTop.

The first 3 functions are in fact building the json arrays, which is a bit different for each action, and then they all call the `request()` function. Code is [here.](https://github.com/nicosalvadore/jira2itop-tickets/blob/main/itop.py)

## The execution
Here is again a link to the [github repo.](https://github.com/nicosalvadore/jira2itop-tickets)
The code is commented so you should be able to read it quite easily.

---
Disclaimer :
- My dev skills are just above ground level, please accept my poor code quality ;)
- There is no error checking on the HTTP/Jira API responses. You might need to tweak the code first.
- Do not blame me if you delete all you Jira / iTop data :D