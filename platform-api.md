# 版本
V1

# 说明

## 简介

本文档用于描述企业客户与「原本」进行系统连接，自动将文章发送到「原本」完成版权认证的方法。

## 三方版权媒体

三方版权媒体是指该媒体发布的文章作者为媒体的签约作者，发布文章的版权归作者所有。三方版权媒体的一般工作流程为作者在媒体平台注册账号、投稿，媒体平台审核作者投稿后发布（或者不审核直接发布），文章署名和版权所有人都是作者。

## 作者选择授权协议

由于文章的版权归属于作者，在作者上传稿件时，需要作者选择其稿件的授权方式。「原本」支持免费的CC转载协议和付费的商业转载协议。为了方便客户集成，「原本」提供了SDK帮客户实现选择协议的界面流程。

## API连接流程

三方版权媒体（以下称为客户）在自有系统中审核文章通过后，调用「原本」提供的API，发送需要认证的文章、对应的作者信息和作者选择的授权方式到「原本」，「原本」会返回经过认证的文章（文章底部增加版权徽章和版权声明文字）到客户系统，客户系统再将增加了版权信息的文章发布出去。

客户系统发送的每一篇文章都要携带作者的邮箱和笔名，「原本」根据作者邮箱和笔名找到该作者在「原本」的账号，将文章认证到该账号下。如果未找到该作者信息对应的账号，「原本」会为该作者创建一个新的账号。

「原本」的API支持批量发送文章，便于客户系统同时审核多篇文章后发布。

