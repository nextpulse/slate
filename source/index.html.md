---
title: Path. API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell


toc_footers:
  - <a href='https://path.tech?doc=1'>Sign Up for a Developer Key</a>
  - last updated jan 17 2019

search: true
---

# Introduction


Path.'s Universal API platform offers a consistent view of data across the crypto ecosystem. The API provides methods for retrieving data from exchanges, wallets and various data sources. Data sources are extracted with read-only mode. No write updates are performed to ensure the integrity and security of all data sources.

The base URL for all API requests is <code>https://api.path.tech/v1</code>


<aside class="notice">We NEVER request exchange login credentials or passwords. We do not store any exchange's API secret keys.</aside>

<aside class="warning">These APIs are at ALPHA stage. It is being used in early integrations and testing. It is expected there will be  changes before our production release. We welcome any feedback or suggestions hello@path.tech </aside>

<aside class="notice">This version of the API offers READ-only functionality.</aside>


## Authentication

> POST /token

> Example Request


```shell
$ curl -H "Content-Type: application/json" -X POST \
	-d '{"name":"myappname", "api_key":"00000000-0..."}' https://api.path.tech/v1/token
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

Path. utilizes cursor-based pagination via the `starting_after` and `ending_before` parameters. Both parameters take an existing object ID value and return objects in reverse chronological order (most recent first). The `ending_before` parameter returns objects listed before the named object. The `starting_after` parameter returns objects listed after the named object. If both parameters are provided, only `ending_before` is used.

Pagination are available for API requests that support bulk fetches. It will be listed in the API's defintion.


Name |  | Description
--------- | ------- | -----------
limit | optional | A limit on the number of objects to be returned, between 1 and 500. (Default 500.)
starting_after | optional | A cursor for use in pagination. `starting_after` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with `obj_foo`, your subsequent call can include `starting_after=obj_foo` in order to fetch the next page of the list.
ending_before| optional | A cursor for use in pagination. `ending_before` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with `obj_bar`, your subsequent call can include `ending_before=obj_bar` in order to fetch the previous page of the list.




## Sandbox

API requests in the sandbox may be slower than production endpoints. To help us debug and iterate, there are minimal caching and extensive debug logs in the sandbox environment.  Data may also be purged on a regular basis.  We maintain the same strong security measure for this environment. 

The base URL for all sanbox API requests is <code>https://sandbox.path.tech/</code>


# User

Path. does not store any data source login credentials or exchange's API keys. So the <code>User</code> object associate data sources to a specific anonymous user. A <code>User</code> is essentially the owner of the data from the different data sources (exchanges, wallets etc) with no specific user identifying information.  All requests for transaction or ledger data will always be associated with a <code>User</code>.  

For example, you may want to fetch data for the same user from Coinbase and Binance.  For your app, you will associate your app's user with the same <code>User</code> object id. 


Data for a <code>User</code> are stored in our system for computational purposes. You may delete it at any time. One advantage of it being in our system is that we can minimize requests to external sources (for example, throttles on exchanges). It will also provide a faster response to requests, as there is minimal recalculation. 


	
## Create a User

> POST /users

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/v1/users
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
$ curl -X DELETE -H "Authorization: mytoken"  https://api.path.tech/v1/users/cf64d...
```
> Example Response

```json
{
	"deleted": true
}
```

Permanently deletes a <code>User</code> object and all related transaction data stored in our system. It cannot be undone. For performance, the deletion may not be immediate but scheduled to execute within 24 hours.



## List all Users
> GET /users

> Example Request

```shell
$ curl -H "Authorization: mytoken"  https://api.path.tech/v1/users
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




 

# Source

<code>Source</code> objects in Path. represent the source to extract information.

<aside class="notice">Any data source keys (for example: api secret) provided are never stored in our system and it is used only during the duration of the request.</aside> 

<aside class="notice">A general rule of thumb: 
	POST requests are typically a fetch from the remote data source. GET requests return cache data from prior fetches. </aside>

## List Sources

> GET /sources/all

> Example Request

```shell
$ curl  -H "Authorization: mytoken"  https://api.path.tech/v1/sources/all
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
			"csv": false
		}
		{...},
		{...}
	]	
}
```

List all currently supported data sources where Path. may extract data information.

### Source Object

Name | Description
--------- | -----------
label | The readable name of the data source.
name | The name of the data source. This is the name used to make API requests.
type | The type of data source.
key | If true, data source's API key/secret is supported for retrieving data.
oauth | If true, supports oauth token for authentication.
csv | If true, support CSV file upload and processing of data


## Retrieve balances

Retrieves the current balances for a <code>Source</code>.

> POST /sources

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/v1/sources \
	-d user_id='d5d...' \
	-d name='coinbase_pro' \
	--data-urlencode key='1a...' \
	--data-urlencode secret='6p...' \
	--data-urlencode extra='4f...'
```

