---
title: "GraphQL and SF4"
date: 2019-07-10T15:24:30-04:00
categories:
  - php
  - graphql
tags:
  - sf4
  - symfony
  - overblog
  - api
---

# Ever heard of GraphQL? 
GraphQL is a query language, fit for the website concept of data. It gives clients the power to ask for exactly what they need and nothing more.
You can check the [graphql](https://graphql.org) website.

Why GraphQL and not REST? I dunno really, I think both have their pros and cons, you can find on medium a lot of stories about benchmarking, how and why teams used graphql etc. 

Last thing about GraphQL is that it works perfectly with node. In the doc you can build a GraphQL relay / server and start creating your api.

Here's what you need to understand about GraphQL :

- Unlike a REST API, the GraphQL API has only one POST endpoint (ex : https://yoursite/graphql ). 
- In GraphQL you can **query** data, or **mutate** (=update) data. These 2 concepts are : Query and Mutation.
- You need to define your schema (as a GraphQL Schema) for all your entities (ex: Post, Comment, Like, User etc).

Let's not dig deep with the theory and let's move to the SF4 implementation.


# GraphQL and Symfony4 : a piece of cake
SF4 is an amazing PHP framework, complying with the PSR-, using the Doctrine ORM and so much cool stuff. I will assume that you already know what are the basics of Symfony.
Now some of you never used a backend symfony Project as an API. By API, I mean that the backend is reachable by any of your frontends (mobile, PWA, React etc.) as a remote API.
Usually, we use TWIG as a front with SF or Encore (<3) for webpack, which is also very nice.

For REST, you may know the fabulous project Api-Platform (with a graphQL extension also). We will use [Overblog](https://github.com/overblog/GraphQLBundle) GraphQL Bundle. It is an amazing project that provides you a very cool graphQL server for Symfony. It uses also the [Webonyx/GraphqlPHP](https://github.com/webonyx/graphql-php), the GraphQL PHP reference today.

I used it for one big project of mine, and I must say THANK YOU GUYS for provinding this Open Source GEM <3. 
Humhum, let's not heat the oven up, and put ourselves to work step by step. 

# Completely working Symfony GraphQL App 
