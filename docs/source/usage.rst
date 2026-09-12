开始使用
========

这份文档面向普通用户、前端开发者以及需要调用 MC Server Web API 的客户端开发者。

服务地址
--------

本文档示例默认后端地址为：

``http://localhost:8080``

如果服务部署在其他服务器，请把示例中的地址替换为实际地址。

认证方式
--------

注册或登录成功后，服务器会返回一个 JWT Token。

访问需要登录的接口时，在 HTTP 请求头加入：

.. code-block:: text

   Authorization: Bearer <token>

例如：

.. code-block:: console

   curl http://localhost:8080/auth/me \
     -H "Authorization: Bearer <token>"

Token 失效后需要重新登录获取新的 Token。

注册账号
--------

使用 ``POST /auth/register`` 注册账号。

请求示例：

.. code-block:: bash

   curl -X POST http://localhost:8080/auth/register \
     -H "Content-Type: application/json" \
     -d '{"username":"steve","displayName":"Steve","password":"123456"}'

注册信息包括：

* ``username``：用户名，最多 50 个字符。
* ``displayName``：显示名称，最多 100 个字符。
* ``password``：密码，长度为 6～72 个字符。

注册成功后会直接返回 Token 和用户信息。

登录
----

使用 ``POST /auth/login`` 登录。

.. code-block:: bash

   curl -X POST http://localhost:8080/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username":"steve","password":"123456"}'

成功后响应中包含：

* ``token``：后续请求使用的 JWT。
* ``user``：当前用户的公开信息。

获取当前用户
------------

使用 ``GET /auth/me`` 获取当前登录用户的信息。

.. code-block:: bash

   curl http://localhost:8080/auth/me \
     -H "Authorization: Bearer <token>"

服务器列表
----------

查看服务器列表不需要登录。

使用 ``GET /server/list/{page}`` 获取分页列表，页码从 ``0`` 开始，每页默认 10 条。

例如查看第一页：

.. code-block:: console

   curl http://localhost:8080/server/list/0

获取指定服务器详情：

.. code-block:: console

   curl http://localhost:8080/server/list/servers/1

添加服务器
----------

只有登录用户可以添加服务器。

使用 ``POST /server/list/create``，请求体主要包括：

* ``name``：服务器名称。
* ``description``：服务器简介，可选。
* ``host``：Minecraft 服务器地址。
* ``port``：Minecraft 服务器端口。
* ``mods``：联机方式说明，可选。
* ``code``：第三方内网联机码，可选。
* ``serverFlag``：在线状态检测方式标记。

示例：

.. code-block:: bash

   curl -X POST http://localhost:8080/server/list/create \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <token>" \
     -d '{"name":"Hypixel","host":"mc.hypixel.net","port":25565,"serverFlag":true}'

注意，服务器所有者由后端根据当前登录用户确定，客户端不应该依赖或修改 ``ownerId``。

在线状态
--------

项目目前支持两种在线状态判定方式。

主动探测
~~~~~~~~

当服务器使用主动探测模式时，后端会使用 MCPing 对 Minecraft 服务器地址和端口进行探测，并根据探测结果判断服务器是否在线。

心跳模式
~~~~~~~~

另一种模式由客户端 Agent 定期向：

``POST /room/agent/heartbeat``

发送心跳。Agent 上报的数据包含房间 ID、Agent 版本、Minecraft 状态、虚拟网络信息以及时间戳等字段。

用户搜索
--------

登录后可以使用：

``GET /users?keyword=xxx``

按照用户名或显示名进行模糊搜索。

搜索结果不会包含当前用户本人，也不会返回被禁用的账户。

好友功能
--------

登录后可以使用好友相关接口：

* ``GET /friends``：获取好友及好友请求列表。
* ``POST /friends/{userId}/request``：向用户发送好友请求。
* ``POST /friends/{friendshipId}/accept``：接受好友请求。
* ``DELETE /friends/{friendshipId}``：删除好友关系。

退出登录
--------

使用 ``POST /auth/logout`` 退出登录：

.. code-block:: bash

   curl -X POST http://localhost:8080/auth/logout \
     -H "Authorization: Bearer <token>"

服务端会将当前 JWT 的 ID 加入注销列表，使令牌立即失效。

下一步
------

如果你需要查看完整的 HTTP 接口、请求方法和权限要求，请继续阅读 :doc:`api`。
