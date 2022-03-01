---
title: KINE API 文档

language_tabs: # must be one of https://git.io/vQNgJ

- java
- python

toc_footers:

- <a href='#'>Copyright by KINE</a>

includes:

- errors

search: true

code_clipboard: true
---

# 更新历史

## 2022-01-07

1. `POST /trade/api/order/v2/place` 返回对象增加fee, profit, profitTakeoffAmount


## 2021-12-22

1. 增加K线历史查询接口， 以及订订阅

## 2021-12-12

1. 增加一键平仓 `POST /trade/api/order/v2/close-all`
2. 增加合仓分仓设置接口 `POST /account/api/set-position-type`
3. 增加合仓分仓获取接口 `GET /account/api/get-position-type`

## 2021-12-04

增加订单状态描述

## 2021-12-03

1. 订单查询，增加条件单，止盈止损单相关字段。  数量，触发价，订单类型
2. Websocket增加订单更新通知。  topic: `order.all`

## 2021-11-29

1. 增加了DEMO链接
https://github.com/kine-api-docs/demo
2. 增加了签名问题排查建议
3. 增加了content-type的说明。 统一使用：application/json;charset=utf-8


## 2021-11-19

* 中文文档

# 简介

欢迎查看KINE API文档。

您可以通过API来访问KINE平台数据，用于下单、撤单、查询订单、获取账户信息等。

我们提供了两种类型的接口: RESTFUL接口和WebSocket订阅。

<aside class="notice">
为了使您快速接入，我们提供了DEMO供您参考。
https://github.com/kine-api-docs/demo
</aside>

## REST API

建议您使用REST API来完成一次性的操作，例如下单，撤单等。

<aside class="notice">
注意: 
我们的REST API请求统一使用 
content-type: application/json;charset=utf-8 
</aside>

## WebSocket API

建议您使用WebSocket API来获取数据更新。例如：市场数据，账户更新，订单更新等。

## 鉴权

REST 和 WebSocket 接口都有公开接口和私有接口:

公开接口: 用于获取基础数据，例如市场数据.

私有接口: 用于用户账户相关的操作。 例如下单，划转，账户余额查询。 
私有接口调用需要使用API Key进行签名（`SIGNED`）以保证用户数据的安全。

# 基本信息

## API URLs

### REST API

`https://api.kine.exchange`

### WebSocket API

`wss://api.kine.exchange/ws`

## Limits 限频

我们对REST接口访问有限频管理，具体请参考接口定义。

公开API根据IP地址限频，非公开接口按照用户ID来限频。

限频错误码为  `HTTP 429` 

关于Websocket长链接，关于连接数，消息发送速率，链接最大存活时间我们也有相关的限制。
具体请参考 `WebSocket API` - `Session limit`

<aside class="warning">
注意: 
如果您调用很频繁可能遇到 403 的错误， 这是被防火墙拦截了。 请您尽量使用websocket来获取最新数据的更新而不是频繁查询接口。
如您确实需要更高频率的调用，请联系我们的客服。  
</aside>

## Security 安全

### 使用签名进行身份认证

`SIGNED` 接口调用，需要额外的Http Request Headers: 

`KINE-API-ACCESS-KEY`, `KINE-API-TS`,`KINE-API-SIGNATURE`.

* `KINE-API-ACCESS-KEY` 是您的API KEY
* `KINE-API-SIGNATURE` 是通过 `HMAC SHA256` 算法得到的签名. 签名算法请参考 `身份验证` 部分.
* `KINE-API-TS` 是当前时间的时间戳（UTC时间），如果时间差距太大，签名验证可能失败，请确保您的服务器时间是准确的。 `毫秒时间戳`

### 如何获取您的API Key？

请前往API Key管理页面生成您的API KEY。 

https://kine.exchange/account/api-key

`API Key` API Key的ID。

`SecretKey` 用来生成接口调用的签名。 (生成后只能查看一次，请妥善保存。！！！！)

API Key 权限

* `ReadOnly` 只读权限。 只能用来查询用户信息。 例如账户余额，订单历史等。
* `Trade` 交易权限。允许使用该API Key进行下单，平仓，划转等。

<aside class="warning">
注意: 
为了确保您账户资产的安全。

1. 请不要将API Key分享给别人。
   
2. 如果API Key被泄露，请尽快在管理页面删除。 
</aside>

### 如何签名您的接口调用?

请参考`身份认证`章节。

# Authentication 身份验证

网络中的请求有可能被拦截并篡改，为了保证数据安全，所有请求个人账户数据的接口调用需要使用用户的Secret Key进行签名，确保您的请求不会被攻击者篡改。

每一个API Key有权限范围，请确保您的 API Key有调用相关接口的权限。

为了使您快速接入，我们提供了DEMO供您参考。
https://github.com/kine-api-docs/demo

请求中，一个合法的签名是根据以下信息计算得出的:

* `Request Method`  GET/POST  (Websocket 签名请使用 GET)
* `Host` 主域名 目前只有一个 `api.kine.exchange`
* `Request path` 接口请求路径，不包含域名。 例如: `/trade/api/order/v2/place`
* `Request Parameters` 请求参数。 这里的请求参数是指HTTP请求参数，拼接在URL中的参数，而不是Request Body中的数据。 不论GET、POST请求， 签名只需要HTTP请求参数，不需要Body中的数据。
* `Timestamp` 毫秒时间戳 需要包含在request header `KINE-API-TS`
* `Payload` 将以上相关信息拼接成固定格式的文本，然后对其进行加密得到最终的签名.  格式请参考 `Payload` 章节
* `Signature` 对Payload加密得到的签名， 需要包含在 request header  `KINE-API-SIGNATURE`
* 
* `API key`  API Key ID，需要包含在Request Header中 `KINE-API-ACCESS-KEY`
* `SecretKey`  用于计算签名的秘钥。
## Signature 签名

### 1. 准备签名用的文本 Payload ， 请参考右侧示例

