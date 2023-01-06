---
title: title

language_tabs: # must be one of https://git.io/vQNgJ


toc_footers:
  - <a href='https://www.vaex.com'>Sign Up for Vaex</a>

includes:

search: true
---

# General

## Introduction

Welcome to Vaex’s trader and developer documentation. These documents outline the exchange functionality, market details, and APIs.

The whole documentation is divided into two parts: REST API and Websocket feed.

The REST API contains four sections: User(private) , Trade(private), Market Data(public) and Others(public).

The WebSocket contains two sections: Public Channels and Private Channels

## Upcoming Changes

To get the latest updates in API, you can click ‘Watch’ on our [Vaex Docs Github](https://github.com/VaexExchange/vaex-api-docs).


**11/01/22**:

- Add API Document


## Reading Guide

2. Read [REST API](#rest-api) to learn how to build a request.
3. Read [Time](#server-time) if you want to make a test request (and receive a sample response) without having to authorize.
4. Read [Service Status](#service-status) to maintain your trading strategy based on service status
5. Read [Authentication](#authentication) to learn how to make an authorized request.
6. Read [Inner Transfer](#inner-transfer) to see how to transfer assets.
7. Read [List Accounts](#list-accounts) to learn how to get the data of your account balance.
8. Read [Place a new order](/#place-a-new-order) to see how to place an order.
9. Read [Order Book](#get-part-order-book-aggregated) to get a snapshot of the order book.
10. Read [Websocket Feed](#websocket-feed) to learn how to establish a websocket connection.
11. Read [Level-2 Market Data](#level-2-market-data) to see how to build a local real-time order book with websocket.
12. Read [Account balance notice](#account-balance-notice) to see how to get a private websocket feed and get real time notice of balance changes.


## Matching Engine

### Order Lifecycle

Valid orders sent to the matching engine are confirmed immediately and are in the **received** state. If an order executes against another order immediately, the order is considered **done**. An order can execute in part or whole. Any part of the order not filled immediately, will be considered **open**. Orders will stay in the open state until canceled or subsequently filled by new orders. Orders that are no longer eligible for matching (filled or canceled) are in the **done** state.

### Self-Trade Prevention

**Self-Trade Prevention** is an option in advanced settings.It is not selected by default. If you specify STP when placing orders, your order won't be matched by another one which is also yours. On the contrary, if STP is not specified in advanced, your order can be matched by another one of your own orders. It should be noted that only the taker's protection strategy is effective.


#### DECREMENT AND CANCEL(DC)

**Market orders are currently not supported for DC**. When two orders from the same user cross, the smaller order will be canceled and the larger order will be decremented by the size of the smaller order. If the two orders are the same size, both will be canceled.

#### CANCEL OLDEST(CO)

Cancel the older (resting) order in full. The new order continues to execute.

#### CANCEL NEWEST(CN)

Cancel the newer (taking) order in full. The old resting order remains on the order book.

#### CANCEL BOTH(CB)

Immediately cancel both orders.



## Client Libraries

Client libraries can help you integrate with our API quickly.

**Official SDK of Vaex**

- [Java SDK（None）](https://github.com/VaexExchange/vaex-java-sdk)


CCXT is our authorized SDK provider and you may access the Vaex API through CCXT. For more information, please visit: [https://ccxt.trade](https://ccxt.trade).


**Examples**

- PHP File Example (GET & POST & DELETE)  [Github Link](https://github.com/VaexExchange/vaex-api-docs/tree/master/examples/php)

- - -



## Request Rate Limit

When a rate limit is exceeded, a status of **429** will be returned.
<aside class="notice">Once the rate limit is exceeded, the system will restrict your use of your IP or account for 10s.</aside>

###REST API

The limit strategy of private endpoints will restrict account by userid. The limit strategy of public endpoints will restrict IP.
<aside class="notice">Note that when an API has a specific rate limit, please refer to the specific limit.</aside>

###WEBSOCKET


### Number of Connections
Number of connections per user ID:   ≤ 50

### Connection Times
Connection Limit: 30 per minute


### Number of Uplink Messages
Message limit sent to the server: 100 per 10 seconds


### Topic Subscription Limit
Maximum number of batch subscriptions at a time: 100 topics

Subscription limit for each connection: 300 topics


## FAQ

### Invalid Signature


* Check if your API-KEY, API-SECRET and API-PASSPHRASE are correct
* Check if the string sequence is correct: timestamp + method + requestEndpoint + body
* Check whether the timestamp in header is the same with the content above
* Check whether you are using the correct encoding in your signature, e.g. base64
* Check whether the **GET** request is submitted as a form
* Check whether the content-type of your POST request is in application/json form and is encoded by charset=utf-8

### Apply Withdraw
* memo


For currencies without memo, the memo field is not required. Please do not pass the parameter when you are applying to withdraw via API, or the system will return: **vaex incorrect withdrawal address**.<br/>

* amount


The precision of the **amount** field shall satisfy the withdrawal precision requirements of the currency. The precision requirements for the currencies can be obtained by [Withdrawals Quotas](#get-withdrawal-quotas).
The withdrawal amount must be an integer multiple of the withdrawal accuracy. If the withdrawal accuracy is 0, the withdrawal amount it can only be an integer.



### WebSocket
* Subscription limit for each connection: 300 topics;
* Validity period for token: 24 hours;
* Number of connections per user: ≤ 50
* Number of Messages of the client side: 100 per 10 seconds;
* Subscribing one symbol means subscribing a topic; (e.g.Topic: /market/level2:{symbol},{symbol}...)

### 403  Error

If an error occurs as follows:
403 "The request could not be satisfied. Bad Request" from Amazon CloudFront<br/>

* Check whether the request is HTTPS
* Remove the RequestBody from the GET request





# REST API

## Base URL

The REST API has endpoints for account and order management as well as public market data.

The base url is **https://api.vaex.com**.  

The request URL needs to be determined by BASE and specific endpoint combination.

## Endpoint of the Interface


Each interface has its own endpoint, described by field **HTTP REQUEST** in the docs.

For the **GET METHOD** API, the endpoint needs to contain the query parameters string.

E.G. For "List Accounts", the default endpoint of this API is **/api/v1/accounts**. If you pass the "currency" parameter(BTC), the endpoint will become **/api/v1/accounts?currency=BTC** and the final request URL will be **https://api.vaex.com/api/v1/accounts?currency=BTC**.


## Request
All requests and responses are **application/json** content type.  

Unless otherwise stated, all timestamp parameters should in milliseconds. e.g. **1544657947759**

### PARAMETERS

For the **GET, DELETE** request, all query parameters need to be included in the request url. (**e.g. /api/v1/accounts?currency=BTC**)

For the **POST, PUT** request, all query parameters need to be included in the request body with JSON. (**e.g. {"currency":"BTC"}**). **Do not include extra spaces in JSON strings.**

### Errors

When errors occur, the HTTP error code or system error code will be returned. The body will also contain a message parameter indicating the cause.

#### HTTP error status codes

```
{
  "code": "400100",
  "msg": "Invalid Parameter."
}

```

Code | Meaning
---------- | -------
400 | Bad Request -- Invalid request format.
401 | Unauthorized -- Invalid API Key.
403 | Forbidden or Too Many Requests -- The request is forbidden or Access limit breached.
404 | Not Found -- The specified resource could not be found.
405 | Method Not Allowed -- You tried to access the resource with an invalid method.
415 | Unsupported Media Type. You need to use: application/json.
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.


#### System error codes

| Code | Meaning |
| ---------- | ------- |
| 200001 | Order creation for this pair suspended       |
| 200002 | Order cancel for this pair suspended     |
| 200003 | Number of orders breached the limit      |
| 200009 | Please complete the KYC verification before you trade XX |
| 200004 | Balance insufficient                           |
| 400001 | Any of VAEX-API-KEY, VAEX-API-SIGN, VAEX-API-TIMESTAMP, VAEX-API-PASSPHRASE is missing in your request header |
| 400002 | VAEX-API-TIMESTAMP Invalid      |
| 400003 | VAEX-API-KEY not exists                      |
| 400004 | VAEX-API-PASSPHRASE error             |
| 400005 | Signature error     |
| 400006 | The requested ip address is not in the api whitelist |
| 400007 | Access Denied        |
| 404000 | Url Not Found                              |
| 400100 | Parameter Error                            |
| 400200 | Forbidden to place an order              |
| 400500 | Your located country/region is currently not supported for the trading of this token |
| 400700 | Transaction restricted, there's a risk problem in your account |
| 400800 | Leverage order failed                          |
| 411100 | User are frozen |
| 500000 | Internal Server Error              |
| 900001 | symbol not exists                              |

If the returned HTTP status code is 200, whereas the operation failed, an error will occur. You can check the above error code for details.

### Success

A successful response is indicated by an HTTP status code 200 and system code 200000. The success response is as follows:


```json
{
  "code": "200000",
  "data": "1544657947759"
}
```



### Pagination

Pagination allows for fetching results with the current page and is well suited for real time data. Endpoints like /api/v1/deposits, /api/v1/orders, /api/v1/fills, will return the latest items by default (50 pages in total). To retrieve more results, users should specify the currentPage number in the subsequent requests to turn the page based on the data previously returned.

#### PARAMETERS

Parameter | Default | Description
---------- | ------- | ------
currentPage | 1 | Current request page.
pageSize | 50 | Number of results per request. Minimum is 10, maximum is 500. 


#### Example
`GET /api/v1/orders?currentPage=1&pageSize=50`


## Types

### Timestamps

Unless otherwise specified, all timestamps from API are returned in **milliseconds**(e.g. **1546658861000**). Most modern languages and libraries will handle this without issues.

But please note that the timestamps between the **matching engine** and the **order** system are in nanoseconds.

### Numbers
Decimal numbers are returned as strings in order to preserve the full precision across platforms. When making a request, it is recommended that you also convert your numbers to strings to avoid truncation and precision errors.

## Authentication

### Generating an API Key

Before being able to sign any requests, you must create an API key via the Vaex website. Upon creating a key you need to write down 3 pieces of information:


* Key
* Secret
* Passphrase

The Key and Secret are generated and provided by Vaex and the Passphrase refers to the one you used to create the Vaex API. Please note that these three pieces of information can not be recovered once lost. If you lost this information, please create a new API KEY.

### API KEY PERMISSIONS

You can manage the API permission on Vaex’s official website. The permissions are:


* **General** - General - Allows a key general permissions. This includes most of the GET endpoints.
* **Trade** -  Allows a key to create orders.
* **Transfer** -  Allows a key to transfer funds (including deposit and withdrawal). Enable with caution - API key transfers WILL BYPASS two-factor authentication.


Please refer to the documentation below to see what API key permissions are required for a specific route.

### Creating a Request

All private REST requests must contain the following headers:

* **VAEX-API-KEY** The API key as a string.
* **VAEX-API-SIGN** The base64-encoded signature (see Signing a Message).
* **VAEX-API-TIMESTAMP** A timestamp for your request.
* **VAEX-API-PASSPHRASE** The passphrase you specified when creating the API key.
* **VAEX-API-KEY-VERSION** You can check the version of API key on the page of [API Management](https://www.vaex.com/account/api)

### Signing a Message

```php
    <?php
    class API {
        public function __construct($key, $secret, $passphrase) {
          $this->key = $key;
          $this->secret = $secret;
          $this->passphrase = $passphrase;
        }

        public function signature($request_path = '', $body = '', $timestamp = false, $method = 'GET') {

          $body = is_array($body) ? json_encode($body) : $body; // Body must be in json format

          $timestamp = $timestamp ? $timestamp : time() * 1000;

          $what = $timestamp . $method . $request_path . $body;

          return base64_encode(hash_hmac("sha256", $what, $this->secret, true));
        }
    }
    ?>
```

```python
    #Example for get balance of accounts in python

    api_key = "api_key"
    api_secret = "api_secret"
    api_passphrase = "api_passphrase"
    url = 'https://api.vaex.com/api/v1/accounts'
    now = int(time.time() * 1000)
    str_to_sign = str(now) + 'GET' + '/api/v1/accounts'
    signature = base64.b64encode(
        hmac.new(api_secret.encode('utf-8'), str_to_sign.encode('utf-8'), hashlib.sha256).digest())
    passphrase = base64.b64encode(hmac.new(api_secret.encode('utf-8'), api_passphrase.encode('utf-8'), hashlib.sha256).digest())
    headers = {
        "VAEX-API-SIGN": signature,
        "VAEX-API-TIMESTAMP": str(now),
        "VAEX-API-KEY": api_key,
        "VAEX-API-PASSPHRASE": passphrase,
        "VAEX-API-KEY-VERSION": "2"
    }
    response = requests.request('get', url, headers=headers)
    print(response.status_code)
    print(response.json())

    #Example for create deposit addresses in python
    url = 'https://api.vaex.com/api/v1/deposit-addresses'
    now = int(time.time() * 1000)
    data = {"currency": "BTC"}
    data_json = json.dumps(data)
    str_to_sign = str(now) + 'POST' + '/api/v1/deposit-addresses' + data_json
    signature = base64.b64encode(
        hmac.new(api_secret.encode('utf-8'), str_to_sign.encode('utf-8'), hashlib.sha256).digest())
    passphrase = base64.b64encode(
        hmac.new(api_secret.encode('utf-8'), api_passphrase.encode('utf-8'), hashlib.sha256).digest())
    headers = {
        "VAEX-API-SIGN": signature,
        "VAEX-API-TIMESTAMP": str(now),
        "VAEX-API-KEY": api_key,
        "VAEX-API-PASSPHRASE": passphrase,
        "VAEX-API-KEY-VERSION": 2
        "Content-Type": "application/json" # specifying content type or using json=data in request
    }
    response = requests.request('post', url, headers=headers, data=data_json)
    print(response.status_code)
    print(response.json())
```

For the header of **VAEX-API-SIGN**:

* Use API-Secret to encrypt the prehash string {timestamp+method+endpoint+body} with sha256 HMAC. The request body is a JSON string and need to be the same with the parameters passed by the API.
* After that, use base64-encode to encrypt the result in step 1 again.

For the **VAEX-API-PASSPHRASE** of the header:

* For API key-V1.0, please pass requests in plaintext.
* For API key-V2.0, please Specify **VAEX-API-KEY-VERSION** as **2** --> Encrypt passphrase with HMAC-sha256 via API-Secret --> Encode contents by base64 before you pass the request."

Notice:

* The encrypted timestamp shall be consistent with the VAEX-API-TIMESTAMP field in the request header.
* The body to be encrypted shall be consistent with the content of the Request Body.  
* The Method should be UPPER CASE.
* For GET, DELETE request, the endpoint needs to contain the query string. e.g. /api/v1/deposit-addresses?currency=XBT. The body is "" if there is no request body (typically for GET requests).



```python
#Example for Create Deposit Address

curl -H "Content-Type:application/json" -H "VAEX-API-KEY:5c2db93503aa674c74a31734" -H "VAEX-API-TIMESTAMP:1547015186532" -H "VAEX-API-PASSPHRASE:QWIxMjM0NTY3OCkoKiZeJSQjQA==" -H "VAEX-API-SIGN:7QP/oM0ykidMdrfNEUmng8eZjg/ZvPafjIqmxiVfYu4=" -H "VAEX-API-KEY-VERSION:2"
-X POST -d '{"currency":"BTC"}' http://api.vaex.com/api/v1/deposit-addresses

VAEX-API-KEY = 5c2db93503aa674c74a31734
VAEX-API-SECRET = f03a5284-5c39-4aaa-9b20-dea10bdcf8e3
VAEX-API-PASSPHRASE = QWIxMjM0NTY3OCkoKiZeJSQjQA==
VAEX-API-KEY-VERSION = 2
TIMESTAMP = 1547015186532
METHOD = POST
ENDPOINT = /api/v1/deposit-addresses
STRING-TO-SIGN = 1547015186532POST/api/v1/deposit-addresses{"currency":"BTC"}
VAEX-API-SIGN = 7QP/oM0ykidMdrfNEUmng8eZjg/ZvPafjIqmxiVfYu4=
```

<aside class="spacer16"></aside>
<aside class="spacer8"></aside>

### Selecting a Timestamp

The **VAEX-API-TIMESTAMP** header MUST be number of **milliseconds** since Unix Epoch in UTC. e.g. 1547015186532

Decimal values are allowed, e.g. 1547015186532. But you need to be aware that timestamp between match and order is **nanosecond**.

The difference between your timestamp and the API service time must be less than **5 seconds** , or your request will be considered expired and rejected. We recommend using the time endpoint to query for the API [server time](#server-time) if you believe there may be time skew between your server and the API server.




# User
Signature is required for this part.



# Account

## List Accounts
```json
[
  {
    "id": "5bd6e9286d99522a52e458de",  //accountId
    "currency": "BTC",  //Currency
    "type": "main",     //Account type, including main and trade
    "balance": "237582.04299",  //Total assets of a currency
    "available": "237582.032",  //Available assets of a currency
    "holds": "0.01099" //Hold assets of a currency
  },
  {
    "id": "5bd6e9216d99522a52e458d6",
    "currency": "BTC",
    "type": "trade",
    "balance": "1234356",
    "available": "1234356",
    "holds": "0"
}]

```
Get a list of accounts.

Please deposit funds to the main account firstly, then transfer the funds to the trade account via [Inner Transfer](#inner-transfer) before transaction.

### HTTP REQUEST
`GET /api/v1/accounts`

### Example
`GET /api/v1/accounts`

### API KEY PERMISSIONS
This endpoint requires the `General` permission.

### PARAMETERS
Param | Type | Description
--------- | ------- | -------
currency | String | *[Optional]* [Currency](#get-currencies)
type | String | *[Optional]* Account type: **main**, **trade**

### RESPONSES
Field | Description
--------- | -------
id | The ID of the account
currency | Currency
type | Account type: **main**, **trade**
balance | Total funds in the account
available | Funds available to withdraw or trade
holds | Funds on hold (not available for use)

### ACCOUNT TYPE
There are three types of accounts: 1) **main** account 2) **trade** account.

No fees will be charged for the funds transfer between these account.

The main account is used for the storage, withdrawal, and deposit of the funds. The assets in the main account cannot be directly used for trading. To trade cryptos, you need to transfer funds from the main account to the trade account.

The trading account is used for transaction. When you place an order, the system will use the balance of the  trade account. You can’t withdraw funds directly from a trade account. To withdraw the funds, you need to transfer the funds from the trade account to the main account firstly.


### FUNDS ON HOLD
When placing an order, the funds for the order will be freezed. The freezed funds cannot be used for other order placement or withdrawal and will remain on hold until the order is filled or cancelled.



## Get Account Ledgers

This interface is for the history of deposit/withdrawal of all accounts, supporting inquiry of various currencies. 

Items are paginated and sorted to show the latest first. See the [Pagination](#pagination) section for retrieving additional entries after the first page.

```json
{
  "currentPage": 1,
  "pageSize": 50,
  "totalNum": 2,
  "totalPage": 1,
  "items": [
    {
      "id": "611a1e7c6a053300067a88d9",//unique key
      "currency": "USDT", //Currency
      "amount": "10.00059547", //Change amount of the funds
      "fee": "0", //Deposit or withdrawal fee
      "balance": "0", //Total assets of a currency
      "accountType": "MAIN", //Account Type
      "bizType": "Loans Repaid", //business type
      "direction": "in", //side, in or out
      "createdAt": 1629101692950, //Creation time
      "context": "{\"borrowerUserId\":\"601ad03e50dc810006d242ea\",\"loanRepayDetailNo\":\"611a1e7cc913d000066cf7ec\"}" //Business core parameters
    },
    {
      "id": "611a18bc6a0533000671e1bf",
      "currency": "USDT",
      "amount": "10.00059547",
      "fee": "0",
      "balance": "0",
      "accountType": "MAIN",
      "bizType": "Loans Repaid",
      "direction": "in",
      "createdAt": 1629100220843,
      "context": "{\"borrowerUserId\":\"5e3f4623dbf52d000800292f\",\"loanRepayDetailNo\":\"611a18bc7255c200063ea545\"}"
    }
  ]
}
```

### HTTP REQUEST
`GET /api/v1/accounts/ledgers`

### Example
`GET /api/v1/accounts/ledgers?currency=BTC&startAt=1601395200000`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is **18 times/3s**.

<aside class="notice">This request is paginated.</aside>


### PARAMETERS
Param | Type | Description
--------- | ------- | -------
currency | String | *[Optional]* Currency ( you can choose more than one currency). You can specify 10 currencies at most for one time. If not specified, all currencies will be inquired by default.
direction | String | *[Optional]*  Side: **in** - Receive, **out** - Send
bizType   | String | *[Optional]*  Business type: **DEPOSIT**, **WITHDRAW**, **TRANSFER**,**TRADE_EXCHANGE**.
startAt| long | *[Optional]*  Start time (milisecond)
endAt| long | *[Optional]* End time (milisecond)

<aside class="notice">the start and end time range cannot exceed 24 hours. An error will occur if the specified time window exceeds the range. If you specify the end time only, the system will automatically calculate the start time as end time minus 24 hours, and vice versa.</aside>
<aside class="notice">Support to obtain 1 year historical data, if need to obtain longer historical data, please submit a ticket: https://vaex.zendesk.com/hc/en-us/requests/new</aside>

### RESPONSES
Field | Description
--------- | -------
id | unique key
currency | The currency of an account
amount | The total amount of assets (fees included) involved in assets changes such as transaction, withdrawal and bonus distribution.
fee | Fees generated in transaction, withdrawal, etc.
balance | Remaining funds after the transaction.
accountType | The account type of the master user: MAIN, TRADE.
bizType | Business type leading to the changes in funds, such as exchange, withdrawal, deposit etc.
direction | Side, **out** or **in**
createdAt | Time of the event
context | Business related information such as order ID, serial No., etc.

### context
If the returned value under bizType is **“trade exchange”**, the additional info. (such as order ID and trade ID, trading pair, etc.) of the trade will be returned in field **context**.

### BizType Description
Field | Description
--------- | -------
Deposit  | Deposit 
Withdrawal  | Withdrawal
Transfer | Transfer
Trade_Exchange | Trade




## Inner Transfer
```json
{
    "orderId": "5bd6e9286d99522a52e458de"
}
```

This API endpoint can be used to transfer funds between accounts internally. Users can transfer funds between their main account, trading account free of charge. 

### HTTP REQUEST
`POST /api/v2/accounts/inner-transfer`

### API KEY PERMISSIONS
This endpoint requires the `Trade` permission.

### PARAMETERS
Param | Type | Mandatory | Description |  
--------- | ------- | -----------| -----------|
clientOid | String | Yes | clientOid, the unique identifier created by the client, use of UUID 
currency | String | Yes | [currency](#Get-Currencies) 
from | String | Yes | Payment Account Type: `main`, `trade`
to | String | Yes | Receiving Account Type: `main`, `trade`
amount | String | Yes | Transfer amount, the precision being a positive integer multiple of the [Currency Precision](#get-currencies) 
fromTag | String | No | Trading pair
toTag | String | No | Trading pair


### RESPONSES
Field | Description
--------- | -------
orderId | The order ID of a funds transfer


# Deposit

## Create Deposit Address

```json
{
	"address": "0x78d3ad1c0aa1bf068e19c94a2d7b16c9c0fcd8b1",
	"memo": "5c247c8a03aa677cea2a251d",   //tag
	"chain": "OMNI"
}
```
Request via this endpoint to create a deposit address for a currency you intend to deposit.

### HTTP REQUEST
`POST /api/v1/deposit-addresses`

### Example
`POST /api/v1/deposit-addresses`

### API KEY PERMISSIONS
This endpoint requires the **"Transfer"** permission.

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
currency | String | Currency
chain | String | *[Optional]* The chain name of currency, e.g.  This only apply for multi-chain currency, and there is no need for single chain currency. 

### RESPONSES
Field | Description
--------- | ------- 
address | Deposit address
memo | Address remark. If there’s no remark, it is empty. When you [withdraw](#apply-withdraw) from other platforms to the Vaex, you need to fill in memo(tag). If you do not fill memo (tag), your deposit may not be available, please be cautious.
chain | The chain name of currency, e.g.

## Get Deposit Addresses
```json
[
  {
    "address": "bc1qaj6kkv85w5d6lr8p8h7tckyce5hnwmyq8dd84d",
    "memo": "",
    "chain": "BTC-Segwit",
    "contractAddress": ""
  },
  {
    "address": "3HwsFot9TW6jL4K4EUHxDSyL8myttxV7Av",
    "memo": "",
    "chain": "BTC",
    "contractAddress": ""
  },
  {
    "address": "TUDybru26JmozStbg2cJGDbR9EPSbQaAie",
    "memo": "",
    "chain": "TRC20",
    "contractAddress": ""
  }
]
```

Get all deposit addresses for the currency you intend to deposit. If the returned data is empty, you may need to create a deposit address first.

### HTTP REQUEST
`GET /api/v2/deposit-addresses`

### Example
`GET /api/v2/deposit-addresses?currency=BTC`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### PARAMETERS
Param | Type | Description
--------- | -------  | -------
currency | String | The currency

### RESPONSES
Field | Description
--------- | ------- 
address | Deposit address
memo | Address remark. If there’s no remark, it is empty. When you [withdraw](#apply-withdraw) from other platforms to the Vaex, you need to fill in memo(tag). If you do not fill memo (tag), your deposit may not be available, please be cautious.
chain | The chain name of currency.
contractAddress | The token contract address.




# Withdrawals

## Get Withdrawals List
```json
{
    "code": "200000",
    "data": {
        "currentPage": 1,
        "pageSize": 50,
        "totalNum": 1,
        "totalPage": 1,
        "items": [
            {
                "id": "63564dbbd17bef00019371fb",
                "currency": "XRP",
                "chain": "xrp",
                "status": "SUCCESS",
                "address": "rNFugeoj3ZN8Wv6xhuLegUBBPXKCyWLRkB",
                "memo": "1919537769",
                "isInner": false,
                "amount": "20.50000000",
                "fee": "0.50000000",
                "walletTxId": "2C24A6D5B3E7D5B6AA6534025B9B107AC910309A98825BF5581E25BEC94AD83B",
                "createdAt": 1666600379000,
                "updatedAt": 1666600511000,
                "remark": "test"
            }
        ]
    }
}
```

### HTTP REQUEST
`GET /api/v1/withdrawals`

### Example
`GET /api/v1/withdrawals`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is **6 times/3s**.

<aside class="notice">This request is paginated.</aside>

### PARAMETERS
Param | Type | Mandatory | Description |  
--------- | ------- | -----------| -----------|
currency | String | No | [Currency](#get-currencies)
status | String | No | Status. Available value: `PROCESSING`, `WALLET_PROCESSING`, `SUCCESS`, and `FAILURE`
startAt| long | No | Start time (milisecond)
endAt| long | No | End time (milisecond)

### RESPONSES
Field | Description
--------- | -------
id | Unique identity
address | Withdrawal address
memo |  Address remark. If there’s no remark, it is empty. When you [withdraw](#apply-withdraw) from other platforms to the Vaex, you need to fill in memo(tag). If you do not fill memo (tag), your deposit may not be available, please be cautious.
currency | Currency
chain | The chain of currency
amount | Withdrawal amount
fee | Withdrawal fee
walletTxId | Wallet Txid
isInner | Internal withdrawal or not
status | status
remark | remark
createdAt | Creation time
updatedAt | Update time




##  Get Withdrawal Quotas
```json
{
	"currency": "ETH",
	"limitBTCAmount": "2.0",
	"usedBTCAmount": "0",
	"remainAmount": "75.67567568",
	"availableAmount": "9697.41991348",
	"withdrawMinFee": "0.93000000",
	"innerWithdrawMinFee": "0.00000000",
	"withdrawMinSize": "1.4",
	"isWithdrawEnabled": true,
	"precision": 8,   //withdrawal precision
	"chain": "OMNI"
}
```

### HTTP REQUEST
`GET /api/v1/withdrawals/quotas`

### Example
`GET /api/v1/withdrawals/quotas?currency=BTC`


### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
currency | String | currency. e.g. BTC
chain | String | *[Optional]* The chain of currency. This only apply for multi-chain currency, and there is no need for single chain currency; you can query the `chain` through the response of the `GET /api/v2/currencies/{currency}` interface.

### RESPONSES
Field | Description
--------- | ------- | -----------
currency | Currency
availableAmount | Current available withdrawal amount
remainAmount | Remaining amount available to withdraw the current day
withdrawMinSize | Minimum withdrawal amount
limitBTCAmount | Total BTC amount available to withdraw the current day
innerWithdrawMinFee | Fees for internal withdrawal
usedBTCAmount | The estimated BTC amount (based on the daily fiat limit) that can be withdrawn within the current day
isWithdrawEnabled | Is the withdraw function enabled or not
withdrawMinFee | Minimum withdrawal fee
precision | Floating point precision.
chain | The chain name of currency, e.g. 

## Apply Withdraw
```json
{
  "withdrawalId": "5bffb63303aa675e8bbe18f9"
}
```

### HTTP REQUEST
`POST /api/v1/withdrawals`

<aside class="notice">On the WEB end, you can open the switch of specified favorite addresses for withdrawal, and when it is turned on, it will verify whether your withdrawal address(including chain) is a favorite address(it is case sensitive); if it fails validation, it will respond with the error message <code>{"msg":"Already set withdraw whitelist, this address is not favorite address","code":"260325"}</code>.</aside>

### Example
`POST /api/v1/withdrawals`

### API KEY PERMISSIONS
This endpoint requires the `Transfer` permission.

### PARAMETERS
Param | Type | Mandatory | Description |  
--------- | ------- | -----------| -----------|
currency   | String | Yes |Currency|
address   | String | Yes | Withdrawal address
amount | number | Yes | Withdrawal amount, a positive number which is a multiple of the amount precision (fees excluded)
memo   | String | No | [Optional] Address remark. If there’s no remark, it is empty. When you withdraw from other platforms to the Vaex, you need to fill in memo(tag). If you do not fill memo (tag), your deposit may not be available, please be cautious.
isInner | boolean | No | [Optional]  Internal withdrawal or not. Default setup: false
remark | String | No | [Optional]  Remark
chain | String | No | *[Optional]* The chain of currency. For a currency with multiple chains, it is recommended to specify chain parameter instead of using the default chain; you can query the `chain` through the response of the `GET /api/v2/currencies/{currency}` interface.
feeDeductType | String | No | Withdrawal fee deduction type: `INTERNAL` or `EXTERNAL` or not specified<br/><br/>1. `INTERNAL`- deduct the transaction fees from your withdrawal amount</br>2. `EXTERNAL`- deduct the transaction fees from your main account</br>3. If you don't specify the `feeDeductType` parameter, when the balance in your main account is sufficient to support the withdrawal, the system will initially deduct the transaction fees from your main account. But if the balance in your main account is not sufficient to support the withdrawal, the system will deduct the fees from your withdrawal amount. For example:  Suppose you are going to withdraw 1 BTC from the Vaex platform (transaction fee: 0.0001BTC), if the balance in your main account is insufficient, the system will deduct the transaction fees from your withdrawal amount. In this case, you will be receiving 0.9999BTC.

### RESPONSES
Field | Description
--------- | -------
withdrawalId | Withdrawal id



# Trade Fee

## Basic user fee

This interface is for the basic fee rate of users

```json
{
    "code": "200000",
    "data": {
        "takerFeeRate": "0.001",
        "makerFeeRate": "0.001"
    }
}
```

### HTTP REQUEST
`GET /api/v1/base-fee`

### Example
`GET /api/v1/base-fee`
<br/>
`GET /api/v1/base-fee?currencyType=1`

### API KEY PERMISSIONS
This endpoint requires the `General` permission.

### PARAMETERS
Param | Type | Mandatory | Description
--------- | ------- | -------| -------
currencyType| String | No | Currency type: `0`-crypto currency, `1`-fiat currency. default is `0`-crypto currency

### RESPONSES
Field | Description
--------- | -------
takerFeeRate | Base taker fee rate
makerFeeRate | Base maker fee rate

## Actual fee rate of the trading pair

This interface is for the actual fee rate of the trading pair. You can inquire about fee rates of 10 trading pairs each time at most.  

```json
{
    "code": "200000",
    "data": [
        {
            "symbol": "BTC-USDT",
            "takerFeeRate": "0.001",
            "makerFeeRate": "0.001"
        },
        {
            "symbol": "ETH-USDT",
            "takerFeeRate": "0.002",
            "makerFeeRate": "0.0005"
        }
    ]
}
```

### HTTP REQUEST
`GET /api/v1/trade-fees`

### Example
`GET /api/v1/trade-fees?symbols=BTC-USDT,ETH-USDT`

### API KEY PERMISSIONS
This endpoint requires the `General` permission.

### PARAMETERS
Param | Type | Mandatory | Description
--------- | ------- | -------| -------
symbols|String|Yes|Trading pair (optional, you can inquire fee rates of `10` trading pairs each time at most)

### RESPONSES
Field | Description
--------- | -------
symbol | The unique identity of the trading pair and will not change even if the trading pair is renamed
takerFeeRate | Actual taker fee rate of the trading pair
makerFeeRate | Actual maker fee rate of the trading pair

# Trade

Signature is required for this part.

# Orders

## Place a new order

```json
{
  "orderId": "5bd6e9286d99522a52e458de"
}
```

You can place two types of orders: **limit** and **market**. Orders can only be placed if your account has sufficient funds. Once an order is placed, your account funds will be put on hold for the duration of the order. How much and which funds are put on hold depends on the order type and parameters specified. See the Holds details below.

<aside class="notice">Placing an order will enable price protection. When the price of the limit order is outside the threshold range, the price protection mechanism will be triggered, causing the order to fail.</aside>


Please note that the system will frozen the fees from the orders that entered the order book in advance. Read [List Fills](#list-fills) to learn more.

Before placing an order, please read [Get Symbol List](#get-symbols-list) to understand the requirements for the quantity parameters for each trading pair.

**Do not include extra spaces in JSON strings**.

###Place Order Limit

The maximum active orders for a single trading pair in one account is **200** (stop orders included).


### HTTP Request
`POST /api/v1/orders`

### Example
`POST /api/v1/orders`

### API KEY PERMISSIONS
This endpoint requires the **"Trade"** permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is **45 times/3s**.

### PARAMETERS
| Param     | type   | Description  |
| --------- | ------ |-------------------------------- |
| clientOid | String | Unique order id created by users to identify their orders, e.g. UUID. |
| side      | String | **buy** or **sell**      |
| symbol    | String | a valid trading symbol code. e.g. ETH-BTC     |
| type      | String | *[Optional]* **limit** or **market** (default is **limit**)          |
| remark    | String | *[Optional]*  remark for the order, length cannot exceed 100 utf8 characters|
| stp       | String | *[Optional]*  self trade prevention , **CN**, **CO**, **CB** or **DC**|
| tradeType | String | *[Optional]* The type of trading : **TRADE**（Spot Trade）. Default is **TRADE**. |

#### LIMIT ORDER PARAMETERS
| Param       | type    | Description                                                  |
| ----------- | ------- | ------------------- |
| price       | String  | price per base currency          |
| size        | String  | amount of base currency to buy or sell         |
| timeInForce | String  | *[Optional]* **GTC**, **GTT**, **IOC**, or **FOK** (default is **GTC**), read [Time In Force](#time-in-force).   |
| cancelAfter | long    | *[Optional]*  cancel after **n** seconds, requires **timeInForce** to be **GTT**                   |
| postOnly    | boolean | *[Optional]*  Post only flag, invalid when **timeInForce** is **IOC** or **FOK**                               |
| hidden      | boolean | *[Optional]*  Order will not be displayed in the order book |
| iceberg    | boolean | *[Optional]*  Only aportion of the order is displayed in the order book |
| visibleSize | String  | *[Optional]*  The maximum visible size of an iceberg order   |


#### MARKET ORDER PARAMETERS
Param | type | Description
--------- | ------- | -----------
size | String | *[Optional]*  Desired amount in base currency
funds | String | *[Optional]*  The desired amount of quote currency to use

* It is required that you use one of the two parameters, **size** or **funds**.

###Advanced Description

###SYMBOL

The symbol must match a valid trading [symbol](#get-symbols-list).

###CLIENT ORDER ID

Generated by yourself, the optional clientOid field must be a unique id (e.g UUID). Only numbers, characters, underline(_) and separator(-) are allowed. The value will be returned in order detail. You can use this field to identify your orders via the public feed. The client_oid is different from the server-assigned order id. Please do not send a repeated client_oid. The length of the client_oid cannot exceed 40 characters. You should record the server-assigned order_id as it will be used for future query order status.


###TYPE

The order type you specify may decide whether other optional parameters are required, as well as how your order will be executed by the matching engine. If order type is not specified, the order will be a **limit** order by default.

**Price** and **size** are required to be specified for a **limit order**.  The order will be filled at the price specified or better, depending on the market condition.   If a limit order cannot be filled immediately, it will be outstanding in the open order book until matched by another order, or canceled by the user.


A **market order** differs from a limit order in that the execution price is not guaranteed. Market order, however, provides a way to buy or sell specific size of order without having to specify the price. Market orders will be executed immediately, and no orders will enter the open order book afterwards. Market orders are always considered takers and incur taker fees.

###TradeType
The platform currently supports spot (**TRADE**) . The system will freeze the funds of the specified account according to your parameter type. If this parameter is not specified, the funds in your trade account will be frozen by default. 

###PRICE
The price must be specified in priceIncrement symbol units. The priceIncrement is the smallest unit of price. For the BTC-USDT symbol, the priceIncrement is 0.00001000. Prices less than 0.00001000 will not be accepted, The price for the placed order should be multiple numbers of priceIncrement, or the system would report an error when you place the order. Not required for market orders.

###SIZE
The size must be greater than the baseMinSize for the symbol and no larger than the baseMaxSize. The size must be specified in baseIncrement symbol units. Size indicates the amount of BTC (or base currency) to buy or sell.

###FUNDS
The funds field indicates the how much of the quote currency you wish to buy or sell. The size of the funds must be specified in quoteIncrement symbol units and the size of funds in order shall be a positive integer multiple of quoteIncrement, ensuring the funds is greater than the quoteMinSize for the symbol but no larger than the quoteMaxSize.


###TIME IN FORCE
Time in force policies provide guarantees about the lifetime of an order. There are four policies: Good Till Canceled **GTC**, Good Till Time **GTT**, Immediate Or Cancel **IOC**, and Fill Or Kill **FOK**.

**GTC** Good Till Canceled orders remain open on the book until canceled. This is the default behavior if no policy is specified.

**GTT** Good Till Time orders remain open on the book until canceled or the allotted cancelAfter is depleted on the matching engine. GTT orders are guaranteed to cancel before any other order is processed after the cancelAfter seconds placed in order book.

**IOC** Immediate Or Cancel orders instantly cancel the remaining size of the limit order instead of opening it on the book.

**FOK** Fill Or Kill orders are rejected if the entire size cannot be matched.

* Note that self trades belong to match as well. For market orders, using the “TimeInForce” parameter has no effect.

### POST ONLY
The post-only flag ensures that the trader always pays the maker fee and provides liquidity to the order book. If any part of the order is going to pay taker fee, the order will be fully rejected.

If a post only order will get executed immediately against the existing orders (except iceberg and hidden orders) in the market, the order will be cancelled.

* For post only orders, it will get executed immediately against the iceberg orders and hidden orders in the market. Users placing the post only order will be charged the maker fees and the iceberg and hidden orders will be charged the taker fees.


### HIDDEN AND ICEBERG


The **Hidden** and **iceberg Orders** are two **options** in advanced settings (note: the iceberg order is a special form of the hidden order). You may select “Hidden” or “Iceberg” when placing a limit or stop limit order.

A hidden order will enter but not display on the orderbook.

Different from the hidden order, an **iceberg order** is divided into visible portion and invisible portion. When placing an iceberg order, you need to set the **visible size**. The minimum visible size is 1/20 of the order size. The minimum visible size shall be greater than the minimum order size, or an error will occur.

In a matching event, the visible portion of an iceberg order will be executed first, and another visible portion will pop up until the order is fully filled.

**Note**:
- The system will charge **taker fees** for **Hidden** and **iceberg Orders**.

- If both "Iceberg" and "Hidden" are selected, your order will be filled as an **iceberg Order** by default.


###HOLDS
For limit buy orders, we will hold the needed portion from your funds (price x size of the order). Likewise, on sell orders, we will also hold the amount of assets that you wish to sell. Actual fees are assessed at the time of a trade. If you cancel a partially filled or unfilled order, any remaining funds will be released from being held.

For market buy or sell orders where the funds are specified, the funds amount will be put on hold. If only size is specified, all of your account balance (in the quote account) will be put on hold for the duration of the market order (usually a trivially short time).

### SELF-TRADE PREVENTION

**Self-Trade Prevention** is an option in advanced settings.It is not selected by default. If you specify **STP** when placing orders, your order won't be matched by another one which is also yours. On the contrary, if **STP** is not specified in advanced, your order can be matched by another one of your own orders.

**Market order is currently not supported for DC**. When the **timeInForce** is set to **FOK**, the **stp** flag will be forcely specified as **CN**.


**Market order is currently not supported for DC**. When *timeInForce* is **FOK**, the stp flag will be forced to be specified as **CN**.

| Flag | Name                          |
| ---- | ----------------------------- |
| DC   | Decrease and Cancel           |
| CO   | Cancel oldest                 |
| CN   | Cancel newest                 |
| CB   | Cancel both                   |

### ORDER LIFECYCLE
The HTTP Request will respond when an order is either rejected (insufficient funds, invalid parameters, etc) or received (accepted by the matching engine). A **200** response indicates that the order was received and is active. **Active** orders may execute immediately (depending on price and market conditions) either partially or fully. A partial execution will put the remaining size of the order in the open state. An order that is filled completely, will go into the **done** state.

Users listening to streaming market data are encouraged to use the order ID field to identify their received messages in the feed.

### PRICE PROTECTION MECHANISM

1. If there are contra orders against the market/limit orders placed by users in the order book, the system will detect whether the difference between the corresponding market price and the ask/bid price will exceed the threshold (you can request via the API symbol interface).
2. For limit orders, if the difference exceeds the threshold, the order placement would fail.
3. For market orders, the order will be partially executed against the existing orders in the market within the threshold and the remaining unfilled part of the order will be canceled immediately.
For example: If the threshold is 10%, when a user places a market order to buy 10,000 USDT in the ETH/USDT market (at this time, the current ask price is 1.20000), the system would determine that the final execution price would be 1.40000. As for (1.40000-1.20000)/1.20000=16.7%>10%, the threshold price would be 1.32000. Therefore, this market order will execute with the existing orders offering prices up to 1.32000 and the remaining part of the order will be canceled immediately.
Notice: There might be some deviations of the detection. If your order is not fully filled, it may probably be led by the unfilled part of the order exceeding the threshold.


###RESPONSES
Field | Description
--------- | -------
orderId | The ID of the order

A successful order will be assigned an order ID. A successful order is defined as one that has been accepted by the matching engine.

<aside class="notice">Open orders do not expire and will remain open until they are either filled or canceled.</aside>



## Place Bulk Orders

```json
//response
{
  "data": [
    {
      "symbol": "ETH-USDT",
      "type": "limit",
      "side": "buy",
      "price": "0.01",
      "size": "0.01",
      "funds": null,
      "stp": "",
      "stop": "",
      "stopPrice": null,
      "timeInForce": "GTC",
      "cancelAfter": 0,
      "postOnly": false,
      "hidden": false,
      "iceberge": false,
      "iceberg": false,
      "visibleSize": null,
      "channel": "API",
      "id": "611a6a309281bc000674d3c0",
      "status": "success",
      "failMsg": null,
      "clientOid": "552a8a0b7cb04354be8266f0e202e7e9"
    },
    {
      "symbol": "ETH-USDT",
      "type": "limit",
      "side": "buy",
      "price": "0.01",
      "size": "0.01",
      "funds": null,
      "stp": "",
      "stop": "",
      "stopPrice": null,
      "timeInForce": "GTC",
      "cancelAfter": 0,
      "postOnly": false,
      "hidden": false,
      "iceberge": false,
      "iceberg": false,
      "visibleSize": null,
      "channel": "API",
      "id": "611a6a309281bc000674d3c1",
      "status": "success",
      "failMsg": null,
      "clientOid": "bd1e95e705724f33b508ed270888a4a9"
    }
  ]
}
```

Request via this endpoint to place 5 orders at the same time. The order type must be a limit order of the same symbol.
**The interface currently only supports spot trading**


### HTTP Request
`POST /api/v1/orders/multi`


### Example
```json
//request
{
  "symbol": "ETH-USDT",
  "orderList": [
    {
      "clientOid": "3d07008668054da6b3cb12e432c2b13a",
      "side": "buy",
      "type": "limit",
      "price": "0.01",
      "size": "0.01"
    },
    {
      "clientOid": "37245dbe6e134b5c97732bfb36cd4a9d",
      "side": "buy",
      "type": "limit",
      "price": "0.01",
      "size": "0.01"
    }
  ]
}
```
`POST /api/v1/orders/multi`

### API KEY PERMISSIONS
This endpoint requires the `Trade` permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is `3 times/3s`.

### PARAMETERS
| Param     | type   | Description  |
| --------- | ------ |-------------------------------- |
| clientOid | String | Unique order id created by users to identify their orders, e.g. UUID. |
| side      | String | **buy** or **sell**      |
| symbol    | String | a valid trading symbol code. e.g. ETH-BTC     |
| type      | String | *[Optional]* only **limit** (default is **limit**)          |
| remark    | String | *[Optional]*  remark for the order, length cannot exceed 100 utf8 characters|
| stop      | String | *[Optional]* Either **loss** or **entry**. Requires **stopPrice** to be defined |
| stopPrice | String | *[Optional]* Need to be defined if stop is specified. |
| stp       | String | *[Optional]*  self trade prevention , **CN**, **CO**, **CB** or **DC**|
| tradeType | String | *[Optional]*  Default is **TRADE** |
| price       | String  | price per base currency          |
| size        | String  | amount of base currency to buy or sell         |
| timeInForce | String  | *[Optional]* **GTC**, **GTT**, **IOC**, or **FOK** (default is **GTC**), read [Time In Force](#time-in-force).   |
| cancelAfter | long    | *[Optional]*  cancel after **n** seconds, requires **timeInForce** to be **GTT**                   |
| postOnly    | boolean | *[Optional]*  Post only flag, invalid when **timeInForce** is **IOC** or **FOK**                               |
| hidden      | boolean | *[Optional]*  Order will not be displayed in the order book |
| iceberg     | boolean | *[Optional]*  Only aportion of the order is displayed in the order book |
| visibleSize | String  | *[Optional]*  The maximum visible size of an iceberg order   |

### RESPONSES
Field | Description
--------- | -------
|status  |  status (success, fail) |
|failMsg |  the cause of failure   |


## Cancel an order

```json
{
     "cancelledOrderIds": [
      "5bd6e9286d99522a52e458de"   //orderId
    ]
}
```

Request via this endpoint to cancel a single order previously placed.

<aside class="notice">This interface is only for cancellation requests. The cancellation result needs to be obtained by querying the order status or subscribing to websocket. It is recommended that you DO NOT cancel the order until receiving the Open message, otherwise the order cannot be cancelled successfully.
</aside>


### HTTP REQUEST
`DELETE /api/v1/orders/{orderId}`

### Example
`DELETE /api/v1/orders/5bd6e9286d99522a52e458de`


### API KEY PERMISSIONS
This endpoint requires the `Trade` permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is `60 times/3s`.

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
orderId | String | [Order ID](#list-orders), unique ID of the order.

### RESPONSES
Field | Description
--------- | -------
orderId | Unique ID of the cancelled order



<aside class="notice">The <b>order ID</b> is the server-assigned order id and not the passed clientOid.</aside>

### CANCEL REJECT
If the order could not be canceled (already filled or previously canceled, etc), then an error response will indicate the reason in the **message** field.

## Cancel Single Order by clientOid

```json
{
  "cancelledOrderId": "5f311183c9b6d539dc614db3",
  "clientOid": "6d539dc614db3"
}
```

Request via this interface to cancel an order via the clientOid.

### HTTP REQUEST
`DELETE /api/v1/order/client-order/{clientOid}`

### Example
`DELETE /api/v1/order/client-order/6d539dc614db3`

### API KEY PERMISSIONS
This endpoint requires the **"Trade"** permission.

### PARAMETERS
| Param | Type | Description                            |
| ------- | ------ | ----------------------------- |
| clientOid | String | Unique order id created by users to identify their orders |

### RESPONSES
| Field | Description     |
| ----------------- | ------- |
| cancelledOrderId | Order ID of cancelled order |
| clientOid | Unique order id created by users to identify their orders |

## Cancel all orders
```json
{
    "cancelledOrderIds": [

        "5c52e11203aa677f33e493fb",  //orderId
        "5c52e12103aa677f33e493fe",
        "5c52e12a03aa677f33e49401",
        "5c52e1be03aa677f33e49404",
        "5c52e21003aa677f33e49407",
        "5c6243cb03aa67580f20bf2f",
        "5c62443703aa67580f20bf32",
        "5c6265c503aa676fee84129c",
        "5c6269e503aa676fee84129f",
        "5c626b0803aa676fee8412a2"
    ]
}
```
Request via this endpoint to cancel all open orders. The response is a list of ids of the canceled orders.

### HTTP REQUEST
`DELETE /api/v1/orders`

### Example
`DELETE /api/v1/orders?symbol=ETH-BTC&tradeType=TRADE`<br/>

### API KEY PERMISSIONS
This endpoint requires the `Trade` permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is `3 times/3s`.

### PARAMETERS
|Param | Type | Description|
|--------- | ------- | -----------|
|symbol | String | *[Optional]* symbol, cancel the orders for the specified trade pair. |
|tradeType| String | *[Optional]*  the type of trading :`TRADE`(Spot Trading)|

### RESPONSES
Field | Description
--------- | -------
orderId | Order ID, unique identifier of an order.



## List Orders

```json
{
    "currentPage": 1,
    "pageSize": 1,
    "totalNum": 153408,
    "totalPage": 153408,
    "items": [
        {
            "id": "5c35c02703aa673ceec2a168",   //orderid
            "symbol": "BTC-USDT",   //symbol
            "opType": "DEAL",      // operation type: DEAL
            "type": "limit",       // order type,e.g. limit,market,stop_limit.
            "side": "buy",         // transaction direction,include buy and sell
            "price": "10",         // order price
            "size": "2",           // order quantity
            "funds": "0",          // order funds
            "dealFunds": "0.166",  // deal funds
            "dealSize": "2",       // deal quantity
            "fee": "0",            // fee
            "feeCurrency": "USDT", // charge fee currency
            "stp": "",             // self trade prevention,include CN,CO,DC,CB
            "stop": "",            // stop type
            "stopTriggered": false,  // stop order is triggered
            "stopPrice": "0",      // stop price
            "timeInForce": "GTC",  // time InForce,include GTC,GTT,IOC,FOK
            "postOnly": false,     // postOnly
            "hidden": false,       // hidden order
            "iceberg": false,      // iceberg order
            "visibleSize": "0",    // display quantity for iceberg order
            "cancelAfter": 0,      // cancel orders time，requires timeInForce to be GTT
            "channel": "IOS",      // order source
            "clientOid": "",       // user-entered order unique mark
            "remark": "",          // remark
            "tags": "",            // tag order source        
            "isActive": false,     // status before unfilled or uncancelled
            "cancelExist": false,   // order cancellation transaction record
            "createdAt": 1547026471000,  // create time
            "tradeType": "TRADE"
        }
     ]
 }
```

Request via this endpoint to get your current order list. Items are paginated and sorted to show the latest first. See the [Pagination](#pagination) section for retrieving additional entries after the first page.

### HTTP REQUEST
`GET /api/v1/orders`

### Example
`GET /api/v1/orders?status=active`<br/>

### API KEY PERMISSIONS
This endpoint requires the `General` permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is `30 times/3s`.

<aside class="notice">This request is paginated.</aside>


### PARAMETERS
You can pinpoint the results with the following query paramaters.

Param | Type | Description
--------- | ------- | -----------
status | String |*[Optional]* `active` or `done`(`done` as default), Only list orders with a specific status .
symbol |String|*[Optional]* Only list orders for a specific symbol.
side | String | *[Optional]* `buy` or `sell`
type | String | *[Optional]* `limit`, `market`, `limit_stop` or `market_stop`
tradeType | String |The type of trading:`TRADE`-Spot Trading.
startAt| long | *[Optional]*  Start time (milisecond)
endAt| long | *[Optional]* End time (milisecond)

### RESPONSES
Field | Description
--------- | -------
id |  Order ID, the ID of an order.
symbol | symbol
opType |  Operation type: DEAL
type | order type
side | transaction direction,include buy and sell
price |  order price
size |  order quantity
funds | order funds
dealFunds |  executed size of funds
dealSize | executed quantity
fee | fee
feeCurrency | charge fee currency
stp |  self trade prevention,include `CN`,`CO`,`DC`,`CB`
stop |  stop type, include entry and loss
stopTriggered |  stop order is triggered or not
stopPrice |  stop price
timeInForce | time InForce,include `GTC`,`GTT`,`IOC`,`FOK`
postOnly | postOnly
hidden | hidden order
iceberg | iceberg order
visibleSize | displayed quantity for iceberg order
cancelAfter | cancel orders time，requires timeInForce to be `GTT`
channel | order source
clientOid | user-entered order unique mark
remark | remark
tags | tag order source
isActive |  order status, true and false. If true, the order is active, if false, the order is fillled or cancelled
cancelExist | order cancellation transaction record
createdAt | create time
tradeType | The type of trading

### ORDER STATUS AND SETTLEMENT
Any order on the exchange order book is in active status. Orders removed from the order book will be marked with done status. After an order becomes done, there may be a few milliseconds latency before it’s fully settled.

You can check the orders in any status. If the status parameter is not specified, orders of done status will be returned by default.

When you query orders in active status, there is no time limit. However, when you query orders in done status, the start and end time range cannot exceed 7* 24 hours. An error will occur if the specified time window exceeds the range. If you specify the end time only, the system will automatically calculate the start time as end time minus 7*24 hours, and vice versa.


The history for cancelled orders is only kept for **one month**. You will not be able to query for cancelled orders that have happened more than a month ago.

<aside class="notice">The total number of items retrieved cannot exceed 50,000. If it is exceeded, please shorten the query time range.</aside>

### POLLING
For high-volume trading, it is highly recommended that you maintain your own list of open orders and use one of the streaming market data feeds to keep it updated. You should poll the open orders endpoint to obtain the current state of any open order.

## Recent Orders

```json
{
    "currentPage": 1,
    "pageSize": 1,
    "totalNum": 153408,
    "totalPage": 153408,
    "items": [
        {
            "id": "5c35c02703aa673ceec2a168",
            "symbol": "BTC-USDT",
            "opType": "DEAL",
            "type": "limit",
            "side": "buy",
            "price": "10",
            "size": "2",
            "funds": "0",
            "dealFunds": "0.166",
            "dealSize": "2",
            "fee": "0",
            "feeCurrency": "USDT",
            "stp": "",
            "stop": "",
            "stopTriggered": false,
            "stopPrice": "0",
            "timeInForce": "GTC",
            "postOnly": false,
            "hidden": false,
            "iceberg": false,
            "visibleSize": "0",
            "cancelAfter": 0,
            "channel": "IOS",
            "clientOid": "",
            "remark": "",
            "tags": "",
            "isActive": false,
            "cancelExist": false,
            "createdAt": 1547026471000,
            "tradeType": "TRADE"
        }
    ]
}
```

Request via this endpoint to get 1000 orders in the last 24 hours.
Items are paginated and sorted to show the latest first. See the [Pagination](#pagination) section for retrieving additional entries after the first page.

### HTTP REQUEST
`GET /api/v1/limit/orders`

### Example
`GET /api/v1/limit/orders`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.


### RESPONSES
Field | Description
--------- | -------
orderId | Order ID, unique identifier of an order.
symbol | symbol
opType |  Operation type: DEAL
type | order type, e.g. limit, market, stop_limit
side | transaction direction,include buy and sell
price |  order price
size |  order quantity
funds | order funds
dealFunds |  deal funds
dealSize | deal quantity
fee | fee
feeCurrency | charge fee currency
stp |  self trade prevention,include CN,CO,DC,CB
stop |  stop type, include entry and loss
stopTriggered |  stop order is triggered
stopPrice |  stop price
timeInForce | time InForce,include GTC,GTT,IOC,FOK
postOnly | postOnly
hidden | hidden order
iceberg | iceberg order
visibleSize | display quantity for iceberg order
cancelAfter | cancel orders time，requires timeInForce to be GTT
channel | order source
clientOid | user-entered order unique mark
remark | remark
tags | tag order source
isActive | order status, true and false. If true, the order is active, if false, the order is fillled or cancelled
cancelExist | order cancellation transaction record
createdAt | create time
tradeType | The type of trading : **TRADE**（Spot Trading）.




<aside class="spacer4"></aside>
<aside class="spacer2"></aside>

## Get an order

```json
{
    "id": "5c35c02703aa673ceec2a168",
    "symbol": "BTC-USDT",
    "opType": "DEAL",
    "type": "limit",
    "side": "buy",
    "price": "10",
    "size": "2",
    "funds": "0",
    "dealFunds": "0.166",
    "dealSize": "2",
    "fee": "0",
    "feeCurrency": "USDT",
    "stp": "",
    "stop": "",
    "stopTriggered": false,
    "stopPrice": "0",
    "timeInForce": "GTC",
    "postOnly": false,
    "hidden": false,
    "iceberg": false,
    "visibleSize": "0",
    "cancelAfter": 0,
    "channel": "IOS",
    "clientOid": "",
    "remark": "",
    "tags": "",
    "isActive": false,
    "cancelExist": false,
    "createdAt": 1547026471000,
    "tradeType": "TRADE"
 }
```
Request via this endpoint to get a single order info by order ID.

### HTTP REQUEST
`GET /api/v1/orders/{order-id}`

### Example
`GET /api/v1/orders/5c35c02703aa673ceec2a168`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
orderId | String | Order ID, unique identifier of an order, obtained via the [List orders](#list-orders).

### RESPONSES
Field | Description
--------- | -------
orderId | Order ID, the ID of an order
symbol | symbol
opType |  operation type,deal is pending order,cancel is cancel order
type | order type,e.g. limit,market,stop_limit.
side | transaction direction,include buy and sell
price |  order price
size |  order quantity
funds | order funds
dealFunds |  deal funds
dealSize | deal quantity
fee | fee
feeCurrency | charge fee currency
stp |  self trade prevention,include CN,CO,DC,CB
stop |  stop type, include entry and loss
stopTriggered |  stop order is triggered
stopPrice |  stop price
timeInForce | time InForce,include GTC,GTT,IOC,FOK
postOnly | postOnly
hidden | hidden order
iceberg | iceberg order
visibleSize | display quantity for iceberg order
cancelAfter | cancel orders time，requires timeInForce to be GTT
channel | order source
clientOid | user-entered order unique mark
remark | remark
tags | tag order source
isActive | order status, true and false. If true, the order is active, if false, the order is fillled or cancelled
cancelExist | order cancellation transaction record
createdAt | create time
tradeType | The type of trading : **TRADE**（Spot Trading）.



<aside class="spacer4"></aside>
<aside class="spacer2"></aside>


## Get Single Active Order by clientOid 

```json
{
  "id": "5f3113a1c9b6d539dc614dc6",
  "symbol": "ETH-BTC",
  "opType": "DEAL",
  "type": "limit",
  "side": "buy",
  "price": "0.00001",
  "size": "1",
  "funds": "0",
  "dealFunds": "0",
  "dealSize": "0",
  "fee": "0",
  "feeCurrency": "BTC",
  "stp": "",
  "stop": "",
  "stopTriggered": false,
  "stopPrice": "0",
  "timeInForce": "GTC",
  "postOnly": false,
  "hidden": false,
  "iceberg": false,
  "visibleSize": "0",
  "cancelAfter": 0,
  "channel": "API",
  "clientOid": "6d539dc614db312",
  "remark": "",
  "tags": "",
  "isActive": true,
  "cancelExist": false,
  "createdAt": 1597051810000,
  "tradeType": "TRADE"
}
```

Request via this interface to check the information of a single active order via clientOid. The system will prompt that the order does not exists if the order does not exist or has been settled.


### HTTP REQUEST
`GET /api/v1/order/client-order/{clientOid}`

### Example
`GET /api/v1/order/client-order/6d539dc614db312`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### PARAMETERS
| Param | Type | Description                           |
| ------- | ------ | ----------------------------- |
| clientOid | String | Unique order id created by users to identify their orders |

### RESPONSES
Field | Description
--------- | -------
id | Order ID, the ID of an order
symbol | symbol
opType |  operation type,deal is pending order,cancel is cancel order
type | order type,e.g. limit,market,stop_limit.
side | transaction direction,include buy and sell
price |  order price
size |  order quantity
funds | order funds
dealFunds |  deal funds
dealSize | deal quantity
fee | fee
feeCurrency | charge fee currency
stp |  self trade prevention,include CN,CO,DC,CB
stop |  stop type, include entry and loss
stopTriggered |  stop order is triggered
stopPrice |  stop price
timeInForce | time InForce,include GTC,GTT,IOC,FOK
postOnly | postOnly
hidden | hidden order
iceberg | iceberg order
visibleSize | display quantity for iceberg order
cancelAfter | cancel orders time，requires timeInForce to be GTT
channel | order source
clientOid | user-entered order unique mark
remark | remark
tags | tag order source
isActive | order status, true and false. If true, the order is active, if false, the order is fillled or cancelled
cancelExist | order cancellation transaction record
createdAt | create time
tradeType | The type of trading : **TRADE**（Spot Trading）.



# Fills

## List Fills

```json
{
    "currentPage":1,
    "pageSize":1,
    "totalNum":251915,
    "totalPage":251915,
    "items":[
        {
            "symbol":"BTC-USDT",    //symbol
            "tradeId":"5c35c02709e4f67d5266954e",   //trade id
            "orderId":"5c35c02703aa673ceec2a168",   //order id
            "counterOrderId":"5c1ab46003aa676e487fa8e3",  //counter order id
            "side":"buy",   //transaction direction,include buy and sell
            "liquidity":"taker",  //include taker and maker
            "forceTaker":true,  //forced to become taker
            "price":"0.083",   //order price
            "size":"0.8424304",  //order quantity
            "funds":"0.0699217232",  //order funds
            "fee":"0",  //fee
            "feeRate":"0",  //fee rate
            "feeCurrency":"USDT",  // charge fee currency
            "stop":"",        // stop type
            "type":"limit",  // order type,e.g. limit,market,stop_limit.
            "createdAt":1547026472000,  //time
            "tradeType": "TRADE"
        }
    ]
}
```

Request via this endpoint to get the recent fills.

Items are paginated and sorted to show the latest first. See the [Pagination](#pagination) section for retrieving additional entries after the first page.


### HTTP REQUEST
`GET /api/v1/fills`

### Example
`GET /api/v1/fills`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is **9 times/3s**.

<aside class="notice">This request is paginated.</aside>


### PARAMETERS
You can request fills for specific orders using query parameters.

Param | Type | Description
--------- | ------- | -----------
orderId | String |*[Optional]* Limit the list of fills to this orderId（If you specify orderId, ignore other conditions）
symbol | String |*[Optional]* Limit the list of fills to this symbol
side | String |*[Optional]* **buy** or **sell**
type | String |*[Optional]* **limit**, **market**, **limit_stop** or **market_stop**
startAt| long | *[Optional]*  Start time (milisecond)
endAt| long | *[Optional]* End time (milisecond)
tradeType | String |The type of trading : **TRADE**（Spot Trading）.


### RESPONSES
Field | Description
--------- | -------
symbol | symbol.
tradeId | trade id, it is generated by Matching engine.
orderId | Order ID, unique identifier of an order.
counterOrderId | counter order id.
side | transaction direction,include buy and sell.
price |  order price
size |  order quantity
funds | order funds
type | order type,e.g. limit,market,stop_limit.
fee | fee
feeCurrency | charge fee currency
stop |  stop type, include entry and loss
liquidity |  include taker and maker
forceTaker |  forced to become taker, include true and false
createdAt | create time
tradeType | The type of trading : **TRADE**（Spot Trading）.


**Data time range**

The system allows you to retrieve data up to one week (start from the last day by default). If the time period of the queried data exceeds one week (time range from the start time to end time exceeded 7*24 hours), the system will prompt to remind you that you have exceeded the time limit. If you only specified the start time, the system will automatically calculate the end time (end time = start time + 7 * 24 hours). On the contrary, if you only specified the end time, the system will calculate the start time (start time= end time - 7 * 24 hours) the same way.

<aside class="notice">The total number of items retrieved cannot exceed 50,000. If it is exceeded, please shorten the query time range.</aside>

**Settlement**

The settlement contains two parts:
- **Transactional settlement**
- **Fee settlement**


After an order is matched, the transactional and fee settlement data will be updated in the data store. Once the data is updated, the system would enable the settlement process and will deduct the fees from your pre-frozen assets. After that, the currency will be transferred to the account of the user.  

**Fees**

Orders on Vaex platform are classified into two types， taker and maker. A taker order matches other resting orders on the exchange order book, and gets executed immediately after order entry. A maker order, on the contrary, stays on the exchange order book and awaits to be matched. Taker orders will be charged taker fees, while maker orders will receive maker rebates. Please note that market orders, iceberg orders and hidden orders are always charged taker fees.

The system will pre-freeze a predicted taker fee when you place an order.The liquidity field indicates if the fill was charged taker or maker fees.

With the leading matching engine system in the market, users placing orders on Vaex platform are classified into two types: **taker** and **maker**. **Taker**s, as the taker in the market, would be charged with taker fees; while **maker**s as the maker in the market, would be charged with less fees than the taker, or even get maker fees from Vaex （The exchange platform would compensate the transaction fees for you）.

After placing orders on the Vaex platform, to ensure the execution of these orders, the system would pre-freeze your assets based on the taker fee charges (because the system could not predict the order types you may choose). Please be noted that the system would deduct the fees from the orders entered the orderbook in advance.

If your order is market order, the system would charge taker fees from you.

If your order is limit order and is immediately matched and executed, the system would charge **taker fees** from you. On the contrary, if the order or part or your order is not executed immediately and enters into the order book, the system would charge **maker** **fees** from you if it is executed before being cancelled

After the order is executed and when the left order funds is **0**, the transaction is completed. If the remaining funds is not sufficient to support the minimum product (min.: 0.00000001), then the left part in the order would be cancelled.

If your order is a maker order, the system would return the left pre-frozen **taker** fees to you.

Notice:

- For a **hidden**/**iceberg** order, if it is not executed immediately and becomes a maker order, the system would still charge **taker fees** from you.
- Post Only order will charge you maker fees. If a post only order would get executed immediately against the existing orders (except iceberg and hidden orders) in the market, the order will be cancelled. If the post only order will execute against an iceberg/hidden order immediately, you will get the maker fees.




For example:

Take **BTC/USDT** as the trading pair, if you plan to buy **1 BTC** in market price, suppose the fee charge is **0.1%** and the data on the order book is as follows:

| Price（USDT） | Size（BTC） | Side |
| ------------- | ----------- | ---- |
| 4200.00       | 0.18412309  | sell |
| 4015.60       | 0.56849308  | sell |
| 4011.32       | 0.24738383  | sell |
| 3995.64       | 0.84738383  | buy  |
| 3988.60       | 0.20484000  | buy  |
| 3983.85       | 1.37584908  | buy  |

 When you placed a buy order in market price, the order would be executed immediately. The transaction detail is as follows:

| Price（USDT） | Size（BTC） | Fee（BTC） |
| ------------- | ----------- | ---------- |
| 4011.32       | 0.24738383  | 0.00024738 |
| 4015.60       | 0.56849308  | 0.00056849 |
| 4200.00       | 0.18312409  | 0.00018312 |




## Recent Fills

```json
{
    "code":"200000",
    "data":[
        {
            "counterOrderId":"5db7ee769797cf0008e3beea",
            "createdAt":1572335233000,
            "fee":"0.946357371456",
            "feeCurrency":"USDT",
            "feeRate":"0.001",
            "forceTaker":true,
            "funds":"946.357371456",
            "liquidity":"taker",
            "orderId":"5db7ee805d53620008dce1ba",
            "price":"9466.8",
            "side":"buy",
            "size":"0.09996592",
            "stop":"",
            "symbol":"BTC-USDT",
            "tradeId":"5db7ee8054c05c0008069e21",
            "tradeType":"TRADE",
            "type":"market"
        },
        {
            "counterOrderId":"5db7ee4b5d53620008dcde8e",
            "createdAt":1572335207000,
            "fee":"0.94625",
            "feeCurrency":"USDT",
            "feeRate":"0.001",
            "forceTaker":true,
            "funds":"946.25",
            "liquidity":"taker",
            "orderId":"5db7ee675d53620008dce01e",
            "price":"9462.5",
            "side":"sell",
            "size":"0.1",
            "stop":"",
            "symbol":"BTC-USDT",
            "tradeId":"5db7ee6754c05c0008069e03",
            "tradeType":"TRADE",
            "type":"market"
        },
        {
            "counterOrderId":"5db69aa4688933000aab8114",
            "createdAt":1572248229000,
            "fee":"1.882148318525",
            "feeCurrency":"USDT",
            "feeRate":"0.001",
            "forceTaker":false,
            "funds":"1882.148318525",
            "liquidity":"maker",
            "orderId":"5db69a9c4e6d020008f03275",
            "price":"9354.5",
            "side":"sell",
            "size":"0.20120245",
            "stop":"",
            "symbol":"BTC-USDT",
            "tradeId":"5db69aa477d8de0008c1efac",
            "tradeType":"TRADE",
            "type":"limit"
        }
    ]
}

```


Request via this endpoint to get a list of 1000 fills in the last 24 hours.


### HTTP REQUEST
`GET /api/v1/limit/fills`

### Example
`GET /api/v1/limit/fills`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### RESPONSES
Field | Description
--------- | -------
symbol | symbol
tradeId | trade id, it is generated by Matching engine.
orderId | Order ID, unique identifier of an order.
counterOrderId | counter order id.
side | transaction direction,include buy and sell.
price |  order price
size |  order quantity
funds | order funds
type | order type,e.g. limit,market,stop_limit.
fee | fee
feeCurrency | charge fee currency
stop |  stop type, include entry and loss
liquidity |  include taker and maker
forceTaker |  forced to become taker, include true and false
createdAt | create time
tradeType | The type of trading : **TRADE**（Spot Trading）.
<aside class="spacer2"></aside>


# Market Data

Signature is not required for this part

# Symbols & Ticker

## Get Symbols List

```json
{
    "code": "200000",
    "data": [
        {
            "symbol": "GALAX-USDT",
            "name": "GALA-USDT",
            "baseCurrency": "GALAX",
            "quoteCurrency": "USDT",
            "feeCurrency": "USDT",
            "market": "USDS",
            "baseMinSize": "10",
            "quoteMinSize": "0.001",
            "baseMaxSize": "10000000000",
            "quoteMaxSize": "99999999",
            "baseIncrement": "0.0001",
            "quoteIncrement": "0.00001",
            "priceIncrement": "0.00001",
            "priceLimitRate": "0.1",
            "minFunds": "0.1",
            "isMarginEnabled": true,
            "enableTrading": true
        },
        {
            "symbol": "XLM-USDT",
            "name": "XLM-USDT",
            "baseCurrency": "XLM",
            "quoteCurrency": "USDT",
            "feeCurrency": "USDT",
            "market": "USDS",
            "baseMinSize": "0.1",
            "quoteMinSize": "0.01",
            "baseMaxSize": "10000000000",
            "quoteMaxSize": "99999999",
            "baseIncrement": "0.0001",
            "quoteIncrement": "0.000001",
            "priceIncrement": "0.000001",
            "priceLimitRate": "0.1",
            "minFunds": "0.1",
            "isMarginEnabled": true,
            "enableTrading": true
        }
    ]
}
```

Request via this endpoint to get a list of available currency pairs for trading.
If you want to get the market information of the trading symbol, please use [Get All Tickers](#get-all-tickers).

### HTTP REQUEST
`GET /api/v2/symbols`

### Example
`GET /api/v2/symbols`

### PARAMETERS
Param | Type | Mandatory  | Description | 
--------- | ------- | -----------| -----------| 
market | String | No | The [trading market](#get-market-list). | 

### RESPONSES
Field |  Description
--------- | -----------
symbol | unique code of a symbol, it would not change after renaming
name | Name of trading pairs, it would change after renaming
baseCurrency |  Base currency,e.g. BTC.
quoteCurrency |  Quote currency,e.g. USDT.
market |  The [trading market](#get-market-list).
baseMinSize |  The minimum order quantity requried to place an order.
quoteMinSize | The minimum order funds required to place a market order.
baseMaxSize |  The maximum order size required to place an order.
quoteMaxSize | The maximum order funds  required to place a market order.
baseIncrement |The increment of the order size. The value shall be a positive multiple of the baseIncrement.
quoteIncrement | The increment of the funds required to place a market order. The value shall be a positive multiple of the quoteIncrement.
priceIncrement |  The increment of the price required to place a limit order. The value shall be a positive multiple of the priceIncrement.
feeCurrency | The currency of charged fees.
enableTrading |  Available for transaction or not.
isMarginEnabled |  Available for margin or not.
priceLimitRate | Threshold for price portection
minFunds | the minimum spot trading amounts

The `baseMinSize` and `baseMaxSize` fields define the min and max order size. The `priceIncrement` field specifies the min order price as well as the price increment.This also applies to `quote` currency.

The order `price` must be a positive integer multiple of this `priceIncrement` (i.e. if the increment is 0.01, the  0.001 and 0.021 order prices would be rejected).

`priceIncrement` and `quoteIncrement` may be adjusted in the future. We will notify you by email and site notifications before adjustment.

Order Type | Follow the rules of minFunds
--------- | ------- | -----------
Limit Buy |	[Order Amount * Order Price] >= `minFunds`
Limit Sell |	[Order Amount * Order Price] >= `minFunds`
Market Buy |	Order Value >= `minFunds`
Market Sell | [Order Amount * Last Price of Base Currency] >= `minFunds`

Note:

* API market buy orders (by amount) valued at [Order Amount * Last Price of Base Currency] <`minFunds` will be rejected.
* API market sell orders (by value) valued at <`minFunds` will be rejected.
* Take profit and stop loss orders at market or limit prices will be rejected when triggered.

## Get Ticker

```json
//Get Ticker
{
    "sequence": "1550467636704",
    "bestAsk": "0.03715004",
    "size": "0.17",
    "price": "0.03715005",
    "bestBidSize": "3.803",
    "bestBid": "0.03710768",
    "bestAskSize": "1.788",
    "time": 1550653727731
}
```

Request via this endpoint to get Level 1 Market Data. The returned value includes the best bid price and size, the best ask price and size as well as the last traded price and the last traded size.

### HTTP REQUEST
`GET /api/v1/market/orderbook/level1`


### Example
`GET /api/v1/market/orderbook/level1?symbol=BTC-USDT`

### PARAMETERS

Param | Type | Description
--------- | ------- | -----------
symbol | String |[symbol](#get-symbols-list)


### RESPONSES
Field |  Description
--------- | -----------
sequence | Sequence
bestAsk |  Best ask price
size | Last traded size
price |  Last traded price
bestBidSize | Best bid size
bestBid | Best bid price
bestAskSize |  Best ask size
time |  timestamp
<aside class="spacer2"></aside>




# Order Book

## Get Part Order Book(aggregated)

```json
{
    "sequence": "3262786978",
    "time": 1550653727731,
    "bids": [["6500.12", "0.45054140"],
             ["6500.11", "0.45054140"]],  //[price，size]
    "asks": [["6500.16", "0.57753524"],
             ["6500.15", "0.57753524"]]   
}
```

Request via this endpoint to get a list of open orders for a symbol.

Level-2 order book includes all bids and asks (aggregated by price), this level returns only one size for each active price (as if there was only a single order for that price).

Query via this endpoint and the system will return only part of the order book to you. If you request level2_20, the system will return you 20 pieces of data (ask and bid data) on the order book. If you request level_100, the system will return 100 pieces of data (ask and bid data) on the order book to you. You are recommended to request via this endpoint as the system reponse would be faster and cosume less traffic.


To maintain up-to-date Order Book, please use [Websocket](#level-2-market-data) incremental feed after retrieving the Level 2 snapshot.



### HTTP REQUEST
`GET /api/v1/market/orderbook/level2_20`<br/>
`GET /api/v1/market/orderbook/level2_100`

### Example
`GET /api/v1/market/orderbook/level2_20?symbol=BTC-USDT`<br/>
`GET /api/v1/market/orderbook/level2_100?symbol=BTC-USDT`

### PARAMETERS

Param | Type | Description
--------- | ------- | -----------
symbol | String | [symbol](#get-symbols-list)

### RESPONSES

Field |  Description
--------- | -----------
sequence | Sequence number
time | Timestamp
bids | bids
asks | asks

### Data Sort

**Asks**: Sort price from low to high

**Bids**: Sort price from high to low

## Get Full Order Book(aggregated)

```json
{
    "sequence": "3262786978",
    "time": 1550653727731,
    "bids": [["6500.12", "0.45054140"],
             ["6500.11", "0.45054140"]],  //[price，size]
    "asks": [["6500.16", "0.57753524"],
             ["6500.15", "0.57753524"]]  
}
```

Request via this endpoint to get the order book of the specified symbol.

Level 2 order book includes all bids and asks (aggregated by price). This level returns only one aggregated size for each price (as if there was only one single order for that price).

This API will return data with **full** depth.

It is generally used by professional traders because it uses more server resources and traffic, and we have strict access frequency control.

To maintain up-to-date Order Book, please use [Websocket](#level-2-market-data) incremental feed after retrieving the Level 2 snapshot.


### HTTP REQUEST
`GET /api/v3/market/orderbook/level2`(Recommend)

### Example
`GET /api/v3/market/orderbook/level2?symbol=BTC-USDT`

### API KEY PERMISSIONS
This endpoint requires the **"General"** permission.

### REQUEST RATE LIMIT
This API is restricted for each account, the request rate limit is **30 times/3s**.

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
symbol | String | [symbol](#get-symbols-list)

### RESPONSES
Field |  Description
--------- | -----------
sequence | Sequence number
time | Timestamp
bids | bids
asks | asks

### Data Sor

**Asks**: Sort price from low to high

**Bids**: Sort price from high to low

<aside class="spacer4"></aside>

# Histories

## Get Trade Histories

```json
[
    {
        "sequence": "1545896668571",
        "price": "0.07",                      //Filled price
        "size": "0.004",                      //Filled amount
        "side": "buy",                        //Filled side. The filled side is set to the taker by default.
        "time": 1545904567062140823           //Transaction time
    },
    {
        "sequence": "1545896668578",
        "price": "0.054",
        "size": "0.066",
        "side": "buy",
        "time": 1545904581619888405
    }
]
```
Request via this endpoint to get the trade history of the specified symbol.


### HTTP REQUEST
`GET /api/v1/market/histories`

### Example
`GET /api/v1/market/histories?symbol=BTC-USDT`

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
symbol | String | [symbol](#get-symbols-list)

### RESPONSES
Field |  Description
--------- | -----------
sequence | Sequence number
time | Transaction time
price | Filled price
size |  Filled amount
side | Filled side. The filled side is set to the taker by default


###SIDE
The trade side indicates the taker order side. A taker order is the order that was matched with orders opened on the order book.

<aside class="spacer2"></aside>

## Get Klines

```json
[
    [
        "1545904980",             //Start time of the candle cycle
        "0.058",                  //opening price
        "0.049",                  //closing price
        "0.058",                  //highest price
        "0.049",                  //lowest price
        "0.018",                  //Transaction volume
        "0.000945"                //Transaction amount
    ],
    [
        "1545904920",
        "0.058",
        "0.072",
        "0.072",
        "0.058",
        "0.103",
        "0.006986"
    ]
]
```
Request via this endpoint to get the kline of the specified symbol. Data are returned in grouped buckets based on requested type.


<aside class="notice"> Klines data may be incomplete. No data is published for intervals where there are no ticks.</aside>

<aside class="warning"> Klines should not be polled frequently. If you need real-time information, use the trade and book endpoints along with the websocket feed.</aside>

### HTTP REQUEST
`GET /api/v1/market/candles`

### Example
`GET /api/v1/market/candles?type=1min&symbol=BTC-USDT&startAt=1566703297&endAt=1566789757`

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
symbol | String | [symbol](#get-symbols-list)
startAt| long | *[Optional]*  Start time (second), default is 0
endAt| long | *[Optional]* End time (second), default is 0
type | String |Type of candlestick patterns: **1min, 3min, 5min, 15min, 30min, 1hour, 2hour, 4hour, 6hour, 8hour, 12hour, 1day, 1week**

<aside class="notice">For each query, the system would return at most **1500** pieces of data. To obtain more data, please page the data by time.</aside>

### RESPONSES
Field |  Description
--------- | -----------
time | Start time of the candle cycle
open |  Opening price
close | Closing price
high | Highest price
low | Lowest price
volume |Transaction volume
turnover | Transaction amount



# Currencies


## Get Currencies

```json
[
  {
    "currency": "CSP",
    "name": "CSP",
    "fullName": "Caspian",
    "precision": 8,
    "confirms": 12,
    "contractAddress": "0xa6446d655a0c34bc4f05042ee88170d056cbaf45",
    "withdrawalMinSize": "2000",
    "withdrawalMinFee": "1000",
    "isWithdrawEnabled": true,
    "isDepositEnabled": true,
    "isMarginEnabled": false,
    "isDebitEnabled": false
  },
  {
    "currency": "LOKI",
    "name": "OXEN",
    "fullName": "Oxen",
    "precision": 8,
    "confirms": 10,
    "contractAddress": "",
    "withdrawalMinSize": "2",
    "withdrawalMinFee": "2",
    "isWithdrawEnabled": true,
    "isDepositEnabled": true,
    "isMarginEnabled": false,
    "isDebitEnabled": true
  }
]
```

Request via this endpoint to get the currency list.

<aside class="notice">Not all currencies currently can be used for trading.</aside>


### HTTP REQUEST
`GET /api/v1/currencies`

### Example
`GET /api/v1/currencies`

### RESPONSES
|Field | Description|
|-----|-------------|
|currency| A unique currency code that will never change|
|name| Currency name, will change after renaming|
|fullName| Full name of a currency, will change after renaming |
|precision| Currency precision |
|confirms| Number of block confirmations |
|contractAddress| Contract address |
|withdrawalMinSize| Minimum withdrawal amount |
|withdrawalMinFee|  Minimum fees charged for withdrawal |
|isWithdrawEnabled| Support withdrawal or not |
|isDepositEnabled| Support deposit or not |
|isMarginEnabled| Support margin or not |
|isDebitEnabled| Support debit or not |


**CURRENCY CODES**

Currency codes will conform to the ISO 4217 standard where possible. Currencies which have or had no representation in ISO 4217 may use a custom code.

Code | Description
--------- | -------  
BTC | Bitcoin
ETH | Ethereum

For a coin, the "**currency**" is a fixed value and works as the only recognized identity of the coin. As the "**name**", "**fullnane**" and "**precision**" of a coin are modifiable values, when the "**name**" of a coin is changed, you should use "**currency**" to get the coin.

For example:

The "**currency**" of XRB is "XRB", if the "**name**" of XRB is changed into "**Nano**", you should use "XRB" (the currency of XRB) to search the coin.

## Get Currency Detail
```json
{
  "currency": "BTC",
  "name": "BTC",
  "fullName": "Bitcoin",
  "precision": 8,
  "confirms": 2,
  "contractAddress": "",
  "withdrawalMinSize": "0.001",
  "withdrawalMinFee": "0.0006",
  "isWithdrawEnabled": true,
  "isDepositEnabled": true,
  "isMarginEnabled": true,
  "isDebitEnabled": true
}
```
Request via this endpoint to get the currency details of a specified currency

### HTTP REQUEST
`GET /api/v1/currencies/{currency}`

### Example
`GET /api/v1/currencies/BTC`

<aside class="notice">Details of the currency.</aside>

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
currency | String | **Path parameter**. [Currency](#get-currencies)
chain | String | *[Optional]* Support for querying the chain of currency, e.g.  This only apply for multi-chain currency, and there is no need for single chain currency.

### RESPONSES
|Field | Description|
|-----|-------------|
|currency| A unique currency code that will never change|
|name| Currency name, will change after renaming |
|fullName| Full name of a currency, will change after renaming |
|precision| Currency precision |
|confirms| Number of block confirmations|
|contractAddress| Contract address|
|withdrawalMinSize| Minimum withdrawal amount |
|withdrawalMinFee| Minimum fees charged for withdrawal |
|isWithdrawEnabled| Support withdrawal or not |
|isDepositEnabled| Support deposit or not |
|isMarginEnabled| Support margin or not |
|isDebitEnabled| Support debit or not |

## Get Currency Detail(Recommend)
```json
{
    "code": "200000",
    "data": {
        "currency": "BTC",
        "name": "BTC",
        "fullName": "Bitcoin",
        "precision": 8,
        "confirms": null,
        "contractAddress": null,
        "isMarginEnabled": true,
        "isDebitEnabled": true,
        "chains": [
            {
                "chainName": "BTC",
                "chain": "btc",
                "withdrawalMinSize": "0.001",
                "withdrawalMinFee": "0.0005",
                "isWithdrawEnabled": true,
                "isDepositEnabled": true,
                "confirms": 2,
                "contractAddress": ""
            },x
            ...
        ]
    }
}
```
Request via this endpoint to get the currency details of a specified currency

### HTTP REQUEST
`GET /api/v2/currencies/{currency}`

### Example
`GET /api/v2/currencies/BTC`

<aside class="notice">Recommended for use</aside>

### PARAMETERS
Param | Type | Description
--------- | ------- | -----------
currency | String | **Path parameter**. [Currency](#get-currencies)
chain | String | *[Optional]* Support for querying the chain of currency, return the currency details of all chains by default.

### RESPONSES
|Field | Description|
|-----|-------------|
|currency| A unique currency code that will never change|
|name| Currency name, will change after renaming |
|fullName| Full name of a currency, will change after renaming |
|precision| Currency precision |
|confirms| Number of block confirmations|
|contractAddress| Contract address|
|withdrawalMinSize| Minimum withdrawal amount |
|chainName| chain name of currency |
|chain| chain of currency |
|withdrawalMinFee| Minimum fees charged for withdrawal |
|isWithdrawEnabled| Support withdrawal or not |
|isDepositEnabled| Support deposit or not |
|isMarginEnabled| Support margin or not |
|isDebitEnabled| Support debit or not |

## Get Fiat Price
```json
{
    "code": "200000",
    "data": {
        "BTC": "3911.28000000",
        "ETH": "144.55492453",
        "LTC": "48.45888179"
    }
}
```
Request via this endpoint to get the fiat price of the currencies for the available trading pairs.  

### HTTP REQUEST
`GET /api/v1/prices`

### Example
`GET /api/v1/prices`

### PARAMETERS
|Param | Type | Description|
|--------- | ------- | -----------|
| base | String |*[Optional]* Ticker symbol of a base currency,eg.USD,EUR. Default is USD |
| currencies | String |*[Optional]* Comma-separated cryptocurrencies to be converted into fiat, e.g.: BTC,ETH, etc. Default to return the fiat price of all currencies.|






# Websocket Feed

While there is a strict access frequency control for REST API, we highly recommend that API users utilize Websocket to get the real-time data.


<aside class="notice">The recommended way is to just create a websocket connection and subscribe to multiple channels.
</aside>

## Apply connect token

```json
{
	"code": "200000",
	"data": {
		"token": "2neAiuYvAU61ZDXANAGAsiL4-iAExhsBXZxftpOeh_55i3Ysy2q2LEsEWU64mdzUOPusi34M_wGoSf7iNyEWJ4aBZXpWhrmY9jKtqkdWoFa75w3istPvPtiYB9J6i9GjsxUuhPw3BlrzazF6ghq4L_u0MhKxG3x8TeN4aVbNiYo=.mvnekBb8DJegZIgYLs2FBQ==",
		"instanceServers": [{
			"endpoint": "wss://ws-api-spot.vaex.com/",	//It is recommended to use a dynamic URL, which may change
			"encrypt": true,
			"protocol": "websocket",
			"pingInterval": 18000,
			"pingTimeout": 10000
		}]
	}
}

```

You need to apply for one of the two tokens below to create a websocket connection.

### Public token (No authentication required):

If you only use public channels (e.g. all public market data), please make request as follows to obtain the server list and temporary public token:

#### HTTP REQUEST
`POST /api/v1/bullet-public`

### Private channels (Authentication request required):

For private channels and messages (e.g. account balance notice), please make request as follows after authorization to obtain the server list and authorized token.

#### HTTP REQUEST
`POST /api/v1/bullet-private`


### Response
|Field | Description|
-----|-----
|endpoint| Websocket server address for establishing connection|
|protocol| Protocol supported|
|encrypt| Indicate whether SSL encryption is used |
|pingInterval| Recommended to send ping interval in millisecond|
|pingTimeout| After such a long time(millisecond), if you do not receive pong, it will be considered as disconnected. |
|token| token|

## Create connection

```javascript
var socket = new WebSocket("wss://ws-api-spot.vaex.com/?token==xxx&[connectId=xxxxx]");
```


When the connection is successfully established, the system will send a welcome message.

<aside class="notice">Only when the welcome message is received will the connection be available</aside>

**connectId**: the connection id, a unique value taken from the client side. Both the id of the welcome message and the id of the error message are connectId.

If you only want to receive private messages of the specified topic, please set privateChannel to true when subscribing.

```json
{
    "id":"hQvf8jkno",
    "type":"welcome"
}
```



<aside class="spacer2"></aside>

## Ping
```json
{
    "id":"1545910590801",
    "type":"ping"
}
```


To prevent the TCP link being disconnected by the server, the client side needs to send ping messages every pingInterval time to the server to keep alive the link.

After the ping message is sent to the server, the system would return a pong message to the client side.

If the server has not received any message from the client for a long time, the connection will be disconnected.


```json
{
    "id":"1545910590801",
    "type":"pong"
}
```
<aside class="spacer3"></aside>

## Subscribe

```json
{
    "id": 1545910660739,                          //The id should be an unique value
    "type": "subscribe",
    "topic": "/market/ticker:BTC-USDT,ETH-USDT",  //Topic needs to be subscribed. Some topics support to divisional subscribe the informations of multiple trading pairs through ",".
    "privateChannel": false,                      //Adopted the private channel or not. Set as false by default.
    "response": true                              //Whether the server needs to return the receipt information of this subscription or not. Set as false by default.
}
```

To subscribe channel messages from a certain server, the client side should send subscription message to the server.

If the subscription succeeds, the system will send ack messages to you, when the response is set as true.


```json
{
    "id":"1545910660739",
    "type":"ack"
}
```
While there are topic messages generated, the system will send the corresponding messages to the client side. For details about the message format, please check the definitions of topics.

### Parameters
#### ID
ID is unique string to mark the request which is same as id property of ack.

#### Topic
The topic you want to subscribe to.

#### PrivateChannel
For some specific topics (e.g. /market/level2), **privateChannel** is available. The default value of **privateChannel** is **False**. If the **privateChannel** is set to **true**, the user will only receive messages related himself on the topic.

#### Response
If the response is set as true, the system will return the ack messages after the subscription succeed.

## UnSubscribe
Unsubscribe from topics you have subscribed to.

```json
{
    "id": "1545910840805",                            //The id should be an unique value
    "type": "unsubscribe",
    "topic": "/market/ticker:BTC-USDT,ETH-USDT",      //Topic needs to be unsubscribed. Some topics support to divisional unsubscribe the informations of multiple trading pairs through ",".
    "privateChannel": false,
    "response": true                                  //Whether the server needs to return the receipt information of this subscription or not. Set as false by default.
}
```

```json
{
    "id": "1545910840805",
    "type": "ack"
}
```

### Parameters

#### ID
Unique string to mark the request.

#### Topic
The topic you want to subscribe.

#### PrivateChannel
If the **privateChannel** is set to **true**, the private topic will be unsubscribed.

#### Response
If the response is set as true, the system would return the ack messages after the unsubscription succeed.

## Multiplex

In one physical connection, you could open different multiplex tunnels to subscribe different **topics** for different data.


For example, enter the command below to open **bt1** multiple tunnel :

 {"id": "1Jpg30DEdU", "type": "openTunnel", "newTunnelId": "bt1", "response": true}

Add “**tunnelId**” in the command:

{"id": "1JpoPamgFM", "type": "subscribe", "topic": "/market/ticker:ETH-USDT"，"tunnelId": "bt1", "response": true}

You would then, receive messages corresponding to the id **tunnelIId**:  

{"id": "1JpoPamgFM", "type": "message", "topic": "/market/ticker:ETH-USDT", "subject": "trade.ticker", "tunnelId": "bt1", "data": {...}}

To close the **tunnel**, you can enter the command below:

{"id": "1JpsAHsxKS", "type": "closeTunnel", "tunnelId": "bt1", "response": true}

##### Limitations

1. The multiplex **tunnel** is provided for API users only.
2. The maximum multiplex tunnels available: **5**.


## Sequence Numbers
The sequence field exists in order book, trade history and snapshot messages by default and the Level 3 and Level 2 data works to ensure the full connection of the sequence. If the sequence is non-sequential, please enable the calibration logic.


## General Logic for Message Judgement in Client Side
1.Judge message type. There are three types of messages at present: message (the commonly used messages for push), notice (the notices generally used), and command (consecutive command).

2.Judge messages by topic. You could judge the message type through the topic.

3.Judge messages by subject. For the same type of messages with the same topic, you could judge the type of messages through their subjects.

# Public Channels

## Symbol Ticker

```json

{
    "id": 1545910660739,
    "type": "subscribe",
    "topic": "/market/ticker:BTC-USDT",
    "response": true
}
```


```json
{
    "type":"message",
    "topic":"/market/ticker:BTC-USDT",
    "subject":"trade.ticker",
    "data":{
        "sequence":"1545896668986", // Sequence number
        "price":"0.08",             // Last traded price
        "size":"0.011",             //  Last traded amount
        "bestAsk":"0.08",          // Best ask price
        "bestAskSize":"0.18",      // Best ask size
        "bestBid":"0.049",         // Best bid price
        "bestBidSize":"0.036"     // Best bid size
    }
}
```

Topic: `/market/ticker:{symbol},{symbol}...`

* Push frequency: once every `100ms`

Subscribe to this topic to get the push of BBO changes.

Please note that more information may be added to messages from this channel in the near future.


<aside class="spacer2"></aside>
<aside class="spacer4"></aside>


## All Symbols Ticker

```json

{
    "id": 1545910660739,                          
    "type": "subscribe",
    "topic": "/market/ticker:all",
    "response": true                              
}
```
```json
{
    "type":"message",
    "topic":"/market/ticker:all",
    "subject":"BTC-USDT",
    "data":{
        "sequence":"1545896668986",
        "bestAsk":"0.08",
        "size":"0.011",
        "bestBidSize":"0.036",
        "price":"0.08",
        "bestAskSize":"0.18",
        "bestBid":"0.049"
    }
}
```
Topic: `/market/ticker:all`

* Push frequency: once every `100ms`

Subscribe to this topic to get the push of all market symbols BBO change.
<aside class="spacer8"></aside>




## Level-2 Market Data

```json
{
    "id": 1545910660740,                          
    "type": "subscribe",
    "topic": "/market/level2:BTC-USDT",
    "response": true                              
}
```

Topic: `/market/level2:{symbol},{symbol}...`

* Push frequency:`real-time`

Subscribe to this topic to get Level2 order book data.

When the websocket subscription is successful,  the system would send the increment change data pushed by the websocket to you.

```json
{
    "type": "message",
    "topic": "/market/level2:BTC-USDT",
    "subject": "trade.l2update",
    "data": {
        "changes": {
            "asks": [
                [
                    "18906",//price
                    "0.00331",//size
                    "14103845"//sequence
                ],
                [
                    "18907.3",
                    "0.58751503",
                    "14103844"
                ]
            ],
            "bids": [
                [
                    "18891.9",
                    "0.15688",
                    "14103847"
                ]
            ]
        },
        "sequenceEnd": 14103847,
        "sequenceStart": 14103844,
        "symbol": "BTC-USDT",
        "time": 1663747970273//milliseconds
    }
}
```

Calibration procedure：

1. After receiving the websocket Level 2 data flow, cache the data.
2. Initiate a [REST](#get-full-order-book-aggregated) request to get the snapshot data of Level 2 order book.
3. Playback the cached Level 2 data flow.
4. Apply the new Level 2 data flow to the local snapshot to ensure that `sequenceStart(new)<=sequenceEnd+1(old)` and `sequenceEnd(new) > sequenceEnd(old)`. The sequence on each record in changes only represents the last modification of the corresponding sequence of the price, and does not serve as a basis for judging message continuity.
5. Update the level2 full data based on sequence according to the price and size. If the price is 0, ignore the messages and update the sequence. If the size=0, update the sequence and remove the price of which the size is 0 out of level 2. For other cases, please update the price.


**Example**

Take BTC/USDT as an example, suppose the current order book data in level 2 is as follows:

After subscribing to the channel, you would receive changes as follows:


<code>
...<br/>
"asks":[<br/>
&nbsp;&nbsp;["3988.59","3", 16], // ignore it because sequence = 16<br/>
&nbsp;&nbsp;["3988.61","0", 19], // Remove 3988.61<br/>
&nbsp;&nbsp;["3988.62","8", 15], // ignore it because sequence < 16 <br/>
]<br/>
"bids":[<br/>
&nbsp;&nbsp;["3988.50", "44", "18"] // Update size of 3988.50 to 44<br/>
]<br/>
"sequenceEnd": 15,<br/>
"sequenceStart": 19,<br/>
...<br/>
</code>

<aside class="notice">The sequence on each record in changes only represents the last modification of the corresponding sequence of the price, not as a basis for judging the continuity of the message; for example, when there are multiple updates at the same price ["3988.50", "20", "17" "], ["3988.50", "44", "18"], at this time only the latest ["3988.50", "44", "18"] will be pushed</aside>

Get a snapshot of the order book through a **REST** request ([Get Order Book](#get-part-order-book-aggregated)) to build a local order book. Suppose that data we got is as follows:

<code>
...<br/>
"sequence": "16",<br/>
"asks":[<br/>
&nbsp;&nbsp;["3988.62","8"],//[Price, Size]<br/>
&nbsp;&nbsp;["3988.61","32"],<br/>
&nbsp;&nbsp;["3988.60","47"],<br/>
&nbsp;&nbsp;["3988.59","3"],<br/>
]<br/>
"bids":[<br/>
&nbsp;&nbsp;["3988.51","56"],<br/>
&nbsp;&nbsp;["3988.50","15"],<br/>
&nbsp;&nbsp;["3988.49","100"],<br/>
&nbsp;&nbsp;["3988.48","10"]<br/>
]<br/>
...<br/>
</code>

The current data on the local order book is as follows:

<code>
| Price | Size | Side |<br/>
|---------|-----|------|<br/>
| 3988.62 | 8&nbsp;&nbsp;    | Sell |<br/>
| 3988.61 | 32&nbsp;   | Sell |<br/>
| 3988.60 | 47&nbsp;   | Sell |<br/>
| 3988.59 | 3&nbsp;&nbsp;    | Sell |<br/>
| 3988.51 | 56&nbsp;   | Buy&nbsp;  |<br/>
| 3988.50 | 15&nbsp;   | Buy&nbsp;  |<br/>
| 3988.49 | 100  | Buy&nbsp;  |<br/>
| 3988.48 | 10&nbsp;  | Buy&nbsp;  |<br/>
</code>

In the beginning, the sequence of the order book is `16`. Discard the feed data of sequence that is below or equals to `16`, and apply playback the sequence `[18,19]` to update the snapshot of the order book. Now the sequence of your order book is `19` and your local order book is up-to-date.

**Diff:**

- Update size of 3988.50 to 44 (Sequence 18)
- Remove 3988.61 (Sequence 19)

Now your current order book is up-to-date and final data is as follows:

<code>
| Price | Size | Side |<br/>
|---------|-----|------|<br/>
| 3988.62 | 8&nbsp;&nbsp;    | Sell |<br/>
| 3988.60 | 47&nbsp;    | Sell |<br/>
| 3988.59 | 3&nbsp;&nbsp; | Sell |<br/>
| 3988.51 | 56&nbsp;    | Buy&nbsp;   |<br/>
| 3988.50 | 44&nbsp;    | Buy&nbsp;   |<br/>
| 3988.49 | 100  | Buy&nbsp;   |<br/>
| 3988.48 | 10&nbsp;   | Buy&nbsp;   |<br/>
</code>


## Level2 - 50 best ask/bid orders
```json
{
    "type": "message",
    "topic": "/spotMarket/level2Depth50:BTC-USDT",
    "subject": "level2",
    "data": {
	      "asks":[
            ["9993","3"],     //price,size
            ["9992","3"],
            ["9991","47"],
            ["9990","32"],
            ["9989","8"]
        ],
        "bids":[
            ["9988","56"],
            ["9987","15"],
            ["9986","100"],
            ["9985","10"],
            ["9984","10"]
        ]
        "timestamp": 1586948108193
    }
}
```

Topic: `/spotMarket/level2Depth50:{symbol},{symbol}...`

* Push frequency: once every `100ms`

The system will return the 50 best ask/bid orders data, which is the snapshot data of every 100 milliseconds (in other words, the 50 best ask/bid orders data returned every 100 milliseconds in real-time).

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer"></aside>

## Klines

```json
{
    "type":"message",
    "topic":"/market/candles:BTC-USDT_1hour",
    "subject":"trade.candles.update",
    "data":{
        "symbol":"BTC-USDT",    // symbol
        "candles":[
            "1589968800",   // Start time of the candle cycle
            "9786.9",       // open price
            "9740.8",       // close price
            "9806.1",       // high price
            "9732",         // low price
            "27.45649579",  // Transaction volume
            "268280.09830877"   // Transaction amount
        ],
        "time":1589970010253893337  // now（us）
    }
}
```
Topic: `/market/candles:{symbol}_{type}`

* Push frequency: `real-time`

Param |  Description
--------- | -------
symbol | [symbol](#get-symbols-list)
type |  `1min`, `3min`, `15min`, `30min`, `1hour`, `2hour`, `4hour`, `6hour`, `8hour`, `12hour`, `1day`, `1week`


Subscribe to this topic to get K-Line data.


## Match Execution Data

```json
{
    "id": 1545910660741,                          
    "type": "subscribe",
    "topic": "/market/match:BTC-USDT",
    "privateChannel": false,                      
    "response": true                              
}
```
Topic: `/market/match:{symbol},{symbol}...`

* Push frequency: `real-time`

Subscribe to this topic to get the matching event data flow of Level 3.

For each order traded, the system would send you the match messages in the following format.

```json
{
    "type":"message",
    "topic":"/market/match:BTC-USDT",
    "subject":"trade.l3match",
    "data":{
        "sequence":"1545896669145",
        "type":"match",
        "symbol":"BTC-USDT",
        "side":"buy",
        "price":"0.08200000000000000000",
        "size":"0.01022222000000000000",
        "tradeId":"5c24c5da03aa673885cd67aa",
        "takerOrderId":"5c24c5d903aa6772d55b371e",
        "makerOrderId":"5c2187d003aa677bd09d5c93",
        "time":"1545913818099033203"
    }
}
```
<aside class="spacer8"></aside>






# Private Channels

Subscribe to private channels require `privateChannel=“true”`.


## Private Order Change Events

Topic: `/spotMarket/tradeOrders`

* Push frequency: `real-time`

This topic will push all change events of your orders.


**Order Status**

“match”: when taker order executes with orders in the order book, the taker order status is “match”;

“open”: the order is in the order book;

“done”: the order is fully executed successfully;


### Message Type

#### open
```json
{
    "type":"message",
    "topic":"/spotMarket/tradeOrders",
    "subject":"orderChange",
    "channelType":"private",
    "data":{
        "symbol":"BTC-USDT",
        "orderType":"limit",
        "side":"buy",
        "orderId":"5efab07953bdea00089965d2",
        "type":"open",
        "orderTime":1670329987026,
        "size":"0.1",
        "filledSize":"0",
        "price":"0.937",
        "clientOid":"1593487481000906",
        "remainSize":"0.1",
        "status":"open",
        "ts":1670329987311000000
    }
}
```
when the order enters into the order book;

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer2"></aside>

#### match
```json
{
    "type":"message",
    "topic":"/spotMarket/tradeOrders",
    "subject":"orderChange",
    "channelType":"private",
    "data":{
        "symbol":"BTC-USDT",
        "orderType":"limit",
        "side":"sell",
        "orderId":"5efab07953bdea00089965fa",
        "liquidity":"taker",
        "type":"match",
        "orderTime":1670329987026,
        "size":"0.1",
        "filledSize":"0.1",
        "price":"0.938",
        "matchPrice":"0.96738",
        "matchSize":"0.1",
        "tradeId":"5efab07a4ee4c7000a82d6d9",
        "clientOid":"1593487481000313",
        "remainSize":"0",
        "status":"match",
        "ts":1670329987311000000
    }
}
```
when the order has been executed;

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer2"></aside>

#### filled
```json
{
    "type":"message",
    "topic":"/spotMarket/tradeOrders",
    "subject":"orderChange",
    "channelType":"private",
    "data":{
        "symbol":"ETH-USDT",
        "orderType":"limit",
        "side":"sell",
        "orderId":"5efab07953bdea00089965fa",
        "type":"filled",
        "orderTime":1670329987026,
        "size":"0.1",
        "filledSize":"0.1",
        "price":"0.938",
        "clientOid":"1593487481000313",
        "remainSize":"0",
        "status":"done",
        "ts":1670329987311000000
    }
}
```
when the order has been executed and its status was changed into DONE;

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer2"></aside>

#### canceled
```json
{
    "type":"message",
    "topic":"/spotMarket/tradeOrders",
    "subject":"orderChange",
    "channelType":"private",
    "data":{
        "symbol":"BTC-USDT",
        "orderType":"limit",
        "side":"buy",
        "orderId":"5efab07953bdea00089965d2",
        "type":"canceled",
        "orderTime":1670329987026,
        "size":"0.1",
        "filledSize":"0",
        "price":"0.937",
        "clientOid":"1593487481000906",
        "remainSize":"0",
        "status":"done",
        "ts":1670329987311000000
    }
}
```
when the order has been cancelled and its status was changed into DONE;

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer2"></aside>

#### update
```json
{
    "type":"message",
    "topic":"/spotMarket/tradeOrders",
    "subject":"orderChange",
    "channelType":"private",
    "data":{
        "symbol":"BTC-USDT",
        "orderType":"limit",
        "side":"buy",
        "orderId":"5efab13f53bdea00089971df",
        "type":"update",
        "oldSize":"0.1",
        "orderTime":1670329987026,
        "size":"0.06",
        "filledSize":"0",
        "price":"0.937",
        "clientOid":"1593487679000249",
        "remainSize":"0.06",
        "status":"open",
        "ts":1670329987311000000
    }
}
```
when the order has been updated;

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>

## Account Balance Notice
```json
{
    "type": "message",
	  "topic": "/account/balance",
	  "subject": "account.balance",
    "channelType":"private",
	  "data": {
		    "total": "88", // total balance
		    "available": "88", // available balance
		    "availableChange": "88", // the change of available balance
		    "currency": "BTC", // currency
		    "hold": "0", // hold amount
		    "holdChange": "0", // the change of hold balance
		    "relationEvent": "trade.setted", //relation event
		    "relationEventId": "5c21e80303aa677bd09d7dff", // relation event id
		    "relationContext": {
            "symbol":"BTC-USDT",
            "tradeId":"5e6a5dca9e16882a7d83b7a4", // the trade Id when order is executed
            "orderId":"5ea10479415e2f0009949d54"
        },  // the context of trade event
		"time": "1545743136994" // timestamp
  }
}

```
Topic: `/account/balance`

* Push frequency: `real-time`

You will receive this message when an account balance changes. The message contains the details of the change.


### Relation Event

| Type    | Description |
|---------| ----------- |
main.deposit | Deposit
main.withdraw_hold | Hold withdrawal amount
main.withdraw_done | Withdrawal done
main.transfer | Transfer (Main account)
main.other | Other operations (Main account)
trade.hold | Hold (Trade account)
trade.setted | Settlement (Trade account)
trade.transfer | Transfer (Trade account)
trade.other | Other operations (Trade account)
other | Others
<aside class="spacer2"></aside>

