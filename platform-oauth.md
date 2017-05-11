# 版本
V1

# 说明

## 简介

本文档用于描述企业客户与「原本」进行系统连接，企业客户的用户发布文章后，自动将文章发送到「原本」完成版权认证的方法。

## 三方版权媒体

三方版权媒体是指该媒体发布的文章作者为媒体的签约作者，发布文章的版权归作者所有。三方版权媒体的一般工作流程为作者在媒体平台注册账号、投稿，媒体平台审核作者投稿后发布（或者不审核直接发布），文章署名和版权所有人都是作者。

## 作者选择授权协议

由于文章的版权归属于作者，在作者上传稿件时，需要作者选择其稿件的授权方式。「原本」支持免费的CC转载协议和付费的商业转载协议。为了方便客户集成，「原本」提供了SDK帮客户实现选择协议的界面流程。

## OAuth授权流程

「原本」支持使用国际通用的OAuth2.0协议完成客户系统和「原本」的连接。
作者在客户平台使用原本的功能前，首先需要将作者在「原本」的账号与客户平台的账号进行绑定，同时授权客户平台代表作者在「原本」发布文章，授权流程的顺序图如下：

![OAuth授权顺序图](https://yb-public.oss-cn-shanghai.aliyuncs.com/openapi-docs/platfrom-oauth-workflow-authorize.png)

在作者编辑文章时，可通过原本SDK提供的[原本协议编辑器](https://github.com/yuanbenio/yb-license-editor)来选择想要的转载授权协议，客户平台需要在保存用户文章的同时保存用户选择的授权协议。

当用户点击发布时，客户平台将用户文章和授权协议，以及用户对应的Access Token一起发送到「原本」进行文章认证，「原本」会返回认证后的徽章和授权说明文字，客户平台需将徽章文字和原文章一同保存起来,并在文章展示页面一起拼接展示出来。

![OAuth发布顺序图](https://yb-public.oss-cn-shanghai.aliyuncs.com/openapi-docs/platfrom-oauth-workflow-publish.png)

由于发送文章到原本认证是一个网络请求，同时原本链的区块构建需要一些时间，为了减少用户点击发布后的等待时间，建议客户系统以异步请求方式实现认证，保存文章后发送一个消息通知给异步队列，由异步队列完成文章的认证和徽章的保存。在文章的展示页面判断只有当徽章已经成功生成后，才显示徽章内容。

# API说明

## OAuth授权接口
[GET] https://openapi.yuanben.io/oauth/authorize

### 参数列表
在用户点击绑定「原本」账号后，客户系统将浏览器重定向到这个API地址，作者在此页面完成账号创建和授权操作。

| 参数名称  | 参数类型 | 是否必填 | 数据类型 | 参数说明 |
| --- | --- | --- | --- | --- |
| client_id | GET | 是 | 字符串 | 客户系统在「原本」的id |
| redirect_uri | GET | 是 | 字符串 | url编码过的回调地址，完成授权后，「原本」会将浏览器重定向到此地址，同时携带授权token |
| response_type | GET | 是 | 字符串 | 固定填写code |
| scope | GET | 是 | 字符串 | 需要获取的权限，目前固定填空值 |

### 返回值

用户完成授权后，「原本」会重定向到redirect_uri指定的地址，同时带上授权

```
http://www.redirect_uri.com/callback?code=xxxxxx
```

## OAuth获取Access Token接口



## 文章发布接口
[ POST ] https://openapi.yuanben.io/v1/platform/articles

### 参数列表

#### 参数同时支持form-data和json两种方式，使用json方式post时，需把Content-type设置为application/json

| 参数名称  | 参数类型 | 是否必填 | 数据类型 | 参数说明 |
| --- | --- | --- | --- | --- |
| articles | POST | 是 | 数组 |每一条为一篇文章 |
| articles[*][author] | POST | 是 | 数组 | 文章的作者信息 |
| articles[*][author][email] | POST | 是 | 字符串 | 作者的邮箱 |
| articles[*][author][pseudonym] | POST | 是 | 字符串 | 作者的笔名 |
| articles[*][title] | POST | 是 | 字符串 | 文章标题
| articles[*][content] | POST | 是 | HTML字符串 | 文章内容，如果文章内容和其他用户的已发布文章高度相似，则接口会返回错误 |
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
| title | 文章标题 |
| content | 文章内容，在原始文章内容后增加了授权徽章图片和版权说明文字 |
| outline | 文章摘要 |
| public_key | 原创认证时使用的公钥（私钥加密后已邮件的形式发送到媒体机构注册时使用的邮箱）
| signature | 原创认证的数字签名
| hash | 文章哈希值
| block_hash | 文章在原本链上的区块地址
| yuanben_id | 完整原本DNA
| short_id | 原本DNA
| url | 文章在原本的详情页地址
| badge_url | 授权徽章图片的URL

### 请求示例
```
curl -X POST \
  https://openapi.yuanben.io/v1/platform/articles \
  -H 'authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...' \
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
  -d '{"articles":[{"author":{"email":"l**@163.com","pseudonym":"一个洋葱"},"title":"测试文章","content":"<p>一篇测试文章</p>","license":{"type":"cc","content":{"adaptation":"sa","commercial":"n"}}}]}'
```

返回值

```
[
  {
    "user_id": 3,
    "author_name": "一个洋葱",
    "title": "测试文章",
    "license": {
      "type": "cc",
      "content": {
        "adaptation": "sa",
        "commercial": "n"
      }
    },
    "content": "<p>一篇测试文章</p><p style=\"margin-top:28px;margin-bottom:0;height:32px\"><img src=\"http://yb-img-staging.oss-cn-shanghai.aliyuncs.com/badges/4GD4OT02A9F4PDUB9AVZ629PDT74A9YFBEAYJ215IIOM5JUW5I.png\" /></p><p style=\"font-size:12px;color:#787878;margin-top:4px\">本文经<a href=\"http://yuanben.io\" target=\"_blank\" style=\"text-decoration:none;color:#007f69\">「原本」</a>原创认证，作者<a href=\"https://yuanben.io/author/3\" style=\"text-decoration:none;color:#007f69\">一个洋葱</a>，访问<a href=\"http://yuanben.io\" target=\"_blank\" style=\"text-decoration:none;color:#007f69\">yuanben.io</a>查询【<a href=\"http://lg.local.yuanben.site/article/4GD4OT02A9F4PDUB9AVZ629PDT74A9YFBEAYJ215IIOM5JUW5I\" target=\"_blank\" style=\"text-decoration:none;color:#007f69\">4GD4OT02</a>】获取授权信息。",
    "outline": "一篇测试文章",
    "created_at": 1491908110,
    "status": 3,
    "public_key": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArrhRNfnh9HAQDTIKGEm6\n1avmtvrJpgWxrlZdW9lTvKSusWJ2YJxHrLmJPXpGmTyZAXjwR13LCe4cZth2Ca2e\nDv/1s/3+2igxQM62rnfVUTqFaW6b7Zmk0M3ht5ncDUTRMZTV8zBmma74UjssPSS/\nfIhc1e75pZ5hIhsOew2tzN7ab3gSpghKunHmOI3KzONlu1LghO1XxfAzskVSJ2uX\nWXIuvzY57q4jPxBv//B2oH1cOk8N4odojXxCvjNuigyXlvcN/wsJWXNLu/5lX1vL\n820XlFosdL5EhO+J4D30ZfOzeIzUj1pPWxCQkb+OMxH865qpGwNMWSkOXp8k5Fie\nXQIDAQAB\n-----END PUBLIC KEY-----\n",
    "signature": "TL9TSUWBpBKAmPxuxwG//pSkrXid1CSg2Lm3NgA6LWDm4b7v4Zar7rP3+mFYB5Obn1WCZ5Kphck1dnxBICfI4t0sl9VwE0ir1GCmOctv+njnVhVkGuvGCNRqvwE7SCZ6reOz3EbPPNU8pSAunrlSdzzg+jI3hB6z1fJVbcoBrfttTvkPFZuFEoGiFZ9vlk+V6WxU24kxtEIZrDwKbkaOghVC/tqcesVZWVabPc4GWeQsZH+gLoAuJ6mzPyTfV5y/2G1hevR9huRuLi2AVM1kq/3SvWP51h7qwaCKxwuGIFBSSmFNr7/h9m5bW6R1+TOtbsuzpIczp96vfAPd1v10bw==",
    "hash": "3K7Q2KTGYP0XFLOLA5IMY607TM27OIG5VEIWQZLFA8VN4V94E5",
    "block_hash": "K8S6YARWZMABPE7S03Z5W8FM67WB904UH4OYPKYK4X5V5EUBZ",
    "yuanben_id": "4GD4OT02A9F4PDUB9AVZ629PDT74A9YFBEAYJ215IIOM5JUW5I",
    "short_id": "4GD4OT02",
    "id": 2714,
    "url": "https://yuanben.io/article/4GD4OT02A9F4PDUB9AVZ629PDT74A9YFBEAYJ215IIOM5JUW5I",
    "badge_url": "https://yb-img.oss-cn-shanghai.aliyuncs.com/badges/4GD4OT02A9F4PDUB9AVZ629PDT74A9YFBEAYJ215IIOM5JUW5I.png"
  }
]
```