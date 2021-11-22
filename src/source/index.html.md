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

## 2021-11-19

* 中文文档 - 进行中

# 简介

欢迎查看KINE API文档。

您可以通过API来访问KINE平台，用于下单，查询您需要的订单、账户信息。

我们提供了两种类型的接口 REST 、 WebSocket， 您可以根据需要选择适合您的。


## REST API

建议您使用REST API来完成一次性的操作，例如下单，提币等。

## WebSocket API

建议您使用WebSocket API来获取数据更新。例如：市场数据，账户更新，订单更新等。

## 鉴权

REST 和 WebSocket 接口都有公开接口和私有接口:

公开接口: 用于获取基础数据，例如市场数据.

私有接口: 用于用户账户相关的操作。 例如下单，划转，账户余额查询。 私有接口调用需要使用API Key进行签名（`SIGNED`）以保证用户数据的安全。

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

## Security 安全

### 使用签名进行身份认证

`SIGNED` 接口调用，需要额外的Http Request Headers: 

`KINE-API-ACCESS-KEY`, `KINE-API-TS`,`KINE-API-SIGNATURE`.

* `KINE-API-ACCESS-KEY` 是您的API KEY
* `KINE-API-SIGNATURE` 是通过 `HMAC SHA256` 算法得到的签名. 签名算法请参考 `身份验证` 部分.
* `KINE-API-TS` 是当前时间的时间戳，如果时间差距太大，签名验证可能失败，请确保您的服务器时间是准确的。 `毫秒时间戳`

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

请求中，一个合法的签名是根据以下信息计算得出的:

* `Request Method`  GET/POST  (Websocket 签名请使用 GET)
* `Host` 主域名 目前只有一个 `api.kine.exchange`
* `Request path` 接口请求路径，不包含域名。 例如: `/trade/api/order/place`
* `API key`  API Key ID，需要包含在Request Header中 `KINE-API-ACCESS-KEY`
* `SecretKey`  用于计算签名的秘钥。
* `Timestamp` 毫秒时间戳 需要包含在request header `KINE-API-TS`
* `Request Parameters` 请求参数。 这里的请求参数是指HTTP请求参数，拼接在URL中的参数，而不是Request Body中的数据。 不论GET、POST请求， 签名只需要HTTP请求参数，不需要Body中的数据。
* `Payload` 将以上相关信息拼接成固定格式的文本，然后对其进行加密得到最终的签名.  格式请参考 `Payload` 章节
* `Signature` 对Payload加密得到的签名， 需要包含在 request header  `KINE-API-SIGNATURE`

## Signature 签名

### 1. 准备签名用的文本 Payload ， 请参考右侧示例

```text
{requestMethod}\n
{host}\n               
{request path}\n       ## 请求路径
{request parameters}\n ## 实际拼接在URL上？右侧的部分， 如果没有参数，请保留 `空行`。
{timestamp}            ## 发起请求时的时间戳
```

<aside class="notice">
For a websocket request, method should be GET
</aside>

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

> Sample Code for signature

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

### 3. Add request headers for signed request.

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

# Market Data API 市场数据

市场数据API是公开接口，不需要签名

## Asset Price 资产价格

查询某个资产最新价格。

### HTTP Request

`GET /market/api/price/{symbol}`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Query Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol, for example 'BTCUSD' |  |

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
symbol | string  |  |  |
price | string  |  |  |
timestamp | long  |  |  |

## Aggregated Asset Prices 全部资产价格

一次查询全部资产的最新价格。

### HTTP Request

`GET /market/api/agg-price`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
prices | Price  | (see below) |  |
ts | long  | aggregated price updated timestamp |  |

Price:

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
ts | long  | price updated timestamp |  |
symbol | string  |  |  |
price | string  |  |  |

Examples.

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

## Funding Rate

This endpoints retrieves the funding rate related information.

### HTTP Request

`GET /market/api/funding-rate/{symbol}`

### Query Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes | false   |  |  |

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
symbol | string  |  |  |
fundingRate | string  | current funding rate |  |
estifundingRate | string  | Estimated next round funding rate |  |
nextFundingTime | long  | next round funding time |  |
timestamp | long  | response time |  |

