HTTP API
========

本文档根据 ``McServerWebByKotlin`` 当前代码整理，面向 API 使用者。

基础信息
--------

默认服务地址：``http://localhost:8080``

除特别说明外，需要登录的接口都使用：

.. code-block:: text

   Authorization: Bearer <token>

认证接口
--------

注册
~~~~

.. code-block:: text

   POST /auth/register

权限：公开

请求 JSON：

.. code-block:: json

   {
     "username": "steve",
     "displayName": "Steve",
     "password": "123456"
   }

密码长度要求为 6～72 个字符。注册成功后返回 Token 和用户信息。

登录
~~~~

.. code-block:: text

   POST /auth/login

权限：公开

请求 JSON：

.. code-block:: json

   {
     "username": "steve",
     "password": "123456"
   }

响应包含 ``token`` 和 ``user``。

获取当前用户
~~~~~~~~~~~~

.. code-block:: text

   GET /auth/me

权限：需要登录

返回当前用户的 ID、用户名、显示名称和头像地址。

退出登录
~~~~~~~~

.. code-block:: text

   POST /auth/logout

权限：需要登录

服务器会使当前 JWT 立即失效。

服务器接口
----------

获取服务器列表
~~~~~~~~~~~~~~

.. code-block:: text

   GET /server/list/{page}

权限：公开

``page`` 从 ``0`` 开始，每页默认 10 条。

获取服务器详情
~~~~~~~~~~~~~~

.. code-block:: text

   GET /server/list/servers/{id}

权限：公开

返回指定服务器及其当前在线状态。

创建服务器
~~~~~~~~~~

.. code-block:: text

   POST /server/list/create

权限：需要登录

请求体示例：

.. code-block:: json

   {
     "name": "Hypixel",
     "description": "Minecraft Server",
     "mods": "Java",
     "host": "mc.hypixel.net",
     "port": 25565,
     "code": null,
     "active": true,
     "serverFlag": true
   }

注意：``ownerId`` 不应由客户端指定，后端会根据当前登录用户设置服务器所有者。

用户接口
--------

搜索用户
~~~~~~~~

.. code-block:: text

   GET /users?keyword=xxx

权限：需要登录

``keyword`` 会同时匹配用户名和显示名称。结果不会包含当前用户本人，也不会返回已禁用用户。

好友接口
--------

获取好友列表
~~~~~~~~~~~~

.. code-block:: text

   GET /friends

权限：需要登录

发送好友请求
~~~~~~~~~~~~

.. code-block:: text

   POST /friends/{userId}/request

权限：需要登录

接受好友请求
~~~~~~~~~~~~

.. code-block:: text

   POST /friends/{friendshipId}/accept

权限：需要登录

删除好友关系
~~~~~~~~~~~~

.. code-block:: text

   DELETE /friends/{friendshipId}

权限：需要登录

Agent 心跳
----------

.. code-block:: text

   POST /room/agent/heartbeat

当前安全配置将该接口设置为公开接口，因此 Agent 可以不携带 JWT 发送心跳。

请求 JSON 对应的数据结构包括：

.. code-block:: json

   {
     "roomId": "1",
     "agentVersion": "1.0.0",
     "status": "online",
     "verified": true,
     "verificationLevel": "verified",
     "minecraft": {
       "reachable": true,
       "latencyMs": 20,
       "playersOnline": 3,
       "playersMax": 20,
       "version": "1.21.x"
     },
     "virtualNetworks": [],
     "timestamp": "2026-09-12T10:00:00Z"
   }

健康检查
--------

.. code-block:: text

   GET /

权限：公开

用于确认后端服务是否正常响应。

接口权限总览
------------

+-----------------------------------+--------+----------+
| 方法                              | 路径   | 权限     |
+===================================+========+==========+
| GET                               | ``/``  | 公开     |
+-----------------------------------+--------+----------+
| POST                              | ``/auth/register`` | 公开 |
+-----------------------------------+--------+----------+
| POST                              | ``/auth/login`` | 公开 |
+-----------------------------------+--------+----------+
| POST                              | ``/auth/logout`` | 登录 |
+-----------------------------------+--------+----------+
| GET                               | ``/auth/me`` | 登录 |
+-----------------------------------+--------+----------+
| GET                               | ``/server/list/{page}`` | 公开 |
+-----------------------------------+--------+----------+
| GET                               | ``/server/list/servers/{id}`` | 公开 |
+-----------------------------------+--------+----------+
| POST                              | ``/server/list/create`` | 登录 |
+-----------------------------------+--------+----------+
| GET                               | ``/users`` | 登录 |
+-----------------------------------+--------+----------+
| GET                               | ``/friends`` | 登录 |
+-----------------------------------+--------+----------+
| POST                              | ``/friends/{userId}/request`` | 登录 |
+-----------------------------------+--------+----------+
| POST                              | ``/friends/{friendshipId}/accept`` | 登录 |
+-----------------------------------+--------+----------+
| DELETE                            | ``/friends/{friendshipId}`` | 登录 |
+-----------------------------------+--------+----------+
| POST                              | ``/room/agent/heartbeat`` | 公开 |
+-----------------------------------+--------+----------+