```text
{requestMethod}\n
{host}\n               
{request path}\n       ## 请求路径
{request parameters}\n ## 实际拼接在URL上？右侧的部分，如果没有参数，请保留 `空行`。
{timestamp}            ## 发起请求时的时间戳
```

> Payload 实例

```text
GET
api.kine.exchange
/trade/api/history
clientOrderId=123&status=2
123123123123
```

### 2. 生成签名.

```signature = sign(payload, secretKey);```

签名方法的入参是Payload和SecretKey

签名方法可以使用不同的语言实现。 右侧提供了Java和Python两种编程语言的实现。

其它编程语言请自行百度 (HMAC SHA256)。

为了使您快速接入，我们提供了DEMO供您参考。
https://github.com/kine-api-docs/demo

> 签名样例

``` java
        String accessKey = "123485552fb24cf49412345688888888";
        String secretKey = "e95a0ba0648215e61d7c29ad6c96c2185c2c15fa3ce173d2b412345688888888";

        // sign:
        Mac hmacSha256 = null;
        try {
            hmacSha256 = Mac.getInstance("HmacSHA256");
            SecretKeySpec secKey =
                    new SecretKeySpec(secretKey.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
            hmacSha256.init(secKey);
        } catch (NoSuchAlgorithmException e) {
            throw new BadSignatureException("No such algorithm: " + e.getMessage());
        } catch (InvalidKeyException e) {
            throw new BadSignatureException("Invalid key: " + e.getMessage());
        }
        String payload = "The payload text";
        byte[] hash = hmacSha256.doFinal(payload.getBytes(StandardCharsets.UTF_8));
        String signature = Base64.getEncoder().encodeToString(hash);
```

### 3. HTTP HEADER中需要增加以下3个：

* `KINE-API-ACCESS-KEY`   The API key
* `KINE-API-TS`           The UTC timestamp
* `KINE-API-SIGNATURE`    The signature generated in Step. 2

> 签名请求示例

```text
GET https://api.kine.exchange/trade/api/history?clientOrderId=123&status=2

Headers:
KINE-API-TS: 1618561349256
KINE-API-ACCESS-KEY: 072285552fb24cf49412345688888888
KINE-API-SIGNATURE: xxxxxxxxxx+jJLJwYSxz7iMbA=
```


### 问题排查

当接口调用返回签名错误时，请检查

