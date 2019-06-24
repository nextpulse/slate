---
title: Path. API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell


toc_footers:
  - <a href='https://path.one/api?doc=1'>Sign Up for a Developer Key</a>
  - last updated june 24 2019

search: true
--- 

# Introduction


Path.'s Universal API platform offers a consistent view of data across the crypto ecosystem. The API provides methods for retrieving data from exchanges, wallets and various data sources. Data sources are extracted with read-only mode. No write updates are performed to ensure the integrity and security of all data sources.

The base URL for all API requests is <code>https://api.path.tech</code>


<aside class="notice">We NEVER request exchange login credentials or passwords. We do not store any exchange's API secret keys.</aside>

<aside class="warning">These APIs are at ALPHA stage. It is being used in early integrations and testing. It is expected there will be  changes before our production release. We welcome any feedback or suggestions hello@path.one </aside>

<aside class="notice">This version of the API offers READ-only functionality.</aside>


## Authentication

> POST /token

> Example Request


```shell
$ curl -H "Content-Type: application/json" -X POST \
	-d '{"name":"myappname", "api_key":"00000000-0..."}' https://api.path.tech/token
```


> Example Response

```json
{
	"access_token":
		{
		  {
			"name": "myappname",
			"token": "eyJ0eXA...",
			"expires": "2018-10-05T19:54:08Z"
		  }
		}
}
```


To verify incoming API requests, Path.’s APIs use access tokens, similar to OAuth application-only authentication.

To obtain the token, you must authenticate your account using your app's name and the app's secret API key. You can manage your API keys in the dashboard. Do not share your secret API keys in publicly accessible areas such as GitHub, client-side code, and so forth. 

Authentication to the API is performed via HTTP Basic Auth. You do not need to provide a password.

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. API requests without authentication will also fail.

The token has an expiration time. Once expired, you will need to re-authenticate to obtain a new token.

## Pagination

Certain bulk requests (fetching  `transactions` or `ledgers`) may return a large number of objects, so results are paginated.  

Pagination is via the `offset` parameter.  Bulk request responses contain `total`  and `offset` fields. Manipulate the `offset` parameter to fetch all available objects. Objects are returned in reverse chronological order (most recent first).

Pagination are available for API requests that support bulk fetches. It will be listed in the API's defintion.

<aside class="notice">Once the <code>total</code> objects are known, it is advisable to use <code>GET</code> to fetch the remaining objects. This will fetch from the cache and avoid re-fetching from the data source.</aside>

Name |  | Description
--------- | ------- | -----------
limit | optional | A limit on the number of objects to be returned, between 1 and 500. (Default 500.)
offset | optional | The number of objects to skip from the first (most recent) record. (Default 0.)



## Webhooks

Many data sources (especially exchanges) have preset rate limits on how much or how often data may be retrieved. To minimize the impact of potentially hitting these limits, we intelligently allocate resources for data-intensive APIs and offer a call back parameter to inform the requesting client once the data is ready.

The parameter <code>callback_url</code> containing the destination URL to post the response data may be provided as part of the API request. The post back to the <code>callback_url</code> is 'as is' with no additional processing. So you may pass in additional application related parameters.

For example <code>callback_url: https://myapp.com/dosomething?myparam=xyz</code>

