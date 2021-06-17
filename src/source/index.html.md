---
title: API Reference

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

# Change Log

## 2021-06-08

* Add new interfaces for market price and account actions. Add holding average price field in account detail interface.

## 2021-06-04

* Rearrange sections regarding signature and correct typos.

## 2021-05-21

* Correct request headers names.

## 2021-04-20

* Initial version.

# Introduction

Welcome to the KINE API.

You can use these APIs introduced in this document to access KINE API endpoints, which provide access to place orders,
and information on your open interest, debt etc. in our system.

We provide two types of interfaces, REST and WebSocket, and you can choose either one according to your scenario and
preferences.

## REST API

It is suggested to use Rest API for one-off operation, like placing order and withdraw.

## WebSocket API

It is suggested to use WebSocket API to get data update, like market data and account status update.

## Authentication

Both REST and WebSocket APIs involve two levels of security types:

Public: For basic information and market data, and do not need authentication.

Private: For account related operation like placing order and account management. Each private API endpoint must be
authenticated with API keys of corresponding permission, i.e., `SIGNED`.

# General Info

## API URLs

### REST API

`https://api.kine.exchange`

### WebSocket API

`wss://api.kine.exchange/ws`

## Limits

For each REST endpoints there is a rate limit, which will be introduced along with its definition. If requests from the
same IP (for open APIs), or the same user ID (for signed APIs) exceeds the limit, an HTTP 429 response will be returned
to the caller.

For websocket connections, there are limits to connections, message rate, and max live time. Please see
the `WebSocket API` - `Session limit` section for details.

## Security

### Signed Endpoint Security

`SIGNED` endpoints require additional headers in request: `KINE-API-ACCESS-KEY`, `KINE-API-TS`,`KINE-API-SIGNATURE`.

* `KINE-API-ACCESS-KEY` should be your API key with proper permission.
* `KINE-API-SIGNATURE` shoule be the `HMAC SHA256` signature. The HMAC SHA256 signature is a keyed HMAC SHA256
  operation. Use your `SecretKey` as the key and all parameters as the value for the HMAC operation. More detail refer
  to `Authentication` section.
* `KINE-API-TS` should be the millisecond timestamp of when the request was created and sent.

### How to generate your API keys?

