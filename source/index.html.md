---
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

# POST /authenticate

To obtain a JWT you need to POST your username and password to this endpoint with the structure as outlined here.

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


# GET /info

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

# POST /batch

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