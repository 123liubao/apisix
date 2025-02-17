---
title: ip-restriction
---

<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

## 描述

`ip-restriction` 可以通过以下方式限制对服务或路线的访问，将 IP 地址列入白名单或黑名单。 单个 IP 地址，多个 IP 地址 或 CIDR 范围，可以使用类似 10.10.10.0/24 的 CIDR 表示法。

## 属性

| 参数名    | 类型          | 可选项 | 默认值 | 有效值 | 描述                             |
| --------- | ------------- | ------ | ------ | ------ | -------------------------------- |
| whitelist | array[string] | 可选   |        |        | 加入白名单的 IP 地址或 CIDR 范围 |
| blacklist | array[string] | 可选   |        |        | 加入黑名单的 IP 地址或 CIDR 范围 |
| message | string | 可选   | Your IP address is not allowed. | [1, 1024] | 在未允许的IP访问的情况下返回的信息 |

只能单独启用白名单或黑名单，两个不能一起使用。
`message` 可以由用户自定义。

## 如何启用

下面是一个示例，在指定的 route 上开启了 `ip-restriction` 插件:

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    },
    "plugins": {
        "ip-restriction": {
            "whitelist": [
                "127.0.0.1",
                "113.74.26.106/24"
            ]
        }
    }
}'
```

当未允许的IP访问时，默认返回 `{"message":"Your IP address is not allowed"}`。如果你想使用自定义的 `message`，可以在插件部分进行配置:

```json
"plugins": {
    "ip-restriction": {
        "whitelist": [
            "127.0.0.1",
            "113.74.26.106/24"
        ],
        "message": "Do you want to do something bad?"
    }
}
```

## 测试插件

通过 `127.0.0.1` 访问：

```shell
$ curl http://127.0.0.1:9080/index.html -i
HTTP/1.1 200 OK
...
```

通过 `127.0.0.2` 访问：

```shell
$ curl http://127.0.0.1:9080/index.html -i --interface 127.0.0.2
HTTP/1.1 403 Forbidden
...
{"message":"Your IP address is not allowed"}
```

## 禁用插件

当你想去掉 `ip-restriction` 插件的时候，很简单，在插件的配置中把对应的 json 配置删除即可，无须重启服务，即刻生效：

```shell
$ curl http://127.0.0.1:9080/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index.html",
    "plugins": {},
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:1980": 1
        }
    }
}'
```

现在就已移除 `ip-restriction` 插件，其它插件的开启和移除也类似。