# Trade API

## Place Order

Place an order

### HTTP Request

`POST /trade/api/order/place`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | symbol |  |
amount | string | yes |    | amount |  |
direct | string | yes |    | direction of trade |  BUY, SELL|
closePosition | boolean | yes |    | close position of curreny asset to zero |  true, false|
clientOrderId | string | no |    | clientOrderId , which given by user | Valid character, A-Z,a-z,0-9,_,- length <= 128|

examples

```json
{
  "amount": "0.25",
  "clientOrderID": "test-321",
  "closePosition": false,
  "direct": "SELL",
  "symbol": "BTCUSD"
}
```

```json
{
  "clientOrderID": "test-322",
  "closePosition": true,
  "symbol": "BTCUSD"
}
```

### Response Content

refer to the sample on the right side

```json
{
  "code": 0,
  "data": {
    "clientOrderID": "string",
    "direct": "string",
    "executedAmount": 0,
    "executedPrice": 0,
    "executedQuoteAmount": 0,
    "fee": 0,
    "orderID": 0,
    "status": 0,
    "symbol": "string",
    "timestamp": 0
  },
  "message": "string",
  "success": true
}
```

## Query Order

This endpoint retrieve the history of order by given clientOrderId. The latest order will be returned if orders have
duplicated client order IDs.

### HTTP Request

`GET /trade/api/history`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Query Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
clientOrderId  | string | yes | false   |  |  |

### Response Content

```json
{
  "code": 200,
  "data": {
    "orderID": 3410874959646425217,
    "clientOrderID": "test-0622-0001",
    "symbol": "BTCUSD",
    "direct": "SELL",
    "executedPrice": "37300",
    "executedAmount": "0.005",
    "executedQuoteAmount": "186.5",
    "fee": "0.1865",
    "timestamp": 1627378607623,
    "status": "EXECUTED",
    "profit": "0"
  },
  "message": "string",
  "success": true
}
```
### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
orderID | number  | Order ID |  |
clientOrderID | string  | Client order ID |  |
symbol | string  | symbol |  |
direct | string  | direct | BUY, SELL |
executedPrice | string  | Executed price |  |
executedAmount | string  | Executed amount |  |
executedQuoteAmount | string  | Executed quote amount |  |
fee | string  | fee |  |
timestamp | number  | executed timestamp |  |
status | string  | order status | EXECUTED, FAILED |
profit | string  | estimated profit (fee not included) |  |

## Query All Orders

This endpoint retrieve the history of all orders with given conditions.

### HTTP Request

`GET /trade/api/all-orders`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Query Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol  | string | yes |  | symbol |  |
orderId  | number | no | 0 | if given, only orders with greater IDs will be queried |  |
startTs  | number | no |  | start timestamp |  |
endTs  | number | no |  | end timestamp |  |
limit  | number | no | 200 | max record returned, max value 500 |  |

### Response Content

```json
{
  "success": true,
  "code": 200,
  "message": null,
  "data": [
    {
      "orderID": 3424812602398605456,
      "clientOrderID": "",
      "symbol": "UNIUSD",
      "direct": "BUY",
      "executedPrice": "0",
      "executedAmount": "9.58",
      "executedQuoteAmount": "0",
      "fee": "0",
      "timestamp": 1634024603901,
      "status": "FAILED",
      "profit": "0"
    },
    {
      "orderID": 3424811865291620496,
      "clientOrderID": "",
      "symbol": "UNIUSD",
      "direct": "SELL",
      "executedPrice": "23.056",
      "executedAmount": "20",
      "executedQuoteAmount": "461.12",
      "fee": "0.46112",
      "timestamp": 1634024242330,
      "status": "EXECUTED",
      "profit": "0.2709"
    }
  ]
}
```
### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
orderID | number  | Order ID |  |
clientOrderID | string  | Client order ID |  |
symbol | string  | symbol |  |
direct | string  | direct | BUY, SELL |
executedPrice | string  | Executed price |  |
executedAmount | string  | Executed amount |  |
executedQuoteAmount | string  | Executed quote amount |  |
fee | string  | fee |  |
timestamp | number  | executed timestamp |  |
status | string  | order status | EXECUTED, FAILED |
profit | string  | estimated profit (fee not included) |  |

