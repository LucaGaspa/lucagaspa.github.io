---
title: Thoughts on designing localized error messages beetween Ruby on Rails and clients
notetype: feed
date: 14-01-2022
---

## Intro

Ruby on Rails defines its way to handle errors and localize them. This works perfectly with a monolithic approach, where views and error strings cohexist in the same repository. Is this the best way to handle errors with microservices on different repositories and in particular using Ruby on Rails with a REST API approach?
Comments are appreciated: refer to this [issue](https://github.com/LucaGaspa/lucagaspa.github.io/issues/1) to discuss how you approach this problem.

## Thoughts

REST APIs should respond to anyone that want to interact with them, regardless who it is. Errors should follow the same approach: anyone should be able to understand the error. So, introducing localization concepts brings some complexities that maybe we do not want to handle. Is it worth to require an `Accept-Language` header in each route to return correctly translated error messages?
Server-side, defining just error codes could be enough. The documentation will let the client be able to formulate the correct human readable error message they prefer. This way the server stays light an cleaner.

## What's the catch?

Considering REST APIs this seems fair. But what about products where backend and multiple frontends must cooperate, possibly without useless code repetitions? If we keep the server light, we have to move the error message logic somewhere else. Assume we have a website and a mobile app, we have at least two options:

- Define the translations dictionary both in the website and in the mobile app
- Define the translations dictionary once and storing it into a CDN

The former solution has lot of pitfalls, mostly in the project management, since we must spend a lot of effort in keeping the code bases equal. The latter is better, but involves more technical skills. We have to keep the translations up to date when we run the client and design carefully when translations are downloaded and what happens in case of errors (i.e. network errors while downloading).

## Conclusions

In my experience, errors have been handled server side. It works, but it's not the best option in my opinion. Starting a new project, I think I'll go with the "repeated code" option, until the team grows in number. Once there are more than 2 people involved in the development, I would probably set a refactor task with high priority. On scaled projects the last option is the best. Each team can be focused on its tasks, without interfere one another (i.e. with copy modifications tasks).
