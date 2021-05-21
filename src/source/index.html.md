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

## 2021-05-21

* Correct Request Headers Name

## 2021-04-20

* April 2021, Draft version

# Introduction

Welcome to the KINE API

You can use our API to access KINE API endpoints, which can get information on open interest, debt etc. from our system.

There are two types of interface, you can choose the proper one according to your scenario and preferences.
REST API

REST (Representational State Transfer) is one of the most popular communication mechanism under HTTP, each URL represents a type of resource.

It is suggested to use Rest API for one-off operation, like trading and withdraw.
WebSocket API

WebSocket is a new protocol in HTML5. It is full-duplex between client and server. The connection can be established by a single handshake, and then server can push the notification to client actively.

It is suggested to use WebSocket API to get data update, like market data and order update.

Authentication

Both API has two levels of authentication:

Public API: It is for basic information and market data. It doesn't need authentication.

Private API: It is for account related operation like trading and account management. Each private API must be authenticated with API Key.

# General Info

## API URLs

### REST API 
`https://api.kine.exchange`

### WebSocket API
`wss://api.kine.exchange/ws`

## Limits
* Introduce the common limit of API. Such as rate limit and other limit.

## Security

### Signed Endpoint Security
* `SIGNED` endpoints require additional headers in request, `KINE-API-ACCESS-KEY` , `KINE-API-TS` ,`KINE-API-SIGNATURE`.
* `Signature` use `HMAC SHA256` signatures. The HMAC SHA256 signature is a keyed HMAC SHA256 operation. Use your secretKey as the key and all parameters as the value for the HMAC operation. More detail refer to `Authentication` section.


### Timing Security

`KINE-API-TS`

* A `SIGNED` endpoint requires a timestamp in headers, to be sent which should be the millisecond timestamp of when the request was created and sent.

### How to generate your API keys?

please contact support@kine.io to generate your API Keys(`AccessKey`, `SecretKey`).

`AccessKey` It is used in API request

`SecretKey` It is used to generate the signature (only visible once after creation, please keep it safe!!!)

API Key Permission

* `ReadOnly` permission: It is used to query the data, such as order query, trade query.
* `Trade` permission: It is used to create order, cancel order and transfer, etc.  It's include ReadOnly permission.
* `Withdraw` permission: It is used to create withdraw order, cancel withdraw order, etc.  It's include Trade/ReadOnly permission.

<aside class="warning">
Warning: 
These two keys are important to your account safety, please don't share both of them together to anyone else. If you find your API Key is disposed, please remove it immediately. 
</aside>

### How to sign the endpoint?

Please refer to below Authentication selection.

# Authentication

The API request may be tampered during internet, therefore all private API must be signed by your API Key (Secrete Key).

Each API Key has permission property, please check the API permission, and make sure your API key has proper permission.

A valid request consists of below parts:

* `Request Method`  GET/POST  (Websocket is GET)
* `host` api.kine.exchange
* `Request Path`  for example: `/trade/api/order/place`
* `accessKey`  The 'Access Key' in your API Key, should be included in request header. `KINE-API-ACCESS-KEY`
* `SecretKey`  Used to sign the request
* `ts`  The UTC timestamp when the request is sent. should be included in request header. `KINE-API-TS`
* `Request Parameters` Each API Method has a group of parameters, you can refer to detailed document for each of them.
    For both GET/POST request, all the parameters must be signed.
* `Payload` the content to be signed. It includes request info. MUST follow the rule to prepare the payload when calculate the signature.
* `signature` The value after signed, it is guarantee the signature is valid, and the request is not be tempered.  `KINE-API-SIGNATURE`


## Signature

> Sample Code for signature

```java
        String accessKey = "xxx";
        String secretKey = "xxx";

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

1. Request `Headers` for signed request.

* `KINE-API-ACCESS-KEY`    The access key
* `KINE-API-TS`           The UTC timestamp
* `KINE-API-SIGNATURE`    The signature

2. Calculate the signature and add it to request header

signature = sign(payload, secretKey);   `MUST follow the payload format rule below.`

**sign() is the method to calculate the signature.(The HMAC SHA256 signature is a keyed HMAC SHA256 operation.)**

Please refer to sample code.


## Payload

> Payload Example

```text
--------
GET
api.kine.exchange
/trade/api/history
clientOrderId=123
123123123123
---------
```

> signed request sample

```text
GET https://api.kine.exchange//trade/api/history?clientOrderId=123