When processing the response, use [pagination](#pagination) to fetch any remaining objects.

Support for webhook callbacks will be listed in the API's definition.

## Sandbox

API requests in the sandbox may be slower than production endpoints. To help us debug and iterate, there are minimal caching and extensive debug logs in the sandbox environment.  Data may also be purged on a regular basis.  We maintain the same strong security measure for this environment. 

The base URL for all sandbox API requests is <code>https://sandbox.path.tech/</code>

## Transaction type

There are currently no standard (or consistent use) of transaction types across the different exchanges. Path. will intelligently map these (based on context) to one of the following:

Name | Description
--------- | -----------
bought | A transaction that is a buy trade.
sold | A transaction that is a sell trade.
received | The asset was received from a destination (source of ownership unknown). 
send | The asset was transferred to another destination (source of ownership unknown).
transfer | The asset was transferred to another destination owned by the same user.


# Source

<code>Source</code> objects in Path. represent the data source to extract information. These are exchanges and wallets.

<aside class="notice">Any data source keys (for example: api secret) provided are never stored in our system and it is used only during the duration of the request.</aside> 

<aside class="notice">A general rule of thumb: 
	POST requests are typically a fetch from the remote data source. GET requests return cache data from prior fetches. </aside>

## List Sources

> GET /sources

> Example Request

```shell
$ curl  -H "Authorization: mytoken"  https://api.path.tech/sources
```
> Example Response

```json
{
	"data":[
		{
			"label": "Coinbase Pro",
			"name": "coinbase_pro",
			"type": "exchange",
			"key": true,
			"oauth": false,
			"csv": false,
			"address": false
		}
		{...},
		{...}
	]	
}
```

List all currently supported data sources where Path. may extract data information. 

No authentication required for this public API.


### Source Object

Name | Description
--------- | -----------
label | The readable name of the data source.
name | The name of the data source. This is the name used to make API requests.
type | The type of data source.
key | If true, data source's API key/secret is supported for retrieving data.
oauth | If true, supports oauth token for authentication.
csv | If true, support CSV file upload and processing of data
address | If true, support fetching transactions with just the wallet address.



# User

Path. does not store any data source login credentials or exchange's API keys. So the <code>User</code> object associate data sources to a specific anonymous user. A <code>User</code> is essentially the owner of the data from the different data sources (exchanges, wallets etc) with no specific user identifying information.  All requests for transaction or ledger data will always be associated with a <code>User</code>.  

For example, you may want to fetch data for the same user from Coinbase and Binance.  For your app, you will associate your app's user with the same <code>User</code> object id. 


Data for a <code>User</code> are stored in our system for computational purposes. You may delete it at any time. One advantage of it being in our system is that we can minimize requests to external sources (for example, throttles on exchanges). It will also provide a faster response to requests, as there is minimal recalculation. 


	
## Create a User

> POST /users

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/users
```
> Example Response

```json
{
	"id": "cf64d...",
	"created": "2018-09-28T22:54:52Z"
}
```

Create a new <code>User</code> object. 



## Delete a User

> DELETE /users/{USER_ID}

> Example Request

```shell
$ curl -X DELETE -H "Authorization: mytoken"  https://api.path.tech/users/cf64d...
```
> Example Response

```json
{
	"deleted": true
}
```

Permanently deletes a <code>User</code> object and all <code>Source</code> objects related transaction data stored in our system. It cannot be undone. For performance, the deletion may not be immediate but scheduled to execute within 24 hours.



## Delete a specific User source

> DELETE /sources

> Example Request

```shell
$ curl -X DELETE -H "Authorization: mytoken"  https://api.path.tech/sources \
	-d user_id='d5d...' \
	-d name='coinbase_pro' 
```
> Example Response

```json
{
	"deleted": true
}
```

Permanently deletes a <code>Source</code> object and all related transaction data stored in our system assoicated with a <code>User</code>. It cannot be undone. 

<aside class="notice">This is a delete ONLY in our system and not data in the data source.</aside>

### Request Parameters

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources](#list-sources)).


## List all Users
> GET /users

> Example Request

```shell
$ curl -H "Authorization: mytoken"  https://api.path.tech/users
```


> Example Response

```json
{
	"name": "myappname",
	"data":[
		{
			"id": "cf64d...",
			"created": "2018-09-28T22:54:52Z"
		}
		{...},
		{...}
	]	
}
```

Return a list of all <code>User</code> objects currently associated with the app sorted by the creation date. Most recent first.


# Balances

Retrieves the current balances for a <code>Source</code>.

### Balance object

Returned as part of the response from a <code>/balances</code> request.

Name | Description
--------- | -----------
name | The label for the currency.
currency | The currency code.
balance | Amount in account/wallet.
ds_id | A data source's internal proprietary id. NOTE: In most cases, this is NOT the wallet addreess.
type | A data source's internal label. Known examples are 'vault' (Coinbase), 'trade', 'main' (Kucoin) 
updated | The date when the balance was retrieved.
usd_amount | The USD value of the balance.
usd_updated | The time of the quoted USD market price.

## Retrieve Balances

> POST /balances

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/balances \
	-d user_id='d5d...' \
	-d name='coinbase_pro' \
	--data-urlencode key='1a...' \
	--data-urlencode secret='6p...' \
	--data-urlencode extra='4f...'
```

> GET /balances   (retrieve from cache)

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/balances?user_id=d5d...&name=coinbase_pro

```

> Example Response

```json
{
	"user_id": "d5d...",
	"name": "coinbase_pro",
	"data":[
		{
			"name": "LTC",
			"currency": "ltc",
			"balance": "1.0",
			"ds_id": "aaaaa...",
			"type": "vault",
			"updated": "2018-10-03T16:59:34Z"
		}
		{...},
		{...}
	]	
}
```


### Request Parameters

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources](#list-sources)).
key | optional | The API key for the data source.
secret | optional | If provided, it will be paired with the api_key.
extra | optional | Additonal information for the data source. Dependent on the data source, examples may include an exchange's customer id.
oauth_token | optional | For data sources that support oauth (for example, Coinbase), a valid oauth <code>access_token</code> can be passed. This takes precedence over <code>key</code>. Our API does not handle the handshake process.
address | optional | The wallet address for a data source that do not require specific keys. For example, non exchange wallet sources like My Ether Wallet

### Response 

Name | Description
--------- | -----------
user_id | The ID of the <code>User</code> object.
name | The name of the data source.
data | Array of [balance objects](#balance-object)

# Transaction

The <code>Transaction</code> objects are transactions from data sources normalized to a single common format.  This enables your application to focus on just one common data structure.  If you would like more advanced data responses (mapping, relationships etc), take a look at [Ledger](#ledger)

### Transaction object

Returned as part of the response from a <code>/transactions</code> request.

Name | Description
--------- | -----------
id | The ID of the <code>Transaction</code> object.
trans_id | A data source's own specific id.
network_hash | The network hash value if applicable.
trans_type | The [transaction type](#transaction-type).  
amount | Amount of the transaction.
currency | The base currency. 
native_amount | The quote amount. 
native_currency | The quote currency. (For example the currency used to obtain the base currency in the transaction.)
quoted_unit_price | If available, the market price per unit quoted by the data source.
quoted_currency | The currency of the quoted unit price.
fee_amount | If available or applicable, fee associated with the transaction. 
fee_currency | The currency of the fee.
to_address | If available or applicable, the destination of the transaction.
to_address_source | If known, the name of the source for <code>to_address</code> (for example, 'Binance').
from_address | If available or applicable, the address of the source.
from_address_source | If known, the name of the source for <code>from_address</code>. 
trans_created | The date of the transaction.
is_fork | If the transaction was part of a fork, this will be returned and set to true.

## Retrieve Transactions


> POST /transactions

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/transactions \
	-d user_id='d5d...' \
	-d name='coinbase_pro' \
	--data-urlencode key='1a...' \
	--data-urlencode secret='6p...' \
	--data-urlencode extra='4f...' \
	-d limit=50
```

> POST /transactions/csv (upload transactions from CSV file)

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/transactions/csv \
    -d user_id='d5d...' \
    -d name='bittrex' \
    -d csvfile=bittrex.csv
```

> GET /transactions (retrieve from cache)

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/transactions?user_id=d5d...&name=coinbase_pro

```


> Example Response

```json
{
	"user_id": "d5d...",
	"name": "coinbase_pro",
	"total": 620,
	"returned": 500,
  "offset": 0,
	"data":[
		{
			"id": 12345
			"trans_id": "LNJBB3-....."
			"trans_type": "sold"
			"amount": 1.0
			"currrency": "ltc",
			"native_amount": 0.004,
			"native_currency": "btc",
      "quoted_unit_price": "6002.12",
      "quoted_currency": "usd",
			"fee_amount": 0,
			"fee_currency": "btc",
			"to_address": null,
      "to_address_source": null,
			"from_address": null,
      "from_address_source": null,
			"trans_created": "2017-10-03T16:59:34Z"
		}
		{...},
		{...}
	]	
}
```
Retrieve the transactions from <code>Source</code>. 

Depending on the data source, there are three possible ways to retrieve transaction information:

* The data source's API keys/token.
* The wallet address (for non exchange wallets).
* Information stored in the official CSV file (typically downloaded from an exchange).

This request supports [Pagination](#pagination)

This request supports [Callbacks](#webhooks)

### Request Parameters

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources](#list-sources)).
key | optional | The API key for the data source. 
secret | optional | If provided, it will be paired with the api_key.
extra | optional | Additonal information for the data source. Dependent on the data source, examples may include an exchange's customer id.
oauth_token | optional | For data sources that support oauth (for example, Coinbase), a valid oauth <code>access_token</code> can be passed. This takes precedence over <code>key</code>. Our API does not handle the handshake process.
address | optional | The wallet address for a data source that do not require specific keys. For example, non exchange wallet sources like My Ether Wallet

### Request Parameters (for CSV)

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources](#list-sources)).
csvfile | required | The CSV file generated by the data source. (For example, the transactions file downloaded from Bittrex exchange.)

### Response 

Name | Description
--------- | -----------
user_id | The ID of the <code>User</code> object.
name | The name of the data source.
total | The total number of <code>Transaction</code> objects available.
returned | The number of <code>Transaction</code> objects returned in this request.
offset | The number of objects from the first (most recent) record.
data | Array of [transaction objects](#transaction-object)

## Retrieve a single Transaction

> GET /transactions/{ID}

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/transactions/145  

```

> Example Response

```json
{
	"user_id": "d5d...",
	"name": "coinbase_pro",
	"data":[
		{
			"id": 145
			"trans_id": "LNJHH3-....."
			"trans_type": "bought"
			"amount": 1.0
			"currrency": "eth",
			"native_amount": 210.00,
			"native_currency": "usd",
      "quoted_unit_price": "210.00",
      "quoted_currency": "usd",
			"fee_amount": null,
			"fee_currency": null,
			"to_address": null,
      "to_address_source": null,
			"from_address": null,
      "from_address_source": null,
			"trans_created": "2018-10-04T17:59:34Z"
		}
	]	
}
```
Retrieve a single <code>Transaction</code> object from previously fetched data. 

### Request Parameters

Name |  | Description
--------- | ------- | -----------
id | required | The ID of the <code>Transaction</code> object. 


# Ledger

This is where the magic happens. <code>Ledger</code> is a transaction object that contains additional information from our smart algorithms. These include mapping of transfer paths, consolidations, and many others to provide a full 360 view.

## Ledger object

Returned as part of the response from a <code>/ledgers</code> request.

Name | Description
--------- | -----------
id | The ID of the <code>Ledger</code> object. As new transactions or data sources are added/deleted, the Path. system will recalculate the mapping relationships. <code>Pledger</code> objects (and hence the ids) are created/deleted accordingly.
transaction_id | The ID of the <code>Transaction</code> object.
network_hash | The network hash value if applicable.
trans_id | Data source's own specific id.
trans_type | The [transaction type](#transaction-type).  
amount | Amount of the transaction.
credit | If true, the transaction is a credit. If false, the transaction is a debit.
currency | The base currency. 
native_amount | The quote amount. 
native_currency | The quote currency. (For example the currency used to obtain the base currency in the transaction.)
fee_amount | If available or applicable, fee associated with the transaction. 
fee_currency | The currency of the fee.
description | Deprecated.
usd_unit_price | The USD price for a single unit of the base currency. (If <code>usd_quoted_at</code> is null, then the value came from the data source.)
usd_quoted_at | If non null, the time of the market price quoted for <code>usd_unit_price</code>.
exchange_transfer | If true, the transaction was a transfer between two exchanges.
internal | If true, the transaction was a movement of assets within an exchange. For example, a transfer from a vault to a wallet.
parent_id | The ID of the parent <code>Ledger</code> object.
child_id | The ID of the child <code>Ledger</code> object.
to_address | If available or applicable, the destination of the transaction.
to_address_source | If known, the name of the source for <code>to_address</code> (for example, 'Binance').
from_address | If available or applicable, the address of the source.
from_address_source | If known, the name of the source for <code>from_address</code>. 
trans_created | The date of the transaction.
consolidations | If set, list of [transaction objects](#transaction-object) that make up this transaction.
trade_pair | If set, for non-fiat, shows the break down of the [trade pair](#tradepair-object) that make up the core transaction.
ds_id | If set, data source specific wallet id. This is not the wallet address, but additional id information that a data soruce may expose for a wallet.
is_fork | If the transaction was part of a fork, this will be returned and set to true.

## TradePair object

Additional information of a trade pair transaction returned from a <code>/ledgers</code> request.


Name | Description
--------- | -----------
currency | The base currency. 
amount | Amount of the transaction.
trans_type | The [transaction type](#transaction-type). 
usd_unit_price | The USD price for a single unit of the base currency. (If <code>usd_quoted_at</code> is null, then the value came from the data source.)
usd_quoted_at | If non null, the time of the market price quoted for <code>usd_unit_price</code>.
## Retrieve Ledgers

> POST /ledgers

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/ledgers \
	-d user_id='d5d...' \
	-d name='kraken' \
	--data-urlencode key='1a...' \
	--data-urlencode secret='6p...' \
	-d limit=50
```

> POST /ledgers/csv (upload transactions from CSV file)

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/ledgers/csv \
    -d user_id='d5d...' \
    -d name='bittrex' \
    -d csvfile=bittrex.csv
```

> GET /ledgers   (retrieve from cache)

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/ledgers?user_id=d5d...&name=kraken

```

> Example Response

```json
{
	"user_id": "d5d...",
	"name": "kraken",
	"total": 240,
	"returned": 240,
	"offset": 0,
	"data":[
		{
			"id": 12345
			"trans_id": "LADBB3-....."
			"trans_type": "sold"
			"amount": 0.036		
			"credit": false,	
			"currrency": "btc",
			"native_amount": 1980.0,
			"native_currency": "xlm",
			"fee_amount": 0.0001,
			"fee_currency": "btc",
			"to_address": null,
      "to_address_source": null,
			"from_address": null,
      "from_address_source": null,
			"trans_created": "2015-01-15T01:12:00Z",
			"usd_unit_price": null,
			"exchange_transfer": false,
			"parent_id": null, 
			"child_id": null,
			"internal": false,
			"trade_pair": [
				{
					"currency": "xlm",
					"amount": 1980.0,
					"usd_unit_price": 0.001,
					"trans_type": "bought"
				},
				{
					"currency": "btc",
					"amount": 0.036,
					"usd_unit_price": 600.00,
					"trans_type": "sold"
				}					
			] 
		}
		{...},
		{...}
	]	
}
```
Retrieves <code>Ledger</code> objects for a <code>User</code>. 

Depending on the data source, there are three possible ways to retrieve transaction information:

* The data source's API keys/token.
* The wallet address (for non exchange wallets).
* Information stored in the official CSV file (typically downloaded from an exchange).


This request supports [Pagination](#pagination)

This request supports [Callbacks](#webhooks)

### Request Parameters

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources](#list-sources)).
key | optional | The API key for the data source.
secret | optional | If provided, it will be paired with the api_key.
extra | optional | Additonal information for the data source. Dependent on the data source, examples may include an exchange's customer id.
oauth_token | optional | For data sources that support oauth (for example, Coinbase), a valid oauth <code>access_token</code> can be passed. This takes precedence over <code>key</code>. Our API does not handle the handshake process.
address | optional | The wallet address for a data source that do not require specific keys. For example, non exchange wallet sources like My Ether Wallet

### Request Parameters (for CSV)

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources](#list-sources)).
csvfile | required | The CSV file generated by the data source. (For example, the transactions file downloaded from Bittrex exchange.)

### Response 

Name | Description
--------- | -----------
user_id | The ID of the <code>User</code> object.
name | The name of the data source.
total | The total number of <code>Ledger</code> objects available.
returned | The number of <code>Ledger</code> objects returned in this request.
offset | The number of objects from the first (most recent) record.
data | Array of [ledger objects](#ledger-object)

## Retrieve a single Ledger

> GET /ledgers/{ID}

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/ledgers/12345 

```

> Example Response

```json
{
	"user_id": "d5d...",
	"name": "kraken",
	"data":[
		{
			"id": 12345
			"trans_id": "LADBB3-....."
			"trans_type": "sold"
			"amount": 0.036		
			"credit": false,	
			"currrency": "btc",
			"native_amount": 1980.0,
			"native_currency": "xlm",
			"fee_amount": 0.0001,
			"fee_currency": "btc",
			"to_address": null,
      "to_address_source": null,
			"from_address": null,
      "from_address_source": null,
			"trans_created": "2015-01-15T01:12:00Z"
			"usd_unit_price": null,
			"exchange_transfer": false,
			"parent_id": null, 
			"child_id": null,
			"internal": false,
			"trade_pair": [
				{
					"currency": "xlm",
					"amount": 1980.0,
					"usd_unit_price": 0.001,
					"trans_type": "bought"
				},
				{
					"currency": "btc",
					"amount": 0.036,
					"usd_unit_price": 600.00,
					"trans_type": "sold"
				}					
			] 
		}
	]	
}
```
Retrieve a single <code>Ledger</code> object from previously fetched data. 

### Request Parameters

Name |  | Description
--------- | ------- | -----------
id | required | The ID of the <code>Ledger</code> object.



# Errors

Path. uses conventional HTTP response codes to indicate the success or failure of an API request. Additionally, a description field may also be set in the json response. This is typically the raw response from the external data source. 

HTTP status code summary

Code| Meaning
-------------- | -------------- 
200 - OK | Everything worked as expected. 
400 - Bad Request | The request was unacceptable, often due to missing or invalid required parameter.
401 - Unauthorized | No valid API key or Token provided. 

> Example Response

```json
{
	"error": "Request to Coinbase Pro failed",
	"description": "{\"message\":\"Invalid API Key\"}"
}
```

# Libraries

Ruby Gem (coming soon)

PHP (coming soon)

Want to help? hello@path.one



# Performance Considerations 

For remote data source requests (for example, exchanges), the speed of the response is very much dependent on the data source. Typically, a GET request is more efficient than a POST request (that will fetch from a remote target).


## Exchange Call Limits

Many exchanges set a limit on the number of requests made within a certain timeframe. 

Where possible, our system will only fetch new data that was not previously recorded.  This ensures we minimize any impact of the call limits that may exist on the data source.

# Known Issues

CSV and Bitfinex support are currently experimental.

Please check our [blog post](https://blog.path.one/known-issues-and-challenges-with-exchanges-api/) for other current known issues.

# Roadmap

- Additional data sources


[Path. Tax](https://path.one/tax)  is our reference application built on top of the Path. API platform. The architecture decouples the business logic from the data layer. Information like gain and loss are not offered as part of the base API platform. If you are interested in learning more please contact us directly at hello@path.one