### Error code

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

# Account API

## Query Account Balances

This endpoint retrieves user's balance

### HTTP Request

`GET /account/api/account-balances`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Response Content

The content consists 3 parts, cross, isolated and total equity.

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
amt | number  | amount |  |
symbol | string | symbol| |
avgPrice| number | average price | |
avgHoldPrice| number | average holding price | |
markValue | number | market value | |
profit | number | profile | |

```json
{
  "code": 200,
  "data": {
    "crossEquity": 12345.67,
    "crossLeverage": 3.45,
    "crossMarginAccounts": [
      {
        "amt": 0.15,
        "avgPrice": 34567.8,
        "avgHoldPrice": 34578.9,
        "markValue": 6851.7,
        "profit": 1234.5,
        "profitRate": 0.3456,
        "symbol": "BTCUSD"
      },
      {
        "amt": -1.23,
        "avgPrice": 2345.67,
        "avgHoldPrice": 2367.89,
        "markValue": -3157.41,
        "profit": -123.45,
        "profitRate": -0.09,
        "symbol": "ETHUSD"
      },
      {
        "amt": 12345.6,
        "avgPrice": 1,
        "avgHoldPrice": 1,
        "markValue": 1,
        "profit": 0,
        "profitRate": 0,
        "symbol": "kUSD"
      }
    ],
    "isolatedEquity": 1234.56,
    "isolatedMarginAccounts": [
      {
        "amt": 12.34,
        "avgPrice": 345.6,
        "avgHoldPrice": 345.1,
        "kUSDAmt": 4567.8,
        "leverage": 2.34,
        "liquidationPrice": 234.5,
        "markValue": 9876.5,
        "profit": 1234.5,
        "profitRate": 0.56,
        "symbol": "BNBUSD"
      }
    ],
    "totalEquity": 23456.7,
    "walletAccounts": [
      {
        "amt": 1234.5,
        "currency": "kUSD",
        "equity": 1234.5
      }
    ],
    "walletEquity": 1234.5
  },
  "message": "string",
  "success": true
}
```


## Leverage Adjustment

Change the leverage of trading accounts. 

### HTTP Request

`POST /account/api/update-leverage`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol | |
leverage | int | yes |    | the leverage, i.e 10, 50, 100 and 125 | |

```json
{
  "symbol": "string",
  "leverage": 10
}
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |   | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## Switch to isolated

Switch to an isolated trading account for a give currency

### HTTP Request

`POST /account/api/swap-to-isolated`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol of target currency | |
leverage | int | yes |    | the leverage | |

```json
{
  "symbol": "ETHUSD",
  "leverage": 10
}
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |   | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## Switch to crossed

Switch an isolated trading account to the crossed trading account for a give currency

### HTTP Request

`POST /account/api/swap-to-crossed`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol of target currency | |

```json
{
  "symbol": "BTCUSD"
}
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |   | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## Account history

Get the history of account changes

### HTTP Request

`GET /account/api/account-history`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Request Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
startTime | number | no | 0 | starting timestamp of the querying scope | |
endTime | number | no | 0 | ending timestamp of the querying scope | |
currency | string | no |  | currency | |
action | number | no |  | | 1 - DEPOSIT, 2 - WITHDRAW, 3 - TRANSFER_WALLET_TO_TRADE, 4 - TRANSFER_TRADE_TO_WALLET, 5 - DEDUCT, 6 - REFUND, 7 - TRANSFER_IN, 8 - TRANSFER_OUT, 9 - REWARD, 10 - REBATE, 11 - EXCHANGE_IN, 12 - EXCHANGE_OUT, 13 - FUNDING_FEE_PAY, 14 - FUNDING_FEE_COLLECT, 17 - FREEZE, 18 - UNFREEZE, 24 - WITHDRAW_REJECTED, 30 - TRADE_MINING_JUNIOR_REWARD, 31 - AIR_DROP, 33 - TRADE_MINING_SENIOR_REWARD, 34 - LOCK, 35 - UNLOCK, 100 - OTHERS |
page | number | no | 1 | page number | |
size | number | no | 20 | number of items per page | 1-200 |

```http request
GET /account/api/account-history?startTime=1619798400000&endTime=1619798460000&currency=kUSD&action=1&page=2&size=10
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |  | |
data  | pagedItems  |  | |

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
userId  | number | user ID| |
startTime | number   | starting timestamp of the querying scope | |
endTime | number   | ending timestamp of the querying scope | |
currency | string  |  |  |
action | string  |  |  |
page | number   | page number | |
size | number   | number of items per page |  |

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

