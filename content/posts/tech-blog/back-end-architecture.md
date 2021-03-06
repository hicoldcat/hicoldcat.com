---
title: 后端网络架构
description: null
author: 李留白
weight: 0
date: 2022-06-26T03:47:38.267Z
lastmod: 2022-07-01T16:47:00.676Z
tags: []
categories:
  - 技术分享
featuredImage: https://hicoldcat.oss-cn-hangzhou.aliyuncs.com/img/gabriel-heinzer-g5jpH62pwes-unsplash.jpg
---

> 原文：https://www.codecademy.com/article/back-end-architecture <br/>
> 翻译：李留白

本文概述了服务器、数据库、路由，以及在客户端发出请求和收到响应之间发生的任何其他事情。

软件工程师似乎总是在讨论他们应用程序的前端和后端。但这到底是什么意思？

前端是在客户端执行的代码。这些代码（通常是HTML、CSS和JavaScript）在用户的浏览器中运行并创建用户界面。

后端是在服务器上运行的代码，它接收来自客户端的请求，并包含将适当的数据送回给客户端的逻辑。后端还包括数据库，它将持久地存储应用程序的所有数据。本文重点介绍服务器端的硬件和软件，使之成为可能。

如果你想复习一下这些话题，可以回顾一下[HTTP](https://www.codecademy.com/articles/http-requests)和[REST](https://www.codecademy.com/articles/what-is-rest)。这些是为客户和服务器之间的请求-响应循环提供结构的主要约定。

让我们先回顾一下客户端和服务器的关系，然后我们就可以开始把所有的碎片放在一起了

### 什么是客户端？

客户端是向后端发送请求的任何东西。它们通常是浏览器，为HTML和JavaScript代码提出请求，它们将执行这些代码来向终端用户显示网站。然而，有许多不同类型的客户端：它们可能是一个移动应用程序，一个运行在其他服务器上的应用程序，甚至是一个支持网络的智能设备。

### 什么是后端？

后端是处理传入的请求并生成和发送响应给客户端所需的所有技术。这通常包括三个主要部分。

- 服务器。这是接收请求的计算机。
- 应用程序。这是运行在服务器上的应用程序，它听从请求，从数据库中检索信息，并发送响应。
- 数据库。数据库是用来组织和保存数据的。

### 什么是服务器？

服务器只是一台听从传入请求的计算机。尽管有一些机器是为这一特定目的而制造和优化的，但任何连接到网络的计算机都可以充当服务器。事实上，在开发应用程序时，你经常使用你自己的计算机作为服务器。

### 应用程序的核心功能是什么？

服务器运行一个应用程序，其中包含如何根据[HTTP verb](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)和 [Uniform Resource Identifier (URI)](https://developer.mozilla.org/en-US/docs/Glossary/URI).来响应各种请求的逻辑。HTTP verb和URI的配对被称为路由，根据请求对它们进行匹配被称为路由。

这些处理函数中的一些将是中间件。在这里，中间件是指在服务器接收请求和发送响应之间执行的任何代码。这些中间件函数可能会修改请求对象，查询数据库，或以其他方式处理收到的请求。中间件函数通常通过将控制权传递给下一个中间件函数来结束，而不是通过发送一个响应。

最终，一个中间件函数将被调用，通过向客户端发送一个HTTP响应来结束请求-响应循环。

通常，程序员会使用Express或Ruby on Rails等框架来简化路由的逻辑。现在，只要想一想，每个路由可以有一个或多个处理函数，每当对该路由的请求（HTTP verb和URI）被匹配时就会被执行。

### 服务器可以发送哪些类型的响应？

服务器发回的数据可以有不同的形式。例如，服务器可能会提供一个HTML文件，以JSON形式发送数据，或者它可能只发回一个[HTTP状态代码](HTTP status code)。当你试图导航到一个不存在的URI时，你可能已经看到了状态代码 "404 - Not Found"，但还有许多状态代码表明服务器收到请求时发生了什么。

### 什么是数据库，以及为什么我们需要使用它们？

数据库通常用在网络应用程序的后端。这些数据库提供了一个接口，以持久的方式将数据保存在内存中。将数据存储在数据库中，既可以减少服务器CPU主内存的负载，又可以在服务器崩溃或断电时检索到数据。

许多发送到服务器的请求可能需要进行数据库查询。一个客户可能会请求存储在数据库中的信息，或者一个客户可能会在提交请求时提交数据，以添加到数据库中。

### 什么是网络API？

API是一个明确定义的不同软件组件之间的通信方法的集合。

更具体地说，Web API是由后端创建的界面：端点的集合和这些端点暴露的资源。

一个网络API的定义是它可以处理的请求类型，这是由它定义的路由决定的，以及客户在击中这些路由后可以期望收到的响应类型。

一个Web API可以用来为不同的前端提供数据。由于Web API可以提供数据而不真正指定数据的查看方式，因此可以创建多个不同的HTML页面或移动应用程序来查看来自Web API的数据。

### 请求-响应周期的其他原则。

- 服务器通常不能在没有请求的情况下启动响应!
- 每个请求都需要一个响应，即使它只是一个404状态代码，表示没有找到内容。否则你的客户端将被挂起（无限期地等待）。
- 服务器不应该为每个请求发送一个以上的响应。这将在你的代码中引发错误。

### 描述一个请求

让我们把这一切变得更具体一些，以一个客户向服务器发出请求时发生的主要步骤为例。

1.Alice 正在SuperCoolShop.com上购物。她点击了一张她的智能手机的封面图片，这个点击事件向http://www.SuperCoolShop.com/products/66432，发出了一个GET请求。

记住，GET描述了请求的种类（客户只是要求提供数据，而不是改变什么）。URI（统一资源标识符）/products/66432指定客户正在寻找关于一个产品的更多信息，而这个产品的ID是66432。

SuperCoolShop有大量的产品，以及许多不同的类别来过滤它们，所以实际的URI会比这更复杂。但这是请求和资源标识符工作的一般原则。

2.Alice 的请求穿过互联网到达SuperCoolShop的一个服务器。这是整个过程中较慢的一个步骤，因为请求的速度不能超过光速，而且它可能有很长的路程要走。由于这个原因，用户遍布世界各地的大型网站会有许多不同的服务器，他们会将用户引向离他们最近的服务器

3.正在积极监听所有用户的请求的服务器收到了Alice的请求!

4.匹配这个请求的事件监听器（HTTP动词：GET，URI：/products/66432）被触发。在请求和响应之间，在服务器上运行的代码被称为中间件。

5.在处理请求时，服务器代码会进行数据库查询，以获得关于这个智能手机案例的更多信息。该数据库包含了Alice想知道的关于这个智能手机外壳的所有其他信息：产品的名称、产品的价格、一些产品评论，以及一个提供产品图片路径的字符串。

6.数据库查询被执行，数据库将请求的数据发回服务器。值得注意的是，数据库查询是这个过程中比较慢的步骤之一。从静态内存中读和写是相当慢的，而且数据库可能是在与原始服务器不同的机器上。这个查询本身可能要穿过互联网

7.服务器从数据库中收到了它所需要的数据，现在它已经准备好构建并向客户发送其响应。这个响应体包含了浏览器所需要的所有信息，以向Alice展示她所感兴趣的手机壳的更多细节（价格、评论、尺寸等）。响应头将包含一个HTTP状态代码200，表示请求已经成功。

8.响应会穿越互联网，回到Alice的电脑。

9.Alice的浏览器收到响应，并使用这些信息来创建和呈现Alice最终看到的视图



