HTTP API
========

本文档面向 API 使用者，并按照当前 ``McServerWebByKotlin`` 后端代码整理。

基础信息
--------

默认服务地址：

.. code-block:: text

   http://localhost:8080

除特别说明外，需要登录的接口都使用 JWT Bearer Token：

.. code-block:: text

   Authorization: Bearer <token>

认证接口
--------

注册
~~~~

.. code-block:: http

   POST /auth/register

权限：公开。

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

.. code-block:: http

   POST /auth/login

权限：公开。

请求 JSON：

.. code-block:: json

   {
     "username": "steve",
     "password": "123456"
   }

成功响应包含 ``token`` 和 ``user``。

获取当前用户
~~~~~~~~~~~~

.. code-block:: http

   GET /auth/me

权限：需要登录。

返回当前用户的 ID、用户名、显示名称和头像地址。

退出登录
~~~~~~~~

.. code-block:: http

   POST /auth/logout

权限：需要登录。

后端会将当前 JWT 的 ``jti`` 加入撤销集合，使该 Token 在当前服务进程中失效。

服务器接口
----------

获取服务器列表
~~~~~~~~~~~~~~

.. code-block:: http

   GET /server/list/{page}

权限：公开。

``page`` 从 ``0`` 开始，当前默认每页 10 条。

当前实现是分页获取服务器记录，并根据服务器状态等条件排序，并不是简单地只返回 ``active=true`` 的服务器。

获取服务器详情
~~~~~~~~~~~~~~

.. code-block:: http

   GET /server/list/servers/{id}

权限：公开。

返回指定服务器的信息以及当前实现计算得到的状态信息。

创建服务器
~~~~~~~~~~

.. code-block:: http

   POST /server/list/create

权限：需要登录。

请求体示例：

.. code-block:: json

   {
     "name": "My Minecraft Server",
     "description": "Minecraft Server",
     "mods": "Java",
     "host": "127.0.0.1",
     "port": 25565,
     "code": null,
     "active": true,
     "serverFlag": true
   }

服务器所有者由后端根据当前登录用户确定，客户端不需要设置 ``ownerId``。

.. note::

   客户端提交的服务器状态字段不应被理解为最终可信的在线状态。后端还会通过 Minecraft Ping 等机制进行状态检测。

用户接口
--------

搜索用户
~~~~~~~~

.. code-block:: http

   GET /users?keyword=xxx

权限：需要登录。

``keyword`` 会同时匹配用户名和显示名称。结果不会包含当前用户本人，也不会返回已禁用用户。

好友接口
--------

获取好友列表
~~~~~~~~~~~~

.. code-block:: http

   GET /friends

权限：需要登录。

发送好友请求
~~~~~~~~~~~~

.. code-block:: http

   POST /friends/{userId}/request

权限：需要登录。

接受好友请求
~~~~~~~~~~~~

.. code-block:: http

   POST /friends/{friendshipId}/accept

权限：需要登录。

只有好友请求的接收方可以接受待处理请求。

删除好友关系
~~~~~~~~~~~~

.. code-block:: http

   DELETE /friends/{friendshipId}

权限：需要登录。

Agent 心跳
----------

.. code-block:: http

   POST /room/agent/heartbeat

权限：公开。

当前接口接收 Agent 上报的数据结构，并根据 ``roomId`` 查找服务器。

请求示例：

.. code-block:: json

   {
     "roomId": "1",
     "agentVersion": "1.0.0",
     "status": "online",
     "verified": true,
     "verificationLevel": 1,
     "minecraft": {
       "reachable": true,
       "latencyMs": 20,
       "playersOnline": 3,
       "playersMax": 20,
       "version": "1.21.x"
     },
     "virtualNetworks": {
       "provider": "example",
       "interfaceName": "Ethernet",
       "ipv4": "192.168.1.100",
       "up": true
     },
     "timestamp": "2026-09-12T10:00:00"
   }

当前后端处理的核心操作是：

#. 将 ``roomId`` 转换为服务器 ID。
#. 查找对应服务器。
#. 找到服务器后更新 ``updatedAt``。
#. 保存服务器实体。

因此，当前心跳接口不能被理解为完整的 Agent 身份认证或服务器验证接口。有关在线状态机制，请参阅 :doc:`status`。

健康检查
--------

.. code-block:: http

   GET /

权限：公开。

用于确认后端服务能够正常响应。

接口权限总览
------------

+--------+------------------------------------+--------+
| 方法   | 路径                               | 权限   |
+========+====================================+========+
| GET    | ``/``                              | 公开   |
+--------+------------------------------------+--------+
| POST   | ``/auth/register``                 | 公开   |
+--------+------------------------------------+--------+
| POST   | ``/auth/login``                    | 公开   |
+--------+------------------------------------+--------+
| GET    | ``/auth/me``                       | 登录   |
+--------+------------------------------------+--------+
| POST   | ``/auth/logout``                   | 登录   |
+--------+------------------------------------+--------+
| GET    | ``/server/list/{page}``            | 公开   |
+--------+------------------------------------+--------+
| GET    | ``/server/list/servers/{id}``      | 公开   |
+--------+------------------------------------+--------+
| POST   | ``/server/list/create``             | 登录   |
+--------+------------------------------------+--------+
| GET    | ``/users``                          | 登录   |
+--------+------------------------------------+--------+
| GET    | ``/friends``                        | 登录   |
+--------+------------------------------------+--------+
| POST   | ``/friends/{userId}/request``      | 登录   |
+--------+------------------------------------+--------+
| POST   | ``/friends/{friendshipId}/accept`` | 登录   |
+--------+------------------------------------+--------+
| DELETE | ``/friends/{friendshipId}``        | 登录   |
+--------+------------------------------------+--------+
| POST   | ``/room/agent/heartbeat``           | 公开   |
+--------+------------------------------------+--------+

状态码建议
----------

客户端可以按照 HTTP 状态码进行基础错误处理：

* ``2xx``：请求处理成功。
* ``4xx``：请求参数、认证或业务条件存在问题。
* ``401``：当前接口需要认证，但没有提供有效的认证信息。
* ``404``：请求的资源不存在。
* ``5xx``：服务端发生异常，应检查后端日志。
