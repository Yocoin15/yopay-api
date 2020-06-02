# Api Overview

This is a tool for integration with yopay.tech - for developers. Allows you to accept crypto payments, learn about the status of operations and much more.
This document contains descriptions of the main scenarios with code examples. All information about the query structure, objects and variables is in the API reference book.
To accept payment, you need only a few lines of code. Let's go 🚀

### Quick Start 

To start working with yopay.tech, you need to register and get access to your personal account. To authenticate requests to the API, you need the secret key from your personal account.

Once registered, you can manage the currencies you want to integrate in the Membership area / Currencies. 
Please enable the currencies there before using this API.

### Get a list of enabled currencies.

![label: GET][~get] */currencies/?token={token}*

**Example:** https://api.yopay.tech/currencies/?token=WKt2skCsqns6666666666CJv0m4DUkX

**Headers:** Content-Type: application/json

**Parameters:**
 - token - API Secret Key

**Response body:**
```json
{
   "success": true,
   "data": [
       "eth",
       "btc"
   ]
}
```

### Create payment request


Payment is the main essence of the yopay.tech API. To create it, you will need the token and URL to which the confirmation will be returned after payment.

![label: GET][~get]  */{crypto}/payment/?token={token}*

![label: GET][~get]  */{crypto}/payment/?callback={callback_url}&token={token}*


**Examples:**

Without callback: 

https://api.yopay.tech/eth/payment/?token=WKt2skCsqns6666666666CJv0m4DUkX

With callback: 

https://api.yopay.tech/eth/payment/?token=WKt2skCsqns6666666666CJv0m4DUkX&callback=https://example.com/cb.php

**Headers:** Content-Type: application/json

**Parameters:**
 - token - API Secret Key
 - crypto - Crypto currency to accept

**Optional parameters:**
 - callback_url - Your server callback url (urlencoded) to get information about payment

**Response body:**
 
*The API always responds with a JSON string. [data] collection contains the important values:*
 - *[address] is the payment address to show to the customer*
 - *[invoice] is our inner payment identifier, keep it in a safe place and never disclose to your clients.*

```json
{
    "success": true,
    "data": {
        "invoice": "d1ddf6e3767030b06666666eae403600",
        "address": "0xf75574f061cd66666666666666666666666666f2"
    }
}
```

**Callback example:**

*A callback is sent every time a new block is mined. It stops when transaction get maxConfirmations confirmations. See code sample below.*

```json
{
	"transaction_amount": 2000000000000000000,
	"transaction_hash": "0xfd4609159efea3804335e4a2666666666666666666666bb07869cc8b5536827c",
	"block_hash": "0xd5800a894ac462cd3338f24068a6666666666666666666664a58fed4b9e15e8f",
	"block_number": 140,
	"address": "0xf75574f061cd66666666666666666666666666f2",
	"callback": "http://example.com/cb.php",
	"blockchain": "eth",
	"status": "complete",
	"confirmations": 13,
	"invoice": "d1ddf6e3767030b06666666eae403600",
	"maxConfirmations": 4
}
```

**PHP callback cb.php example:**

```php
$orderId = $_GET['orderId'];
$data = file_get_contents('php://input');
if ($data) {
    $params = json_decode($data, true);
    $invoice = $params['invoice'];

    //save number of confirmations to DB: $params['confirmations']

    if ($params['confirmations'] >= $params['maxConfirmations']) {
        $amountPaid = $params['transaction_amount'];
        // check is order not already marked as paid
        // compare $amountPaid with order total
        // compare $invoice with one saved in the database to ensure callback is legitimate
        // do other required checks
        // mark the order as paid
        die('ok'); //stop further callbacks
    } else {
        die("waiting for confirmations");
    }
}
```


### Get Invoice Info and Status 

A payment can be considered successful as soon as it has moved to the complete status. If the user changes his mind to pay or something goes wrong, the payment will go to status failed.
To find out the status of a payment, you can request information about a payment at a frequency convenient for you. To do this, you will need the payment identifier {invoice} (the value of the id parameter in the created payment object).

![label: GET][~get]  */invoice/{invoice}/?token={token}*

Obtain information about invoice which already stop sent callback requests

**Example:** https://api.yopay.tech/invoice/d1ddf6e3767030b06666666eae403600/?token=WKt2skCsqns6666666666CJv0m4DUkX 

**Headers:** Content-Type: application/json

**Parameters:**
 - token - API Secret Key
 - invoice - Invoice ID from Create payment request
 
**Response body:**

*The API returns a JSON string containing the information about invoice that same as data sent to callback url.*

