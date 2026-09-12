MC Server Web 用户文档
=======================

欢迎使用 **MC Server Web**。

MC Server Web 是一个面向 Minecraft 服务器的在线列表与房间管理服务。它提供账号系统、服务器列表、服务器状态检测、用户搜索和好友关系等 HTTP API，并支持 Agent 通过心跳上报服务器状态。

后端项目使用 Kotlin + Spring Boot + JPA + MySQL 构建。

项目地址
--------

后端源代码：

`CVKNO80098/McServerWebByKotlin <https://github.com/CVKNO80098/McServerWebByKotlin>`_

文档仓库：

`CVKNO80098/McServersDoc <https://github.com/CVKNO80098/McServersDoc>`_

快速开始
--------

如果你第一次使用 MC Server Web，请先阅读 :doc:`usage`，其中包含注册、登录、认证以及常用 API 的基本使用方式。

如果你正在开发 Web 前端、桌面客户端、Minecraft Agent 或其他第三方程序，请直接阅读 :doc:`api`。

核心概念
--------

* **用户**：通过账号登录 API，并拥有自己创建的服务器。
* **服务器**：Minecraft 服务器在 MC Server Web 中的登记信息，包括名称、地址、端口和在线状态等。
* **主动探测**：后端主动通过 Minecraft Ping 检测服务器是否可以访问。
* **Agent 心跳**：服务器所在环境运行 Agent，由 Agent 定期向后端上报状态。
* **好友关系**：用户之间可以发送、接受和删除好友关系。

.. note::

   本文档描述的是当前后端代码的实际行为。如果后端接口发生变化，应以代码和最新文档为准。

目录
----

.. toctree::
   :maxdepth: 2
   :caption: 文档

   usage
   api
   concepts
   faq