Please go to API management page of Kine exchange (https://kine.exchange/account/api-key) to generate your API keys.

`API Key` is used in an API request for identification.

`SecretKey` is used to generate the signature. (Only visible once after creation. Please keep it safe!!!)

API Key Permissions

* `ReadOnly` permission grants the access to data queries, such as order history and account status.
* `Trade` permission allows API key to place orders, cancel order, transfer assets between accounts etc.

[comment]: <> (* `Withdraw` permission: It is used to create withdraw order, cancel withdraw order, etc. It's include Trade/ReadOnly permission)

<aside class="warning">
Warning: 
These two keys are important to your account safety. Please don't share them with anyone else. If you find your API Key exposed, please remove it from your account immediately. 
</aside>

### How to sign the endpoint?

Please refer to Authentication section below.

# Authentication

The API request may be tampered during traveling through internet. Therefore, all calls to private API endpoints must be
signed with your API key (secrete key).

Each API key would be assigned with a permission during application. Please check the permission, and make sure your API
key has proper permission to access the target API endpoints.

A valid signature for a request is generated based on the following information:

* `Request Method`  GET/POST  (for Websocket authentication please use GET)
* `Host` The host of the API endpoints, for now it is only `api.kine.exchange`
* `Request path` For example: `/trade/api/order/place`
* `API key`  Your API key, which should also be included in request header `KINE-API-ACCESS-KEY` as mentioned above
* `SecretKey`  Used to sign the request
* `Timestamp` The UTC timestamp when the request is sent, which should also be included in request header `KINE-API-TS`
  as mentioned above
* `Request Parameters` The parameters of the request. For both GET and POST request, all the parameters must be included
  in signing.
* `Payload` The actual content to be signed. MUST be organized following the rule instructed in the `Payload` section
  below.
* `Signature` The generated hash value after signing, which will be verified at server side to guarantee the request has
  not been tempered, and should be included in request header  `KINE-API-SIGNATURE`

## Signature

The signature is calculated according to a particular algorithm, based on the information mentioned above.

### 1. Prepare the payload, in the format to right side.

```text
{requestMethod}\n

{host}\n

{request path}\n

{request parameters}\n #If no paramters, keep it as empty line

{timestamp}
```

<aside class="notice">
For a websocket request, method should be GET
</aside>

> Payload Example

```text
GET
api.kine.exchange
/trade/api/history
clientOrderId=123
123123123123
```

### 2. Generate the signature.

```signature = sign(payload, secretKey);```

**sign() is the method to calculate the signature. (HMAC SHA256)**

Please refer to sample code.

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

> signed request sample

```text
GET https://api.kine.exchange/trade/api/history?clientOrderId=123

Headers:
KINE-API-TS: 1618561349256
KINE-API-ACCESS_KEY: 072285552fb24cf49412345688888888
KINE-API-SIGNATURE: xxxxxxxxxx+jJLJwYSxz7iMbA=
```

# Market Data API

Market data APIs are all with open access.

## Asset Price

This endpoint retrieves asset prices, which are updated every second.

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

## Aggregated Asset Prices

This endpoint retrieves aggregated mark prices of all assets, which are updated every second.

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
amt | string | yes |    | amount |  |
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
    "clientOrderID": "test-123",
    "direct": "SELL",
    "executedAmount": 2,
    "executedPrice": 60321,
    "executedQuoteAmount": 120642,
    "fee": 12.0642,
    "orderID": 1234567890123,
    "status": 4,
    "symbol": "BTCUSD",
    "timestamp": 1618531200000
  },
  "message": "string",
  "success": true
}
```

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

## Max and min target isolated risk rate

Get the max and min target risk rate, when switch the trading account of a given currency to isolated

### HTTP Request

`GET /account/api/get-riskrate-when-swap-to-isolated`

### Rate Limit

5/s

### Required Permission

`ReadOnly`

### Request Parameters

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol of the target currency| |

```http request
GET /account/api/get-riskrate-when-swap-to-isolated?symbol=BTCUSD
```

### Response Content

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
success | boolean  | true: the request be processed, false: fail | |
message | string  |  description of the code | |
code  | string  |  | |
data  | map  | min and max risk rate | |

```json
{
  "code": 200,
  "data": {
    "minRiskRate": "0.1",
    "maxRiskRate": "0.85"
  },
  "message": "string",
  "success": true
}
```

## Leverage Adjustment

Change leverage of isolated positions

### HTTP Request

`POST /account/api/update-isolated-target-leverage`

### Rate Limit

5/s

### Required Permission

`Trade`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | the symbol | |
targetLeverage | number | yes |    | the target leverage | |

```json
{
  "symbol": "string",
  "targetLeverage": 0
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
targetLeverage | number | yes |    | the target leverage | |

```json
{
  "symbol": "ETHUSD",
  "riskRate": "0.85"
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

WebSocket API provide market price, order, account update data stream subscription.

## Authentication

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

A websocket session requires a signed `AUTH` message as its first message. With this signed auth message verified, the
websocket session will be marked as authorised, after which, it can subscribe projected streams (Order/Account).

The signed auth message is a json message. The payload is similar with REST API.

### Auth Message

> websocket auth message

```json
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

Response:

```json
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

The signature is same as Rest API. The auth message refer to the example.

## Live check (Ping/Pong)

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

We use a customized ping/pong message, rather than websocket protocol ping/pong frame.

We support both direction ping/pong, server ping and client ping. Once the client/server received the ping message, they
must reply a pong message ASAP.


<aside class="warning">
MUST reply a pong when you received a ping.
Websocket server sends ping to client every `5s`. If no pong message is received within `15s`, the session will be closed from server side.
</aside>

## Session limit

There are some limits about the websocket session.

Limit | Limit Type | Limit Value | Deesc |
--------- | ----------- | -----------| ----------|
Sessions per IP | count  |  50  | Max number of sessions per IP |
Sessions per API key | count  |  10 | Max number of authed sessions per API key |
Message rate per session | Rate  |  10/s | Incoming message rate limit per session (No limit for output)  |
Max live time | Duration  |  24h | Session max live time. Server will close the session once it's expired  |

## Subscribe Topics

> subscribe message

```json
{
  "op": "SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "{topic name}"
  }
}
```

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

```json
{
  "op": "UN_SUB",
  "ts": 1618646119181,
  "data": {
    "topic": "{topic name}"
  }
}
```

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

Client send subscribe message to subscribe data stream.

| Topic Name |  Comment |
|--- | ----|
|  md.index-price.aggregated   | Aggregated Price, witch include all asset price    |
|  account.all   |  Account Update   |

## Price Data Stream

Price topic : `md.index-price.aggregated`

The process to establish price consuming is:

* Client initiate the connection by sending the subscription message
* Server will respond a success message if the request is accepted
* Prices will send to client from server every 500ms.

### Message examples

Please find them on the right side.

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

## Account Update

Topic: `account.all`

The account snapshots will be published to client on two conditions

1. On price change, snapshots of all accounts will be published to client every 3 seconds, recalculated by latest
   prices.
2. Asset change. Some user activities, such as placing an order, transferring / withdrawing assets, will lead to a
   publish.

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
    "crossLeverage": "1.00000000",
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