```json
{
    "transaction_amount": 2000000000000000000,
    "transaction_hash": "0xfd4609159efea3804335e4a2666666666666666666666bb07869cc8b5536827c",
    "block_hash": "0xd5800a894ac462cd3338f24068a6666666666666666666664a58fed4b9e15e8f",
    "block_number": 140,
    "address": "0xf75574f061cd66666666666666666666666666f2",
    "callback": "http://example.com/cb.php",
    "blockchain": "eth",
    "status": "complete",
    "confirmations": 13,
    "invoice": "d1ddf6e3767030b06666666eae403600",
    "maxConfirmations": 4
}
```

**Gongratulation, you accepted the first payment!**

### Get Currencies exchange rates

![label: GET][~get]  */rates/{fiat}/?token={token}* 

Obtain information about currencies exchange rates against fiat from markets.

**Example:** https://api.yopay.tech/rates/usd/?token=WKt2skCsqns6666666666CJv0m4DUkX

**Headers:** Content-Type: application/json

**Parameters:**
 - token - API Secret Key
 - fiat - Comma separated fiat currencies in which you obtain rates

**Response body:**

*The API returns a JSON string containing the information about selected currencies exchange rates.*

```json
{
    "success": true,
    "data": {
        "USD": {
            "eth": {
                "coinmarketcap": 0.0203020243,
                "mid": 0.0203020243
            }
        },
        "EUR": {
            "eth": {
                "coinmarketcap": 1.4806298998,
                "mid": 1.4806298998
            }
        }
    }
}
```

#### What to use as a payout address?

You will need payout addresses for all crypto currencies you want to accept. Only you will have access to your payout wallets.
You can use any online wallet, service or exchange of your choice.


### Retrieve wallet balance

![label: GET][~get]  /{crypto}/wallet/?token={token}

**Example:** https://api.yopay.tech/eth/wallet/?token=WKt2skCsqns6666666666CJv0m4DUkX

**Headers:** Content-Type: application/json

**Parameters:**
 - token - API Secret Key
 - crypto - Crypto currency to accept
 
**Response body:**
```json
{
    "success": true,
    "data": {
        "balance": "16220000000000000",
        "wallet": {
            "user": "5afe757f6666666666664d00",
            "cold_wallet": "0x2b30D2903C1366666666666666666666113E27E4",
            "api_access": true,
            "max_amount": null,
            "max_confirmations": 3,
            "min_amount": null,
            "is_active": true,
            "blockchain": "eth"
        }
    }
}
```

### Request withdrawal to cold wallet

![label: POST][~post]  */eth/withdraw/?token={token}*

![label: POST][~post]  */eth/withdraw/{callback_url}/?token={token}*

Request withdrawal to cold wallet (You configure it in your cabinet)

**Examples:** 

 - Without callback: 
 
 https://api.yopay.tech/eth/withdraw/?token=WKt2skCsqns6666666666CJv0m4DUkX
 
 - With callback: 
 
 https://api.yopay.tech/eth/withdraw/http%3A%2F%2Fexample.com%2Fcb.php/?token=WKt2skCsqns6666666666CJv0m4DUkX

**Headers:** Content-Type: application/json

**Parameters:**
 - token - API Secret Key
- callback_url - Your server callback url (urlencoded) to get information about withdrawal
 

**Request body:**
```json
{
 "address": "0xf75574f061cd66666666666666666666666666f2",
 "amount": "0.02"
}
```

**Response body:**
```json
{
    "success": true,
    "data": "Transaction sent",
    "id": "5b5666666666666666662ed21"
}
```

**Callback example:**

```json
{
  "withdrawal": {
    "txid": "0x8e81cc436666666666666666643801d8f98d4d093c1e5381c031eb55048f92ed",
    "address": "0xf75574f061cd66666666666666666666666666f2",
    "blockchain": "eth",
    "amount": "0.33",
    "callback": "http:/example.com/cb.php",
    "status": "complete",
    "created": "2018-07-23T12:35:35.394Z",
    "id": "5b55cbb7566666666689fcdb"
  }
}
```


### Retrieve withdrawal details

![label: GET][~get]  */withdraw/{withdraw}/?token={token}*

**Example:** https://api.yopay.tech/withdraw/5b13c41166666fff666666e7/?token=WKt2skCsqns6666666666CJv0m4DUkX

**Headers:** Content-Type: application/json

**Response body:**

```json
{
    "success": true,
    "data": {
        "txid": "0x8e81cc436666666666666666643801d8f98d4d093c1e5381c031eb55048f92ed",
        "address": "0xf75574f061cd66666666666666666666666666f2",
        "blockchain": "eth",
        "amount": "0.33",
        "callback": "http:/example.com/cb.php",
        "status": "complete",
        "created": "2018-07-23T12:35:35.394Z",
        "id": "5b55cbb7566666666689fcdb"
      }
    }
```


Before usage:
Open settings and populate password and upload you json file
toggle api access on


[~get]: https://img.shields.io/badge/-GET-success.svg
[~post]: https://img.shields.io/badge/-POST-blue.svg