![API连接顺序图](https://yb-public.oss-cn-shanghai.aliyuncs.com/openapi-docs/platform-api-workflow.png)

# API说明

## 身份认证

调用API需要身份认证，客户需要首先联系「原本」获取access token，在后续的API调用中使用access token进行身份认证。

认证方式为，在调用接口时，在HTTP请求头部增加Authorization Header，值为"Bearer access_token"。

## 文章发布接口
[ POST ] https://openapi.yuanben.io/v1/platform/articles
[ POST ] http://openapi.staging.yuanben.site/v1/platform/articles (测试线)

### 参数列表

#### 参数同时支持form-data和json两种方式，使用json方式post时，需把Content-type设置为application/json

| 参数名称  | 参数类型 | 是否必填 | 数据类型 | 参数说明 |
| --- | --- | --- | --- | --- |
| articles | POST | 是 | 数组 |每一条为一篇文章 |
| articles[*][client_id] | POST | 是 | 整数 | 文章 ID |
| articles[*][author] | POST | 是 | 数组 | 文章的作者信息 |
| articles[*][author][email] | POST | 是 | 字符串 | 作者的邮箱 |
| articles[*][author][pseudonym] | POST | 是 | 字符串 | 作者的笔名 |
| articles[*][title] | POST | 是 | 字符串 | 文章标题
| articles[*][content] | POST | 是 | HTML字符串 | 文章内容，如果文章内容和其他用户的已发布文章高度相似，则接口会返回错误 |
| articles[*][closed] | POST | 否 | 布尔型 | 文章是否对外开放，如果设置为true，文章则不对其他用户开放，默认为false |
| articles[*][original_url] | POST | 否 | 字符串 | 文章原文发布地址，默认为空 |
| articles[*][original_publish_time] | POST | 否 | 字符串 | 文章原文发布时间，默认为空 |
| articles[*][license] | POST | 是 | 数组 |文章授权协议，包括CC协议和商业协议两种
| articles[*][license][type] | POST | 是 | 字符串 | 授权类型，值为'cc'或者'cm'，分别对应CC协议和商业协议 |
| articles[*][license][content] | POST | 是 | 数组 | 授权方式具体参数 |
| articles[*][license][content][adaptation] | POST | 是 | 字符串 | 是否允许演绎，对于商业协议，可选'y'或者'n'，对于CC协议，可选'y','n'或者'sa' ('sa'表示相同方式共享) |
| articles[*][license][content][commercial] | POST | cc协议必填 | 字符串 | 是否允许商业使用， 'y'或者'n' |
| articles[*][license][content][price] | POST | 商业协议必填 | 整数 | 商业协议的授权转载价格，单位为“分”

### 返回值

状态码：若请求成功，返回200，提交数据有误返回422

数据：返回数据类型为json数组，其中每一项为一篇文章的信息，顺序与发送请求时的文章顺序一致，具体每一篇文章包含的字段有：

| 参数名称 | 参数说明 |
| --- | --- |
| client_id| 文章ID|
| public_key | 原创认证时使用的公钥（私钥加密后已邮件的形式发送到媒体机构注册时使用的邮箱）
| signature | 原创认证的数字签名
| hash | 文章哈希值
| block_hash | 文章在原本链上的区块地址
| yuanben_id | 完整原本DNA
| short_id | 原本DNA
| url | 文章在原本的详情页地址
| badge_html| 授权徽章的 html
| badge_url | 授权徽章图片的URL

### 请求示例
```
curl -X POST \
  https://openapi.yuanben.io/v1/platform/articles \
  -H 'authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...' \
-F 'articles[0][client_id]=5' \
  -F 'articles[0][author][email]=l**@163.com
  -F 'articles[0][author][pseudonym]=一个洋葱
  -F 'articles[0][title]=测试文章' \
  -F 'articles[0][content]=<p>一篇测试文章</p>' \
  -F 'articles[0][license][type]=cc' \
  -F 'articles[0][license][content][adaptation]=sa' \
  -F 'articles[0][license][content][commercial]=n'
```

或

```
curl -X POST \
  https://openapi.yuanben.io/v1/platform/articles \
  -H 'authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...' \
  -H 'content-type: application/json' \
  -d '{"articles":[{"client_id": 5, "author":{"email":"l**@163.com","pseudonym":"一个洋葱"},"title":"测试文章","content":"<p>一篇测试文章</p>","license":{"type":"cc","content":{"adaptation":"sa","commercial":"n"}}}]}'
```

返回值

```
[
  {
    "status": {
      "success": true,
      "message": "ok"
    },
    "article": {
      "client_id": 1,
      "public_key": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsZhLKMRGp10UXavQwmKM\n4Q25JX5aYiswhKG50/hbq7MZ7PrcpVgx1wVAnhG6r50B5SlWopeL8SQtFE6CzEk7\nD2tTjScZqw4yq75sHReOKqt01rFa63/ke1nShZuk4rekeENpWLRjwztGVv5QbOx7\nUbuEJqfibFugeaWCW2t4y4EZIKoGTWV++cMbk8/qIGqAHmj40Vg4/k7w2Co5K4+w\nz+puJU/bR2/tIf5tnOUtmZF/i/iGGPmwNSa+NJANa1EqyTLqYMTQJugZNBdEkofp\nwx5Uzr5fa+4rn7dr1LmbV0CDoP6/MBq96a+5ULrTrdZQwe6alsbOaaAxBvV8wuuC\nPwIDAQAB\n-----END PUBLIC KEY-----\n",
      "signature": "rLKnrpHp4C/37sWTWeMKElBk6yJC31/0KFL7H4dwi3t56eIoDmLy/bY549wA8x6b3quc9x8dCP6aHZKs8Qfrk24cTunsTgh8zdp0S3Kx/XaMh0torFkNKGjEQGyTvfrNbWn+84DspXXQ28m2os7uYo8ir/fSoRay3/6tUKrbTdX+/okHouAD9UqorFdsXURwS4LpTyJiJr0pg+pRVnxkX+/isWHw/3ZZDfq+Rah4w8EOHbJRbAekC0taa6SxBxjvZfnpIFZA2VkxBnR94BqBHPmKd+LGLCoGApVBHa8ehGYJHDIJo34Q23u0UHSTVRACOOxWumrzgaVMuDqzWdtBVg==",
      "hash": "14BWY2WLX50R4R86H3FZQH4HQLMA9G0OZJBDIF4MQOY9SL3ONM",
      "block_hash": "4CFVLGPMKC7OE7FNFM9OEPFH71UPIVOP8TQBPN2M716GJAUH1X",
      "yuanben_id": "3LGUMDDAIB0R3I1ZIG7QQJU2HQMKDRT1UCFJKDOBY7BMQ6UYI5",
      "short_id": "3LGUMDDA",
      "url": "http://www.work.io/article/3LGUMDDAIB0R3I1ZIG7QQJU2HQMKDRT1UCFJKDOBY7BMQ6UYI5",
      "badge_html": "<p style=\"margin-top:28px;margin-bottom:0;height:32px;text-align:left\"><img src=\"http://img.yuanben.org/badges/3LGUMDDAIB0R3I1ZIG7QQJU2HQMKDRT1UCFJKDOBY7BMQ6UYI5.png\" style=\"margin:0!important\" /></p><p style=\"font-size:12px;color:#787878;margin-top:4px\">本文经<a href=\"http://yuanben.io\" target=\"_blank\" style=\"text-decoration:none;color:#007f69\">「原本」</a>原创认证，作者<a href=\"https://yuanben.io/author/1424\" style=\"text-decoration:none;color:#007f69\">习近平</a>，访问<a href=\"http://yuanben.io\" target=\"_blank\" style=\"text-decoration:none;color:#007f69\">yuanben.io</a>查询【<a href=\"http://www.work.io/article/3LGUMDDAIB0R3I1ZIG7QQJU2HQMKDRT1UCFJKDOBY7BMQ6UYI5\" target=\"_blank\" style=\"text-decoration:none;color:#007f69\">3LGUMDDA</a>】获取授权信息。",
      "badge_url": "http://img.yuanben.org/badges/3LGUMDDAIB0R3I1ZIG7QQJU2HQMKDRT1UCFJKDOBY7BMQ6UYI5.png"
    }
  }
]
```


## 图片发布接口
[ POST ] https://openapi.yuanben.io/v1/platform/images
[ POST ] http://openapi.staging.yuanben.site/v1/platform/images (测试线)

### 参数列表

#### 参数同时支持form-data和json两种方式，使用json方式post时，需把Content-Type设置为application/json 
**post数据大小不可大于50MB（包含文件）**


| 参数名称  | 参数类型 | 是否必填 | 数据类型 | 参数说明 |
| --- | --- | --- | --- | --- |
| images | POST | 是 | 数组 |每一条为一张图片 |
| images[*][client_id] | POST | 是 | 整型 | 图片 ID |
| images[*][title] | POST | 是 | 字符串 | 图片标题|
| images[*][description] | POST | 是 | 字符串 |图片描述 |
| images[*][tags] | POST | 是 | 数组 |图片标签 |
| images[*][image] | POST | 是 | file/string |图片,当Content-Type为json时传图片based64后的字符串，当为form-data时传文件 |
| images[*][license] | POST | 是 | 数组 |文章授权协议，包括CC协议和商业协议两种
| images[*][license][type] | POST | 是 | 字符串 | 授权类型，值为'cc'或者'cm'，分别对应CC协议和商业协议 |
| images[*][license][content] | POST | 是 | 数组 | 授权方式具体参数 |
| images[*][license][content][adaptation] | POST | 是 | 字符串 | 是否允许演绎，对于商业协议，可选'y'或者'n'，对于CC协议，可选'y','n'或者'sa' ('sa'表示相同方式共享) |
| images[*][license][content][commercial] | POST | cc协议必填 | 字符串 | 是否允许商业使用， 'y'或者'n' |
| images[*][license][content][price] | POST | 商业协议必填 | 整数 | 商业协议的授权转载价格，单位为“分”

### 返回值

状态码：若请求成功，返回200，提交数据有误返回422

返回数据为json对象

| 参数名称 | 参数说明 |
| --- | --- |
| status | 认证状态 |
| status.code | 认证是否成功 200为成功 |
| status.message | 认证失败的错误消息|
| data | 数据信息，为JSON数组，其中每一项为一张图片的信息，顺序和发送的图片顺序一致： |

data中每一组的image参数是JSON Object，包含图片的详细信息，具体包含的字段有：

| 参数名称 | 参数说明 |
| --- | --- |
| client_id | 发送的ID，原样返回 |
| public_key | 原创认证时使用的公钥（私钥加密后已邮件的形式发送到媒体机构注册时使用的邮箱）|
| signature | 原创认证的数字签名 |
| content_hash | 图片哈希值 |
| block_hash | 文章在原本链上的区块地址 |
| yuanben_id | 完整原本DNA |
| short_id | 原本DNA |
| url | 图片在原本的详情页地址 |

### 请求示例
```
curl -X POST \
  https://openapi.yuanben.io/v1/media/images \
  -H 'authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...' \
 -F 'images[0][client_id]=文章ID' \
  -F 'images[0][title]=测试文章' \
  -F 'images[0][description]=<p>一篇测试文章</p>' \
  -F 'images[0][license][type]=cc' \
  -F 'images[0][license][content][adaptation]=sa' \
  -F 'images[0][license][content][commercial]=n'
```

或

```
curl -X POST \
  https://openapi.yuanben.io/v1/media/images \
  -H 'authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...' \
  -H 'content-type: application/json' \
  -d '{"images":[{"client_id": 5,"title":"测试文章","description":"一篇测试文章","image":"based64后值","license":{"type":"cc","content":{"adaptation":"sa","commercial":"n"}}}]}'
```

返回值

```
{
    "status": {
        "code": 200,
        "message": ""
    },
    "data": [
        {
            "status": {
                "success": true,
                "message": "ok"
            },
            "image": {
                "client_id": "1",
                "public_key": "02636d79fc7c7cc37b9cb348037eec28743d030e5d570d0d69837a3587bed31880",
                "signature": "22bcfa7512c9426c3737b36994e30bb4c08eecfe8c50da0a0d0fd2e4cef5027310d91bafb30fc3cb255d9f543b6572db2b8a64662f487edde4b012719e039c161c",
                "content_hash": "9d966886063e20405224c94d012e032c79446ee67b76273d095b8c3983f8807d",
                "block_hash": null,
                "yuanben_id": "3NI02V0ZKLHF5QIQTHEHCOMUMX5T244SJ2GYNPADF4SWJZDQHE",
                "short_id": "3NI02V0Z",
                "url": "http://staging.yuanben.site/image/3NI02V0ZKLHF5QIQTHEHCOMUMX5T244SJ2GYNPADF4SWJZDQHE"
            }
        }
    ]
}

```