1. Request Header 是否包含了文档中提到的必填项  KINE-API-XXX
2. 请打印出计算签名用的payload并仔细检查。 注意换行、空格
3. 请参考DEMO代码，检查你的代码逻辑。 https://github.com/kine-api-docs/demo
4. 如果依然无法解决，请联系我们的客服，我们的技术人员将帮助你解决。  
   [电报群](https://t.me/kine_api_support) 


# Market Data API 市场数据

市场数据API是公开接口，不需要签名


## 查询所有交易对以及交易规则
### HTTP Request
`GET /market/api/trading-rules`

### Required Permission
`ReadOnly`

### 请求参数
无

### 返回值
字段 | 数据类型 | 描述 | 举例 |
--------- | ----------- | --------|  ----- | 
symbol        | string  |  交易对 | ETHUSD，BTCUSD |
enabled       | boolean |  是否可交易 true:可交易，false不可交易 |  |
priceDecimal  | int     |  价格精度 |  |
qtyDecimal    | int     |  数量精度 |  |
amountDecimal | int     |  金额精度 |  |
minQty        | decimal  |  最小下单数量 |  |
minAmount         | decimal  |  最小下单金额 |  |
defaultMarginType | enum     |  默认账户类型 全仓:CROSSED 逐仓:ISOLATED |  |
defaultLeverage   | decimal  |  默认杠杆 |  |
maxLeverage       | decimal  |  最大杠杆 |  |


## 查询某个交易对的指数价格
### HTTP Request
`GET /market/api/price/{symbol}`

### Required Permission

`ReadOnly`

### 请求参数

参数 | 数据类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | 交易对  |  BTCUSD |


### 返回值

字段 | 数据类型 | 描述 | 举例 |
--------- | ----------- | --------|  ----- | 
symbol    | string  |  交易对 |  |
price     | string  |  价格 |  |
timestamp | long    |  指数时间 |  |

## 获取全部交易对价格

### HTTP Request

`GET /market/api/agg-price`

### Required Permission

`ReadOnly`

### 返回值

参数 | 数据类型  | 默认值 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| ------ |
prices | Price  |  |  | | 
ts     | long   | 指数时间 |  | | 

Price:

Field | 数据类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
ts     | long     | 指数时间 |   |  |
symbol |string    | 交易对   |   |  |
price  |bignumber | 价格     |   |  |


```json
{
  "ts": 1619798400000,
  "prices": [
    {
      "ts": 1619798360000,
      "symbol": "BTCUSD",
      "price": "46578"
    },
    {
      "ts": 1619798367000,
      "symbol": "ETHUSD",
      "price": "1234.5"
    }
  ]
}
```

## 获取资金费率

### HTTP Request

`GET /market/api/funding-rate/{symbol}`

### 请求参数

参数 | 类型 | 是否必须 |默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes | false   |  交易对 |  BTCUSD |

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
symbol | string  |  交易对 |  |
fundingRate | string  | 当前资金费率 |  |
estifundingRate | string  | 预测资金费率 |  |
nextFundingTime | long  | 资金费率收取时间 |  |
timestamp | long  | 响应时间 |  |

## 查询K线历史

### HTTP Request

`GET /market/api/kline`

该接口用于查询k线历史。
如需获取实时k线，请通过websocket订阅实时k线更新。

>K线数据示例

```json
{
  "success": true,
  "code": 200,
  "message": null,
  "data": {
    "lines": [
      [
        1639973820000,  // k线时间戳， 时间区间的起始时间戳
        46854,          // open  起始价格
        46928,          // high  最高价
        46833,          // low 最低价
        46928,          // close 结束价格
        0               // 交易量 金额USD
      ],
      [
        1639973880000,
        46906,
        46926,
        46873,
        46904,
        0
      ],
      [
        1639973940000,
        46902,
        46938,
        46902,
        46912,
        0
      ]
    ]
  }
}

```

### 请求参数

参数 | 类型 | 是否必须 |默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes | false   |  交易对 |  BTCUSD |
period | string | yes | false   |  k线周期 |  "1min","5min","15min","30min","60min","4hour","1day","1week","1mon" |
size | string | false | 150   |  kline历史条数， 默认150， 最大2000 |  查询历史多少根k线 |

### 返回值

为减少数据大小， 这里使用了精简的数据结构。 二维数组来表示k线数据。
每一行是一根k线， 数据中每个位置的数据定义，请参考下列表格， 或者右侧示例。

** 返回数据包含历史数据（size） + 1 (最新实时kline)

字段索引 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
0 | long  |  k线起始时间戳 |  |
1 | number  | open ， 起始价格 |  |
2 | number  | high ， 最高价格 |  |
3 | number  | low， 最低价格 |  |
4 | number  | close， 结束价格 |  |
5 | number  | 交易金额USD |  |

# 交易API

## 下单（市价单，限价单）

### HTTP Request

`POST /trade/api/order/v2/place`

> 下单请求参数示例

```json
// 市价买入 (type = 1)
{
  "symbol": "BTCUSD",
  "direct": "SELL",
  "type" : 1,
  "baseAmount": "0.25",
  "clientOrderID": "test-321"
}
```

```json
//限价买入  (type = 7)
{
  "symbol":"BTCUSD",
  "direct":"BUY",
  "type":"7",
  "baseAmount":"0.001",
  "triggerPrice": "50000",
  "price": "55000"
}
```

```json
//市价卖出平仓  (type = 1, 平掉仓位 10000265)
{
  "baseAmount":0.001,
  "direct":"SELL",
  "symbol":"BTCUSD",
  "type":"1",
  "positionId":10000265
}
```

### Required Permission

`Trade`

### Request Body(Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol          | string | yes |    | 交易对 |  |
baseAmount      | string | yes |    | 交易数量 |  |
direct          | string | yes |    | 交易方向 |  BUY买入, SELL卖出|
type            | int    | yes |    | 订单类型 |  1 市价单，7 条件单(限价单)|
positionId      | long   | no  |    | 开仓0，加仓，减仓，平仓时需要传入要执行的当前仓位ID |  0 |
clientOrderId   | string | no |     | clientOrderId , which given by user | Valid character, A-Z,a-z,0-9,_,- length <= 128|
stopProfitPrice | string | no |     | 如果市价单同时下止盈单，需要指定止盈价格 |  0 |
stopLossPrice   | string | no |     | 如果市价单同时下止损单，需要指定止损价格 |  0 |
price           | string | no |     | 当前指数价格 |  0 |
triggerPrice    | string | no |     | 当下条件单（限价单）时需要传入触发价格 |  0 |



### 返回值
字段 | 类型 | 描述 | Value 举例 |
--------- | ----------- | -----------| ----------| 
result           | json格式  | 含有：  市价单下单结果 success: true表示成功，success:false表示失败 code表示返回码, data中为下单的orderId        |  |
stopProfitResult | json格式  | 止盈订单下单结果，数据格式同上 |  |
stopLossResult   | json格式  | 止损订单下单结果，数据格式同上 |  |

```json
{
  "success":true,
  "code":200,
  "message":null,
  "data":{
    "result":{
        "success":true,
        "code":200,
        "message":null,
        "data": {
          "orderId": 1000000,
          "positionId": 1000001,
          "orderID" : 1000000,
          "fee": 0,
          "profit": 0,
          "profitTakeoffAmount": 0
        }
      },
    "stopProfitResult":null,
    "stopLossResult":null
    }
  }
```


## 下单（止盈止损）

### HTTP Request

`POST /trade/api/order/v2/place-profit-loss`

> 下单请求参数示例

```json
{
  "symbol": "BTCUSD",
  "direct": "SELL",
  "positionId": "100000",
  "clientOrderID": "test-321",
  "clientPrice": "50000",
  "stopProfitPrice" : "60000",
  "stopProfitAmount" : "0.1",
  "stopLossPrice" : "55000",
  "stopLossAmount" : "0.1"
}
```

### Required Permission

`Trade`

### Request Body(Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol           | string | yes |    | 交易对     |  |
direct           | string | yes |    | 交易方向   |  BUY买入, SELL卖出|
positionId       | long   | yes |    | 开仓0，加仓，减仓，平仓时需要传入要执行的当前仓位ID |  0 |
clientOrderId    | string | no  |    | clientOrderId , which given by user | Valid character, A-Z,a-z,0-9,_,- length <= 128|
clientPrice      | string | yes |    | 指数价格   |  0 |
stopProfitPrice  | string | no  |    | 止盈价     |  0 |
stopProfitAmount | string | no  |    | 止盈数量   |  0 |
stopLossPrice    | string | no  |    | 止损价     |  0 |
stopLossAmount   | string | no  |    | 止损数量   |  0 |


### 返回值
字段 | 类型 | 描述 | Value 举例 |
--------- | ----------- | -----------| ----------| 
stopProfitResult | string  | 止盈订单下单结果，空字符串表示成功，非空表示失败 |  |
stopLossResult   | string  | 止损订单下单结果，空字符串表示成功，非空表示失败 |  |

```json
{
  "success":true,
  "code":200,
  "message":null,
  "data":{
    "stopProfitResult":"",
    "stopLossResult":""
    }
  }
```


## 一键平仓
 - 支持一键平掉用户的所有仓位
 - 支持一键平掉用户在某个交易对上的所有仓位
 - 支持一键平掉某一个指定仓位

### HTTP Request

`POST /trade/api/order/v2/close-all`

> 平掉 BTC上的指定仓位

```json
{
  "symbols": ["BTCUSD"],
  "positionId": 10000001
}
```


> 平掉 BTC上的所有仓位

```json
{
  "symbols": ["BTCUSD"]
}
```

> 平掉 BTC,ETH上的所有仓位

```json
{
  "symbols": ["BTCUSD","ETHUSD"]
}
```

> 平掉用户的所有仓位

```json
{
  
}
```

### Required Permission

`Trade`

### Request Body(Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbols          | array  | no |    | 交易对     |  |
positionId       | long   | no |    | 仓位Id |  0 |


### 返回值
字段 | 类型 | 描述 | Value 举例 |
--------- | ----------- | -----------| ----------| 

```json
{
  "success":true,
  "code":200,
  "message":null,
  "data": ""
```

## 单个订单查询

根据订单orderId或者clientOrderId获取订单数据

### HTTP Request

`GET /trade/api/order/v2/order`

### Required Permission

`ReadOnly`

### 请求参数

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
orderId        | long   | no |    |  订单的OrderId|  |
clientOrderId  | string | no |    |  订单的clientOrderId|  |


### 返回值
字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
orderID             | long       | 订单ID|  |
clientOrderID       | string     | 订单ClientOrderId|  |
positionId          | long       |  0 表示没有对应的仓位ID 例如挂条件单 |   |
symbol              | string     | 交易对|  |
direct              | string     | BUY买入，SELL卖出 |  |
type                | int        | 1 市价单 2 条件单（<=触发价）3 条件单（>=触发价） 4 强平单 5 强制减仓单 9 多仓止盈单 10 空仓止盈单 11 空仓止损单 12 多仓止损单  |  |
triggerPrice        | string     | 触发价 |  |
conditionalBaseAmount  | string     | 条件单、止盈止损单请求数量 |  |
executedPrice       | string     | 成交价格 |  |
executedAmount      | string     | 成交数量 |  |
executedQuoteAmount | string     | 成交金额  |  |
fee                 | string     | 手续费 |  |
status              | string     | 订单状态     NEW:新订单, EXPIRED:已过期, CANCELED:已取消, EXECUTED:已执行, REJECTED:已拒绝, FAILED:失败,PENDING:挂单未成交|  |
profit              | string     | 盈亏 |  |
timestamp           | long       | 订单成交时间 |  |

```json
{
  "code": 200,
  "data": {
      "orderID": 3432259521202880500,
      "clientOrderID": "",
      "positionId": 123323443,
      "symbol": "ETHUSD",
      "direct": "SELL",
      "executedPrice": "4170.5",
      "executedAmount": "1",
      "executedQuoteAmount": "4170.5",
      "fee": "4.1705",
      "timestamp": 1637575561094,
      "status": "EXECUTED",
      "profit": "-37.7455",
      "profitTakeoffAmount": "0"
  },
  "message": "string",
  "success": true
}
```

## 查询历史订单列表

### HTTP Request

`GET /trade/api/order/v2/all-orders`

### Required Permission

`ReadOnly`

### 请求参数

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol   | string | no |  | 交易对 |  |
orderId  | number | no | 0 | 其实订单ID |  |
startTs  | number | no |  | 开始时间 |  |
endTs    | number | no |  | 结束时间 |  |
limit    | number | no | 200 | 最大返回条数 |  |

### 返回值
字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
orderID             | long       | 订单ID|  |
clientOrderID       | string     | 订单ClientOrderId|  |
positionId          | long       | 0 表示没有对应的仓位ID 例如挂条件单 |   |
symbol              | string     | 交易对|  |
direct              | string     | BUY买入，SELL卖出 |  |
type                | int        | 1 市价单 2 条件单（<=触发价）3 条件单（>=触发价） 4 强平单 5 强制减仓单 9 多仓止盈单 10 空仓止盈单 11 空仓止损单 12 多仓止损单  |  |
triggerPrice        | string     | 触发价 |  |
conditionalBaseAmount  | string     | 条件单、止盈止损单请求数量 |  |
executedPrice       | string     | 成交价格 |  |
executedAmount      | string     | 成交数量 |  |
executedQuoteAmount | string     | 成交金额  |  |
fee                 | string     | 手续费 |  |
status              | string     | 订单状态 NEW:新订单, EXPIRED:已过期, CANCELED:已取消, EXECUTED:已执行, REJECTED:已拒绝, FAILED:失败,PENDING:挂单未成交 |  |
profit              | string     | 盈亏 |  |
timestamp           | long       | 订单成交时间 |  |

```json
{
  "code": 200,
  "data": {
      "orderID": 3432259521202880500,
      "clientOrderID": "",
      "symbol": "ETHUSD",
      "direct": "SELL",
      "executedPrice": "4170.5",
      "executedAmount": "1",
      "executedQuoteAmount": "4170.5",
      "fee": "4.1705",
      "timestamp": 1637575561094,
      "status": "EXECUTED",
      "profit": "-37.7455",
      "profitTakeoffAmount": "0"
  },
  "message": "string",
  "success": true
}
```


## 查询委托订单

### HTTP Request
`GET /trade/api/order/v2/open-orders`

### Required Permission

`ReadOnly`

### 请求参数
无

### 返回值

返回数据格式参考 `GET /trade/api/order/v2/all-orders`


## 撤单

### HTTP Request
`POST /trade/api/order/v2/cancel-order`

### Required Permission

`ReadOnly`

### 请求参数
参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
orderId        | long   | no |    |  订单的OrderId|  |
clientOrderId  | string | no |    |  订单的clientOrderId|  |

```json
{
  "orderId": 123456,
  "clientOrderId": ""
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误描述 | |
code  | string  |   错误码 | |

```json
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```


### 下单错误编码

Error Code | Description |
--------- |  ----------| 
31102  | Max OI exceeded | 
31103 | Less than min OI|
31104 | Invalid symbol|
31105 | Quantity precision error|
31106 | Price precision error|
31107 | No position for one-click liquidation |
31108 | Invalid parameter|
31109 | Max OI per trade exceeded|
31110 | Max OI per user exceeded|
31111 | System max OI per symbol exceeded|
31112 | Max OI by risk rate exceeded|
31113 | Symbol not enabled|
31201 | Can't place order when settlement |
31202 | Invalid amount |
31203 | Invalid direction |
31204 | Illegal client order ID |


# 账户API

## 查询用户UID

### HTTP Request

`GET /account/api/v2/user-id`

### Required Permission

`ReadOnly`

### 请求参数
无

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean |  true: 成功, false: 失败 | |
message | string  |  错误描述     | |
code    | string  |  错误码       | |
data    | long    |  当前用户的UID| |

```json
{
  "code": 0,
  "data": 1000000,
  "message": "string",
  "success": true
}
```


## 查询用户资产

### HTTP Request

`GET /account/api/v2/account-positions`

### Required Permission

`ReadOnly`

### 返回值

```json
{
  "success": true,
  "code": 200,
  "message": null,
  "data": {
    "tradingMarginCurrency": "USDT",    用户交易保证金类型，目前支持USDT，USDC，kUSD三种
    "positionType": 1,                  用户仓位类型 1 合仓:每个交易对只允许1个仓位  2 分仓:每个交易对可以有多个仓位
    "intradayProfit": 0.408419,         当日盈亏（已实现盈亏 + 未实现盈亏)
    "totalTradingValue": 499.072519,    交易账户总保证金
    "totalWalletValue": 1322994.310008, 资金账户总价值（每个币种数量*价格之和）
    "totalNetValue": 1323493.382527,    交易账户 + 资金账户 总价值
    "unrealizedProfit": 4.6,            当前未实现盈亏
    "intradayProfitRate": 0,            当日收益率
    "walletAccountSummary": {           资金账户每个币种详情：币种，余额
      "accounts": [
        {
          "userId": 10015805,
          "currency": "KINE",
          "available": 10000.0168,
          "freeze": 0,
          "locked": 0,
          "price": 1.435,
          "totalValue": 14350.024108
        },
        {
          "userId": 10015805,
          "currency": "kUSD",
          "available": 1298623.6389,
          "freeze": 0,
          "locked": 0,
          "price": 1,
          "totalValue": 1298623.6389
        },
        {
          "userId": 10015805,
          "currency": "USDC",
          "available": 10.647,
          "freeze": 0,
          "locked": 0,
          "price": 1,
          "totalValue": 10.647
        },
        {
          "userId": 10015805,
          "currency": "USDT",
          "available": 10010,
          "freeze": 0,
          "locked": 0,
          "price": 1,
          "totalValue": 10010
        }
      ],
      "totalValue": 1322994.310008
    },
    "tradingAccountSummary": {   交易账户每个账户详情
      "accounts": 
        {
          "symbol": "BTCUSD",           交易对
          "marginType": "CROSSED",      保证金类型 CROSSED全仓 ISOLATED逐仓
          "leverage": 10,               账户设置的杠杆
          "baseCurrency": "BTC",        基础币种
          "baseAvailable": 0,           账户中计价币种的数量（即多仓，空仓的数量之和，多仓用正数，空仓用负数，如持有1个多仓，-0.5空仓，则结果为0.5）
          "baseGrossAvailable": 0,      账户中计价币种的数量绝对值之和（即多仓，空仓的数量绝对值之和，如持有1个多仓，-0.5空仓，则结果为1.5）
          "quoteCurrency": "kUSD",      计价币种，无论是USDT，USDC，kUSD保证金，此处统一为kUSD
          "quoteAvailable": 0,          计价币种保证金数量，如果是全仓则为0，如果是逐仓则为所有此逐仓仓位的保证金之和
          "price": 57373,               当前交易对指数价格
          "initialMargin": null,        持仓情况下账户的初始保证金
          "positionMargin": null,       持仓情况下账户的仓位保证金
          "unRealizedPnl": null,        持仓情况下所有此账户下仓位的未实现盈亏
          "maintMargin": null,          持仓情况下账户所需要的维持保证金，当维持保证金低于可用保证金（marginAvailable）是账户会被强平
          "marginBalance": 452.556519,   账户的保证金余额
          "marginAvailable": 452.556519, 账户的可用保证金（可转出）
          "marginRatio": 0               维持保证金/保证金余额
        },
        {
          "symbol": "ETHUSD",
          "marginType": "ISOLATED",
          "leverage": 100,
          "baseCurrency": "ETH",
          "baseAvailable": 1,
          "baseGrossAvailable": 1,
          "quoteCurrency": "kUSD",
          "quoteAvailable": 41.916,
          "price": 4196.2,
          "initialMargin": 41.916,
          "positionMargin": 41.962,
          "unRealizedPnl": 4.6,
          "maintMargin": 20.981,
          "marginBalance": 46.516,
          "marginAvailable": 4.554,
          "marginRatio": 0.4511
        }
      ],
      "positions": [   所有仓位详情
        {
          "id": 10000312,             仓位ID
          "symbol": "ETHUSD",         交易对
          "marginType": "ISOLATED",   保证金类型 CROSSED全仓 ISOLATED逐仓
          "baseCurrency": "ETH",      基础币种
          "quoteCurrency": "kUSD",    计价币种，无论是USDT，USDC，kUSD保证金，此处统一为kUSD
          "leverage": 100,            杠杆
          "amount": 1,                持仓数量，正数表示多仓，负数表示空仓
          "avgPrice": 4191.6,         开仓均价
          "avgHoldPrice": 4195.8,     持仓均价
          "liquidationPrice": 4170.5, 强平价格
          "markPrice": 4196.2,        当前指数价格
          "profit": 4.6,              未实现盈亏
          "profitRate": 0.1098,       盈亏比例
          "initialMargin": 41.916,    初始保证金
          "positionMargin": 41.962,   仓位保证金
          "maintRate": 0.005,         维持保证金率，根据仓位市值大小会对应不同的比例，具体看https://kine.io相应的文档
          "maintMargin": 20.981,      所需要的维持保证金
          "unRealizedPnl": 4.6,       未实现盈亏
          "marginBalance": 46.516,    保证金余额
          "marginRatio": 0.4511,      维持保证金/保证金余额
          "marginAvailable": 4.554    可用保证金（可转出）
        }
      ],
      "crossedInitialMargin": 0,           所有全仓仓位的初始保证金之和
      "crossedPositionMargin": 0,          所有全仓仓位的仓位保证金之和
      "crossedMaintMargin": 0,             所有全仓仓位所需要的维持保证金之和
      "crossedUnRealizedPnl": 0,           所有全仓仓位的未实现盈亏之和
      "crossedMarginBalance": 452.556519,  全仓账户的保证金余额
      "crossedMarginRatio": 0,             全仓整体的保证金比例
      "crossedMarginAvailable": 452.556519, 全仓账户的可用保证金
      "isolatedUnRealizedPnl": 4.6,        所有逐仓仓位的未实现盈亏之和
      "isolatedMarginBalance": 46.516,     所有逐仓仓位的保证金余额之和
      "isolatedPositionMargin": 41.962,    所有逐仓仓位的仓位保证金之和
      "isolatedMarginAvailable": 4.554,    所有逐仓仓位的可用保证金之和
      "marginBalanceTotal": 499.072519,    全仓+逐仓的保证金余额之和
      "unRealizedPnlTotal": 4.6,           全仓+逐仓的未实现盈亏之和
      "positionMarginTotal": 41.962,       全仓+逐仓的保证金余额之和
      "marginAvailableTotal": 457.110519   全仓+逐仓的可用保证金之和
    }
  }
}
```

## 调整杠杆
- 分仓模式下，调整账户杠杆，只影响新的开仓，不影响已有仓位的杠杆
- 合仓模式下，会改变账户和已存在仓位的杠杆

### HTTP Request

`POST /account/api/update-leverage`

### Required Permission

`Trade`

### Request Body(Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | 交易对 | BTCUSD |
leverage | int | yes |    | 杠杆倍数, i.e 10, 50, 100 and 125 | |

```json
{
  "symbol": "string",
  "leverage": 10
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误描述 | |
code  | string  |   错误码 | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## 账户 全仓切换逐仓

全仓账户切换逐仓账户：当前账户不能有仓位和挂单

### HTTP Request

`POST /account/api/swap-to-isolated`

### Required Permission

`Trade`

### 参数 (Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例|
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | 交易对 | |

```json
{
  "symbol": "ETHUSD"
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误描述 | |
code  | string  |   错误码 | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## 账户 逐仓切换全仓

逐仓账户切换全仓账户：当前账户不能有仓位和挂单

### HTTP Request

`POST /account/api/swap-to-crossed`


### Required Permission

`Trade`

### 参数 (Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例|
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | 交易对 | |

```json
{
  "symbol": "ETHUSD"
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误描述 | |
code  | string  |   错误码 | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```


## 账户 设置合仓,分仓

- 合仓：一个交易对上最多只能有一个仓位
- 分仓：一个交易对上支持有多个仓位

### HTTP Request

`POST /account/api/set-position-type`


### Required Permission

`Trade`

### 参数 (Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例|
--------- | ------- | ------- | ----------- | -----------| ----------| 
positionType | int | yes |    | 合仓: 1, 分仓: 2 | |

```json
{
  "positionType": 2
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string   | 错误描述 | |
code    | string   | 错误码 | |
data    | boolean  | true: 成功, false: 失败 | | 

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```


## 账户 获取合仓,分仓

- 合仓：一个交易对上最多只能有一个仓位
- 分仓：一个交易对上支持有多个仓位

### HTTP Request

`GET /account/api/get-position-type`


### Required Permission

`Trade`

### 参数 (Json)

参数 | 类型 | 是否必须 | 默认值 | 描述 | 举例|
--------- | ------- | ------- | ----------- | -----------| ----------| 


### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误描述 | |
code    | int  |   错误码 | |
data    | int  |   合仓: 1, 分仓: 2 | | 

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## 账户财务历史

### HTTP Request

`GET /account/api/account-history`

### Required Permission

`ReadOnly`


```text
操作枚举
1 - DEPOSIT,
2 - WITHDRAW, 
3 - TRANSFER_WALLET_TO_TRADE, 
4 - TRANSFER_TRADE_TO_WALLET, 
5 - DEDUCT, 6 - REFUND, 
7 - TRANSFER_IN, 
8 - TRANSFER_OUT, 
9 - REWARD, 
10 - REBATE, 
11 - EXCHANGE_IN, 
12 - EXCHANGE_OUT, 
13 - FUNDING_FEE_PAY, 
14 - FUNDING_FEE_COLLECT, 
17 - FREEZE, 
18 - UNFREEZE, 
24 - WITHDRAW_REJECTED, 
30 - TRADE_MINING_JUNIOR_REWARD, 
31 - AIR_DROP, 
33 - TRADE_MINING_SENIOR_REWARD, 
34 - LOCK, 
35 - UNLOCK, 
100 - OTHERS
```

### 请求参数 

参数 | 类型 | 是否必须 | 默认值| 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
startTime | number | no | 0 | 开始时间 | |
endTime   | number | no | 0 | 结束时间 | |
currency  | string | no |  | 币种 | |
action    | string | no |  | 操作 | 操作枚举，请参考右侧示例 |
page      | number | no | 1 | 第几页 | |
size      | number | no | 20 | 每页多少条 | 最多50条 |




```http request
GET /account/api/account-history?startTime=1619798400000&endTime=1619798460000&currency=kUSD&action=1&page=2&size=10
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
userId    | number   | 用户UID | |
startTime | number   | 操作开始时间 | |
endTime   | number   | 操作结束时间 | |
currency  | string   | 币种|  |
action    | string   | 操作 |  |
amount    | number   | 金额 |  |
status    | string   | 状态 |  |


```json
{
  "success": true,
  "code": 200,
  "message": null,
  "data": {
    "page": 1,
    "size": 4,
    "total": 123,
    "items": [
      {
        "createTime": 1623722199450,
        "updateTime": 1623722199450,
        "userId": 10077439,
        "action": "EXCHANGE_OUT",
        "source": "",
        "currency": "kUSD",
        "amount": 1000.000000000000000000,
        "status": "SUCCESS"
      },
      {
        "createTime": 1623722199450,
        "updateTime": 1623722199450,
        "userId": 10077439,
        "action": "EXCHANGE_IN",
        "source": "",
        "currency": "USDC",
        "amount": 1000.000000000000000000,
        "status": "SUCCESS"
      },
      {
        "createTime": 1623722189929,
        "updateTime": 1623722189929,
        "userId": 10077439,
        "action": "TRANSFER_WALLET_TO_TRADE",
        "source": "",
        "currency": "kUSD",
        "amount": 1000.000000000000000000,
        "status": "SUCCESS"
      },
      {
        "createTime": 1623665804197,
        "updateTime": 1623665804197,
        "userId": 10077439,
        "action": "TRANSFER_TRADE_TO_WALLET",
        "source": "",
        "currency": "kUSD",
        "amount": 1000.000000000000000000,
        "status": "SUCCESS"
      }
    ]
  }
}
```

## 资金账户与交易账户之间的划转

1. 划转保证金到交易账户（全仓），即增加全仓保证金
2. 交易账户（全仓）划转到资金账户，即减少全仓保证金

### HTTP Request

`POST /account/api/transfer`

### Required Permission

`Trade`

### 请求 (Json)

参数 | 类型 | 是否必须 | 默认值| 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
amount | string | yes |    | 大于0的数字，最多支持4位小数 | |
direction | number | yes |    | 划转方向 | 1 - 划转资金账户保证金到交易账户（全仓）, 2 - 交易账户（全仓）划转到资金账户|

```json
{
  "amount": "1234.56",
  "direction": 1
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误信息 | |
code  | string  |   错误码 |  |

```json
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## 全仓交易账户与逐仓交易账户之间的划转

3. 全仓交易账户转入逐仓交易账户，即增加逐仓账户保证金，只有在逐仓有仓位情况下才可以划转
4. 逐仓交易账户转入全仓交易账户，即减少逐仓账户保证金，只有在逐仓有仓位情况下才可以划转

### HTTP Request

`POST /account/api/isolated-transfer`

### Required Permission

`Trade`

### 请求 (Json)

参数 | 类型 | 是否必须 | 默认值| 描述 | 举例 |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | 交易对对应的账户 | |
amount | string | yes |    | 大于0的数字，最多支持4位小数 | |
direction | number | yes |    |  | 3 - 全仓到逐仓, 4 - 逐仓到全仓|
positionId | long | yes |    | 要增加保证的逐仓仓位ID，或者要减少保证金的逐仓仓位ID | |

```json
{
  "symbol": "ETHUSD",
  "amount": "1234.56",
  "direction": 3,
  "positionId" : 10000
}
```

### 返回值

字段 | 类型 | 描述 | 举例 |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: 成功, false: 失败 | |
message | string  |  错误描述 | |
code  | string  |   错误码 | |

```json
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

# WebSocket API

WebSocket API 提供了市场数据，账户更新的订阅。 以topic进行订阅，订阅后将收到相应数据的更新。

为了使您快速接入，我们提供了DEMO供您参考。
https://github.com/kine-api-docs/demo

## 身份认证 (Authentication)

> Sample Code for websocket signature

```javascript

var awsAccessKeyId = "xxx";
var awsAccessKeySecret = "xxxx";

var signatureMethod = "HmacSHA256";

var timestamp = Date.now();

var requestMethod = 'GET';

var host = "api.kine.exchange";

var path = "/ws";


var payload = requestMethod.toUpperCase() + "\n" +

  host.toLowerCase() + "\n" +

  path + "\n" +

  "accessKey=" + awsAccessKeyId + "\n" +

  "" + timestamp;
;

var signatureBytes = CryptoJS.HmacSHA256(payload, awsAccessKeySecret);

var signature = CryptoJS.enc.Base64.stringify(signatureBytes);

var authMessage = {};
authMessage.op = "AUTH";
authMessage.ts = timestamp;
authMessage.data = {};
authMessage.data.accessKey = awsAccessKeyId;
authMessage.data.ts = timestamp;
authMessage.data.signature = signature;
;

console.log("app-id", awsAccessKeyId);

console.log("app-key", awsAccessKeySecret);

console.log("signature", signature);

console.log(payload);

console.log(JSON.stringify(authMessage));


```

建立websocket会话后，需要发送Auth消息进行身份认证， 认证通过后才能订阅私有数据（账户、订单更新）。否则只能订阅市场公开数据。

The signed auth message is a json message. The payload is similar with REST API.

### Auth Message 身份认证消息

> websocket auth message

身份认证消息包含使用API Key进行的签名， 签名算法请参考REST接口签名算法。

Payload以及认证消息示例， 具体请参照右侧示例。

```json
// 身份认证消息
{
  "op": "AUTH",
  "ts": 1618232922010,
  "data": {
    "accessKey": "96b514aee78f1a1ed1a24453fc8e5c52b1c04361",
    "ts": 12345678789,
    "signature": "4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o="
  }
}
```

```json
// 认证响应消息
{
  "status": "success",
  "op": "AUTH_RESULT",
  "ts": 1618805541051,
  "data": 10079469
}
```

> Payload Example

```text
--------
GET
api.kine.exchange
/ws
accessKey=xxxxxxxxxxxx
123432345
---------
```

## 存活检查 (Ping/Pong)

> Ping Message

```json
{
  "op": "PING",
  "ts": 1618232485224,
  "params": {}
}
```

> Pong Message

```json
{
  "status": "success",
  "op": "PONG",
  "ts": 1618232485337,
  "data": {
    "op": "PING",
    "ts": 1618232485224
  }
}
```

我们使用自定义的Ping/Pong消息进行存活检查。 （非websocket协议内的ping/pong）

我们同时支持双向的Ping/Pong。

1. 服务端会定时(5/s)Ping客户端，检查客户端是否存活。 

2. 客户端根据需要，也可以主动Ping服务端，检查链接是否存活。

消息格式请参考右侧示例。

<aside class="warning">
客户端收到PING后请立刻回复PONG，如不及时回复(大于15s)，服务端将主动断开链接。
</aside>

## Websocket回话限制

关于Websocket回话有一定的限制，请留意。

Limit | Limit Type | Limit Value | Deesc |
--------- | ----------- | -----------| ----------|
每个IP允许创建的最大会话数 | count  |  50  |  |
每个API Key允许创建的最大会话数| count  |  10 | 每个API KEY最多创建的且认证后的websocket会话数|
最大允许客户端发送消息的速率 | Rate  |  10/s | Incoming message rate limit per session (No limit for output)  |
会话最大存活时间 | Duration  |  24h | 超时服务端将主动断开链接，客户端请重新链接  |

## 订阅Topic

> subscribe message 订阅消息示例



```json
//订阅消息
{
  "op": "SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "{topic name}"
  }
}
```



```json
//订阅成功响应消息
{
  "status": "success",
  "op": "SUB_RESULT",
  "ts": 1618805541048,
  "data": {
    "topic": "{topic name}"
  }
}
```

> un-subscribe message

```json
//取消订阅消息
{
  "op": "UN_SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "{topic name}"
  }
}
```



```json
//取消订阅成功响应消息
{
  "status": "success",
  "op": "UN_SUB_RESULT",
  "ts": 1618805541048,
  "data": {
    "topic": "{topic name}"
  }
}
```

客户端发送订阅消息，订阅响应的数据流。

| Topic Name |  Comment |
|--- | ----|
|  md.index-price.aggregated   | 聚合价格，包括所有资产的价格更新    |
|  account-v2.all   |  账户更新 V2   |
|  order.all   |  订单更新 ， 数据格式与订单查询接口保持一致  |
|  md.kline.{symbol}.{period}   |  订阅实时K线推送 |

## 价格更新数据流

Price topic : `md.index-price.aggregated`

订阅价格更新数据流的流程:

* 客户端创建链接，并发送订阅消息。
* 服务端返回订阅成功消息
* 服务端发送最新价格 每500ms

### Message examples 消息示例

消息示例，请参考右侧。

> Subscribe the prices

```json
{
  "op": "SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "md.index-price.aggregated"
  }
}
```

> Subscribe Response

```json
{
  "status": "success",
  "op": "SUB_RESULT",
  "ts": 1618646119344,
  "data": {
    "topic": "md.index-price.aggregated"
  }
}
```

> Example of price response

```json
{
  "topic": "md.index-price.aggregated",
  "status": "success",
  "op": "DATA",
  "ts": 1618646334452,
  "data": [
    {
      "symbol": "BTCUSD",
      "price": "62120",
      "ts": 1618644164511
    },
    {
      "symbol": "RAZEUSD",
      "price": "1.59",
      "ts": 1618646262217
    },
    {
      "symbol": "ETHUSD",
      "price": "2455.3",
      "ts": 1618644164511
    },
    ....
  ]
}
```

#### Response of price

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
symbol | String  | Asset Symbol |  |
price | String  | Current price |  |
ts | long  | UTC timestamp |  |


## 账户更新V2

Topic: `account-v2.all`

V2版本的账户快照推送。
数据结构定义与/account/api/v2/account-positions接口保持一致。

你账户的快照数据将在下面两个条件触发推送更新。

1. 每3秒钟，将根据最新价格，计算后进行推送。
2. 您的资产变更后将立即推送。 例如：下单，平仓，划转等

> Subscribe account

```json
{
  "op": "SUB",
  "ts": 1618646724055,
  "data": {
    "topic": "account-v2.all"
  }
}
```

> Subscribe response

```json
{
  "status": "success",
  "op": "SUB_RESULT",
  "ts": 1618646119344,
  "data": {
    "topic": "account-v2.all"
  }
}
```

> User accounts

```json
{
  "topic": "account-v2.all",
  "status": "success",
  "op": "DATA",
  "ts": 1618990231304,
  "data": {
    <<<数据格式与/account/api/v2/account-balances接口数据保持一致，请参考其接口定义>>>
  }
}
```

## 订单更新

Topic: `order.all`

订单更新消息， 每次订单状态的变化都会触发订单更新推送。

返回数据格式参考 `GET /trade/api/order/v2/all-orders`

> Subscribe Order update

```json
{
  "op": "SUB",
  "ts": 1618646724055,
  "data": {
    "topic": "order.all"
  }
}
```

> Subscribe response

```json
{
  "status": "success",
  "op": "SUB_RESULT",
  "ts": 1618646119344,
  "data": {
    "topic": "order.all"
  }
}
```

> Order

```json
{
  "topic": "order.all",
  "status": "success",
  "op": "DATA",
  "ts": 1618990231304,
  "data": {
    <<<返回数据格式参考 `GET /trade/api/order/v2/all-orders`>>>
  }
}
```

## 实时k线推送

> 实时k线推送数据
```json
{
    "topic": "md.kline.btcusd.30min",
    "status": "success",
    "op": "DATA",
    "ts": 1639975098550,
    "data": [
      1639974600000, 
      46826, 
      46953, 
      46808, 
      46844, 
      0]
}
```

Topic: `md.kline.{symbol}.{period}`

实时k线推送。

返回数据格式参考 `GET /market/api/kline`



# Reference

API对接技术支持(技术不在线时请耐心等待或创建工单) [电报群](https://t.me/kine_api_support) 

创建技术支持工单(24h内回复) [创建工单](https://github.com/kine-api-docs/zh-cn/issues/new/choose)
