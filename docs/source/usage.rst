开始使用
========

这份文档面向普通用户、前端开发者以及需要调用 MC Server Web API 的客户端开发者。

服务地址
--------

本文档示例默认后端地址为：

.. code-block:: text

   http://localhost:8080

如果服务部署在其他服务器，请把示例中的地址替换为实际地址。

第一次调用 API
--------------

推荐按照下面的顺序开始：

#. 调用注册接口创建账号。
#. 调用登录接口获取 JWT Token。
#. 在需要登录的接口中携带 ``Authorization`` 请求头。
#. 调用服务器、用户或好友相关接口。

认证方式
--------

注册或登录成功后，服务器会返回 JWT Token。

访问需要登录的接口时，在 HTTP 请求头加入：

.. code-block:: text

   Authorization: Bearer <token>

例如：

.. code-block:: console

   curl http://localhost:8080/auth/me \
     -H "Authorization: Bearer <token>"

当前后端默认 Token 有效期为 1440 分钟（24 小时）。Token 失效后需要重新登录获取新的 Token。

注册账号
--------

使用 ``POST /auth/register`` 注册账号。

.. code-block:: bash

   curl -X POST http://localhost:8080/auth/register \
     -H "Content-Type: application/json" \
     -d '{"username":"steve","displayName":"Steve","password":"123456"}'

注册信息包括：

* ``username``：用户名，最多 50 个字符。
* ``displayName``：显示名称，最多 100 个字符。
* ``password``：密码，长度为 6～72 个字符。

注册成功后会直接返回 Token 和用户信息，因此通常不需要再次登录。

登录
----

使用 ``POST /auth/login`` 登录：

.. code-block:: bash

   curl -X POST http://localhost:8080/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username":"steve","password":"123456"}'

成功响应中包含：

* ``token``：后续请求使用的 JWT。
* ``user``：当前用户的公开信息。

获取当前用户
------------

使用 ``GET /auth/me`` 获取当前登录用户的信息：

.. code-block:: bash

   curl http://localhost:8080/auth/me \
     -H "Authorization: Bearer <token>"

服务器列表
----------

查看服务器列表和服务器详情不需要登录。

使用 ``GET /server/list/{page}`` 获取分页列表，页码从 ``0`` 开始，每页默认 10 条：

.. code-block:: console

   curl http://localhost:8080/server/list/0

获取指定服务器详情：

.. code-block:: console

   curl http://localhost:8080/server/list/servers/1

.. note::

   当前列表接口不是“只返回在线服务器”。离线服务器也可能出现在列表中；后端会根据当前实现对服务器进行排序。

添加服务器
----------

只有登录用户可以添加服务器。

使用 ``POST /server/list/create``，请求体主要包括：

.. code-block:: json

   {
     "name": "My Minecraft Server",
     "description": "Minecraft Server",
     "mods": "Java",
     "host": "127.0.0.1",
     "port": 25565,
     "code": null,
     "serverFlag": true
   }

服务器所有者由后端根据当前登录用户确定，客户端不应该依赖或修改 ``ownerId``。

在线状态
--------

项目目前存在两种状态检测机制：

主动探测
~~~~~~~~

后端定时使用 Minecraft Ping 探测启用了相应检测标记的服务器，并根据连接结果更新 ``active`` 状态。

当前定时任务的执行间隔为 10 分钟，因此数据库中的状态可能存在延迟。

Agent 心跳
~~~~~~~~~~

服务器所在环境可以运行 Agent，并向：

.. code-block:: text

   POST /room/agent/heartbeat

发送心跳。该接口当前为公开接口，不要求 JWT。

Agent 可以上报房间 ID、Agent 版本、Minecraft 状态、虚拟网络信息以及时间戳等字段。当前后端收到心跳后的核心操作是根据 ``roomId`` 找到服务器并更新 ``updatedAt``。

更多细节请参阅 :doc:`status`。

用户搜索
--------

登录后可以使用：

.. code-block:: text

   GET /users?keyword=xxx

按照用户名或显示名搜索用户。

搜索结果不会包含当前用户本人，也不会返回被禁用的账户。

好友功能
--------

登录后可以使用好友相关接口：

* ``GET /friends``：获取好友关系。
* ``POST /friends/{userId}/request``：向用户发送好友请求。
* ``POST /friends/{friendshipId}/accept``：接受好友请求。
* ``DELETE /friends/{friendshipId}``：删除好友关系。

只有好友请求的接收方可以接受待处理的请求。

退出登录
--------

使用 ``POST /auth/logout`` 退出登录：

.. code-block:: bash

   curl -X POST http://localhost:8080/auth/logout \
     -H "Authorization: Bearer <token>"

服务端会将当前 JWT 的 ``jti`` 加入内存中的撤销集合，使该 Token 在当前服务进程中失效。

下一步
------

如果你需要查看完整的 HTTP 接口、请求方法和权限要求，请继续阅读 :doc:`api`。

如果你正在开发服务器 Agent 或需要理解在线状态判定，请阅读 :doc:`status`。

常见问题可以查看 :doc:`faq`。