## Max transferable of wallet account

Get the max available amount (in kUSD) that can be transferred from wallet account

### HTTP Request

`GET /account/api/max-wallet-transfer-out`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |  | |
data  | decimal  | max available amount | |

```
{
  "code": 200,
  "data": "1324.56",
  "message": "string",
  "success": true
}
```

## Max transferable of crossed account

Get the max available amount (in kUSD) that can be transferred from the crossed trading account

### HTTP Request

`GET /account/api/max-crossed-transfer-out`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |  | |
data  | decimal  | max available amount | |

```json
{
  "code": 200,
  "data": "1324.56",
  "message": "string",
  "success": true
}
```

## Max transferable of isolated account

Get the max available amount (in kUSD) that can be transferred from a specified isolated trading account

### HTTP Request

`GET /account/api/max-isolated-transfer-out`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Request Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol of the target currency | |
positionId | long | yes |    | which position you want to transfer out | |

```http request
GET /account/api/max-isolated-transfer-out?symbol=BTCUSD
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |  | |
data  | map  | max available amount to transfer out | |

```json
{
  "code": 200,
  "data": {
    "maxTransOut": "5432.1"
  },
  "message": "string",
  "success": true
}
```

## Transfer between the wallet and crossed trading account

Transfer (in kUSD) between the wallet and crossed trading account

### HTTP Request

`POST /account/api/transfer`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
amount | string | yes |    | the amount to be transferred | |
direction | number | yes |    | the direction to be transferred | 1 - wallet to trading account, 2 - trading to wallet account|

```json
{
  "amount": "1234.56",
  "direction": 1
}
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |   | |

```json
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```

## Transfer between crossed/isolated trading accounts

Transfer (in kUSD) between the crossed trading account and a specified isolated trading account

### HTTP Request

`POST /account/api/isolated-transfer`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the isolated trading account to be transferred from or to| |
amount | string | yes |    | the amount to be transferred | |
direction | number | yes |    | the direction to be transferred | 3 - crossed to isolated trading account, 4 - isolated to crossed trading account|
positionId | long | yes |    | which position you want to transfer out | |

```json
{
  "symbol": "ETHUSD",
  "amount": "1234.56",
  "direction": 3
}
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |   | |

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

## Authentication 身份认证

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

## Live check (Ping/Pong) 存活检查

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

## Session limit 回话限制

关于Websocket回话有一定的限制，请留意。

Limit | Limit Type | Limit Value | Deesc |
--------- | ----------- | -----------| ----------|
每个IP允许创建的最大会话数 | count  |  50  |  |
每个API Key允许创建的最大会话数| count  |  10 | 每个API KEY最多创建的且认证后的websocket会话数|
最大允许客户端发送消息的速率 | Rate  |  10/s | Incoming message rate limit per session (No limit for output)  |
会话最大存活时间 | Duration  |  24h | 超时服务端将主动断开链接，客户端请重新链接  |

## Subscribe Topics 订阅Topic

> subscribe message 订阅消息示例

订阅消息
```json
{
  "op": "SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "{topic name}"
  }
}
```

订阅成功响应消息
```json
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
取消订阅消息
```json
{
  "op": "UN_SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "{topic name}"
  }
}
```

