cd---
title: Hardtofind API Reference

language_tabs:
  - json

toc_footers:
  - <a href='https://www.hardtofind.com.au' target='_blank'>www.hardtofind.com.au</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the hardtofind.com.au API documentation!

The base URL for all API endpoints is: `https://api.hardtofind.com.au/`

A staging environment is available at: `https://staging-api.hardtofind.com.au/`. Please be aware that the database is not kept up-to-date and may be purged at any time.

# Authentication

The API uses [JSON Web Tokens](https://jwt.io/) for authentication where necessary. 
For the rest of this documentation we refer to these as JWT's. 
To obtain a JWT use the `/authenticate` endpoint.

Tokens are issued for a lifetime of 15 minutes which should be enough to undertake any action via the API. 
We may implement long lived tokens based on feedback, but as this is carries an inherent security risk, we require
re-authentication with each set of actions.

API requests expect for the API token to be included in all API requests to the server in an `X-Auth-Token` header as follows:

`X-Auth-Token: <your_jwt>`

<aside class="notice">
You must replace <code>your_jwt</code> with your personal API key.
</aside>

## POST /authenticate

To obtain a JWT you need to POST your username and password to this endpoint with the structure as outlined here.

The JWT is valid for a period of 15 minutes from the time of issue which should be sufficient to undertake any operations
using the API while minimising security risks.

Note: the username and password belong to the store owner. There is currently a limitation of one user account per store.

```json
{
  "username": "<your_username>", 
  "password": "<your_password>"
}
```
> If the supplied credentials are valid the above command returns a JSON response with the JWT as outlined below:

```json
{
  "person": {
    "id": "<your_id>",
    "role": "<your_role>"
  },
  "token": "<your_jwt>"
}
```


## GET /info

Once you have a JWT you can check its contents using the `/info` endpoint. It will return the details as outlined here:

```json
{
    "user_id": 111,
    "seller_account_id": 222,
    "created": "<created_date>",
    "expires": "<expiry_date>",
    "roles": [
        "<your_role>",
        "<your_other_role>"
    ]
}
```

# Paged data (pagination)

```json
{
    "data": [
        ...
    ],
    "page_size": 10,
    "page_number": 1,
    "total_results": 4,
    "pages": 1
}
```

Throughout the API when a large amount of data can be retrieved a standard return format is used to facilitate paging through the results.

The `data` element will contain JSON representations of the appropriate entities for your request, the other elements 
are fairly self explinatory and provide sufficient information to craft navigation links. 

To navigate through the pages, append a `page_number` query string parameter to your next `GET` request.
 
# Batch operations

For easy management of frequently changing data and to facilitate keeping your in-house systems in sync with hardtofind.com.au
a series of batch update jobs can be created. They are outlined below.

## Variant inventory updates:

```json
{  
   "action":"update",
   "entity_type": "variant",
   "entities":[  
      {  
         "sku":"<SKU-001>",
         "inventory": 199
      },
      {  
         "sku":"<SKU-002>",
         "inventory": 99
      },
      {  
         "sku":"<SKU-003>",
         "inventory": 55
      }
   ]
}
```
To update variant inventory, send a request to `POST /batch` with the JSON as outlined here within the body.

If the supplied SKU's are not unique within your catalog, they will be ignored. We recommed ensuring your SKU's are 
unique within the system in any case. This business rule is not yet enforced, but may be in future.

If the supplied SKU's do not exist within your catalog, they will be ignored.

# Order Management

<aside class="warning">
Although the API enables you to easily import Order data into your in-house system, it is still a requirement that
you include an official hardtofind packing slip within each of your shipments.

A direct download link is provided within the <code>_links.packing_slip</code> element to allow you to download the PDF packing slip directly.
</aside>

## Standard Order structure

```json
{
    "id": "123456",
    "status": "READY",
    "_links": {
        "self": "https://api.hardtofind.com.au/order/123456",
        "manage": "https://www.hardtofind.com.au/mfadmin/sales_order_part/view/123456",
        "packing_slip": "https://api.hardtofind.com.au/order/123456/packing-slip"
    },
    "dates": {
        "created_at": "2016-08-18T14:43:22+1000",
        "updated_at": "2016-08-18T14:43:22+1000",
        "dispatch_due_at": "2016-08-23T12:18:00+1000"
    },
    "payment_summary": {
        "price": 99.95,
        "shipping": 0,
        "value_addition": 0,
        "discount": 0,
        "credit": 0,
        "payment": 99.95,
        "total": 99.95
    },
    "lines": [
        {
            "description": "Product title here",
            "price": 99.95,
            "product_id": "111",
            "quantity": 1,
            "sku": "SKU-111",
            "variant_data": [
                {
                    "label": "Clothing size",
                    "value": "XL"
                }
            ],
            "personalisations": [
                {
                    "label": "First personalisation",
                    "value": "Something personal"
                },
                {
                    "label": "Second personalisation",
                    "value": "Something even more personal"
                }
            ]
        }
    ],
    "gift_wrapping": {
        "description": "Free gift wrap with personal message",
        "message": "Have a lovely time!",
        "price": 0,
        "quantity": 1
    },
    "shipping": {
        "address": {
            "first_name": "Tim",
            "last_name": "Massey",
            "line_one": "12 Address Street",
            "line_two": "",
            "suburb": "Suburbia",
            "state": "ACT",
            "postcode": "1001",
            "country": "Australia",
            "company": "Company Name"
        },
        "method": {
            "description": "Shipping - FREE standard shipping",
            "price": 0,
            "quantity": 1
        },
        "instructions": "If no-one home - please leave on front step."
    },
    "shipments": [
        {
            "shipment_date": "2016-09-08T14:57:37+1000",
            "tracking_number": "TRACK-1234",
            "shipping_provider_name": "Provider Name Here"
        }
    ]
}
```

The JSON structure to the right is a representation of an Order within the system and should provide you with all the data
necessary to fulfill the order.

## Order States

The available Order states within hardtofind.com.au are as follows:

Order State | Description
----------- | -----------
READY | The Order has been paid for fully and is awaiting fulfillment by the Seller
PROCESSED | The Order has been picked up by the Seller and is undergoing fulfillment
SHIPPED | The Order items have been fully shipped by the Seller.
RETURNED | The Order has been returned in full
REFUNDED | The Order has had the monies refunded to the Shopper
CANCELLED | The Order has been cancelled by the Customer Support team and the finances manually corrected.

## GET /orders
  
To view a list of Orders use this endpoint. It is possible to filter the list based on the following query string parameters.

By default the status of `PENDING` will be returned. Orders are ordered by last updated date (descending), most recently updated first.

Orders will be returned using the standard paged data structure as outlined [here](#paged-data-pagination).

Query parameter | Available options | Description
--------------- | ----------------- | ------------
status | One of the status listed [here](#order-states) | When supplied, only orders matching the status will be returned.
page_number | <code>[0-9]</code> | To be used for paging through large result sets

## GET /order/{id}

To retrieve a specific Order, issue a GET request to this endpoint. The return structure will match the standard Order as outlined [here](#order-management) 

## POST /order/{id}/transition

```json
{
    "action": "<action name>",
    "data":{}
}
```

To move an order through its lifecycle, create a transition using this endpoint.

Available actions are:

* process
* ship

### Process transition

```json
{
    "action": "process"
}
```

The process transition only requires an action property of process.

### Ship transition

```json
{
    "action": "ship",
    "data":{
        "tracking_number": "TRACK-123456",
        "shipping_provider_name": "Royal Mail",
        "shipment_date": "2016-09-15T14:41:25+1000"
    }
}
```

The structure of a shipment transition requires an action of ship and the data contains the shipment provider and tracking information.