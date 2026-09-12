MC Server Web 用户文档
=======================

欢迎使用 MC Server Web。

这是一个 Minecraft 服务器列表服务，用户可以通过 Web API 注册账号、登录、浏览服务器列表、添加服务器，并使用好友与用户搜索功能。

后端项目基于 Kotlin + Spring Boot + JPA + MySQL 构建。

项目地址
--------

后端源代码：

`CVKNO80098/McServerWebByKotlin <https://github.com/CVKNO80098/McServerWebByKotlin>`_

快速开始
--------

如果你只是想使用接口，可以先阅读 :doc:`usage`。

如果你正在开发前端或其他客户端，建议直接查看 :doc:`api`。

.. note::

   本文档面向 API 使用者，而不是后端开发者。文档中的示例默认后端运行在 ``http://localhost:8080``。

目录
----

.. toctree::
   :maxdepth: 2
   :caption: 用户文档

   usage
   api
