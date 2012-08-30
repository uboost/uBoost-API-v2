uBoost API (v2)
======================
*Updated: 2012-08-24*

--------------------------------------------------------------------------------

Welcome Developers
------------------
### Introduction

This guide is intended for developers who need to communicate remotely with the uBoost platform via web services. It assumes basic knowledge of network terminology, programming, and RESTful interfaces. By consuming web services through this API (application programming interface), you can automatically create accounts, award points, single-sign-on (SSO) your student from your LMS to uBoost, manage student groups, and award custom badges.


### General Description

uBoost web services are designed around [RESTful principles](http://en.wikipedia.org/wiki/Representational_State_Transfer). In order to access these web services, you will need login credentials for HTTP Basic Authentication and the ability to communicate over HTTPS/TLS. uBoost only accepts web service calls encrypted over the HTTPS protocol. 

All responses from the API are in [JSON](http://en.wikipedia.org/wiki/Json) format. The original uBoost API returned XML, but for version 2 support has been dropped in favor of JSON. For POST and PUT requests, both JSON and form-encoded formats are supported. JSONP (JSON with padding) is supported using the `callback` parameter for certain APIs that require cross domain calls, 

HTTP status codes are used to indicate success/failure. 200 indicates successful viewing, updating, or deleting of data. 201 indicates the successful creation of a new resource. Currently used failure codes include: 401 indicates that authorization is needed, 404 indicates the resource requested was not found, 422 indicates a failure to create or update the resource with the parameters given. Human-readable error information will be returned in the `message` key of the JSON response, which is useful for debugging.

The programming examples are given as [cURL](http://en.wikipedia.org/wiki/CURL) commands, which are useful for testing, experimenting, and conceptualizing. However, in actual development you should use your favorite RESTful web service package for your particular development platform. These packages provide higher-level calls that effectively generate the same curl commands listed in this guide.

Capitalized placeholders in the examples, such as `API_CREDENTIALS` and `SUBDOMAIN`, are meant to be replaced with the actual credentials and values that are supplied to you by uBoost.

--------------------------------------------------------------------------------

* [Accounts](#accounts)
  * [Create](#accounts-create)
  * [Select](#accounts-select)
  * [Update](#accounts-update)
  * [Remove](#accounts-remove)
  * [Find](#accounts-find)
  * [Get Single Sign On (SSO) Token for an Account](#accounts-sso)
* [Points](#points)
  * [List Points Transactions Belonging to an Account](#points-list)
  * [Add Points to Account](#points-add)
* [Groups](#groups)
  * [List](#groups-list)
  * [Select](#groups-select)
  * [Create](#groups-create)
  * [Update](#groups-update)
  * [Destroy](#groups-destroy)
* [Badges](#badges)
  * [Create](#badges-create)
  * [Unaward](#badges-unaward)
* [Widgets](#widgets)
  * [Profile](#widgets-profile)
  * [Badge Categories](#widgets-badge-categories)
  * [My Badges](#widgets-my-badges)
  * [Unearned Badges](#widgets-unearned-badges)
  * [List of Leaderboards](#widgets-leaderboards)
  * [Leaderboard](#widgets-leaderboard)

--------------------------------------------------------------------------------

### Typical Development Process

uBoost will create your site in our demonstration environment, and issue you your uBoost web service login credentials. Upon receiving your credentials, you may start your integration effort. In the meantime, we will also be asking you to provide optional custom design elements you may wish to supply, as well as your corporate logo and colors. During development, there are usually 4 touch points between your system and uBoost:

* Automatically create users in uBoost when an account is created on your site.
* Have a link(s) for users to navigate to uBoost from your site via SSO.
* Display user's current point balance at select areas on your site.
* Automatically issue points to users in uBoost when they complete activities you wish to reward them for.

Setting up and creating users under applicable groups or tags is highly recommended. Optionally, if you have custom badges setup with uBoost you will need to capture the events in your system that lead to badge award and trigger the award itself in uBoost via web services.

We will be creating your site in our production environment as your roll-out date nears. After your integration and QA is completed, final deployment may need to be coordinated with uBoost if you have existing users and point balances that need to be imported. During actual deployment, your site will need to be temporarily brought down to freeze any new account creation or point-earning activity while initial users are being populated in uBoost. Alternatively, you may handle this with one-time scripts against uBoost web services to account for deltas in accounts and points after initial cut-over. From start to finish, the entire development process can take anywhere from 2-8 weeks.


API Version
------------------

### Specifying a Version to Use

```
Accept: application/vnd.uboost.api.v2
```

To request for a specific version of the API, send the version number within the `Accept` header. Currently supported version is `v2`, which is the default if not specified.

### What's New in Version 2.0

* Consolidated under /api
* All response data in JSON format
* XML no longer supported
* POST and PUT request parameters can be in either JSON or form-encoded formats
* Accounts API
  * New `tag_list` parameter for tag-based sites
  * `group_id` and `group_name` parameters are valid only for group-based sites
* Groups API
  * Not available for tag-based sites
* Points Transactions API
  * Changed parameter name from `transaction_description` to `description`
  * Removed `transaction_type`, because it is automatically set to `Direct Deposit` for all points created via the API
  * Added the `limit`(max 50) and `offset` parameters to the list method
  * Additional attributes are returned in the JSON data (XP and transaction type)
* Badges API
  * JSONP support for cross domain API calls
  * Initially based on uBar API


--------------------------------------------------------------------------------

Accounts API
------------------

### Create

```
POST https://SUBDOMAIN.uboost.com/api/accounts
```

cURL example - POST request in JSON format

```
curl -X POST -i \
-H 'Content-Type: application/json' \
-d '{ "account" : { "user_name" : "isaacnewtonx", "password" : "Gr4v1Ty!" } }' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts


HTTP/1.1 201 Created
Location: https://SUBDOMAIN.uboost.com/api/accounts/921785562
Content-Type: application/json; charset=utf-8
Content-Length: 626

{
  "student": {
    "id":921785562,
    "user_name":"isaacnewtonx",
    "password":"Gr4v1Ty!",
    ...
  }
}
```

cURL example - POST request in form-encoded format

```
curl -X POST -i \
-d account[user_name]="isaacnewton" \
-d account[password]="App73CORE" \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts


HTTP/1.1 201 Created 
Location: https://SUBDOMAIN.uboost.com/api/accounts/921785565
Content-Type: application/json; charset=utf-8
Content-Length: 627

{
  "student": {
    "id":921785565,
    "user_name":"isaacnewton",
    "password":"App73CORE",
    ...
  }
}
```

**Required Parameters**

* account[user_name]


**Optional Parameters**

*  account[password] # not required if single sign on (use sign_in_user method call)
*  account[first_name] # highly recommended for reporting purposes
*  account[middle_name]
*  account[last_name] # highly recommended for reporting purposes
*  account[birth_date]
*  account[email1]
*  account[address_line1]
*  account[address_line2]
*  account[postal_code]
*  account[city]
*  account[state]
*  account[district]
*  account[school]
*  account[parent_name]
*  account[parent_email]
*  account[external_id] # store account id from your system here. Useful for ID mapping cross-check as needed.
*  account[licensed] # true or false. True by default. False makes this a trial account and user will only be able to browse and not redeem.
*  account[verified] # indicates that the account email should be considered verified by the account
*  account[active] # indicates that the account can be logged in to. True by default.


**Group or Tags**

The new tagging feature allows you to add free-form tags to accounts. The tags can be used to create custom leaderboards and reports. The tagging feature will eventually be added to other parts of uBoost as well, such as filtering badges by tags. We encourage all new implementations to utilize tagging, as it provides a flexible and easy way to add metadata to your accounts.

Sites are setup to use either tags or groups, but not both. If tagging is enabled, please use the `tag_list` parameter.

*  account[tag_list] # comma deliminated list of tags, for example `"state-California, city-San Francisco, program-Algebra"`

If groups are enabled, then `group_id` and `group_name` are valid parameters. If no `group_name` is given, an account is created that does not belong to any group. If no group can be found for the given `group_name`, then a 422 is returned.

*  account[group_id] -OR- account[group_name]

**Return**

* 201: account created, JSON data. The URI to the newly created resource is returned in the location header.
* 422: invalid parameter name or value. Human readable errors are returned in the `message` key.

**Note:**

If the `password` parameter is not sent in the POST request, then a password will automatically be generated.



### Select

```
GET https://SUBDOMAIN.uboost.com/api/accounts/ACCOUNT_ID
```

cURL example - GET request

```
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/921785565

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 616

{
  "student":{
    "id":921785565,
    "user_name":"isaacnewton",
    "password":"App73CORE",
    ...
  }
}
```

**Required Parameters**

* ACCOUNT_ID (in URI path - replace with actual resource ID)


**Return**

* 200: account as JSON data
* 404: account not found. Human readable errors are returned in the `message` key.


### Update

```
PUT https://SUBDOMAIN.uboost.com/api/accounts/ACCOUNT_ID
```

cURL example - PUT request in JSON format

```
curl -X PUT -i \
-H 'Content-Type: application/json' \
-d '{ "account" : { "first_name" : "Isaac", "last_name" : "Newton" } }' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/921785565


HTTP/1.1 200 OK 
Content-Type: application/json; charset=utf-8
Content-Length: 2

{}
```

**Required Parameters**

* ACCOUNT_ID (in URI path - replace with actual resource ID)

**Optional Parameters**

* account[user_name]
* account[password]
* account[active]
* account[first_name]
* account[middle_name]
* account[last_name]
* account[birth_date]
* account[email1]
* account[address_line1]
* account[address_line2]
* account[postal_code]
* account[city]
* account[state]
* account[district]
* account[school]
* account[licensed] # setting to true turns off the trial period if this account was created with account[licensed]=false

**Return**

* 200: OK
* 422: invalid parameter name or value. Human readable errors are returned in the `message` key.


### Remove

```
DELETE https://SUBDOMAIN.uboost.com/api/accounts/ACCOUNT_ID
```

cURL example - DELETE request

```
curl -X DELETE -i \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/921785565


HTTP/1.1 200 OK 
Content-Type: application/json; charset=utf-8
Content-Length: 2

{}
```

**Required Parameters**

* ACCOUNT_ID (in URI path - replace with actual resource ID)


**Return**

* 200: OK
* 404: account not found. Human readable errors are returned in the `message` key.

### Find

```
GET https://SUBDOMAIN.uboost.com/api/accounts/find
```

cURL example - GET request

```
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/find?user_name=isaacnewton
-OR-
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/find?external_id=12345

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 616

{
  "student":{
    "id":921785565,
    "user_name":"isaacnewton",
    "password":"App73CORE",
    ...
  }
}
```

**Required Parameters**

* user_name  -OR-
* external_id


**Return**

* 200: account as JSON data
* 404: account not found. Human readable errors are returned in the `message` key.


**Note:**

Find will only look for active accounts. If the account is not active, use the Select method.


### Get Single Sign On (SSO) Token for an Account

```
GET https://SUBDOMAIN.uboost.com/api/accounts/ACCOUNT_ID/sign_in_user
```

cURL example - GET request

```
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/921785565/sign_in_user

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 134

{
  "student":{
    "id":921785565,
    "sso_token":"d33cf92f65f23e6b9b11a162dad407ea038f994e",
    "sso_token_expires_at":"2012/08/25 00:35:55 +0000"
  }
}
```

**Required Parameters**

* ACCOUNT_ID (in URI path - replace with actual resource ID)


**Return**

* 200: OK
* 404: account not found. Human readable errors are returned in the `message` key.


**Notes:**

The SSO token has a 3 minute TTL and can be used only once.

When your user clicks a link within your LMS to go to rewards, make a web service call to this method, and add the `?sso_token=SSO_TOKEN` parameter to the rewards URL before redirecting. The user will arrive at the rewards site and be automatically logged in.

```
Redirect user to
http://SUBDOMAIN.uboost.com?sso_token=SSO_TOKEN
```

--------------------------------------------------------------------------------

Points API
------------------

### List Points Transactions Belonging to an Account

```
GET https://SUBDOMAIN.uboost.com/api/accounts/ACCOUNT_ID/points_transactions
```

-OR-

```
GET https://SUBDOMAIN.uboost.com/api/points_transactions?account_id=ACCOUNT_ID
```

cURL example - GET request

```
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/accounts/921785565/points_transactions

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 7901

{
  "points_transactions":[
    {
      "account_id":921785565,
      "created_at":"2012/08/22 10:14:50 -1000",
      "description":"Test from the new API",
      "id":1068511516,
      "points_change":50,
      "source_account_id":921542448,
      "updated_at":"2012/08/22 10:14:50 -1000",
      "transaction_time":"2012/08/22 10:14:50 -1000",
      "webservice":true,
      "points_transaction_type":{
        "id":52,
        "name":"Direct Deposit"
      }
    },
    ...
  ]
}
```


**Required Parameters**

* ACCOUNT_ID (in URI path - replace with actual resource ID)


**Optional Parameters**

* limit # max 50
* offset


**Return**

* 200: OK
* 404: account not found. Human readable errors are returned in the `message` key.

### Add Points to Account

```
POST https://SUBDOMAIN.uboost.com/api/points_transactions
```

cURL example - POST request in JSON format

```
curl -X POST -i \
-H 'Content-Type: application/json' \
-d '{ "points_transaction" : { 
"account_id" : "921785565",
"points_change" : 50,
"description" : "Test from the new API",
"transaction_time" : "2012-08-01 08:00:00 -1000"}
}' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/points_transactions


HTTP/1.1 201 Created
Location: https://SUBDOMAIN.uboost.com/api/points_transactions/1068511523
Content-Type: application/json; charset=utf-8
Content-Length: 7901

{
  "points_transaction":{
    "account_id":921785565,
    "created_at":"2012/08/24 15:54:06 -1000",
    "description":"Test from the new API",
    "id":1068511523,
    "points_change":50,
    "redemption_id":null,
    "site_id":75,
    "source_account_id":921542448,
    "transaction_time":"2012/08/01 08:00:00 -1000",
    "updated_at":"2012/08/24 15:54:06 -1000",
    "webservice":true,
    "account":{
      "points":800,
      "points_accumulated":1250,
      "xp":1250
    },
    "points_transaction_type":{
      "id":52,
      "name":"Direct Deposit"
    }
  }
}
```


**Required Parameters**

* points_transaction[account_id]
* points_transaction[points_change] # Positive to credit account, and negative to debit account.


**Optional Parameters**

* points_transaction[description] # Free form text. Highly recommended that this is indicative to student as to why they received this award.
* points_transaction[transaction_time] # Will default to the resource creation date if not specified. Format: `YYYY-MM-DD hh:mm:ss ±[hh][mm]`

**Return**

* 201: points transaction created, JSON data. The URI to the newly created resource is returned in the location header.
* 404: account not found
* 422: invalid parameter name or value. Human readable errors are returned in the `message` key.


--------------------------------------------------------------------------------

Groups API
------------------

* Not available for tag-based sites

### List

```
GET https://SUBDOMAIN.uboost.com/api/groups
```

cURL example - GET request

```
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/groups

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 880

{
  "groups":[
    {
      "accounts_count":0,
      "created_at":"2010/03/04 23:49:57 -1000",
      "id":635,
      "name":"Demo Group 1",
      "site_id":75,
      "updated_at":"2010/05/21 21:21:36 -1000"
    },
    ...
  ]
}
```

**Return**

* 200: OK
* 403: forbidden if tag-based site. Human readable errors are returned in the `message` key.


### Select

```
GET https://SUBDOMAIN.uboost.com/api/groups/GROUP_ID
```

cURL example - GET request

```
curl -i https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/groups/635

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 156

{
  "group":{
    "accounts_count":0,
    "created_at":"2010/03/04 23:49:57 -1000",
    "id":635,
    "name":"Demo Group 1",
    "site_id":75,
    "updated_at":"2010/05/21 21:21:36 -1000"
  }
}
```

**Required Parameters**

* GROUP_ID (in URI path - replace with actual resource ID)


**Return**

* 200: group as JSON data
* 404: group not found. Human readable errors are returned in the `message` key.


### Create

```
POST https://SUBDOMAIN.uboost.com/api/groups
```

cURL example - POST request in JSON format

```
curl -X POST -i \
-H 'Content-Type: application/json' \
-d '{ "group" : { "name" : "Demo Group 1" } }' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/groups


HTTP/1.1 201 Created
Location: https://SUBDOMAIN.uboost.com/api/groups/635
Content-Type: application/json; charset=utf-8
Content-Length: 156

{
  "group":{
    "accounts_count":0,
    "created_at":"2010/03/04 23:49:57 -1000",
    "id":635,
    "name":"Demo Group 1",
    "site_id":75,
    "updated_at":"2010/05/21 21:21:36 -1000"
  }
}
```


**Required Parameters**

* group[name]


**Return**

* 201: group created, JSON data. The URI to the newly created resource is returned in the location header.
* 422: invalid parameter name or value. Human readable errors are returned in the `message` key.


### Update

```
PUT https://SUBDOMAIN.uboost.com/api/groups/GROUP_ID
```

cURL example - PUT request in JSON format

```
curl -X PUT -i \
-H 'Content-Type: application/json' \
-d '{ "group" : { "name" : "New Group Name" } }' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/groups/635


HTTP/1.1 200 OK 
Content-Type: application/json; charset=utf-8
Content-Length: 2

{}
```

**Required Parameters**

* GROUP_ID (in URI path - replace with actual resource ID)
* group[name]

**Return**

* 200: OK
* 404: group not found
* 422: invalid parameter name or value. Human readable errors are returned in the `message` key.


### Destroy

```
DELETE https://SUBDOMAIN.uboost.com/api/groups/GROUP_ID
```

cURL example - DELETE request

```
curl -X DELETE -i \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/groups/635


HTTP/1.1 200 OK 
Content-Type: application/json; charset=utf-8
Content-Length: 2

{}
```

**Required Parameters**

* GROUP_ID (in URI path - replace with actual resource ID)


**Return**

* 200: OK 
* 404: group not found.
* 422: group must be empty. Human readable errors are returned in the `message` key.


--------------------------------------------------------------------------------

Badges API
------------------

### Create

```
POST https://SUBDOMAIN.uboost.com/api/badges
```

cURL example - POST request in JSON format

```
curl -X POST -i \
-H 'Content-Type: application/json' \
-d '{ "badge" : { "account_id" : "921785565", "badge_type_id" : "81" } }' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/badges


HTTP/1.1 201 Created
Location: https://SUBDOMAIN.uboost.com/api/badges/1216481
Content-Type: application/json; charset=utf-8
Content-Length: 148

{
  "badge":{
    "account_id":921785565,
    "badge_type_id":81,
    "created_at":"2012/08/24 17:44:58 -1000",
    "id":1216481,
    "updated_at":"2012/08/24 17:44:58 -1000"
  }
}
```


**Required Parameters**

* badge[account_id]
* badge[badge_type_id] # IDs for custom badges (if applicable) will be provided


**Return**

* 201: badge created, JSON data. The URI to the newly created resource is returned in the location header.
* 404: account or badge type not found
* 422: invalid parameter name or value. Human readable errors are returned in the `message` key.


### Unaward

```
DELETE https://SUBDOMAIN.uboost.com/api/badges/unaward
```

cURL example - DELETE request

```
curl -X DELETE -i \
-H 'Content-Type: application/json' \
-d '{ "badge" : { "account_id" : "921785565", "badge_type_id" : "81" } }' \
https://API_CREDENTIALS@SUBDOMAIN.uboost.com/api/badges/unaward


HTTP/1.1 200 OK 
Content-Type: application/json; charset=utf-8
Content-Length: 2

{}
```

**Required Parameters**

* badge[account_id]
* badge[badge_type_id]


**Return**

* 200: OK 
* 404: account or badge type not found. Human readable errors are returned in the `message` key.


**Note:**

Unaward will delete all badges of the same badge type for the specified account.


--------------------------------------------------------------------------------

Widgets API
------------------

### JSONP Support

You can send the `?callback` parameter to any Widgets API GET call to have the results wrapped in a function. This is typically used to get around cross domain issues when accessing the Widgets API from a widget embedded within your LMS.

### SSO and Cookies

An SSO token can be sent in using the `?sso_token` parameter. The token can only be used once and has a 3 minute TTL. Once the token is used, the response from our API will include the `_uboost_session_id` cookie that can be used for future calls to the Widgets API. All Widgets API calls using the `_uboost_session_id` cookie will return data for the authenticated account.


* Reference: [Get Single Sign On (SSO) Token for an Account](#accounts-sso)

### User Credentials

In the cURL examples, substitue `USER_CREDENTIALS` with an active student account's user name and password. The Widgets API will return data used to populate widgets that are associated with the authenticated account.

* Do not use the `API_CREDENTIALS` to access the Widgets API. The `API_CREDENTIALS` account does not have the proper data to populate a widget, and is used to access all APIs other than the Widgets API.



### Profile

```
GET https://SUBDOMAIN.uboost.com/api/widgets/profile
```

cURL example - GET request

```
curl -i https://USER_CREDENTIALS@SUBDOMAIN.uboost.com/api/widgets/profile

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 322

{
  "profile":{
    "id":921785565,
    "user_name":"isaacnewton",
    "avatar_head":"http://dev-test1.uboost.com:3000/images/avatar/no_avatar_head.png",
    "avatar_body":"http://dev-test1.uboost.com:3000/images/avatar/no_avatar_body.png",
    "level":1,
    "currency_name":"Credits",
    "points":"0",
    "xp":0,
    "to_next_level_xp":999,
    "badges":0,
    "power_ups":0
  }
}
```

### Badge Categories

```
GET https://SUBDOMAIN.uboost.com/api/widgets/badges/categories
```

cURL example - GET request

```
curl -i https://USER_CREDENTIALS@SUBDOMAIN.uboost.com/api/badges/categories

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 375

{
  "badge_categories":[
    {
      "id":7,
      "name":"General Activity"
    },
    ...
  ]
}
```

### My Badges

```
GET https://SUBDOMAIN.uboost.com/api/widgets/badges/mine/BADGE_CATEGORY_ID
```

cURL example - GET request

```
curl -i https://USER_CREDENTIALS@SUBDOMAIN.uboost.com/api/badges/mine/7

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 375

{
  "my_badges":[
    {
      "name":"Sandals of Fleeting Passage",
      "description":"Visited recognition and rewards at least 5 times.",
      "xp":100,
      "icon":"http://s3.amazonaws.com/ub-dev/images/334111/Sandals_of_Fleeting_Passage_small.png"
    },
    ...
  ]
}
```

### Unearned Badges

```
GET https://SUBDOMAIN.uboost.com/api/widgets/badges/unearned/BADGE_CATEGORY_ID
```

cURL example - GET request

```
curl -i https://USER_CREDENTIALS@SUBDOMAIN.uboost.com/api/badges/unearned/7

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 375

{
  "unearned_badges":[
    {
      "name":"Highlighter of the Product Analyst",
      "description":"Rated at least 25 different rewards.",
      "xp":50,
      "icon":"http://s3.amazonaws.com/ub-dev/images/334086/Highlighter_of_the_Product_Analyst_small.png",
      "status":9,
      "threshold":25
    },
    ...
  ]
}
```

### List of Leaderboards

```
GET https://SUBDOMAIN.uboost.com/api/widgets/leaderboards
```

cURL example - GET request

```
curl -i https://USER_CREDENTIALS@SUBDOMAIN.uboost.com/api/leaderboards

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 329

{
  "leaderboards":[
    {
      "id":1459,
      "name":"Top Ranked",
      "api":"https://SUBDOMAIN.uboost.com/api/widgets/leaderboards/1459"
    },
    {
      "id":1460,
      "name":"Top Donors",
      "api":"https://SUBDOMAIN.uboost.com/api/widgets/leaderboards/1460"
    },
    {
      "id":1461,
      "name":"Top Spenders",
      "api":"https://SUBDOMAIN.uboost.com/api/widgets/leaderboards/1461"
    }
  ]
}
```
**Note:**

Various types of leaderboards are available via the API. Currently the leaderboard types include: Top Ranked (based on XP), Top Donors, and Top Spenders. Additional types may be added per client request.

Each site is setup with three default leaderboards, one of each type. The number of leaderboards are configurable per site. And for tag-based sites, each leaderboard type can be filtered or customized by tags.

### Leaderboard

```
GET https://SUBDOMAIN.uboost.com/api/widgets/leaderboards/LEADERBOARD_ID
```

cURL example - GET request

```
curl -i https://USER_CREDENTIALS@SUBDOMAIN.uboost.com/api/leaderboards/1459

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 375

{
  "leaderboard":[
    {
      "name":"top ranked user's name",
      "icon":"http://s3.amazonaws.com/ub-dev/image_files/517096/head_921613786_1345773020_small.png",
      "rank":1,
      "points":"141160 xp"
    },
    {
      "name":"2nd place user",
      "icon":"/images/avatar/no_avatar_head.png",
      "rank":2,
      "points":"93430 xp"
    },
    {
      "name":"3rd place user",
      "icon":"/images/avatar/no_avatar_head.png",
      "rank":3,
      "points":"90084 xp"
    },
    {
      "name":"The user one rank above you",
      "icon":"/images/avatar/no_avatar_head.png",
      "rank":25,
      "points":"86981 xp"
    },
    {
      "name":"YOU",
      "icon":"/images/avatar/no_avatar_head.png",
      "rank":26,
      "points":"72851 xp"
    },
    {
      "name":"The rest",
      "icon":"/images/avatar/no_avatar_head.png",
      "rank":27,
      "points":"54013 xp"
    },
    ...
  ]
}
```

**Optional Parameters**

* limit # default 10, max 50

**Note:**

By default each leaderboard will return a list of ten users: the top three ranked users, a user ranked above, the current user, and 5 users ranked below the current user. 