> GET /sources/{USER_ID}/{NAME}   (retrieve from cache)

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/v1/sources/d5d.../coinbase_pro

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
			"vault": false,
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
name | required | The name of the data source (obtained via [/sources/all](#list-sources)).
key | optional | Must be provided if <code>oauth_token</code> is missing. 
secret | optional | If provided, it will be paired with the api_key.
extra | optional | Additonal information for the data source. Dependent on the data source, examples may include an exchange's customer id.
oauth_token | optional | For data sources that support oauth (for example, Coinbase), a valid oauth <code>access_token</code> can be passed. This takes precedence over <code>key</code>. Our API does not handle the handshake process.

## Delete a Source

> DELETE /sources

> Example Request

```shell
$ curl -X DELETE -H "Authorization: mytoken"  https://api.path.tech/v1/sources \
	-d user_id='d5d...' \
	-d name='coinbase_pro' 
```
> Example Response

```json
{
	"deleted": true
}
```

Permanently deletes a <code>Source</code> object and all related transaction data stored in our system. It cannot be undone. 

<aside class="notice">This is a delete ONLY in our system and not data in the data source.</aside>
	
# Transaction

The <code>Transaction</code> objects are transactions from data sources normalized to a single common format.  This enables your application to focus on just one common data structure.  If you would like more advanced data responses (mapping, relationships etc), take a look at [Ledger](#ledger)

### Transaction object

Returned as part of the response from a <code>/transactions</code> request.

Name | Description
--------- | -----------
id | The ID of the <code>Transaction</code> object.
trans_id | A data source's own specific id.
trans_type | The [transaction type](#transaction-type).  
amount | Amount of the transaction.
currency | The base currency. 
native_amount | The quote amount. 
native_currency | The quote currency. (For example the currency used to obtain the base currency in the transaction.)
fee_amount | If available or applicable, fee associated with the transaction. 
fee_currency | The currency of the fee.
to_address | If available or applicable, the destination of the transaction.
from_address | If available or applicable, the address of the source.
trans_created | The date of the transaction.

### Transaction type

There are currently no standard (or consistent use) of transaction types across the different exchanges. Path. will intelligently map these (based on context) to one of the following:

Name | Description
--------- | -----------
bought | A transaction that is a buy trade.
sold | A transaction that is a sell trade.
received | The asset was received from a destination (source of ownership unknown). 
send | The asset was transferred to another destination (source of ownership unknown).
transfer | The asset was transferred to another destination owned by the same user.

## Retrieve Transactions


> POST /transactions

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/v1/transactions \
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
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/v1/transactions/csv \
    -d user_id='d5d...' \
    -d name='bittrex' \
    -d csvfile=bittrex.csv
```

> GET /transactions/{USER_ID}/{NAME} (retrieve from cache)

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/v1/transactions/d5d.../coinbase_pro

```


> Example Response

```json
{
	"user_id": "d5d...",
	"name": "coinbase_pro",
	"data":[
		{
			"id": 12345
			"trans_id": "LNJBB3-....."
			"trans_type": "sold"
			"amount": 1.0
			"currrency": "ltc",
			"native_amount": 0.004,
			"native_currency": "btc",
			"fee_amount": 0,
			"fee_currency": "btc",
			"to_address": null,
			"from_address": null,
			"trans_created": "2017-10-03T16:59:34Z"
		}
		{...},
		{...}
	]	
}
```
Retrieves the transactions for a <code>Source</code>. Depending on the data source, there are two possible ways to retrieve transaction information. Using the data source's API keys/token or via information stored in the official CSV file.

This request supports [Pagination](#pagination)

### Request Parameters

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources/all](#list-sources)).
key | optional | Must be provided if <code>oauth_token</code> is missing. 
secret | optional | If provided, it will be paired with the api_key.
extra | optional | Additonal information for the data source. Dependent on the data source, examples may include an exchange's customer id.
oauth_token | optional | For data sources that support oauth (for example, Coinbase), a valid oauth <code>access_token</code> can be passed. This takes precedence over <code>key</code>. Our API does not handle the handshake process.

### Request Parameters (for CSV)

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources/all](#list-sources)).
csvfile | required | The CSV file generated by the data source. (For example, the transactions file downloaded from Bittrex exchange.)

### Response 

Name | Description
--------- | -----------
user_id | The ID of the <code>User</code> object.
name | The name of the data source.
data | Array of [transaction objects](#transaction-object)

## Retrieve a single Transaction

Retrieve a single <code>Transaction</code> object from previously fetched data. 


> GET /transactions/{ID}

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/v1/transactions/145  

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
			"fee_amount": null,
			"fee_currency": null,
			"to_address": null,
			"from_address": null,
			"trans_created": "2018-10-04T17:59:34Z"
		}
	]	
}
```

# Ledger

This is where the magic happens. <code>Ledger</code> is a transaction object that contains additional information from our smart algorithms. These include mapping of transfer paths, consolidations, and many others to provide a full 360 view.

## Ledger object

Returned as part of the response from a <code>/ledgers</code> request.

Name | Description
--------- | -----------
id | The ID of the <code>Ledger</code> object. 
trans_id | Data source's own specific id.
trans_type | The [transaction type](#transaction-type).  
amount | Amount of the transaction.
credit | If true, the transaction is a credit. If false, the transaction is a debit.
currency | The base currency. 
native_amount | The quote amount. 
native_currency | The quote currency. (For example the currency used to obtain the base currency in the transaction.)
fee_amount | If available or applicable, fee associated with the transaction. 
fee_currency | The currency of the fee.
description | Additional information to describe the transaction.
usd_unit_price | The USD price for a single unit of the base currency. (A closing price on the date of the transaction will be used if none is available in the original transaction.)
exchange_transfer | If true, the transaction was a transfer between two exchanges.
internal | If true, the transaction was a movement of assets within an exchange. For example, a transfer from a vault to a wallet.
parent_id | The ID of the parent <code>Ledger</code> object. 
child_id | The ID of the child <code>Ledger</code> object.
to_address | If available or applicable, the destination of the transaction.
from_address | If available or applicable, the address of the source.
trans_created | The date of the transaction.
consolidations | If set, list of [transaction objects](#transaction-object) that make up this transaction.
trade_pair | If set, for non-fiat, shows the break down of the [trade pair](#tradepair-object) that make up the core transaction.

## TradePair object

Additional information of a trade pair transaction returned from a <code>/ledgers</code> request.


Name | Description
--------- | -----------
currency | The base currency. 
amount | Amount of the transaction.
trans_type | The [transaction type](#transaction-type). 
usd_unit_price | The USD price for a single unit of the currency. (A closing price on the date of the transaction will be used if none is available in the original transaction.)
## Retrieve Ledgers



> POST /ledgers

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/v1/ledgers \
	-d user_id='d5d...' \
	-d name='kraken' \
	--data-urlencode key='1a...' \
	--data-urlencode secret='6p...' \
	-d limit=50
```

> POST /ledgers/csv (upload transactions from CSV file)

> Example Request

```shell
$ curl -X POST -H "Authorization: mytoken"  https://api.path.tech/v1/ledgers/csv \
    -d user_id='d5d...' \
    -d name='bittrex' \
    -d csvfile=bittrex.csv
```

> GET /ledgers/{USER_ID}/{NAME}   (retrieve from cache)

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/v1/ledgers/d5d.../kraken

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
			"from_address": null,
			"trans_created": "2015-01-15T01:12:00Z"
			"description": "sold BTC and received XLM",
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

Retrieves <code>Ledger</code> objects for a <code>User</code>. Depending on the data source, there are two possible ways to retrieve transaction information. Using the data source's API keys/token or via information stored in the official CSV file.

This request supports [Pagination](#pagination)


### Request Parameters

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources/all](#list-sources)).
key | optional | Must be provided if <code>oauth_token</code> is missing.
secret | optional | If provided, it will be paired with the api_key.
extra | optional | Additonal information for the data source. Dependent on the data source, examples may include an exchange's customer id.
oauth_token | optional | For data sources that support oauth (for example, Coinbase), a valid oauth <code>access_token</code> can be passed. This takes precedence over <code>key</code>. Our API does not handle the handshake process.

### Request Parameters (for CSV)

Name |  | Description
--------- | ------- | -----------
user_id | required | The ID of the <code>User</code> object.
name | required | The name of the data source (obtained via [/sources/all](#list-sources)).
csvfile | required | The CSV file generated by the data source. (For example, the transactions file downloaded from Bittrex exchange.)

### Response 

Name | Description
--------- | -----------
user_id | The ID of the <code>User</code> object.
name | The name of the data source.
data | Array of [ledger objects](#ledger-object)

## Retrieve a single Ledger


Retrieve a single <code>Ledger</code> object from previously fetched data. 


> GET /ledgers/{ID}

> Example Request

```shell
$ curl -G -H "Authorization: mytoken"  https://api.path.tech/v1/ledgers/12345 

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
			"from_address": null,
			"trans_created": "2015-01-15T01:12:00Z"
			"description": "sold BTC and received XLM",
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


## Retrieve Ledger path

Retrieve the full parent/child mapping for a <code>Ledger</code>. Is this useful? Let us know hello@path.tech


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

Want to help? hello@path.tech



# Performance Considerations 

For remote data source requests (for example, exchanges), the speed of the response is very much dependent on the data source. Typically, a GET request is more efficient than a POST request (that will fetch from a remote target).


<aside class="notice">We plan to offer async requests in the next major release.</aside> 



## Exchange Call Limits

Many exchanges set a limit on the number of requests made within a certain timeframe. 

Where possible, our system will only fetch new data that was not previously recorded.  This ensures we minimize any impact of the call limits that may exist on the data source.

# Known Issues

Please check our [blog post](https://blog.path.tech/known-issues-and-challenges-with-exchanges-api/) for current known issues.

# Roadmap

- Additional data sources

- Async/callback requests


- API to retrieve the spot price for an asset or a trade pair. Is this useful? Let us know hello@path.tech


[Path.Tax](https://path.tax)  is our reference application built on top of the Path. API platform. The architecture decouples the business logic from the data layer. So information like gain and loss are not offered as part of the base API platform. If you are interested in learning more please contact us directly at hello@path.tech



