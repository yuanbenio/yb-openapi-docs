# 版本
V1

# 说明

## 简介

本文档用于描述「原本」在媒体客户的授权下，通过接口形式，交付媒体客户已存证内容至外部内容需求方的方法

# API说明

## 身份认证

调用API需要获取身份认证的 access_token，客户需要提供的材料如下：

1. 发起API调用的**客户端IP**：针对一个特定的 access_token，只允许一个固定的IP使用其发起API调用；
2. 获得内容的媒体信息：即客户希望从哪一个媒体平台获取内容。

根据客户提供的信息，「原本」将生成以下授权数据：
1. access_token：在调用接口时，在HTTP请求头部增加Authorization Header，值为"Bearer access_token"
2. uuid：代表内容媒体的uuid，作为API请求地址的一部分


## 内容获取接口
> [ GET ] http://openapi.yuanben.io/v1/content/articles/{uuid} 

> [ GET ] http://openapi.staging.yuanben.site/v1/content/articles/{uuid} (测试线)

### 参数列表
| 参数名称 | 是否必填 | 数据类型 | 参数说明 |
| --- | --- | --- | --- |
|page|否|整型|内容分页数，从1开始|
|page_size|否|整型|每页内容数量，默认20条，最大50条|
|start_time|否|整型|内容发布的最早时间|
|end_time|否|整型|内容发布的最晚时间|

### 返回值

状态码：若请求成功，返回200，授权有误返回401

返回数据为JSON数组
| 参数名称 | 参数说明 |
| --- | --- |
| status | 状态 |
| status.code | 状态码 |
| status.message | 状态消息 |
| data | 内容 |
| data.name | 内容媒体名称 |
| data.total | 本次API调用返回的内容总条数 |
| data.articles | 内容数组 |

其中data.articles为json数组，其内容格式如下：
| 参数名称 | 参数说明 |
| --- | --- |
| id | 「原本」端稿件ID |
| title | 标题 |
| content | 内容 |
| created_at | 「原本」发布时间 |
| original_publish_time | 内容发布时间 |
| original_url | 原文发布地址 |
| yuanben_id | 「原本」端DNA编号 |

### 请求示例
```shell
curl -v 'http://openapi.yuanben.io/v1/content/articles/9cc040eb-5631-435d-8c79-35f6cf1bce83?page=1&page_size=2' -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciO...'
```

返回值

```json
{
    "status": {
        "code": 200,
        "message": "success"
    },
    "data": {
        "name": "中国经济周刊-测试号111",
        "total": 200,
        "articles": [
            {
                "id": 957134,
                "title": "fsdfasdfsdfsdfsdf",
                "content": "<p>fvkujukmhnfbsdavz哦了，开 u 蒋敏你哈再靠近梦想</p>",
                "created_at": 1578313196,
                "original_publish_time": 1578313196,
                "original_url": "",
                "yuanben_id": "Z2QMZO7J722LWLIAMBN5XZYO3EUDUOZW6F1Y8P2LR3P7D67V9"
            },
            {
                "id": 957135,
                "title": "123",
                "content": "<p>fsfafadsfasfaf  fas sa</p>",
                "created_at": 1578375763,
                "original_publish_time": 1578375763,
                "original_url": "",
                "yuanben_id": "4D3ZWQQKM47N0XFBM98024WV8JF646CEY0VIQXXDW2RR79JBFU"
            }
        ]
    }
}
```

