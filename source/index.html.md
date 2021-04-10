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

# Introduction

Welcome to the KINE API! You can use our API to access KINE API endpoints, which can get information on open interest, debt etc. from our system.


# Authentication

> To authorize, use this code:

```java
import com.kine;




```

```python
import kine

api = kine.authorize('yourkeys')
```


> Make sure to replace `yourkeys` with your API key.



`Authorization: yourkeys`

<aside class="notice">
You must replace <code>yourkeys</code> with your personal API key.
</aside>

# General System Information

## Get All 

```java
import com.kine;

```

```python
import kine

api = kittn.authorize('yourkeys')
api.generalInfo.get()
```


> The above command returns JSON structured like this:

```json
{
"underlying":[
  {
    "symbol": "BTCKUSD",
    "price": "31234.456",
    "position": "220.96"
   },
  {
    "symbol": "ETHKUSD",
    "price": "1234.456",
    "position": "-200.56",
  }
],
"MCD":{
    "supply": "643.55",
    "price":"2.1",
    "priceUnit": "kUSD"
},
"timestamp":  1609314639576

}
```

This endpoint retrieves prices and MCD (Multi-Collateral Debt) from system. The data will update every 30 seconds.

### HTTP Request

`GET http://kine.io/api/general`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_mdc | false | If set to true, the result will also include MCD values.


<aside class="success">
Remember — This is a draft version of API. 
</aside>


