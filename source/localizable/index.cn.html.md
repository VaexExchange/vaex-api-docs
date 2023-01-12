---
title: title

language_tabs: # must be one of https://git.io/vQNgJ


toc_footers:
  - <a href='https://www.vaex.com'>Sign Up for Vaex</a>

includes:

search: true
---

# 快速開始

## 簡介

歡迎使用Vaex開發者文檔。

本文檔概述了交易功能、市場行情和其他應用開發接口。


API分爲兩部分：**REST API和Websocket 實時數據流**

 -  REST API包含四個類別：**[用戶模塊](#c7e3890dd9)（私有），[交易模塊](#ce6402891a)（私有），[市場數據](#d63a2ca8ac)（公共），[其他接口](#cd67660573)（公共）**
 -  Websocket包含兩類：**公共頻道和私人頻道**

## 更新預告

爲了您能獲取到最新的API 變更的通知，請在 [Vaex Docs Github](https://github.com/VaexExchange/vaex-api-docs)添加關注【Watch】



**29/12/22**:

- 【新增】API文檔


## 閱讀指南

2.  [REST&nbsp;API](#rest-api) 如何創建一個REST&nbsp;API。
3.  [服務器時間](#cfc829dcfc) 在無需驗籤的情況下，可以獲取服務器時間。（可用作連接測試）
4.  [服務狀態](#b92b50d79d) 根據服務狀態來維護交易策略。
5.  [接口認證](#44936cf54e) 如何進行接口認證。
6.  [內部資金劃轉](#d92edf6866) 資金賬戶和交易賬戶之間資產的相互劃轉。
7.  [賬戶列表](#88d7ca20ae) 獲取個人的賬戶資產指南。
8.  [下單](#6a338fcba8) 獲取下單操作指南。
9.  [委託買賣盤](#0063487c69) 獲取買賣盤的快照信息。
10. [Websocket](#websocket-3) 如何創建Websocket 連接
11. [Level-2 市場行情](#level-2-3) 如何使用Websocket 在本地構建一個實時的買賣盤。
12. [餘額變更](#c8ed95427a) 通過Websocket實時獲取賬戶餘額變更信息


## 撮合引擎

### 訂單生命週期

當訂單進入撮合引擎時，訂單狀態爲 **received** 。如果一個訂單與另外一個訂單完全撮合，訂單狀態更新爲 **done** 。一個訂單可以部分成交或全部成交，未被撮合的訂單狀態爲 **open**。當訂單被取消（ **canceled** ）或成交（ **filled** ），訂單狀態更新爲 **done**, 否則訂單會一直保持 **open** 狀態。

### 自成交保護（STP）

您可在高級設置中設置**自成交保護**策略。您的訂單將不會發生自成交。如果您在再下單時沒有指定**STP**，否則您的訂單可能會與被自己的訂單成交。需要注意的是，只有taker方的保護策略生效。

#### 減量和取消(DC)

目前，**市價單不支持減量和取消（DECREMENT AND CANCEL）**。針對同一個用戶，同一個交易對，買賣訂單同時存在，數量少的訂單取消，數量多的訂單減去少的一方的數量，繼續執行撮合。如果數量相同，兩個訂單都會取消。

#### 取消舊的訂單(CO)

**Cancel Old** 全部取消舊的訂單，新的訂單繼續執行撮合。

#### 取消新的訂單(CN)

**Cancel New** 全部取消新的訂單，舊的訂單仍然在買賣盤中。

#### 雙方都取消(CB)

**Cancel Both** 買賣方都取消。



## 客戶端開發庫

客戶端庫可以幫助您快速集成我們的API。

**官方**

- [Java SDK(暫無)](https://github.com/VaexExchange/vaex-java-sdk)

CCXT 是我們官方SDK提供方，您可以使用CCXT來對接Vaex API。
更多信息, 請訪問: [https://ccxt.trade](https://ccxt.trade).



**代碼樣例**

- PHP 文件樣例 (GET & POST & DELETE)  [Github Link](https://github.com/VaexExchange/vaex-api-docs/tree/master/examples/php)

- - -


## 請求頻率限制

當請求頻率超過限制頻率時，系統將返回code爲**429**的超頻錯誤碼。
<aside class="notice">如果接口請求頻率超過限制，你的IP或賬戶會被限制使用10s。</aside>

###REST API

對需要校驗API權限的私有接口，限制賬號userid。不需要檢驗權限API，則限制IP。
<aside class="notice">接口有特定請求頻率限制說明，以特定說明爲準。</aside>

###WEBSOCKET

### 連接數量
每個用戶ID同時建立的連接數： ≤ 50個

### 連接次數
連接請求次數限制：每分鐘 30次 請求

### 上行消息條數
向服務器發送指令條數限制：每10秒 100條

### 訂閱topic數量
單次最多批量訂閱數量限制：100個topic

每個連接最大可訂閱topic數量限制：300個topic

## 常見問題

### 簽名錯誤

* 檢查API-KEY，API-SECRET，API-PASSPHRASE是否正確
* 檢查簽名內容順序 timestamp + method + requestEndpoint + body
* 檢查header中timestamp是否與生成signature一致
* 檢查簽名生成是否爲base64編碼
* get請求是否以表單方式提交
* post請求的數據格式是否是json格式（application/json; charset=utf-8）

### 申請提現
* memo字段<br/>
  個別幣種提現需要memo字段，該字段在其他平臺可能會表示成tag或paymentId<br/>
  對於沒有 memo 的幣種，在使用API提現的時候是不能傳遞memo值，否則，接口會返回 **vaex incorrect withdrawal address**
* amount字段<br/>
  amount需要符合該幣種提現的precision，可以通過[獲取提現額度](#fb40b060ad)獲取<br/>
  提現金額必須爲提現精度的整數倍，如果爲0表示只能爲整數。



### WebSocket 限制
* 一個連接最多訂閱300個topic；
* token有效期24小時；
* 一個用戶最多50個連接；
* 客戶端每10秒最多上行100個消息；
* 一個symbol就是一個topic; e.g.Topic: /market/level2:{symbol},{symbol}...

### 返回 403 問題

如返回以下錯誤消息：
403 "The request could not be satisfied. Bad Request" from Amazon CloudFront<br/>

* 檢查請求是否爲HTTPS
* 移除GET請求中的RequestBody

# REST API

## API服務器地址
REST API 對於賬戶、訂單、和市場數據均提供了接口。

基本URL: **https://api.vaex.com**。

<aside class="notice">爲了遵守當地法律要求，使用中國大陸IP的用戶不允許訪問以上URL。</aside>

請求URL由基本URL和接口端點組成。

## 接口端點(Endpoint)
每個接口都提供了對應的端點（Endpoint），可在HTTP請求模塊下獲取。

對於**GET** 請求, 端點需要要包含請求參數。

例如，對於"[賬戶列表](#88d7ca20ae)"接口，其默認端點爲 **/api/v1/accounts**。
如果您的請求參數currency=BTC，則該端點將變爲 **/api/v1/accounts?currency=BTC**。因此，您最終請求的URL應爲：**https://api.vaex.com/api/v1/accounts?currency=BTC**。


## 請求
所有的POST請求和返回內容類型都是 **application/json**.  

除非另行說明，所有的時間戳參數均以Unix時間戳毫秒計算。如：**1544657947759**

### 請求參數
對於 **GET和DELETE** 請求, 需將參數拼接在請求URL中。(**例如， /api/v1/accounts?currency=BTC**)

對於 **POST和PUT** 請求, 需將參數以JSON格式拼接在請求主體中(**例如， {"currency":"BTC"}**)。
注意：不要在JSON字符串中添加空格。

### 錯誤返回
系統會返回HTTP錯誤代碼或系統錯誤代碼。您可根據返回的參數消息排查錯誤原因。

#### HTTP錯誤碼
```json
{
    "code":"400100",
    "msg":"Invalid Parameter."
}
```

| 代碼  | 意義                                                                 |
| --- | ------------------------------------------------------------------ |
| 400 | Bad Request -- 無效的請求格式                                             |
| 401 | Unauthorized -- 無效的API-KEY                                         |
| 403 | Forbidden 或 Too Many Requests -- 請求被禁止 或 超過[請求頻率限制](#7250787b00)     |
| 404 | Not Found -- 找不到指定資源                                               |
| 405 | Method Not Allowed -- 您請求資源的方法不正確                                  |
| 415 | Content-Type: application/json -- [請求類型](#466c5b7be2)必須爲application/json類型 |
| 500 | Internal Server Error -- 服務器出錯，請稍後再試                               |
| 503 | Service Unavailable -- 服務器維護中，請稍後再試                                |



#### 系統錯誤碼
| 代碼   | 意義                                                         |
| ------ | ------------------------------------------------------------ |
| 200001 | Order creation for this pair suspended--交易對暫停交易       |
| 200002 | Order cancel for this pair suspended--交易對暫停取消訂單     |
| 200003 | Number of orders breached the limit--委託中訂單數量過多      |
| 200009 | Please complete the KYC verification before you trade XX--需要通過KYC高級認證才能交易該幣對 |
| 200004 | Balance insufficient--賬戶餘額不足                           |
| 400001 | Any of VAEX-API-KEY, VAEX-API-SIGN, VAEX-API-TIMESTAMP, VAEX-API-PASSPHRASE is missing in your request header -- 請求頭中缺少驗籤參數 |
| 400002 | VAEX-API-TIMESTAMP Invalid -- 請求時間與服務器時差超過5秒      |
| 400003 | VAEX-API-KEY not exists -- API-KEY 不存在                      |
| 400004 | VAEX-API-PASSPHRASE error -- API-PASSPHRASE 不正確             |
| 400005 | Signature error -- [簽名](#99f215f459)錯誤，請檢查您的簽名     |
| 400006 | The requested ip address is not in the api whitelist -- 請求IP不在API白名單中 |
| 400007 | Access Denied -- API權限不足，無法訪問該URI目標地址。        |
| 404000 | Url Not Found -- 找不到請求資源                              |
| 400100 | Parameter Error -- 請求參數不合法                            |
| 400200 | Forbidden to place an order--禁止在該交易對下單              |
| 400500 | Your located country/region is currently not supported for the trading of this token--該數字資產不支持您所在區域的用戶參與，感謝您的理解 |
| 400700 | Transaction restricted, there's a risk problem in your account--您的賬戶存在風險問題，暫時不允許進行交易 |
| 400800 | Leverage order failed--槓槓下單失敗                          |
| 411100 | User are frozen -- 用戶被凍結，請聯繫[幫助中心](https://vaex.zendesk.com/hc/zh-cn/requests/new) |
| 500000 | Internal Server Error -- 服務器出錯，請稍後再試              |
| 900001 | symbol not exists--交易對不存在                              |

如果系統返回HTTP狀態碼爲200，但業務失敗，系統會報錯。您可根據返回的參數消息排查錯誤。

### 成功返回
當系統返回HTTP狀態碼200和系統代碼200000時，表示響應成功，返回結果如下：

```json
{
    "code":"200000",
    "data":"1544657947759"
}
```

### 分頁

Pagination允許使用當前頁數獲取結果，非常適用於獲取實時數據。如/api/v1/deposits、/api/v1/orders及/api/v1/fills端點均默認返回第一頁結果，共50條數據。如需獲取更多數據，請根據當前返回的數據指定其他分頁，然後再進行請求。


#### 請求參數
| 參數名稱    | 默認值 | 含義                            |
| ----------- | ------ | ------------------------------- |
| currentPage | 1      | 當前頁碼                        |
| pageSize    | 50     | 每頁記錄數，最小值10，最大值500 |

#### 示例
`GET /api/v1/orders?currentPage=1&pageSize=50`


## 類型說明

### 時間戳

所有時間戳參數都應以毫秒爲單位，除非另有說明。例如， **1544657947759**。

撮合引擎和訂單系統的時間戳使用的是納秒爲單位。

### 數字

爲了保證跨平臺的數字的精確度，Decimal轉化爲字符串返回。請在發出請求時，將數字轉換爲字符串來避免數字被截斷或者精度錯誤。

## 接口認證

### 創建API-KEY
通過接口進行請求前，您需在Web端創建API-KEY。創建成功後，您需妥善保管好以下三條信息：


* Key
* Secret
* Passphrase

Key和Secret由Vaex隨機生成並提供，Passphrase是您在創建API時使用的密碼。以上信息若遺失將無法恢復，需要重新申請API KEY。

### API權限

您可在Vaex Web端管理API權限。API權限分爲以下幾類：

* **通用權限** - 允許API訪問大部分的GET請求。
* **交易權限** - 允許API具有下單權限。
* **提現權限** - 允許API劃轉資金，包含充值和提現。
  授權提現權限時請注意，不需要郵箱驗證和谷歌驗證就可以使用API進行轉賬。



請參考下方API文檔，看接口具體需要哪些權限。

### 創建請求

Rest請求頭必須包含以下內容:

* **VAEX-API-KEY** API-KEY以字符串傳遞
* **VAEX-API-SIGN** [簽名](#99f215f459)
* **VAEX-API-TIMESTAMP** 請求的時間戳
* **VAEX-API-PASSPHRASE** 創建API時填的API-KEY的密碼
* **VAEX-API-KEY-VERSION** API-KEY版本號，可通過[API管理](https://www.vaex.cc/account/api)頁面查看版本號

### 簽名

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
        "VAEX-API-KEY-VERSION": 2,
        "Content-Type": "application/json" # specifying content type or using json=data in request
    }
    response = requests.request('post', url, headers=headers, data=data_json)
    print(response.status_code)
    print(response.json())
```
請求頭中的 **VAEX-API-SIGN**:

1. 使用 API-Secret 對
    {timestamp + method + endpoint + body} 拼接的字符串進行**HMAC-sha256**加密。
2. 再將加密內容使用 **base64** 編碼。

請求頭中的 **VAEX-API-PASSPHRASE**:

1. 對於V1版的API-KEY，請使用明文傳遞
2. 對於V2版的API-KEY，需要將VAEX-API-KEY-VERSION指定爲2，並將passphrase使用API-Secret進行**HMAC-sha256**加密，再將加密內容通過**base64**編碼後傳遞

注意：

* 加密的 timestamp 需要和請求頭中的VAEX-API-TIMESTAMP保持一致
* 用於加密的body需要和請求中的Request Body的內容保持一致
* 請求方法需要大寫
* 對於 GET, DELETE 請求，endpoint 需要包含請求的參數（/api/v1/deposit-addresses?currency=BTC）。如果沒有請求體（通常用於GET請求），則請求體使用空字符串””。



```python
#Example for Create Deposit Address

curl -H "Content-Type:application/json" -H "VAEX-API-KEY:5c2db93503aa674c74a31734" -H "VAEX-API-TIMESTAMP:1547015186532" -H "VAEX-API-PASSPHRASE:QWIxMjM0NTY3OCkoKiZeJSQjQA==" -H "VAEX-API-SIGN:7QP/oM0ykidMdrfNEUmng8eZjg/ZvPafjIqmxiVfYu4="  -H "VAEX-API-KEY-VERSION:2"
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

### 選擇時間戳

請求頭中的 **VAEX-API-TIMESTAMP** 必須爲[Unix 時間](http://en.wikipedia.org/wiki/Unix_time)，精確到毫秒，例如，1547015186532。

服務器請求的時間戳與API服務器時差必須控制在5秒以內，否則請求會因過期而被服務器拒絕。如果服務器與API服務器之間存在時間偏差，請使用平臺提供的服務器時間接口，獲取API[服務器的時間](#cfc829dcfc)。

# 用戶模塊

此部分需進行[簽名驗證](#99f215f459)。


##
# 賬戶

## 賬戶列表
```json
[
    {
        "id":"5bd6e9286d99522a52e458de", //accountId
        "currency":"BTC",  //幣種
        "type":"main", //賬戶類型，資金（main）賬戶、交易(trade)賬戶
        "balance":"237582.04299",//資金總額
        "available":"237582.032", //可用金額
        "holds":"0.01099" //凍結金額
    },
    {
        "id":"5bd6e9216d99522a52e458d6",
        "currency":"BTC",
        "type":"trade",
        "balance":"1234356",
        "available":"1234356",
        "holds":"0"
    }
]
```
獲取賬號下賬戶詳情列表。

交易前請先[充值](#fa03c46253)到資金賬戶，再使用[內部資金劃轉](#d92edf6866)將資金從資金賬戶劃轉到交易賬戶。


### HTTP請求
`GET /api/v1/accounts`

### 請求示例
`GET /api/v1/accounts`

### API權限
此接口需要**通用權限**。


### 請求參數

請求參數 | 類型 | 含義
--------- | -------| -------
currency | String | [可選] [幣種](#47f0f7e8df)
type | String |[可選] 賬戶類型 **main**、**trade**

### 返回值
字段 | 含義
--------- | -------
id | accountId 賬戶ID
currency | 賬戶對應的幣種
type |賬戶類型 ，**main**（資金賬戶）、**trade**（交易賬戶）
balance | 賬戶資金總額
available | 賬戶可用的資金
holds | 賬戶凍結的資金

### 賬戶類型
賬戶劃分爲: 資金賬戶、交易賬戶。

賬戶之間的資金劃轉不收取手續費。

資金賬戶主要用於資金的提現和充值，資金賬戶裏面的資金不可以直接用於交易。交易之前需要將資金從資金賬戶轉到交易賬戶。

交易賬戶主要用於交易。下單扣除的是交易賬戶裏面的資金，交易賬戶裏面的資金不可以直接提現，必須要轉到資金賬戶纔可以提現。


賬戶之間資金劃轉請參考[內部資金劃轉](#d92edf6866)。

### 凍結資金
當下單時，您用於下單的資金會被凍結。凍結的資金不可以用作再次下單或者提現。當訂單取消或成交後，資金才能解凍回退或解凍支付。



## 賬戶流水記錄
此接口返回所有賬戶的出入賬流水記錄，支持多幣種查詢。
返回值是[分頁](#95d51b1f3b)後的數據，根據時間降序排序。

```json
{
  "currentPage": 1,
  "pageSize": 50,
  "totalNum": 2,
  "totalPage": 1,
  "items": [
    {
      "id": "611a1e7c6a053300067a88d9",//唯一鍵
      "currency": "USDT", //幣種
      "amount": "10.00059547", //資金變動值
      "fee": "0", //充值或提現費率
      "balance": "0", //金額變動
      "accountType": "MAIN", //賬戶類型
      "bizType": "Loans Repaid", //業務類型
      "direction": "in", //出入賬方向入賬或出賬（in or out）
      "createdAt": 1629101692950, //創建時間
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
      "context": "{\"borrowerUserId\":\"5e3f4623dbf52d000800292f\",\"loanRepayDetailNo\":\"611a18bc7255c200063ea545\"}"// 業務核心參數
    }
  ]
}
```

### HTTP請求
`GET /api/v1/accounts/ledgers`


### 請求示例
`GET /api/v1/accounts/ledgers?currency=BTC&startAt=1601395200000`

### API權限
此接口需要**通用權限**。

### 頻率限制
此接口針對每個賬號請求頻率限制爲**18次/3s**

<aside class="notice">這個接口需要使用分頁</aside>

### 請求參數
請求參數 | 類型 | 含義
--------- | ------- | -------
currency | String |  [可選] 幣種，選填，可多選，以逗號分隔，最多支持選擇10個幣種，若不填寫，默認查詢所有幣種
direction | String | [可選] 出入賬方向: **in** -入賬, **out** -出賬
bizType   | String | [可選] 業務類型: **DEPOSIT** -充值, **WITHDRAW** -提現, **TRANSFER** -轉賬, **TRADE_EXCHANGE** -交易
startAt   | long   | [可選] 開始時間（毫秒）
endAt     | long   | [可選] 截止時間（毫秒）

<aside class="notice">您只能獲取 24 小時時間範圍內的數據（即：查詢時，開始時間到結束時間的時間範圍不能超過24小時）。若超出時間範圍，系統會報錯。如果您只指定了結束時間，沒有指定開始時間，系統將按照24小時的範圍自動計算開始時間（開始時間=結束時間-24小時）並返回相應數據，反之亦然。</aside>
<aside class="notice">最多獲取1年的歷史數據，如需要獲取更久遠的歷史數據，請提交工單查詢：https://vaex.zendesk.com/hc/en-us/requests/new</aside>

### 返回值
字段 | 含義
--------- | -------
id | 唯一鍵
currency | 幣種
amount | 資金變動值
fee | 充值或提現費率
balance | 變動後的資金總額
accountType | 母賬號賬戶類型MAIN、TRADE
bizType | 業務類型，比如交易，提現等
direction | 出入賬方向 **out** 或 **in**
createdAt | 創建時間
context | 業務核心參數

### context

如果 **bizType** 是trade exchange，那麼 **context** 字段會包含交易的額外信息（訂單id，交易id，交易對）。

### BizType 含義
值 | 含義
--------- | -------
Deposit  | 獲取充值入賬記錄
Withdrawal  | 獲取提現記錄
Transfer | 獲取資金劃轉記錄
Trade_Exchange | 獲取幣幣交易記錄



## 獲取可劃轉資金
```json
{
    "currency":"BTC",
    "balance":"0",
    "available":"0",
    "holds":"0",
    "transferable":"0"
}
```
此接口可獲取指定賬戶和幣種下的可劃轉的資金。

### HTTP請求
`GET /api/v1/accounts/transferable`

### 請求示例
`GET /api/v1/accounts/transferable?currency=BTC&type=MAIN`

### API權限
此接口需要`通用權限`。

### 請求參數

請求參數 | 類型 | 是否必須 |含義
--------- | ------- |  ------- | -------
currency | String | 是 | [幣種](#47f0f7e8df)
type | String | 是 | 賬戶類型：`MAIN`、`TRADE`

### 返回值
字段 | 含義
--------- | -------
currency | 幣種
balance | 資金總額
available | 可用資金
holds | 凍結資金
transferable | 可劃轉資金



## 內部資金劃轉
```json
{
    "orderId":"5bd6e9286d99522a52e458de"
}
```
此接口用於平臺內部賬戶資金劃轉，用戶可以將資金在資金賬戶、交易賬戶之間免費劃轉。

### HTTP請求
`POST /api/v2/accounts/inner-transfer`

### API權限
此接口需要`交易權限`。

### 請求參數
請求參數 | 類型 | 是否必須 |含義
--------- | ------- |  ------- | -------
clientOid | String | 是 | Client Order Id，客戶端創建的唯一標識，建議使用UUID
currency | String | 是 | [幣種](#47f0f7e8df)
from | String |  是 | 付款賬戶類型: `main`、`trade`
to | String |  是 | 收款賬戶類型: `main`、`trade`
amount | String | 是 | 轉賬金額，精度爲[幣種精度](#47f0f7e8df)正整數倍
fromTag | String | 否 | 交易對
toTag | String | 否 | 交易對

### 返回值
字段 | 含義
--------- | -------
orderId | 內部資金劃轉的訂單ID

# 充值

## 申請充值地址
```json
{
    "address":"0x78d3ad1c0aa1bf068e19c94a2d7b16c9c0fcd8b1",
    "memo":"5c247c8a03aa677cea2a251d",
    "chain":"OMNI"
}
```

此接口可用於申請充值地址

### HTTP請求
`POST /api/v1/deposit-addresses`

### 請求示例
`POST /api/v1/deposit-addresses`

### API權限
此接口需要**提現權限**。

### 請求參數

請求參數 | 類型 | 含義
--------- | ------- |  -------
currency | String | [幣種](#47f0f7e8df)
chain | String | [可選] 幣種的鏈名。這個參數用於區分多鏈的幣種，單鏈幣種不需要。 

### 返回值
字段 | 含義
--------- | -------
address | 充值地址
memo | 地址標籤memo(tag)，如果返回爲空，則該幣種沒有memo。對於沒有memo的幣種，在[提現](#5a3e3653c8-2)的時候不可以傳遞memo
chain | 幣種的鏈名。

## 獲取充值地址
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

此接口可以獲取幣種所有支持鏈的充值地址，如果返回數據爲空，請先[申請充值地址](#c424eb7b8c)。

### HTTP請求
`GET /api/v2/deposit-addresses`

### 請求示例
`GET /api/v2/deposit-addresses?currency=BTC`

### API權限
此接口需要**通用權限**。

### 請求參數
請求參數 | 類型 | 含義
--------- | -------  | -------
currency | String |[幣種](#47f0f7e8df)

### 返回值
字段 | 含義
--------- | -------
address | 充值地址
memo | 地址標籤memo(tag)，如果返回爲空，則該幣種沒有memo。對於沒有memo的幣種，在[提現](#081737423d)的時候不可以傳遞memo
chain | 幣種的鏈名
contractAddress | 合約地址



## 獲取充值列表
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
                "currency": "XRP",
                "chain": "xrp",
                "status": "SUCCESS",
                "address": "rNFugeoj3ZN8Wv6xhuLegUBBPXKCyWLRkB",
                "memo": "1919537769",
                "isInner": false,
                "amount": "20.50000000",
                "fee": "0.00000000",
                "walletTxId": "2C24A6D5B3E7D5B6AA6534025B9B107AC910309A98825BF5581E25BEC94AD83B@e8902757998fc352e6c9d8890d18a71c",
                "createdAt": 1666600519000,
                "updatedAt": 1666600549000,
                "remark": "Deposit"
            }
        ]
    }
}
```
此端點，可獲取充值分頁列表。
返回值是[分頁](#95d51b1f3b)後的數據，根據時間降序排序。

### HTTP請求
`GET /api/v1/deposits`

### 請求示例
`GET /api/v1/deposits`

### API權限
此接口需要**通用權限**。

### 頻率限制
此接口針對每個賬號請求頻率限制爲**6次/3s**

<aside class="notice">這個接口需要使用分頁</aside>

### 請求參數
請求參數 | 類型 | 是否必須 | 含義 |
--------- | ------- | -----------| -----------|
currency | String | 否 | [幣種](#47f0f7e8df)
startAt| long | 否 | 開始時間（毫秒）
endAt| long | 否 | 截止時間（毫秒）
status | String | 否 | 狀態。可選值: `PROCESSING`, `SUCCESS`, `FAILURE`

### 返回值
字段 | 含義
--------- | -------
address | 充值地址
memo | 地址標籤memo(tag)，如果返回爲空，則該幣種沒有memo。對於沒有memo的幣種，在[提現](#081737423d)的時候不可以傳遞memo
amount | 充值金額
fee | 充值手續費
currency | 幣種
isInner | 是否爲平臺內部充值
walletTxId | 錢包交易Id
status | 狀態
remark | 備註
createdAt | 創建時間
updatedAt | 修改時間



# 提現

## 獲取提現列表
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

### HTTP請求
`GET /api/v1/withdrawals`

### 請求示例
`GET /api/v1/withdrawals`

### API權限
此接口需要**通用權限**。

### 頻率限制
此接口針對每個賬號請求頻率限制爲**6次/3s**

<aside class="notice">這個接口需要使用分頁</aside>

### 請求參數
請求參數 | 類型 | 是否必須 | 含義 |
--------- | ------- | -----------| -----------|
currency | String | 否 | [幣種](#47f0f7e8df)
status | String | 否 | 狀態。可選值: `PROCESSING`, `WALLET_PROCESSING`, `SUCCESS`, `FAILURE`
startAt| long | 否 | 開始時間（毫秒）
endAt| long | 否 | 截止時間（毫秒）

### 返回值
字段 |  含義
--------- | -------
id | 唯一標識
address | 提現地址
memo | 提現地址標識
currency | 幣種
chain | 幣種chain
amount | 提現金額
fee | 提現手續費
walletTxId | 錢包交易Id
isInner | 是否爲平臺內部提現
status | 狀態
remark | 備註
createdAt | 創建時間
updatedAt | 修改時間



##  獲取提現額度
```json
{
    "currency":"BTC",
    "limitBTCAmount":"2.0",
    "usedBTCAmount":"0",
    "remainAmount":"75.67567568",
    "availableAmount":"9697.41991348",
    "withdrawMinFee":"0.93000000",
    "innerWithdrawMinFee":"0.00000000",
    "withdrawMinSize":"1.4",
    "isWithdrawEnabled":true,
    "precision":8,
    "chain":"OMNI"
}
```

### HTTP請求
`GET /api/v1/withdrawals/quotas`

### 請求示例
`GET /api/v1/withdrawals/quotas?currency=BTC`

### API權限
此接口需要**通用權限**。

### 請求參數
請求參數 | 類型 | 含義
--------- | ------- | ---------
currency | String | [幣種](#47f0f7e8df)
chain | String | [可選] 幣種chain。這個參數用於區分多鏈的幣種，單鏈幣種不需要；你可通過`GET /api/v2/currencies/{currency}`接口查詢幣種的`chain`值.

### 返回值
字段 | 含義
--------- | -------
currency | 幣種
availableAmount | 可提現的金額
remainAmount | 當日剩餘可提現的額度
withdrawMinSize | 最小提現金額
limitBTCAmount | 當日剩餘可提現的額度，摺合爲BTC
innerWithdrawMinFee | 內部提現手續費
usedBTCAmount | 當日BTC摺合提現
isWithdrawEnabled | 是否可提現
withdrawMinFee | 最小提現手續費
precision | 提現的精度
chain | 幣種的鏈名。

## 申請提現
```json
{
    "withdrawalId":"5bffb63303aa675e8bbe18f9"
}
```

### HTTP請求
`POST /api/v1/withdrawals`

<aside class="notice">在WEB端可以開啓指定常用地址提現，開啓後會校驗你的提現地址(地址區分字母大小寫,包括chain的設置)是否爲常用地址；若校驗不通過會提示<code>{"msg":"Already set withdraw whitelist, this address is not favorite address","code":"260325”}</code>錯誤信息。</aside>

### API權限
此接口需要`提現權限`。

### 請求參數
請求參數 | 類型 | 是否必須 |含義
--------- | ------- |  ------- | -------
currency  | String | 是 | 幣種
address   | String | 是 | 提現地址
amount | number | 是 | 提現總額，必須爲提現精度的正整數倍
memo | String | 否 | 地址標籤memo(tag)，如果返回爲空，則該幣種沒有memo。對於沒有memo的幣種，在[提現](#081737423d)的時候不可以傳遞memo
isInner | boolean | 否 | [可選] 是否爲平臺內部提現。默認爲`false`
remark | String | 否 | [可選] 備註信息
chain | String | 否 | [可選] 針對一幣多鏈的幣種，推薦指定chain參數；不指定則使用默認鏈；你可通過`GET /api/v2/currencies/{currency}`接口查詢幣種的`chain`值。
feeDeductType | String | 否 | 提現手續費扣除方式: `INTERNAL` 或 `EXTERNAL` 或不指定 <br/><br/>1. `INTERNAL`- 從提現金額中扣除手續費</br>2. `EXTERNAL`- 從資金賬戶中扣除手續費</br>3. 不指定`feeDeductType`參數時, 當您的資金賬戶的餘額足以支持支付提現手續費時，首先從您的資金賬戶中扣除手續費，反之，從您的提現金額中扣除手續費。比如，您從Vaex提現 1 個BTC(提現手續費爲：0.0001BTC)，如果您資金賬戶裏的餘額不支持支付手續費，系統將會自動從您的提現金額中扣除手續費，您實際到賬金額爲0.9999個BTC。

### 返回值
字段 | 含義
--------- | -------
withdrawalId | 提現Id 唯一標識




## 取消提現
提現狀態爲提現中纔可以取消。

### HTTP請求
`DELETE /api/v1/withdrawals/{withdrawalId}`

### 請求示例
`DELETE /api/v1/withdrawals/5bffb63303aa675e8bbe18f9`

### API權限
此接口需要**提現權限**。

### 請求參數
請求參數 | 類型 | 含義
--------- | ------- | -------
withdrawalId | String | 路徑參數，[提現Id](#5a3e3653c8-2) 唯一標識





# 手續費

## 用戶基礎手續費

此接口返回用戶的基礎費率。

```json
{
    "code": "200000",
    "data": {
        "takerFeeRate": "0.001",
        "makerFeeRate": "0.001"
    }
}
```

### HTTP請求
`GET /api/v1/base-fee`

### 請求示例
`GET /api/v1/base-fee`
<br/>
`GET /api/v1/base-fee?currencyType=1`

### API權限
此接口需要`通用權限`。

### 請求參數  
請求參數|數據類型|是否必需|含義
---|---|---|---
currencyType|String|否|幣種類型: `0`-數字貨幣, `1`-法幣. 默認爲`0`-數字貨幣

### 返回值
字段 |  含義
--------- | -------
takerFeeRate | 用戶吃單基礎手續費率
makerFeeRate | 用戶掛單基礎手續費率

## 交易對實際費率
此接口返回用戶交易時實際費率，一次限制最多查10個交易對。

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

### HTTP請求
`GET /api/v1/trade-fees`

### 請求示例
`GET /api/v1/trade-fees?symbols=BTC-USDT,ETH-USDT`

### API權限
此接口需要`通用權限`。

### 請求參數  
請求參數|數據類型|是否必需|含義
---|---|---|---
symbols|String|是| 交易對，可多填，逗號分割，一次限制最多查`10`個交易對

### 返回值
字段 |  含義
--------- | -------
symbol | 交易對唯一標識碼，重命名後不會改變
takerFeeRate | 交易對吃單實際手續費率
makerFeeRate | 交易對掛單實際手續費率


# 交易模塊

以下請求需要校驗[簽名](#99f215f459)。


# 訂單

## 下單

```json
{
    "orderId":"5bd6e9286d99522a52e458de"
}
```

訂單有兩種類型：
限價單（**limit**）: 指定價格和數量進行交易。
市價單(**market**) : 指定資金或數量進行交易。

在下單前，請確保您的[交易賬戶](#88d7ca20ae)有足夠的資金。一旦下單成功，您下單的金額會被凍結。[凍結金額](#HOLDS)的多少取決於您下單的類型和具體的請求參數。
<aside class="notice">下單將啓用價格保護機制。當限價單的價格在閾值範圍之外時，會觸發價格保護機制，導致下單失敗。</aside>


請悉知，當您的訂單進入買賣盤，系統會提前凍結[訂單的手續費](#052bf0ef58)。

在下單之前，請充分了解每一個[交易對](#d6402cad41)的參數含義。

**請求體中的JSON字符串中不要有多餘的空格**

### 下單限制
對於一個賬號，每一個交易對最大活躍委託訂單數量 **200** （包含止損單）。

### HTTP 請求
`POST /api/v1/orders`

### 請求示例
`POST /api/v1/orders`

### API權限
此接口需要**交易權限**。

### 頻率限制
此接口針對每個賬號請求頻率限制爲**45次/3s**

### 請求參數

下單公有的請求參數

| 請求參數      | 類型     | 含義                                                                                    |
| --------- | ------ | ------------------------------------------------------------------------------------- |
| clientOid | String | Client Order Id，客戶端創建的唯一標識，建議使用UUID                                                   |
| side      | String | **buy**（買） 或 **sell**（賣）                                                              |
| symbol    | String | [交易對](#d6402cad41) 比如，ETH-BTC                                                         |
| type      | String | [可選] 訂單類型 **limit** 和  **market** (默認爲 **limit**)                                     |
| remark    | String | [可選] 下單備註，長度不超過100個字符（UTF-8）                                                          |
| stp       | String | [可選] [自成交保護](#stp)（self trade prevention）分爲**CN**, **CO**, **CB** , **DC**四種策略 |
| tradeType       | String | [可選] 交易類型，爲**TRADE**（現貨交易）（默認爲**TRADE** ）。 |
#### **limit** 限價單額外所需請求參數

| 請求參數        | 類型      | 含義                                                          |
| ----------- | ------- | ----------------------------------------------------------- |
| price       | String  | 指定幣種的價格                                                     |
| size        | String  | 指定幣種的數量                                                     |
| timeInForce | String  | [可選] 訂單時效策略 **GTC**, **GTT**, **IOC**, **FOK** (默認爲**GTC**) |
| cancelAfter | long    | [可選] **n** 秒之後取消，訂單時效策略爲 **GTT**                            |
| postOnly    | boolean | [可選] 被動委託的標識, 當訂單時效策略爲 **IOC** 或 **FOK** 時無效                |
| hidden      | boolean | [可選] 是否隱藏（買賣盤中不展示）                                          |
| iceberg     | boolean | [可選] 冰山單中是否僅顯示訂單的可見部分                                       |
| visibleSize | String  | [可選] 冰山單最大的展示數量                                             |

#### **market** 市價單額外所需請求參數

請求參數 | 類型 | 含義
--------- | ------- | ------- | ---------
size | String | 否（size和funds 二選一） | 下單數量
funds | String |  否（size和funds 二選一）| 下單資金

* 下市價單，需定買賣數量或資金。

###術語解釋

###交易對(Symbol)

交易對必須是Vaex支持的[交易對](#d6402cad41)。

###Client Order Id(clientOid)

ClientOid字段是客戶端創建的唯一ID（推薦使用UUID），只能包含數字、字母、下劃線（_） 和 分隔線（-）。這個字段會在獲取訂單信息時返回。您可使用clientOid來標識您的訂單。ClientOid不同於服務端創建的訂單ID。請不要使用同一個clientOid發起請求。clientOid最長不得超過40個字符。

請妥善記錄服務端創建的orderId，以用於查詢訂單狀態的更新。

###訂單類型(type)

您在下單時指定的訂單類型，決定了您是否需要請求其他參數，同時還會影響到撮合引擎的執行。如果您在下單時未指定訂單類型，系統將默認按照限價單執行。

下限價單時，您需指定限價單的價格（**price**）和數量（**size**）。系統將根據市場行情以指定或更優價格撮合該訂單。如果訂單未能被立即撮合，將繼續留買賣盤中，直至被撮合或被用戶取消。

與限價單不同，市價單價格會隨着市場價格波動而變化。下市價單時，您無需指定價格，只需指定數量。市價單會立即成交，不會進入買賣盤。所有市價單都是taker，需支付taker費用。

###交易類型(tradeType)
目前平臺支持現貨（**TRADE**）交易下單。系統根據您的參數類型，將對指定賬戶資金進行凍結。若未傳遞該參數，將默認按照現貨凍結您交易賬戶資金。

###價格(Price)
下限價單時，price 必須以交易對的[價格增量 priceIncrement](#d6402cad41)爲基準，價格增量是交易對的價格的精度。比如，對BTC-USDT這個交易對, 它的 priceIncrement 爲0.00001000。那麼你下單的 price 不可以小於0.00001000，且爲 priceIncrement 的正整數倍，否則下單時會報錯，invalid priceIncrement。

###數量(Size)
下限價單時，size 是指交易對的交易對象(即寫在靠前部分的資產名)的數量。size 必須以交易對中的[數量增量 baseIncrement](#d6402cad41)爲基準，數量增量是交易對的數量的精度。下單的 size 爲 baseIncrement 的正整數倍並且必須在 baseMinSize 和 baseMaxSize 之間。

###資金(Funds)
下市價單時，funds 字段是指交易對的定價資產(即寫在靠後部分資產名)的資金。funds 必須以交易對中的[資金增量quoteIncrement](#d6402cad41)爲基準，資金增量是交易對的資金的精度。下單的 funds 爲 quoteIncrement 的正整數倍且必須在 quoteMinSize 和 quoteMaxSize 之間。

###訂單時效策略(TimeInForce)
訂單時效是一種交易時使用的特殊策略，用於設定訂單在被撮合或取消前的生效時間。訂單時效策略分爲四種：

| 縮寫  | 全稱                  | 含義                         |
| --- | ------------------- | -------------------------- |
| GTC | Good Till Canceled  | 主動取消才過期                    |
| GTT | Good Till Time      | 指定時間後過期                    |
| IOC | Immediate Or Cancel | 立即成交可成交的部分，然後取消剩餘部分，不進入買賣盤 |
| FOK | Fill Or Kill        | 如果下單不能全部成交，則取消             |

* 注意，成交也包含自成交。市價單並不支持訂單時效策略(TimeInForce)

###被動委託(PostOnly)

postOnlys只是一個標識，如果下單有能立即成交的對手方，則取消。
* 當用戶所下訂單是postonly訂單時，如果訂單進入撮合引擎後遇到冰山單和隱藏單可以立即成交，postonly 訂單收maker 手續費，冰山單和隱藏單收taker手續費。

###隱藏單和冰山單(Hidden & Iceberg)

您可在高級設置中設置隱藏單和冰山單（冰山單是一種特殊形式的隱藏單）。進行限價單和限價止損單交易時，您可選擇按照隱藏單或冰山單執行。

隱藏單不會展示在買賣盤上。

與隱藏單不同，冰山單分爲可見和隱藏兩部分。進行冰山單交易時，需設置可見訂單數量。冰山單最小可見數量是總訂單量的1/20。

進行撮合時，冰山單的可見部分會首先被撮合，當可見部分被全部撮合後，另一部分隱藏訂單將浮出，直至訂單全部成交。

**注意**：

- 系統將對隱藏和冰山單收取taker費用。
- 如果您同時設定了冰山單和隱藏單，您的訂單將默認作爲冰山單處理。

###凍結策略(Hold)
對於限價買單，我們會從您的資金裏面凍結您買單的金額(price * size)。同樣，對於限價賣單，我們也會凍結您賣單的資產。在成交那一刻評估實際的手續費。如果您取消了一個部分成交或未成交的訂單，那麼剩餘金額會解凍會退到您的賬戶。
對於市價買/賣單，需要指定**funds(資金)**，我們會根據funds來凍結您賬戶裏的資金。如果只指定了**size**，在成交或取消之前，您的賬戶所有資金會被凍結（通常凍結時間非常短）。

###自成交保護(SelfTradePrevention)

您可在高級設置中設置自成交保護策略。您的訂單將不會發生自成交。如果您在下單時沒有指定STP，否則您的訂單可能會被自己的訂單成交。市價單現不支持DC策略。

**市價單現不支持 DC**，當*timeInForce* 爲**FOK**， 那麼stp會指定爲**CN**。

| 縮寫  | 全稱                  | 含義                           |
| --- | ------------------- | ---------------------------- |
| DC  | Decrease and Cancel | 取消數量少的一方的訂單，並將數量多的一方數量改爲新舊差值 |
| CO  | Cancel old          | 取消舊的訂單                       |
| CN  | Cancel new          | 取消新的訂單                       |
| CB  | Cancel both         | 雙方都取消                        |

### 訂單生命週期(ORDER LIFECYCLE)

當下單請求因請求成功（撮合引擎已收到訂單）或（因餘額不足、參數不合法等原因）被拒絕時，HTTP 請求會進行響應。下單成功，返回訂單ID，訂單將被撮合，可能會部分或全部成交。部分成交後，訂單剩餘爲未成交部分會變成等待撮合（**Active**）狀態（不包括使用立即成交或取消[IOC]的訂單）。已完全成交的訂單會變成“已完成”（**Done**）狀態。

訂閱市場數據頻道的用戶可使用訂單ID（**orderId**）和用戶訂單ID（**clientOid**）來識別消息。

### 價格保護機制

下單將啓用價格保護機制。具體規則如下

- 若用戶在幣幣交易所下的市價單/限價單可以與當前買賣盤內訂單直接成交，那麼系統會判斷成交深度對應的價格與同方向盤口價的偏差是否超出閾值（閾值可根據API symbol接口獲取）；

- 若超過，當您是限價單時，該訂單會提示下單失敗；

- 若是市價單則此訂單將被系統部分執行，執行上限爲閾值對應的價格內的訂單數量，其他剩餘訂單將不被成交。

舉例說明：若閾值爲10%，當某用戶在ETH/USDT交易區下了10,000 USDT的市價買單時(此時賣一價爲1.20000)，系統會判斷訂單成交後最新成交價爲1.40000。(1.40000-1.20000)/1.20000=16.7%>10%，而閾值價格爲1.32000，此時，用戶的這筆市價買單將最多被成交至1.32000，其他剩餘訂單則不會和買賣盤內訂單進行撮合。
請注意：該功能對深度的探測可能存在偏差，若您的訂單未被完全成交有可能是因爲超出了閾值的部分未成交。


### 返回值
| 字段                                | 含義   |
| --------------------------------- | ---- |
| orderId                           | 訂單Id |
| 下單成功後，會返回一個orderId字段，意味這訂單進入撮合引擎。 |      |



## 批量下單
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

該接口支持在一個接口中批量下單，每次可同時下5個訂單，訂單類型必須爲相同交易對的限價單

### HTTP 請求
`POST /api/v1/orders/multi`

### 請求示例
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
`POST /api/v1/orders/multi`

### API權限
此接口需要`交易權限`。

### 頻率限制
此接口針對每個賬號請求頻率限制爲`3次/3s`

### 請求參數

| 請求參數     | 類型     | 含義                                                                                    |
| ---------   | ------  | -------------------------------------------------------------------------------------- |
| clientOid   | String  | Client Order Id，客戶端創建的唯一標識，建議使用UUID                                          |
| side        | String  | **buy**（買） 或 **sell**（賣）                                                           |
| symbol      | String  | [交易對](#d6402cad41) 比如，ETH-BTC                                                       |
| type        | String  | [可選] 訂單類型 只能是**limit**(默認爲**limit**)                                |
| remark      | String  | [可選] 下單備註，長度不超過100個字符（UTF-8）                                                 |
| stop        | String  | [可選] 止盈止損單，觸發條件， **loss**（小於等於） 或 **entry**（大於等於）。設置後，就必須設置stopPrice參數。               |
| stopPrice   | String  | [可選] 觸發價格，只要設置stop參數，就必須設置此屬性。                                                        |
| stp         | String  | [可選] [自成交保護](#selftradeprevention)（self trade prevention）分爲**CN**, **CO**, **CB** , **DC**四種策略 |
| tradeType   | String  | [可選] 交易類型，默認爲**TRADE**                                                             |
| price       | String  | 指定貨幣的價格                                                     |
| size        | String  | 指定貨幣的數量                                                     |
| timeInForce | String  | [可選] 訂單時效策略 **GTC**, **GTT**, **IOC**, **FOK** (默認爲**GTC**) |
| cancelAfter | long    | [可選] **n** 秒之後取消，訂單時效策略爲 **GTT**                            |
| postOnly    | boolean | [可選] 被動委託的標識, 當訂單時效策略爲 **IOC** 或 **FOK** 時無效                |
| hidden      | boolean | [可選] 是否隱藏（買賣盤中不展示）                                          |
| iceberg     | boolean | [可選] 冰山單中是否僅顯示訂單的可見部分                                       |
| visibleSize | String  | [可選] 冰山單最大的展示數量                                             |

### 返回值
| 字段    | 含義                        |
| -------| --------------------------- |
|status  |  訂單創建結果 (success, fail) |
|failMsg |  失敗原因                    |


## 單個撤單
```json
{
    "cancelledOrderIds":[
        "5bd6e9286d99522a52e458de"
    ]
}
```

此端點可以取消單筆訂單。

<aside class="notice">此接口只提交取消請求。實際取消結果需要通過查詢訂單狀態或訂閱websocket獲取訂單消息。建議您在收到Open消息後再進行撤單，否則會導致訂單取消不成功。
</aside>

### HTTP請求
`DELETE /api/v1/orders/{orderId}`

### 請求示例
`DELETE /api/v1/orders/5bd6e9286d99522a52e458de`

### API權限
此接口需要`交易權限`。

### 頻率限制
此接口針對每個賬號請求頻率限制爲`60次/3s`

### 請求參數
| 請求參數    | 類型     | 含義                            |
| ------- | ------ | ----------------------------- |
| orderId | String | 路徑參數，[訂單Id](#6a338fcba8) 唯一標識 |

### 返回值
| 字段                | 含義      |
| ----------------- | ------- |
| cancelledOrderIds | 取消的訂單id |


<aside class="notice">The <b>orderId</b> 是服務器生成的訂單唯一標識，不是客戶端生成的clientOId</aside>

### 撤單被拒
如果訂單不能撤銷（已經成交或已經取消），會返回錯誤信息，可根據返回的msg獲取原因。


## 基於clientOid 單個撤單
```json
{
  "cancelledOrderId": "5f311183c9b6d539dc614db3",
  "clientOid": "6d539dc614db3"
}
```
此接口發送一個通過clientOid撤銷訂單的請求。

### HTTP請求
`DELETE /api/v1/order/client-order/{clientOid}`

### 請求示例
`DELETE /api/v1/order/client-order/6d539dc614db3`

### API權限
此接口需要**交易權限**。

### 請求參數
| 請求參數    | 類型     | 含義                            |
| ------- | ------ | ----------------------------- |
| clientOid | String | 路徑參數，客戶端生成的標識 |

### 返回值
| 字段                | 含義      |
| ----------------- | ------- |
| cancelledOrderId | 取消的訂單id |
| clientOid | 客戶端生成的標識 |


## 全部撤單
```json
{
    "cancelledOrderIds":[

        "5c52e11203aa677f33e493fb",
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
此接口，可以取消所有狀態爲**open**的訂單，返回值是是已取消訂單的ID列表。

### HTTP請求
`DELETE /api/v1/orders`

### 請求示例
`DELETE /api/v1/orders?symbol=ETH-BTC&tradeType=TRADE`<br/>

### API權限
此接口需要`交易權限`。

### 頻率限制
此接口針對每個賬號請求頻率限制爲`3次/3s`

### 請求參數
| 請求參數   | 類型     | 含義                                 |
| ------ | ------ | ---------------------------------- |
| symbol | String | [可選] 取消指定[交易對](#d6402cad41)的open訂單 |
| tradeType| String | [可選] 取消指定交易類型的open訂單:`TRADE`（現貨交易）, 默認爲取消`TRADE`(現貨交易)訂單                                 |

### 返回值
| 字段                | 含義      |
| ----------------- | ------- |
| cancelledOrderIds | 取消的訂單id |

## 獲取訂單列表
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
此接口，可獲取訂單列表
返回值是[分頁](#95d51b1f3b)後的數據，根據時間降序排序。

### HTTP請求
`GET /api/v1/orders`

### 請求示例
`GET /api/v1/orders?status=active`<br/>

### API權限
此接口需要`通用權限`。

### 頻率限制
此接口針對每個賬號請求頻率限制爲`30次/3s`

<aside class="notice">這個接口需要使用分頁</aside>

### 請求參數
| 請求參數    | 類型     | 含義                                                                                            |
| ------- | ------ | --------------------------------------------------------------------------------------------- |
| status  | String | [可選] `active`（活躍） 或 `done`（完成）,默認爲`done`。只返回指定狀態的訂單信息                           |
| symbol  | String | [可選] 只返回指定[交易對](#d6402cad41)的訂單信息                                                             |
| side    | String | [可選] `buy`（買）或 `sell`（賣）                                                                 |
| type    | String | [可選] 訂單類型: `limit`（限價單）, `market`(市價單), `limit_stop`(限價止盈止損單), `market_stop`（市價止盈止損單） |
| tradeType    | String | [可選] 交易類型: `TRADE`（現貨交易）|
| startAt | long   | [可選] 開始時間（毫秒）                                                                                 |
| endAt   | long   | [可選]  截止時間（毫秒）                                                                                |

### 返回值
| 字段            | 含義                                          |
| ------------- | ------------------------------------------- |
| id            | 訂單id，訂單唯一標識                                 |
| symbol        | 交易對                                         |
| opType        | 操作類型: DEAL                                  |
| type          | 訂單類型                                        |
| side          | 買或賣                                         |
| price         | 訂單價格                                        |
| size          | 下單數量                                        |
| funds         | 下單金額                                        |
| dealFunds     | 成交額                                         |
| dealSize      | 成交數量                                        |
| fee           | 手續費                                         |
| feeCurrency   | 計手續費幣種                                      |
| stp           | 自成交保護                                       |
| stop          | 止盈止損類型， entry:止盈; loss:止損                   |
| stopTriggered | 是否觸發止盈止損                                    |
| stopPrice     | 止盈止損觸發價格                                    |
| timeInForce   | 訂單時效策略                                      |
| postOnly      | 是否爲被動委託                                     |
| hidden        | 是否爲隱藏單                                      |
| iceberg       | 是否爲冰山單                                      |
| visibleSize   | 冰山單在買賣盤可見數量                                 |
| cancelAfter   | timeInForce 爲 GTT n秒後過期                     |
| channel       | 下單來源                                        |
| clientOid     | 客戶端生成的標識                                    |
| remark        | 訂單說明                                        |
| tags          | 訂單標籤                                        |
| isActive      | 訂單狀態 true: 訂單狀態爲 open;<br/> false: 訂單已成交或取消 |
| cancelExist   | 訂單是否存在取消記錄                                  |
| createdAt     | 創建時間                                        |
| tradeType     | 交易類型                                       |

### 訂單狀態和結算

在買賣盤上，所有委託都處於活躍（**Active**）狀態，從買賣盤上移除的訂單則被標記爲已完成（**Done**）狀態。

訂單被成交後到入賬，因系統清算可能會有毫秒級別的延遲。

您可發送請求，查詢任一狀態的訂單。如果您未指定狀態參數，系統將默認返回“已完結”（Done）狀態的訂單。

查詢“**active**”狀態的訂單，沒有時間限制。但查詢“已完成”狀態的訂單時，您只能獲取 7 * 24 小時時間範圍內的數據（即：查詢時，開始時間到結束時間的時間範圍不能超過24 * 7小時）。若超出時間範圍，系統會報錯。如果您只指定了結束時間，沒有指定開始時間，系統將按照 24小時的範圍自動計算開始時間（開始時間=結束時間-7*24小時）並返回相應數據，反之亦然。

已取消訂單的歷史記錄僅保留**一個月**。您將無法查詢一個月以前已取消的訂單。
已完成訂單的歷史記錄僅保留**六個月**。您將無法查詢六個月以前已完成的訂單。

<aside class="notice">檢索的總條目不能超過5萬條，如果超過，請縮短查詢時間範圍。</aside>
###訂單輪詢(Polling)

對於高頻交易的用戶，建議您在本地緩存和維護一份自己的活動委託列表，並使用市場數據流實時更新自己的訂單信息。

## 最近訂單記錄
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

此接口，可獲取最近24小時的1000條訂單數據。
返回值是[分頁](#95d51b1f3b)後的數據，根據時間降序排序。

### HTTP請求
`GET /api/v1/limit/orders`

### 請求示例
`GET /api/v1/limit/orders`

### API權限
此接口需要**通用權限**。

### 返回值
| 字段            | 含義                                          |
| ------------- | ------------------------------------------- |
| id            | 訂單id，訂單唯一標識                                 |
| symbol        | 交易對                                         |
| opType        | 操作類型: DEAL                                 |
| type          | 訂單類型                                        |
| side          | 買或賣                                         |
| price         | 訂單價格                                        |
| size          | 訂單數量                                        |
| funds         | 下單金額                                        |
| dealFunds     | 成交額                                         |
| dealSize      | 成交數量                                        |
| fee           | 手續費                                         |
| feeCurrency   | 計手續費幣種                                      |
| stp           | 自成交保護                                       |
| stop          | 止盈止損類型， entry:止盈; loss:止損                   |
| stopTriggered | 是否觸發止盈止損                                    |
| stopPrice     | 止盈止損觸發價格                                    |
| timeInForce   | 訂單時效策略                                      |
| postOnly      | 是否爲被動委託                                     |
| hidden        | 是否爲隱藏單                                      |
| iceberg       | 是否爲冰山單                                      |
| visibleSize   | 冰山單在買賣盤可見數量                                 |
| cancelAfter   | timeInForce 爲 GTT n秒後過期                     |
| channel       | 下單來源                                        |
| clientOid     | 客戶端生成的標識                                    |
| remark        | 訂單說明                                        |
| tags          | 訂單標籤                                        |
| isActive      | 訂單狀態 true: 訂單狀態爲 open;<br/> false: 訂單已成交或取消 |
| cancelExist   | 訂單是否存在取消記錄                                  |
| createdAt     | 創建時間                                        |
| tradeType     | 交易類型: TRADE（現貨交易）       |
<aside class="spacer4"></aside>
<aside class="spacer2"></aside>

## 單個訂單詳情
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
此接口，可以通過訂單id獲取單個訂單信息。

### HTTP請求
`GET /api/v1/orders/{orderId}`

### 請求示例
`GET /api/v1/orders/5c35c02703aa673ceec2a168`

### API權限
此接口需要**通用權限**。

### 請求參數
| 請求參數    | 類型     | 含義                            |
| ------- | ------ | ----------------------------- |
| orderId | String | 路徑參數，[訂單Id](#6a338fcba8) 唯一標識 |

### 返回值
| 字段            | 含義                                          |
| ------------- | ------------------------------------------- |
| id            | 訂單id，訂單唯一標識                                 |
| symbol        | 交易對                                         |
| opType        | 操作類型: DEAL                                  |
| type          | 訂單類型                                        |
| side          | 買或賣                                         |
| price         | 訂單價格                                        |
| size          | 訂單數量                                        |
| funds         | 下單金額                                        |
| dealFunds     | 成交額                                         |
| dealSize      | 成交數量                                        |
| fee           | 手續費                                         |
| feeCurrency   | 計手續費幣種                                      |
| stp           | 自成交保護                                       |
| stop          | 止盈止損類型， entry:止盈; loss:止損                   |
| stopTriggered | 是否觸發止盈止損                                    |
| stopPrice     | 止盈止損觸發價格                                    |
| timeInForce   | 訂單時效策略                                      |
| postOnly      | 是否爲被動委託                                     |
| hidden        | 是否爲隱藏單                                      |
| iceberg       | 是否爲冰山單                                      |
| visibleSize   | 冰山單在買賣盤可見數量                                 |
| cancelAfter   | timeInForce 爲 GTT n秒後觸發                     |
| channel       | 下單來源                                        |
| clientOid     | 客戶端生成的標識                                    |
| remark        | 訂單說明                                        |
| tags          | 訂單標籤                                        |
| isActive      | 訂單狀態 true: 訂單狀態爲 open;<br/> false: 訂單已成交或取消 |
| cancelExist   | 訂單是否存在取消記錄                                  |
| createdAt     | 創建時間                                        |
| tradeType     | 交易類型: TRADE（現貨交易）                                       |

<aside class="spacer4"></aside>
<aside class="spacer2"></aside>


## 基於clientOid 獲取單個訂單詳情
```json
{
  "id": "5f3113a1c9b6d539dc614dc6",
  "symbol": "BTC-USDT",
  "opType": "DEAL",
  "type": "limit",
  "side": "buy",
  "price": "0.00001",
  "size": "1",
  "funds": "0",
  "dealFunds": "0",
  "dealSize": "0",
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
此接口，可以通過clientOid查詢單個訂單的信息，若訂單不存在則提示訂單不存在。

### HTTP請求
`GET /api/v1/order/client-order/{clientOid}`

### 請求示例
`GET /api/v1/order/client-order/6d539dc614db312`

### API權限
此接口需要**通用權限**。

### 請求參數
| 請求參數    | 類型     | 含義                            |
| ------- | ------ | ----------------------------- |
| clientOid | String | 路徑參數，客戶端生成的標識 |

### 返回值
| 字段            | 含義                                          |
| ------------- | ------------------------------------------- |
| id            | 訂單id，訂單唯一標識                                 |
| symbol        | 交易對                                         |
| opType        | 操作類型: DEAL                                  |
| type          | 訂單類型                                        |
| side          | 買或賣                                         |
| price         | 訂單價格                                        |
| size          | 訂單數量                                        |
| funds         | 下單金額                                        |
| dealFunds     | 成交額                                         |
| dealSize      | 成交數量                                        |
| fee           | 手續費                                         |
| feeCurrency   | 計手續費幣種                                      |
| stp           | 自成交保護                                       |
| stop          | 止盈止損類型， entry:止盈; loss:止損                   |
| stopTriggered | 是否觸發止盈止損                                    |
| stopPrice     | 止盈止損觸發價格                                    |
| timeInForce   | 訂單時效策略                                      |
| postOnly      | 是否爲被動委託                                     |
| hidden        | 是否爲隱藏單                                      |
| iceberg       | 是否爲冰山單                                      |
| visibleSize   | 冰山單在買賣盤可見數量                                 |
| cancelAfter   | timeInForce 爲 GTT n秒後觸發                     |
| channel       | 下單來源                                        |
| clientOid     | 客戶端生成的標識                                    |
| remark        | 訂單說明                                        |
| tags          | 訂單標籤                                        |
| isActive      | 訂單狀態 true: 訂單狀態爲 open					 |
| cancelExist   | 訂單是否存在取消記錄                                  |
| createdAt     | 創建時間                                        |
| tradeType     | 交易類型: TRADE（現貨交易）                                        |

# 成交明細

## 成交記錄
```json
{
    "currentPage":1,
    "pageSize":1,
    "totalNum":251915,
    "totalPage":251915,
    "items":[
        {
            "symbol":"BTC-USDT",
            "tradeId":"5c35c02709e4f67d5266954e",
            "orderId":"5c35c02703aa673ceec2a168",
            "counterOrderId":"5c1ab46003aa676e487fa8e3",
            "side":"buy",
            "liquidity":"taker",
            "forceTaker":true,
            "price":"0.083",
            "size":"0.8424304",
            "funds":"0.0699217232",
            "fee":"0",
            "feeRate":"0",
            "feeCurrency":"USDT",
            "stop":"",
            "type":"limit",
            "createdAt":1547026472000,
            "tradeType": "TRADE"
        }
    ]
}
```

此接口，可獲取最近的成交明細列表
返回值是[分頁](#95d51b1f3b)後的數據，根據時間降序排序。

### HTTP請求
`GET /api/v1/fills`

### 請求示例
`GET /api/v1/fills`

### API權限
此接口需要**通用權限**。

### 頻率限制
此接口針對每個賬號請求頻率限制爲**9次/3s**

<aside class="notice">這個接口需要使用分頁</aside>

### 請求參數
| 請求參數    | 類型     | 含義                                                                                            |
| ------- | ------ | --------------------------------------------------------------------------------------------- |
| orderId | String | [可選] 查詢該[訂單Id](#6a338fcba8) 的成交明細（如果指定了orderId，請忽略其他查詢條件）                                     |
| symbol  | String | [可選] 查詢指定[交易對](#d6402cad41)的成交明細                                                              |
| side    | String | [可選] **buy（買）** 或 **sell（賣）**                                                                 |
| type    | String | [可選] 訂單類型: **limit（限價單）**, **market(市價單)**, **limit_stop(限價止盈止損單)**, **market_stop（市價止盈止損單）** |
| startAt | long   | [可選] 開始時間（毫秒）                                                                                 |
| endAt   | long   | [可選]  截止時間（毫秒）                                                                                |
| tradeType  | String   | 交易類型: TRADE（現貨交易）                                                                                 |
### 返回值

| 字段             | 含義                       |
| -------------- | ------------------------ |
| symbol         | 交易對                      |
| tradeId        | 交易Id                     |
| orderId        | 訂單Id                     |
| counterOrderId | 對手方訂單Id                  |
| side           | 買或賣                      |
| forceTaker     | 是否強制作爲taker處理            |
| liquidity      | 流動性類型: taker 或 maker     |
| price          | 訂單價格                     |
| size           | 訂單數量                     |
| funds          | 成交額                      |
| fee            | 手續費                      |
| feeRate        | 手續費率                     |
| feeCurrency    | 計手續費幣種                   |
| stop           | 止盈止損類型，entry:止盈; loss:止損 |
| type           | 訂單類型limit 或 market       |
| createdAt      | 創建時間                     |
| tradeType     | 交易類型: TRADE（現貨交易）           |

**查詢時間範圍**
您可檢索一週時間範圍內的數據您範圍內檢索數據（默認從最近一天開始算起）。 若檢索時間範圍超過一週，系統將提示您超過時間限制。如果查詢只提供開始時間沒有提供結束時間，系統將自動計算結束時間（結束時間=開始時間+ 7*24小時），反之亦然。

<aside class="notice">檢索的總條目不能超過5萬條，如果超過，請縮短查詢時間範圍。</aside>
**結算**
結算分爲兩部分，一部分是成交結算，一部分是費用結算。當撮合完成後，這些數據將立即更新到我們的數據存儲區，系統將啓動結算並從您的預凍結資金中進行扣除。

**手續費**

Vaex平臺上的訂單分爲兩種類型：Taker 和 Maker。Taker單會與買賣盤上的已有訂單立即成交，而Maker單則相反，會一直留在買賣盤中等待撮合。Taker單消耗了市場的流動性，因此會被收取taker費用，而Maker單增加了市場的流動性，會被收取較低的手續費甚至獲得手續費補貼。請注意：市價單、冰山單和隱藏單都會被扣除taker手續費。

下單時，系統會預凍結您賬戶中的taker費用。流動性（liquidity）字段中的參數說明了訂單將會被收取taker還是maker費用。

假設您的訂單是限價單，當您下單後在撮合引擎中被立即撮合，我們將收取您taker費用，而如果您的訂單沒有被立即撮合或有部分剩餘未被撮合都會進入買賣盤，進入買賣盤的訂單在未被取消前成交都會收取您maker手續費。

進入撮合後與對手盤訂單撮合，當指令訂單剩餘金額爲0，交易完成，如果剩餘資金不足以購買最低數量（0.00000001）的商品，則取消指令訂單。

如果您的訂單作爲maker被成交，我們會將剩餘預凍結的taker費用返還給您。

但需要注意的是:

- 當您創建了一個隱藏委託/冰山委託訂單時，即使它未被撮合引擎立即成交而被被動成交，仍然會收取taker費用
- 被動委託收取maker費用。如果該委託下單後會立刻與市場已有委託(除冰山/隱藏訂單外)撮合，那麼該委託將被取消。如果被動委託下單後與冰山/隱藏訂單立即成交，被動委託訂單將收取**maker**費用  

舉例：

以BTC/USDT爲例，假設您想市價買入1BTC，手續費率爲0.1%，市場買賣盤數據如下：

| Price（USDT） | Size（BTC）  | Side |
| ----------- | ---------- | ---- |
| 4200.00     | 0.18412309 | sell |
| 4015.60     | 0.56849308 | sell |
| 4011.32     | 0.24738383 | sell |
| 3995.64     | 0.84738383 | buy  |
| 3988.60     | 0.20484000 | buy  |
| 3983.85     | 1.37584908 | buy  |

 當您下一個買入市價單時，市場會立即成交，成交明細將分爲3筆，如下圖所示：

| Price（USDT） | Size（BTC）  | Fee（BTC）   |
| ----------- | ---------- | ---------- |
| 4011.32     | 0.24738383 | 0.00024738 |
| 4015.60     | 0.56849308 | 0.00056849 |
| 4200.00     | 0.18312409 | 0.00018312 |

## 最近成交記錄
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

此接口，可以獲取最近24小時1000條成交明細的列表
返回值是[分頁](#95d51b1f3b)後的數據，根據時間降序排序。

### HTTP請求
`GET /api/v1/limit/fills`

### 請求示例
`GET /api/v1/limit/fills`

### API權限
此接口需要**通用權限**。

### 返回值

| 字段             | 含義                        |
| -------------- | ------------------------- |
| symbol         | 交易對                       |
| tradeId        | 交易Id                      |
| orderId        | 訂單Id                      |
| counterOrderId | 對手方訂單Id                   |
| side           | 買或賣                       |
| forceTaker     | 是否強制作爲taker處理             |
| liquidity      | 流動性類型: taker 或 maker      |
| price          | 訂單價格                      |
| size           | 訂單數量                      |
| funds          | 成交額                       |
| fee            | 手續費                       |
| stop           | 止盈止損類型， entry:止盈; loss:止損 |
| tradeType     | 交易類型: TRADE（現貨交易）                                        |
| type           | 訂單類型，limit 或 market       |
| createdAt      | 創建時間                      |

<aside class="spacer2"></aside>



# 市場數據

市場數據是公共的，不需要驗證簽名。


# 交易對 & 行情快照

## 交易對列表
```json
[
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
    "enableTrading": true
  },
  {
    "symbol": "VET-USDT",
    "name": "VET-USDT",
    "baseCurrency": "VET",
    "quoteCurrency": "USDT",
    "feeCurrency": "USDT",
    "market": "USDS",
    "baseMinSize": "10",
    "quoteMinSize": "0.01",
    "baseMaxSize": "10000000000",
    "quoteMaxSize": "99999999",
    "baseIncrement": "0.0001",
    "quoteIncrement": "0.000001",
    "priceIncrement": "0.0000001",
    "priceLimitRate": "0.1",
    "minFunds": "0.1",
    "enableTrading": true
  }
]
```
此接口，可獲取交易對列表。如果您想獲取更多交易對的市場信息，可在[全局行情快照](#f3027c9902)中獲取。


### HTTP請求
`GET /api/v2/symbols`

### 請求示例
`GET /api/v2/symbols`

### 請求參數
請求參數 | 類型 | 是否必須 | 含義 |
--------- | ------- | -----------| -----------|
market | String | 否 | [交易市場](#5666d37373) |

### 返回值
| 字段             | 含義                            |
| -------------- | ----------------------------- |
| symbol         | 交易對唯一標識碼，重命名後不會改變             |
| name           | 交易對名稱，重命名後會改變                 |
| baseCurrency   | 商品貨幣，指一個交易對的交易對象，即寫在靠前部分的資產名  |
| quoteCurrency  | 計價幣種，指一個交易對的定價資產，即寫在靠後部分資產名   |
| market         | [交易市場](#5666d37373)                   |
| baseMinSize    | 下單時size的最小值                   |
| quoteMinSize   | 下市價單，funds的最小值                |
| baseMaxSize    | 下單，size的最大值                   |
| quoteMaxSize   | 下市價單，funds的最大值                |
| baseIncrement  | 數量增量，下單的size必須爲數量增量的正整數倍      |
| quoteIncrement | 市價單：資金增量，下單的funds必須爲資金增量的正整數倍 |
| priceIncrement | 限價單：價格增量，下單的price必須爲價格增量的正整數倍 |
| feeCurrency    | 交易計算手續費的幣種                   |
| enableTrading  | 是否可以用於交易                      |
| priceLimitRate | 價格保護閾值                          |
| minFunds       | 最小交易金額                          |

- `baseMinSize` 和 `baseMaxSize` 這兩個字段規範了下單size的最小值和最大值。
- `priceIncrement` 字段規範了下單的price的最小值和價格增量。

下單的`price`必須爲價格增量的正整數倍（如果增量爲 0.01，下單價格是0.001或0.021的請求會被拒絕，返回invalid priceIncrement）

`priceIncrement` 和 `quoteIncrement` 以後可能會調整，調整前我們會提前以郵件和全站通知的方式進行通知。

委託單 | 遵循minFunds的規則
--------- | ------- | -----------
限價買單 | [委託數量*委託價格] >= `minFunds`
限價賣單 | [委託數量*委託價格] >= `minFunds`
市價買單 | 委託金額 >= `minFunds`
市價賣單 | [委託數量*BASE幣種最新成交價] >= `minFunds`

注：

* 對於API中按數量進行市價買單的情況，採用[委託數量*BASE幣種最新成交價]<`minFunds`進行判斷
* 對於API中按照金額進行市價賣單的情況，採用委託金額<`minFunds`進行判斷。
* 對於市價及限價止盈止損委託，委託時不受此限制，但委託觸發之後的下單後亦會受到此限制

## 行情快照
```json
//Get Ticker
{
    "sequence":"1550467636704",
    "price":"0.03715005",
    "size":"0.17",
    "bestAsk":"0.03715004",
    "bestAskSize":"1.788",
    "bestBid":"0.03710768",
    "bestBidSize":"3.803",
    "time":1550653727731
}
```
此接口，會返回level-1市場行情快照，買/最佳賣一價，買/賣一數量，最近成交價，最近成交數量。

### HTTP請求
`GET /api/v1/market/orderbook/level1`

### 請求示例
`GET /api/v1/market/orderbook/level1?symbol=BTC-USDT`

### 請求參數
請求參數 | 類型 | 含義
--------- | ------- | -------
symbol | String |  [交易對](#d6402cad41)

### 返回值
字段 | 含義
--------- | -------
sequence | 序列號
price |  最新成交價格
size | 最新成交數量
bestAsk |  最佳賣一價
bestAskSize |  最佳賣一數量
bestBid |  最佳買一價
bestBidSize | 最佳買一數量
time |  時間戳
<aside class="spacer2"></aside>




# 委託買賣盤

## Level-2部分買賣盤(價格聚合)
```json
{
    "sequence":"3262786978",
    "bids":[
        [
            "6500.12",//price
            "0.45054140"//size
        ],
        [
            "6500.11",
            "0.45054140"
        ]
    ],
    "asks":[
        [
            "6500.16",
            "0.57753524"
        ],
        [
            "6500.15",
            "0.57753524"
        ]
    ],
    "time":1550653727731
}
```
此接口，可獲取指定交易對的買賣盤數據。

買賣盤上的買單和賣單均按照價格彙總，每個價格下僅返回一個根據價格彙總的掛單量。

此接口，只會返回部分的買賣盤數據，level2_20是指返回買賣方各20條數據，level_100 是指返回買賣方各100條數據。推薦您使用這個接口，因爲響應速度更快，流量消耗小。



### HTTP請求
`GET /api/v1/market/orderbook/level2_20`<br/>
`GET /api/v1/market/orderbook/level2_100`

### 請求示例
`GET /api/v1/market/orderbook/level2_20?symbol=BTC-USDT`<br/>
`GET /api/v1/market/orderbook/level2_100?symbol=BTC-USDT`

### 請求參數
請求參數 | 類型 | 含義
--------- | -------  | -------
symbol | String |  [交易對](#d6402cad41)

### 返回值
字段 | 含義
--------- | -------
sequence | 序列號
time | 時間戳
bids | 買盤
asks | 賣盤

###數據排序方式

**Asks**: 買盤，根據價格從低到高

**Bids**: 賣盤，根據價格從高到低


## Level-2全部買賣盤(價格聚合)
```json
{
    "sequence":"3262786978",
    "bids":[
        [
            "6500.12", //price
            "0.45054140"
        ],
        [
            "6500.11",
            "0.45054140"
        ]
    ],
    "asks":[
        [
            "6500.16",
            "0.57753524"
        ],
        [
            "6500.15",
            "0.57753524"
        ]
    ],
    "time":1550653727731
}
```
此接口獲取指定交易對的所有活動委託的快照。

Level 2 買賣盤上的買單和賣單均按照價格彙總，每個價格下僅返回一個根據價格彙總的掛單量。

此接口將返回全部的買賣盤數據。

該功能適用於專業交易員，因爲該過程將使用較多服務器資源及流量，訪問頻率受到了嚴格控制。

爲保證本地買賣盤數據爲最新數據，在獲取Level 2快照後，請使用[Websocket](#level-2-3)推送的增量消息來更新Level 2買賣盤。

### HTTP請求
`GET /api/v3/market/orderbook/level2`

### 請求示例
`GET /api/v3/market/orderbook/level2?symbol=BTC-USDT`

### API權限
此接口需要**通用權限**。

### 頻率限制
此接口針對每個賬號請求頻率限制爲**30次/3s**

### 請求參數
請求參數 | 類型 | 含義
--------- | ------- | -------
symbol | String |  [交易對](#d6402cad41)

### 返回值
字段 | 含義
--------- | -------
sequence | 序列號
time | 時間戳
bids | 買盤
asks | 賣盤

###數據排序方式

**Asks**: 買盤，根據價格從低到高

**Bids**: 賣盤，根據價格從高到低



<aside class="spacer4"></aside>

# 歷史數據

## 成交歷史
```json
[
    {
        "sequence":"1545896668571",
        "price":"0.07",//成交價格
        "size":"0.004", //成交數量
        "side":"buy", //成交方向
        "time":1545904567062140823
    },
    {
        "sequence":"1545896668578",
        "price":"0.054",
        "size":"0.066",
        "side":"buy",
        "time":1545904581619888405
    }
]
```
此接口，可獲取指定交易對的成交歷史列表。

### HTTP請求
`GET /api/v1/market/histories`

### 請求示例
`GET /api/v1/market/histories?symbol=BTC-USDT`

### 請求參數
請求參數 | 類型 | 含義
--------- | -------  | -------
symbol | String |  [交易對](#d6402cad41)

### 返回值

字段 | 含義
--------- | -------
sequence | 序列號
time | 交易時間戳
price | 訂單價格
size | 訂單數量
side |賣方 或 買方

###SIDE
Taker訂單的成交方向。Taker訂單指立刻與買賣盤上的已有訂單成交的訂單類型。



<aside class="spacer2"></aside>

## 歷史蠟燭圖數據
```json
[
    [
        "1545904980", //k線週期的開始時間
        "0.058",  //開盤價
        "0.049",  //收盤價
        "0.058", //最高價
        "0.049", //最低價
        "0.018",  //成交量
        "0.000945"  //成交額
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
此接口，返回指定交易對的kline(蠟燭圖），返回數據根據時間粒度劃分。

<aside class="notice">  歷史蠟燭圖數據可能不完整，請勿輪詢調用此接口，可以通過Websocket訂閱實時數據</aside>


### HTTP請求
`GET /api/v1/market/candles`

### 請求示例
`GET /api/v1/market/candles?type=1min&symbol=BTC-USDT&startAt=1566703297&endAt=1566789757`

### 請求參數
請求參數 | 類型 | 含義
------------- | ------- | -------
symbol | String |  [交易對](#d6402cad41)
startAt| long | [可選] 開始時間（秒）默認值爲0
endAt| long | [可選]  截止時間（秒）默認值爲0
type | String | 時間粒度，也就是每根蠟燭的時間區間:<br/>**1min, 3min, 5min, 15min, 30min, 1hour, 2hour, 4hour, 6hour, 8hour, 12hour, 1day, 1week**

<aside class="notice"> 每次查詢系統最多返回1500條數據。要獲得更多數據，請按時間分頁數據。</aside>

### 返回值
字段 | 含義
--------- | -------
time | k線週期的開始時間
open | 開盤價
close | 收盤價
high | 最高價
low | 最低價
volume | 成交量
turnover | 成交額


# 幣種

## 幣種列表
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
    "isDepositEnabled": true
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
    "isDepositEnabled": true
  }
]
```
此接口，返回幣種詳情列表。

<aside class="notice">並不是所有的幣種可以用於交易</aside>

### HTTP請求
`GET /api/v1/currencies`

### 請求示例
`GET /api/v1/currencies`

### 返回值
| 字段  | 含義                |
| --- | ----------------- |
|currency| 幣種唯一標識，不會改變|
|name| 幣種名，可變更|
|fullName| 幣種全稱，可變更|
|precision| 幣種精度 |
|confirms| 區塊鏈確認數|
|contractAddress| 合約地址 |
|withdrawalMinSize| 提現最小值 |
|withdrawalMinFee| 提現最小手續費 |
|isWithdrawEnabled| 是否可提現 |
|isDepositEnabled| 是否可充值|


### 幣種標識(currency code)
幣種標識（code）均符合 ISO 4217 的標準，不符合ISO 4217標準中無法標識的幣種，將採取自定義標識。

| Code | 含義            |
| ---- | ------------- |
| BTC  | Bitcoin       |
| ETH  | Ethereum      |

返回值中的**currency**是不會改變的，而name、fullname、precision等都可能會變動，當一個幣種更換name時，您仍可以使用currency去獲取該幣種的信息。

例如：XRB更名後變爲Nano，但它的currency仍然是XRB，而它的name變更爲Nano，此時您仍然需要通過XRB去查詢該幣種。

## 幣種詳情
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
  "isDepositEnabled": true
}
```
此接口，返回可交易幣種的貨幣詳細信息

### HTTP請求
`GET /api/v1/currencies/{currency}`

### 請求示例
`GET /api/v1/currencies/BTC`

### 請求參數
| 請求參數     | 類型     | 含義                                                              |
| -------- | ------ | --------------------------------------------------------------- |
| currency | String | 路徑參數，[幣種標識](#47f0f7e8df)                                        |
| chain    | String | [可選] 針對一幣多鏈的幣種，可通過chain獲取幣種詳情。 |

### 返回值
|字段 | 含義|
|--------- | -------|
|currency| 幣種唯一標識，不會改變|
|name| 幣種名，可變更|
|fullName| 幣種全稱，可變更|
|precision| 幣種精度 |
|confirms| 區塊鏈確認數|
|contractAddress| 合約地址|
|withdrawalMinSize| 提現最小值 |
|withdrawalMinFee| 提現最小手續費 |
|isWithdrawEnabled| 是否可提現 |
|isDepositEnabled| 是否可充值|

## 幣種詳情(推薦使用)
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
            },
            ...
        ]
    }
}
```
此接口，返回可交易幣種的貨幣詳細信息

### HTTP請求
`GET /api/v2/currencies/{currency}`

### 請求示例
`GET /api/v2/currencies/BTC`

<aside class="notice">推薦使用</aside>

### 請求參數
| 請求參數     | 類型     | 含義                                                              |
| -------- | ------ | --------------------------------------------------------------- |
| currency | String | 路徑參數，[幣種標識](#47f0f7e8df)                                        |
| chain    | String | [可選] 可通過chain獲取幣種指定鏈的詳情，不傳默認返回所有鏈的幣種詳情。 |

### 返回值
|字段 | 含義|
|--------- | -------|
|currency| 幣種唯一標識，不會改變|
|name| 幣種名，可變更|
|fullName| 幣種全稱，可變更|
|precision| 幣種精度 |
|confirms| 區塊鏈確認數|
|contractAddress| 合約地址|
|withdrawalMinSize| 提現最小值 |
|chainName| 幣種chain名字 |
|chain| 幣種chain |
|withdrawalMinFee| 提現最小手續費 |
|isWithdrawEnabled| 是否可提現 |
|isDepositEnabled| 是否可充值|



## 法幣換算價格
此接口，返回法幣換算後的價格

```json
{
    "code":"200000",
    "data":{

        "BTC":"3911.28000000",
        "ETH":"144.55492453",
        "LTC":"48.45888179"
    }
}
```
### HTTP 請求
`GET /api/v1/prices`

### 請求示例
`GET /api/v1/prices`

### 請求參數
請求參數 | 類型 | 含義
--------- | ------- | -------
 base | String | [可選] 法幣貨幣代碼。比如，USD，EUR 默認爲USD |
 currencies | String  |[可選] 需轉換的數字貨幣（多個幣種，請使用“,“進行間隔）。比如，BTC,ETH 。默認爲返回所有幣種的法幣價格|





# Websocket
REST API的使用受到了訪問頻率的限制，因此推薦您使用Websocket獲取實時數據。

<aside class="notice">推薦您只創建一個Websocket連接，在這個連接上訂閱多個通道的數據</aside>

## 申請連接令牌
在創建Websocket連接前，您需申請一個令牌（Token）。

### 公共令牌 (不需要驗證簽名):
```json
{
	"code": "200000",
	"data": {
		"token": "2neAiuYvAU61ZDXANAGAsiL4-iAExhsBXZxftpOeh_55i3Ysy2q2LEsEWU64mdzUOPusi34M_wGoSf7iNyEWJ4aBZXpWhrmY9jKtqkdWoFa75w3istPvPtiYB9J6i9GjsxUuhPw3BlrzazF6ghq4L_u0MhKxG3x8TeN4aVbNiYo=.mvnekBb8DJegZIgYLs2FBQ==",
		"instanceServers": [{
			"endpoint": "wss://ws-api-spot.vaex.com/",	//建議使用動態URL，該URL可能會改變
			"encrypt": true,
			"protocol": "websocket",
			"pingInterval": 18000,
			"pingTimeout": 10000
		}]
	}
}

```
如果您只訂閱公共頻道的數據，請按照以下方式請求獲取服務實例列表和公共令牌。

#### HTTP請求
`POST /api/v1/bullet-public`

### 私有令牌 (需要驗證簽名):
如您需請求私有頻道的數據（如：賬戶資金變化），請在簽名驗證後按照以下方式獲取Websocket的服務實例和已驗籤的令牌。

#### HTTP請求
`POST /api/v1/bullet-private`

### 返回值
|字段 | 含義|
-----|-----
|endpoint| Websocket建立連接的服務器地址 |
|protocol| 支持的協議 |
|encrypt| 表示是否使用了SSL加密 |
|pingInterval| 建議發送ping的時間間隔（毫秒）|
|pingTimeout| 如果在pingTimeout時間後，未收到pong消息，那麼連接可能已斷開了 |
|token | 令牌 |

## 建立連接
```javascript
var socket = new WebSocket("wss://ws-api-spot.vaex.com/?token==xxx&[connectId=xxxxx]");
```
成功建立連接後，您將會收到系統向您發出的歡迎（welcome）消息。

<aside class="notice">客戶端需要等待welcome消息，只有收到了welcome消息，才表示連接可用。</aside>

```json
{
    "id":"hQvf8jkno",
    "type":"welcome"
}
```
**connectId**：連接ID，是客戶端生成的唯一標識。您在創建連接時收到的歡迎（welcome）消息的ID以及錯誤消息的ID都屬於連接ID（connectId）。

如果你想只接收指定topic的私人消息，請在訂閱時使用privateChannel:true。

<aside class="spacer2"></aside>

## Ping
```json
{
    "id":"1545910590801",
    "type":"ping"
}
```
爲防止服務器斷開TCP連接，建議客戶端每間隔pingInterval時間發送一條ping指令。
在服務器收到ping消息後，系統會向客戶端返回一條pong消息。
如果服務器長時間沒有收到來自客戶端的消息，連接將被斷開。

```json
{
    "id":"1545910590801",
    "type":"pong"
}
```
<aside class="spacer3"></aside>

## 訂閱
```json
  // 訂閱
{
    "id":"123456789",
    "type":"subscribe",
    "topic":"/market/ticker:BTC-USDT,ETH-USDT",
    "privateChannel":false,
    "response":true
}
```
使用服務器訂閱消息時，客戶端應向服務器發送訂閱消息。

訂閱成功後，當**response**參數爲**true**時，系統將向您發出**“ack”消息**。

```json
{
    "id":"1545910660739",
    "type":"ack"
}
```

當訂閱頻道產生新消息時，系統將向客戶端推送消息。瞭解消息格式，請查看頻道介紹。

### 參數
#### ID
ID用於標識請求和ack的唯一字符串。

#### Topic
您訂閱的頻道內容。

#### PrivateChannel
您可通過privateChannel參數訂閱以一些特殊的topic（如：/market/level2）。該參數默認設置爲“false”。設置爲“true”時，則您只能收到與您訂閱的topic相關的內容推送。

#### Response
若設置爲True, 用戶成功訂閱後，系統將返回ack消息。

客戶端需要發送訂閱消息到服務端，獲取指定topic的消息。

但系統會將相應topic的消息發送到客戶端，詳情返回值請參照指定的topic。

## 退訂
用於取消您之前訂閱的topic

```json
  // 取消訂閱
{
    "id":"1545910840805",  //要求唯一
    "type":"unsubscribe", //類型: unsubscribe
    "topic":"/market/ticker:BTC-USDT,ETH-USDT",
    "privateChannel":false, //是否使用該頻道的私有頻道，默認爲false
    "response":true //是否需要服務端返回本次訂閱的回執信息，默認爲false   
}

```

```json
{
    "id":"1545910840805",
    "type":"ack"
}
```

### 參數

#### ID
用於標識請求的唯一字符串。

#### Topic
您訂閱的topic內容。

#### PrivateChannel
設置爲“true”，您可以退訂相關的私有頻道推送。


#### Response
退訂成功後，當**response**參數爲**true**時，系統將向您發出“ack”消息。


## 多路複用
 在一條物理連接上，您可開啓多條多路複用通道，以訂閱不同topic，獲取多種數據推送。

### 開啓
例如： 請輸入以下指令定開啓多條bt1通道 {"id": "1Jpg30DEdU", "type": "openTunnel", "newTunnelId": "bt1", "response": true}

在指定中添加參數tunnelId： {"id": "1JpoPamgFM", "type": "subscribe", "topic": "/market/ticker:BTC-USDT"，"tunnelId": "bt1", "response": true}

### 返回值
請求成功後，您將收到 tunnelIId 對應的消息推送：{"id": "1JpoPamgFM", "type": "message", "topic": "/market/ticker:BTC-USDT", "subject": "trade.ticker", "tunnelId": "bt1", "data": {...}}

### 關閉
請輸入以下指令：{"id": "1JpsAHsxKS", "type": "closeTunnel", "tunnelId": "bt1", "response": true}

### 限制
- 多路複用僅限API用戶使用。
- 最多可開啓的多路複用通道條數：5條。

## 定序編號
買賣盤數據化、成交歷史數據以及快照消息均會默認返回sequence字段。您可以從Level 2和Level 3市場行情數據中的sequence來判斷數據是否丟失，連接是否穩定。如果連接不穩定，請按照校準流程進行校準。

## 消息處理邏輯

- 判斷消息的type: 目前有三類消息，message（常用的推送消息），notice（一般的通知），command（連接的命令）
- 判斷消息topic: 通過topic判斷是哪一類消息
- 判斷subject: 同一個topic的不同類型消息用subject區分。


# 公共頻道

## 交易瞬時行情
```json
{
    "id":1545910660739,
    "type":"subscribe",
    "topic":"/market/ticker:BTC-USDT",
    "response":true
}
```
```json
{
    "type":"message",
    "topic":"/market/ticker:BTC-USDT",
    "subject":"trade.ticker",
    "data":{
        "sequence":"1545896668986", //序列號
        "price":"0.08",             // 最近成交價格
        "size":"0.011",             // 最近成交數量
        "bestAsk":"0.08",           //最佳賣一價
        "bestAskSize":"0.18",       // 最佳賣一數量
        "bestBid":"0.049",          //最佳買一價
        "bestBidSize":"0.036",      //最佳買一數量
    }
}
```
Topic: `/market/ticker:{symbol},{symbol}...`

* 推送頻率: `100ms`一次

訂閱此topic可獲取指定[交易對](#d6402cad41)的BBO(最佳買一和賣一)數據的推送。

平臺後期可能會向該渠道推送更多的信息。

<aside class="spacer2"></aside>
<aside class="spacer4"></aside>


## 全部交易對瞬時行情

```json
{
    "id":1545910660739,
    "type":"subscribe",
    "topic":"/market/ticker:all",
    "response":true
}
```
```json
{
    "type":"message",
    "topic":"/market/ticker:all",
    "subject":"BTC-USDT",
    "data":{
        "sequence":"1545896668986",
        "price":"0.08",
        "size":"0.011",
        "bestAsk":"0.08",
        "bestAskSize":"0.18",
        "bestBid":"0.049",
        "bestBidSize":"0.036"
    }
}
```
Topic: `/market/ticker:all`

* 推送頻率: `100ms`一次

訂閱此topic可獲取所有的BBO(最佳買一和賣一)數據的推送。
<aside class="spacer8"></aside>




## Level-2市場行情

```json
{
    "id":1545910660740,
    "type":"subscribe",
    "topic":"/market/level2:BTC-USDT",
    "response":true
}
```

Topic: `/market/level2:{symbol},{symbol}...`

* 推送頻率: `實時推送`

訂閱此topic可獲取指定[交易對](#d6402cad41)Level-2買賣盤數據。訂閱成功後，服務端會推送增量的市場數據給您。


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

校準流程:

1. 將Websocket推送的Level 2數據緩存在本地;
2. 通過REST請求拉取[Level 2](#level-2-2)買賣盤的快照;
3. 回放緩存的Level 2數據流;
4. 將Level 2數據流應用到快照上，只需滿足 sequenceStart(new)<=sequenceEnd+1(old) 並且 seqenceEnd(new) > sequenceEnd(old)，即可認爲是有效的L2增量消息，可以用changes中的數據覆蓋本地數據。changes中每條記錄上的sequence僅表示該價格的數量最後一次修改對應的sequence，不作爲消息連續的判斷依據；
5. 根據訂單的價格和數量更新買賣盤。如果價格爲0，忽略這條消息，只更新順序號；如果數量爲0，則需要將該數量對應的訂單價格從買賣盤中移除。如遇其他情況，正常更新買賣盤即可。

[Level 2](#level-2-2) 的Change屬性是一個“price, size, sequence”的字符串值,即：[“價格”,“數量”,“sequence”]。

請注意: size指的是price對應的最新size。當size爲0時，需要將其對應的price從買賣盤中刪除。


**示例**

以BTC/USDT爲例，假設level 2當前買賣盤數據如下:

步驟1.成功訂閱此topic，您會收到如下買賣盤數據流:

<code>
...<br/>
"asks":[<br/>
&nbsp;&nbsp;["3988.59","3", 16], // 摒棄 sequence = 16<br/>
&nbsp;&nbsp;["3988.61","0", 19], // 移除 price 爲 3988.61 的數據<br/>
&nbsp;&nbsp;["3988.62","8", 15], // 摒棄 sequence <16 <br/>
]<br/>
"bids":[<br/>
&nbsp;&nbsp;["3988.50", "44", "18"] // 更新 price 爲 3988.50 的size<br/>
]<br/>
"sequenceEnd": 15,<br/>
"sequenceStart": 19,<br/>
...<br/>
</code>

<aside class="notice">changes中每條記錄上的sequence僅表示該價格的數量最後一次修改對應的sequence，不作爲消息連續的判斷依據；比如當在相同價位有多個數量更新["3988.50", "20", "17"]、["3988.50", "44", "18"]，此時只會推送最新的["3988.50", "44", "18"]</aside>

步驟2.通過REST請求拉取[Level-2買賣盤快照信息](#level-2-2)

<code>
...<br/>
"sequence": "16",<br/>
"asks":[<br/>
&nbsp;&nbsp;["3988.62","8"],// [“價格”,“數量”]<br/>
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

當前拉取的買賣盤的快照數據如下:

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

當前買賣盤快照信息的sequence爲`16`，摒棄買賣盤數據流中sequence <= 16的數據，回放sequence爲`[18,19]`的數據，更新買賣盤快照信息。現在，您本地的sequence爲`19`。

變更:

- 將價格3988.50對應的數量變更爲44（sequence順序號爲18）
- 移除價格爲3988.61的數據（sequence順序號爲19）

變更後，當前買賣盤數據爲最新數據，具體數據如下:

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


## Level2 - 50檔深度頻道
```json
{
    "type": "message",
    "topic": "/spotMarket/level2Depth50:BTC-USDT",
    "subject": "level2",
    "data": {
	    "asks":[
            ["9993","3"],    //價格, 數量
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

* 推送頻率: `100ms`一次

每次返回前50檔的深度數據，此數據爲每100毫秒的快照數據，即每隔100毫秒，快照當前時刻市場買賣盤的50檔深度數據並推送

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer"></aside>


## K線

```json
{
    "type":"message",
    "topic":"/market/candles:BTC-USDT_1hour",
    "subject":"trade.candles.update",
    "data":{
        "symbol":"BTC-USDT",    // 交易對
        "candles":[
            "1589968800",   // candle的開盤時間
            "9786.9",       // open開票價
            "9740.8",       // close收盤價
            "9806.1",       // high最高價
            "9732",         // low最低價
            "27.45649579",  // volume成交量
            "268280.09830877"   // turnover成交額
        ],
        "time":1589970010253893337  // 當前時間，納秒
    }
}
```
Topic: `/market/candles:{symbol}_{type}`

* 推送頻率: `實時推送`

參數 |  含義
--------- | -------
symbol | 交易對
type |  `1min`, `3min`, `15min`, `30min`, `1hour`, `2hour`, `4hour`, `6hour`, `8hour`, `12hour`, `1day`, `1week`


訂閱此topic可獲取指定 symbol的指定 type 的K線數據。

訂閱成功後，服務端會推送的市場K線數據給您。

<aside class="spacer2"></aside>







## 撮合執行數據

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

* 推送頻率: `實時推送`

訂閱此topic，可獲取撮合執行數據。


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




# 私有頻道

訂閱私人頻道需要`privateChannel=“true”`。

## 私有訂單變更事件

Topic: `/spotMarket/tradeOrders`

* 推送頻率: `實時推送`

該topic將推送所有有關您的訂單的變更事件。

**訂單狀態**

"match": 訂單爲taker時與買賣盤中訂單成交，此時該taker訂單狀態爲match；

"open": 訂單存在於買賣盤中；

"done": 訂單完成；


### 消息類型


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

訂單進入買賣盤時發出的消息。

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
訂單成交時發出的消息

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
        "symbol":"BTC-USDT",
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
訂單因成交後狀態變爲DONE時發出的消息

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
訂單因被取消後狀態變爲DONE時發出的消息

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
訂單因被修改發出的消息

<aside class="spacer4"></aside>
<aside class="spacer4"></aside>
<aside class="spacer2"></aside>


## 餘額變更事件
```json
{
    "type":"message",
    "topic":"/account/balance",
    "subject":"account.balance",
    "channelType":"private",
    "data":{
        "total": "88", //總額
        "available": "88", // 可用餘額
        "availableChange": "88", // 可用餘額變化值
        "currency": "BTC", // 幣種
        "hold": "0", // 凍結金額
        "holdChange": "0", // 可用凍結金額變化值
        "relationEvent": "trade.setted", // 關聯事件
        "relationEventId": "5c21e80303aa677bd09d7dff", // 關聯事件id
        "relationContext": {
            "tradeId":"5e6a5dca9e16882a7d83b7a4", // 成交了纔會有tradeId
            "orderId":"5ea10479415e2f0009949d54",
            "symbol":"BTC-USDT"
        }, // 交易事件的上下文
    "time": "1545743136994" // 時間戳
  }
}

```
Topic: `/account/balance`

* 推送頻率: `實時推送`

當您的賬戶餘額變更時，您會收到詳細的賬戶變更信息。

### Relation Event

類型 | 描述
--------- | -------
main.deposit | 充值入賬
main.withdraw_hold | 提現凍結
main.withdraw_done | 提現完成
main.transfer | 資金賬戶轉賬
main.other | 資金賬戶其他操作
trade.hold | 交易賬戶凍結
trade.setted | 交易賬戶入賬
trade.transfer | 交易賬戶轉賬
trade.other | 交易賬戶其他操作
other | 其他操作
<aside class="spacer2"></aside>
