核心概念
========

服务器
------

服务器是 MC Server Web 中用于展示 Minecraft 房间的信息对象。创建服务器时，后端会根据当前登录用户确定服务器所有者，客户端不需要也不应该自行指定 ``ownerId``。

服务器信息主要包括：

* ``name``：服务器名称。
* ``description``：服务器描述。
* ``mods``：服务器使用的 Mod 信息。
* ``host``：Minecraft 服务器地址。
* ``port``：Minecraft 服务器端口。
* ``code``：第三方局域网或虚拟网络连接所使用的标识信息。
* ``active``：服务器当前的在线状态。
* ``serverFlag``：是否启用后端的服务器状态检测扩展。

在线状态
--------

当前系统存在两种状态来源：

#. 后端主动使用 Minecraft Ping 检测服务器。
#. Agent 向后端发送心跳。

两者解决的问题不同。Minecraft Ping 更适合判断 Minecraft 服务端本身是否能够被访问；Agent 心跳则可以携带运行环境、虚拟网络和玩家数量等额外信息。

详细说明请参阅 :doc:`status`。

Agent
-----

Agent 是运行在服务器所在环境中的辅助程序。它可以向 ``POST /room/agent/heartbeat`` 发送 JSON 数据，用于告诉后端对应房间仍然存在，并上报 Agent、Minecraft 和虚拟网络信息。

当前后端实现中，心跳接口本身不要求 JWT Bearer 认证。后端收到心跳后，会根据 ``roomId`` 查找服务器，并更新该服务器的 ``updatedAt``。

.. warning::

   当前实现不会根据心跳请求中的 ``verified``、``verificationLevel`` 等字段自动完成完整的服务器认证流程。不要将发送心跳理解为服务器已经通过全部验证。

JWT 认证
--------

需要登录的接口使用 HTTP ``Authorization`` 请求头传递 JWT：

.. code-block:: text

   Authorization: Bearer <token>

注册和登录成功后都会返回 Token。当前后端默认 Token 有效期为 1440 分钟；实际部署环境可能通过配置修改该值。

好友关系
--------

好友系统使用关系记录表示用户之间的状态。常见状态包括：

* ``PENDING``：等待对方处理。
* ``ACCEPTED``：双方已经成为好友。
* ``REJECTED``：请求被拒绝。
* ``BLOCKED``：关系被阻止。

接受好友请求时，只有请求的接收方可以执行接受操作。