Headers:
KINE-API-TS: 1618561349256
KINE-API-ACCESS_KEY: xxxxxxx
KINE-API-SIGNATURE: xxxxxxxxxx+jJLJwYSxz7iMbA=
```

{requestMethod}\n

{host}\n

{request path}\n

{request parameters}\n    #If no paramters, keep it as empty line

{timestamp}

<aside class="notice">
Websocket Request Method should be GET
</aside>

# Market Data API

The market data api is open api.

## Asset Price

This endpoint retrieves asset realtime prices. The data will update every second.

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
estifundingRate | string  | current funding rate |  |
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

`Write`

### Request Body(Json)

Parameter | DataType | Required | Default Value | Description | Value Range |
--------- | ------- | ------- | ----------- | -----------| ----------| 
symbol | string | yes |    | symbol |  |
amt | string | yes |    | amount |  |
direct | string | yes |    | direction of trade |  BUY, SELL|
closePosition | boolean | yes |    | close position of curreny asset to zero |  true, false|
clientOrderId | string | no |    | clientOrderId , which given by user | Valid character, A-Z,a-z,0-9,_,-   length <= 128|

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

This endpoint retrieve the history of order by given clientOrderId
The latest order will be returned if orders have duplicated client order IDs.

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

## Query Account Balance

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
markValue | number | market value | |
profit | number | profile | |

```json
{
  "code": 0,
  "data": {
    "crossEquity": 0,
    "crossLeverage": 0,
    "crossMarginAccounts": [
      {
        "amt": 0,
        "avgPrice": 0,
        "markValue": 0,
        "profit": 0,
        "profitRate": 0,
        "symbol": "string"
      }
    ],
    "isolatedEquity": 0,
    "isolatedMarginAccounts": [
      {
        "amt": 0,
        "avgPrice": 0,
        "kUSDAmt": 0,
        "leverage": 0,
        "liquidationPrice": 0,
        "markValue": 0,
        "profit": 0,
        "profitRate": 0,
        "symbol": "string"
      }
    ],
    "totalEquity": 0,
    "walletAccounts": [
      {
        "amt": 0,
        "currency": "string",
        "equity": 0
      }
    ],
    "walletEquity": 0
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

`Write`

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
code  | string  |  description of the code | |

```
{
  "code": 0,
  "data": true,
  "message": "string",
  "success": true
}
```


# WebSocket API

WebSocket API provide market price, order , account update data stream subscription.

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


var payload = requestMethod.toUpperCase()+"\n"+

            host.toLowerCase()+"\n"+

            path+"\n"+

            "accessKey=" + awsAccessKeyId + "\n" +
            
            "" + timestamp ;
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

The websocket api authentication required a signed auth message. After verified the signed auth message, the websocket session will be marked at authed session.
So that, it can subscribe projected stream(Order/Account).

The signed auth message is a json message. the payload is similar with REST API.

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

Response:
{"status":"success","op":"AUTH_RESULT","ts":1618805541051,"data":10079469}
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
{"op":"PING","ts":1618232485224,"params":{}}
```

> Pong Message

```json
{"status":"success","op":"PONG","ts":1618232485337,"data":{"op":"PING","ts":1618232485224}}
```

It's a customized ping/pong message, rather than websocket protocol ping/pong frame.

We support both direction ping/pong, Server ping and Client ping.
Once the client/server received the ping message, must reply a pong message ASAP.


<aside class="warning">
MUST reply pong when you received the ping.
Websocket server sent ping to client every `5s`, if don't receive pong more than `15s`, then server will close the session.
</aside>


## Session limit

There are some limit about the websocket session.

Limit | Limit Type | Limit Value | Deesc |
--------- | ----------- | -----------| ----------|
Count per IP | count  |  50  | Session count limit per IP |
Count per AccessKey | count  |  10 | Authed Session count limit per accessKey |
Msg Rate per session | Rate  |  10/s | InComing Msg Rate limit per session, No limit for output  |
Max live time | Duration  |  24h | Session max live time, server will close the session once it's expired  |


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

{"status":"success","op":"SUB_RESULT","ts":1618805541048,"data":{"topic":"{topic name}"}}
```

> un-subscribe message

```json
{
	"op": "UNSUB",
	"ts": 1618646119181,
	"data": {
		"topic": "{topic name}"
	}
}

{"status":"success","op":"UNSUB_RESULT","ts":1618805541048,"data":{"topic":"{topic name}"}}
```

Client send subscribe message to subscribe data stream. 

| Topic Name |  Comment |
|--- | ----|
|  md.index-price.aggregated   | Aggregated Price, witch include all asset price    |
|  account.all   |  Account Update   |

## Price Data Stream

Price topic : `md.index-price.aggregated`


To establish the price consuming, the process as below:

  * client initate the connection by send the subscribe message out
  * server will response the success if the request accepted
  * prices will send to client from server to client every 500ms.

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
	"data": [{
		"symbol": "BTCUSD",
		"price": "62120",
		"ts": 1618644164511
	}, {
		"symbol": "RAZEUSD",
		"price": "1.59",
		"ts": 1618646262217
	}, {
		"symbol": "ETHUSD",
		"price": "2455.3",
		"ts": 1618644164511
	},
	....
	]
}
```
####  Response of price

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
symbol | String  | Asset Symbol | Asset Reference |
price | String  | Current price |  |
ts | long  | UTC timestamp | Asset Reference |

## Account Update

Topis: `account.all`

the accounts will be published to client on two behaviours 

1. price change, all accounts will be published to client every 3 seconds, recalculated by lastest price.
2. asset change, some user activites , such as place an order, transfer/withdraw assets, will leads to a accounts publish.

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
		"walletAccounts": [{
			"currency": "KINE",
			"amt": "57.28470000",
			"equity": "161.47164911"
		}, {
			"currency": "kUSD",
			"amt": "0.00006000",
			"equity": "0.00006000"
		},...],
		"crossLeverage": "1.00000000",
		"crossEquity": "2872.90626000",
		"crossMarginAccounts": [{
			"symbol": "BTCUSD",
			"amt": "0.00000000",
			"markValue": "0.00000000",
			"profit": null,
			"profitRate": null,
			"avgPrice": null
		}, ...],
		"isolatedEquity": "0.00000000",
		"isolatedMarginAccounts": [{
			"symbol": "XRPUSD",
			"amt": "0.00000000",
			"markValue": "0.00000000",
			"avgPrice": null,
			"profit": null,
			"profitRate": null,
			"kUSDAmt": "0.00000000",
			"leverage": null,
			"liquidationPrice": null
		}, ..]
	}
}
```

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------| 
totalEquity | Number  |  walletEquity + crossEquity + isolatedEquity |  |
walletEquity | Number  |  walletEquity |  |
crossEquity | Number  |  crossEquity |  |
crossLeverage | Number  |  crossLeverage |  |
isolatedEquity | Number  |  walletEquity |  |
walletAccounts | List<WalletAccount>  |  Wallet Account Detail for all asset |  |
crossMarginAccounts | List<CrossMarginAccount>  |  Cross Margin Account Detail for all asset |  |
isolatedMarginAccounts | List<IsolatedMarginAccount>  |  Isolated Margin Account Detail for all asset |  |

**WalletAccount**

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------|
currency | String  | The currency  | Asset Reference |
amt | Number  |  The amount |  |
equity | Number  |  The equity |   |

**CrossMarginAccount**

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------|
symbol | String  |  The symbol of the asset | Asset Reference |
amt | Number  |  The amount |  |
markValue | Number  |  The markValue |  |
profit | Number  |  The profit |  |
profitRate | Number  |  The profitRate |  |
avgPrice | Number  |  The avgPrice |  |

**IsolatedMarginAccount**

Field | DataType | Description | Value Range |
--------- | ----------- | -----------| ----------|
symbol | String  |  The symbol of the asset | Asset Reference |
amt | Number  |  The amount |  |
markValue | Number  |  The markValue |  |
profit | Number  |  The profit |  |
profitRate | Number  |  The profitRate |  |
kUSDAmt | Number  |  The kUSD Amount |  |
leverage | Number  |  The leverage |  |
liquidationPrice | Number  |  The liquidationPrice |  |


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
