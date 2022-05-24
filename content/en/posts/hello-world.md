---
title: "Managing multi-cluster secrets with Sealed Secrets"
date: 2022-05-04T18:00:38-03:00
draft: false
---
One of the most hard problems i have found dealing with multiple-cluster in my jobs is dealing with multiple secrets, create secrets for each cluster, being sure that secret works, and store them safely.

Having a proper control of this issue always be a big mess, mostly because this easily become a safety issue 
most of the time in companies i have passed, someone have all the secrets stored in their machine or in some dark and scared place, or all of the secrets start to be stored in one private repository where every one in a point of the time will have the access or other horrible solution like that.

In my researches about developer experience i have found some people talking about one open-source project called <b>Sealed-Secrets</b> and in the moment i looked for it, i instantly fall in love.

the concept is really simple and powerful i an application focused on kubernetes which receive some data and encrypt this data in custom kubernetes resource.

At first you may think "why do i need this, i can only place my secret in my cluster right?"
well yes, but when we start to create custom pipelines to improve our deployment process, create multiple clusters 

**TL;DR**

If you already know how sealed-secrets works and only wants the commands to encrypt your secrets just look this [Github Gist](https://gist.github.com/MSBarbieri/168552946b6f831b5b1887a4eb74b899)