取消订阅成功响应消息
```json
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
|  account.all   |  账户更新   |

## Price Data Stream 价格更新数据流

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

## Account Update 账户更新

Topic: `account.all`

你账户的快照数据将在下面两个条件触发推送更新。

1. 每3秒钟，将根据最新价格，计算后进行推送。
2. 您的资产变更后将立即推送。 例如：下单，平仓，划转等

> Subscribe account

```json
{
  "op": "SUB",
  "ts": 1618646724055,
  "data": {
    "topic": "account.all"
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
    "topic": "account.all"
  }
}
```

> User accounts

```json
{
  "topic": "account.all",
  "status": "success",
  "op": "DATA",
  "ts": 1618990231304,
  "data": {
    "totalEquity": "3034.37796911",
    "walletEquity": "161.47170911",
    "walletAccounts": [
      {
        "currency": "KINE",
        "amt": "57.28470000",
        "equity": "161.47164911"
      },
      {
        "currency": "kUSD",
        "amt": "0.00006000",
        "equity": "0.00006000"
      },
      ...
    ],
    "crossEquity": "2872.90626000",
    "crossMarginAccounts": [
      {
        "symbol": "BTCUSD",
        "amt": "0.00000000",
        "markValue": "0.00000000",
        "profit": null,
        "profitRate": null,
        "avgPrice": null
      },
      ...
    ],
    "isolatedEquity": "0.00000000",
    "isolatedMarginAccounts": [
      {
        "symbol": "XRPUSD",
        "amt": "0.00000000",
        "markValue": "0.00000000",
        "avgPrice": null,
        "profit": null,
        "profitRate": null,
        "kUSDAmt": "0.00000000",
        "leverage": null,
        "liquidationPrice": null
      },
      ...
    ]
  }
}
```

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
totalEquity | Number  |  walletEquity + crossEquity + isolatedEquity |  |
walletEquity | Number  |  wallet equity |  |
crossEquity | Number  |  cross equity |  |
crossLeverage | Number  |  leverage cross account |  |
isolatedEquity | Number  |  wallet equity |  |
walletAccounts | List<WalletAccount>  |  Wallet account details for all assets |  |
crossMarginAccounts | List<CrossMarginAccount>  |  Cross margin account details for all assets |  |
isolatedMarginAccounts | List<IsolatedMarginAccount>  |  Isolated margin account details for all assets |  |

**WalletAccount**

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------|
currency | String  | currency  | Asset Reference |
amt | Number  |  amount |  |
equity | Number  |  equity |   |

**CrossMarginAccount**

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------|
symbol | String  |  symbol of the asset | Asset Reference |
amt | Number  |  amount |  |
markValue | Number  | current mark value |  |
profit | Number  |  profit |  |
profitRate | Number  |  profit rate |  |
avgPrice | Number  |  average price |  |
leverage | Number  |  leverage |  |
liquidationPrice | Number  |  liquidation price |  |
initialMargin | Number  |  position initial margin |  |
positionMargin | Number  |  position margin |  |
maintMargin | Number  |  maintenance margin |  |
marginBalance | Number  |  margin balance |  |
marginRatio | Number  |  margin ratio |  |
marginAvailable | Number  | available margin |  |

**IsolatedMarginAccount**

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------|
symbol | String  |  symbol of the asset | Asset Reference |
amt | Number  |  amount |  |
markValue | Number  |  mark value |  |
profit | Number  |  profit |  |
profitRate | Number  |  profit rate |  |
kUSDAmt | Number  |  kUSD Amount |  |
leverage | Number  |  leverage |  |
liquidationPrice | Number  |  liquidation price |  |
initialMargin | Number  |  position initial margin |  |
positionMargin | Number  |  position margin |  |
maintMargin | Number  |  maintenance margin |  |
marginBalance | Number  |  margin balance |  |
marginRatio | Number  |  margin ratio |  |
marginAvailable | Number  | available margin |  |

# Reference

## Asset Reference

Asset Symbol | Base Currency | Quote Currency | Comment |
--------- | ----------- | -----------| ----------| 
BTCUSD | BTC  | kUSD | xx |
ETHUSD | ETH  | kUSD | xx |
LTCUSD | LTC  | kUSD | xx |
FILUSD | FIL  | kUSD | xx |
XRPUSD | XRP  | kUSD | xx |
LINKUSD | LINK  | kUSD | xx |
UNIUSD | UNI  | kUSD | xx |
COMPUSD | COMP  | kUSD | xx |
SNXUSD | SNX  | kUSD | xx |

[]: https://kine.exchange/account/api-key
