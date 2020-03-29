See also: [Platform Building Cheat Sheet](https://github.com/mze3e/platform-cheat-sheet#platform-building-cheat-sheet)

# API Design Cheat Sheet

## General
1.  Build the API with consumers in mind--as a product in its own right.
    -   Not for a specific UI.
    -   Embrace flexibility / tunability of each endpoint (see #5, 6 & 7).
    -   Eat your own dogfood, even if you have to mockup an example UI.
1. **Date/Time/Timestamp:**: There is a standard: Use it: ISO 8601 in `UTC`:  `{ "createdTimestamp": "2017-11-15T18:10:24.343Z" }`
1. **I18N** Add support for different Languages: 
`Accept-Language: fr-ca, fr-fr not ?language=fr`
1. **KISS** 
Anyone should be able to use your API without having to refer to the documentation.
	* Use standard, concrete and shared terms, not your specific business terms or acronyms.
	* Never allow application developers to do things more than one way.
	* Design your API for your clients (application developers), not for your data.
	* Target major uses cases first, deal with exceptions later. 
	`GET /orders, GET /users, GET /products, ...`
1. **CURL** 
You should use CURL to share examples, which can be easily copy/paste. 
	```bash
	CURL –X POST \ 
	-H "Accept: application/json" \ 
	-H "Authorization: Bearer at-80003004-19a8-46a2-908e-33d4057128e7" \ 
	-d '{"state":"running"}' \
	https://api.foo.com/v1/users/007/orders?client_id=API_KEY_003
	```

## Status Codes
* `200 OK`	Successful | get, patch (return a JSON object) | Basic success code. Works for the general case. Especially used on successful first GET requests, or PUT/PATCH updated content.
* `201 Created`	Successful | post (return a JSON object) | Indicates that a resource was created. Typically responding to PUT and POST request.
* `202 Accepted`	Successful | post, delete, path - async | Indicates that the request has been accepted for processing. Typically responding to an asynchronous processing call (for better UX and performance)
* `204 No content`	Successful | delete | The request succeeded but there’s really nothing to show. Usually sent after a successful DELETE.
* `206 Partial content`	Successful | get - async | The returned resource is incomplete. Typically used with paginated resources.

## Error Status Codes
* `400 Bad Request` | General error for any request (if it doesn’t fit in any other). A good practice will be to manage two kind of errors: request behavior errors (`invalid_request`) , and application condition errors (`invalid_user`).
* `401 Unauthorized` Not authenticated (`no_credentials`) | I don’t know you, tell me who you are and I will check your permission.
* `403 Forbidden` Authenticated, but no permissions (`not_allowed`) | Your access permissions aren’t sufficient to access this resource.
* `404 - Not found` Resource not found on GET (`not_found`) | The resource you’re requesting doesn’t exist
* `405 - Method not allowed` (`method_does_not_make_sense`) Either it doesn’t make sense to call such a method on this resource or the authenticated user doesn’t have the right to do it.
* `406 - Not Acceptable` (`not_acceptable`) There’s nothing to send that matches the Accept or Accept-Language headers of the request. For example, you requested a resource in XML and the resource is only available in JSON. This also works for i18n. `{"error": "not_acceptable", "available_languages":["us-en", "de", "kr-ko"]}`
* `409 - Conflict` (`data_state_error`) Duplicate data or invalid data state would occur
* `422 Unprocessable Entity` (`semantic_error`) A `422` status code occurs when a request is well-formed, however, due to semantic errors it is **unable to be processed**.
* `500 Internal Server Error` (`server_error`) The request call is right, but a problem is encountered. The client can’t really do anything about that, so we suggest to return a 500 status code.

## Errors
Make sure errors contain as much Information as possible (Developers are your customers)

* Example: `POST /directories` -> `409 Conflict`
	```json
  {
    "status": 409,
    "code": 40924,
    "property": "name",
    "message": "A directory named 'Avengers' already exists.",
    "developerMessage": "A directory named 'Avengers' already exists. If you have a stale local cache, please expire it now.",
    "moreInfo": "https://www.foo.com/docs/api/errors/40924"
  }
  ```
  

## URL
* **Base-URL:** `https://api.foo.com` (simpler -> better as customers (developers) want the easiest path to adoption) vs. `https://www.foo.com/dev/service/api/rest`
	* You may consider the following five subdomains
		* Production - https://api.foo.com
		* Tests - https://api.sandbox.foo.com
		* Developer portal - https://developers.foo.com
		* Production - https://oauth2.foo.com
		* Tests - https://oauth2.sandbox.foo.com
* **Versioning: URL:** `https://api.foo.com/v1` (simpler -> pragmatic approach - although discouraged in general) vs. Media-Type `application/json+foo;application,v=1` (Ideal way -> URL doesn't change but it's hard for client developers to implement / understand it)
* **HREF:** Every Resource contains its own unique link in the response body: `{ "href": "https://api.foo.com/v1/accounts/123" }`
* **IDs:** Should be globally unique. Avoid sequential numbers (possible security risk). Good candidates: UUIDs or Url64
* **HTTP Method Overrides:** If clients don't support anything other than GET or POST: `POST /accounts/123?_method=DELETE`
* **Caching & Concurreny Control:**  ETag == Version Number for a specific Ressource:
  * Example:
  * Server (initial response): `ETag: "2344afa23432"`
  * Client (later request): `If-None-Match: "2344afa23432"`
  * Server (later response): `304 Not Modified`
* **Security:** 
	* Avoid sessions when possible (stateless scales better). 
	* Authorize based on Resource content, and not on URLs (they can change). 
	* Use [OAuth2](http://oauth.net/2/) in addition to SSL/TLS
		* OAuth2 matches 99% of requirements and client typologies, don't reinvent the wheel, you'll fail.
		* Use a Bearer token for authentication.
		* Require HTTPS / TLS / SSL to access your APIs. OAuth2 Bearer tokens demand it. 
		* Unencrypted communication over HTTP allows for simple eavesdroppping and impersonation.
* **Cross-origin	requests:** Use	`CORS`	standard	to	support	REST	API	requests	from	browsers	(js	SPA…). 
	* If you need to support older browsers such as IE7 or IE8 or IE9 then you need to use JSONP endpoints as well. DON'T!
* **Maintenance:** 
	* Use HTTP Redirects (e.g. for moving Ressources -> migrate URLs or deprecate them). 
	* Create abstraction layers in your code to minimize change in the API.
	* Use well defined custom Media-Types (If you can do it: It is most resilient to changes over time).
* **URL reserved words:**
	* first - Use /first to get the 1st element  `GET /orders/first`
	* last - Use /last to retrieve the latest resource of a collection `GET /orders/last`
	* count - Use /count to get the current size of a collection  `GET /orders/count`
		*  200 OK {"2"}

## Mention Media Format Specification and API Version in the Accept Header 

* **Content Negotiation:** The client can ask for the required content in the `Accept` Header: `Accept: application/json, text/plain` vs. Resource Extension: Like `/applications/123.json` or `applications/123.csv` (it conventionally overrides the `Accept` Header
	* eg. `Accept: application/json;`
	* eg. `Accept: application/foo+json;`
	* eg. `Accept: application/json, text/plain not /orders.json`

## Mention Parsing Rules in the Content-Type Header for Client
* Contains the MIME-Type of the body being sent: 
	* e.g. `application/json` 
	* or custom MIME-Types like `application/foo+json` (the body is structured to the foo media type specification)
	* or `application/foo+json;application` (add addons/fragments to express not only the format but also the resource inside of it. This is a `foo` JSON document that happens to be an application Resource)
* eg. `Content-Type: application/json`
## Use Content-Type negotiation to describe incoming request payloads from clients for Server
* For example, let's say you're doing ratings, including a thumbs-up/thumbs-down and five-star rating. You have one route to create a rating:  `POST /ratings`
	* How do you distinguish the incoming data to the service so it can determine which rating type it is: thumbs-up or five star?
	* The temptation is to create one route for each rating type:  `POST /ratings/five_star`  and  `POST /ratings/thumbs_up`
	* However, by using Content-Type negotiation we can use our same  `POST /ratings`  route for both types. 
	* By setting the  `Content-Type`  header on the request, the server can determine how to process the incoming rating data to something like  
		* eg. `Content-Type: application/vnd.company.rating.thumbsup`
		* eg.  `Content-Type: application/vnd.company.rating.fivestar` 

## Resources
* **Resource Names:** no verbs -> nouns only (vs SOAP-RPC).
	* `GET /orders` **NOT** `GET /getAllOrders`
* Use plural forms (‘orders’ instead of ‘order’)
* If you have to use more than one word in URL, you should use spinal-case (some servers ignore case). `POST /specific-orders`
 * Keep your API coarse grained to be scaleable to future requirements there is no one size fits all. It has to be aligned to the business requirements and future vision.
 * Fine grained will need more api calls to serve a single page with all the contents, whereas Coarse grained would return all the required data in one go.
 * Resources shouldn’t be nested more than two level deep : `GET /users/007`
	 ```json
	 { 	
		"id":"007",
		"firstName":"James",
		"name":"Bond",
		"address":	{ 
						"street":"H.Ferry Rd.", 
						"country":{"name":"London"} 
						} 
	}
	 ```
	 * **Properties:** Use camelCase notation for properties or snake_case - whichever you pick, be consistent.

### Use the Collection Metaphor
 * Two URLs (endpoints) per resource:
	* Collection of Resources (e.g. with Links `/orders` , `/applications`)
	* One instance of a Resource  (e.g. `/orders/{orderId}`, `/applications/a1b2c3`)
 * **Hierarchical structure** You should leverage the hierarchical nature of the URL to imply structure (aggregation or composition). Alternate resource names with IDs as URL nodes `(e.g. /orders/{orderId}/items/{itemId})` an order is composed of items.
* Keep URLs as short as possible. Preferably, no more-than three nodes per URL.
 * `GOOD` use HTTP methods eg. `/applications` or `/applications/a1b2c3`
 * `BAD` don't use verbs or specify operations in the url eg. `/getAccount` or `/searchAccounts` or `/createDirectory`
* **For Resources:** Be as specific as possible: `/customers` vs. `/newsletter-customers` and `/registered-customers`


## Resource References (Linking):
* Make resource representations meaningful.
    * **No Naked IDs!** No plain IDs embedded in responses. Use links and reference objects.
	* **Design resource representations** Don’t simply represent database tables.
	* **Merge representations** Don’t expose relationship tables as two IDs.
*  Support link expansion of relationships. Allow clients to expand the data contained in the response by including additional representations instead of, or in addition to, links.
 * **Instance Reference:** `GET /accounts/123` 
	```json
	 {
	 "href": "https://api.foo.com/v1/accounts/123",
	 "name": "Tony", ... 
	 }
	 ```
 * **Collection Reference:** `GET /accounts/123` 
	```json
	{ 
	 "href": "https://api.foo.com/v1/accounts/123", 
	 "name": "Tony", 
	 ..., 
	 "directory": 
		 { "href": "https://api.foo.com/v1/directories/345"}
	}
	 ```
* **Reference Expansion (aka Entity Expansion or Link Expansion):** `GET /accounts/123?expand=directory` 
	```json
	 { 
	 "href": "https://api.foo.com/v1/accounts/123",
	 "name": "Tony", 
	 ..., 
	 "directory": 
		 { "href": "https://api.foo.com/v1/directories/345",
		 "name": "Avengers", 
		 ...
		 } 
	 }
	```

* **Many to Many:** Each `n:m` Mapping is a Resource: 
	* e.g. **Group to Account:** A Group contains `Accounts` and an `Account` contains `Groups`: 
	* The Resource would be `GroupMembership`:
	  * Example: `GET /groupMemberships/678`
	```json
	{
	    "href": "https://api.foo.com/groupMemberships/678",
	    "account": { "href": "https://api.foo.com/accounts/123" },
	    "group": { "href": "https://api.foo.com/groups/234" },
	    ...
	}
	```
 * Another Example with the Resource `Account`: It contains the `groups` directly as well as the `groupMemberships`: `GET /accounts/134`
  ```json
  {
    "href": "https://api.foo.com/account/123",
    "name": "Tony",
    ...,
    "groups": [{ "href": "https://api.foo.com/groups/234" }],
    "groupMemberships": { "href": "https://api.foo.com/groupMemberships?accountId=123" }
  }
  ```
 * Consider connectedness by utilizing a linking strategy. Some popular examples are:
    
    -   [HAL](http://stateless.co/hal_specification.html)
    -   [Siren](https://github.com/kevinswiber/siren)
    -   [JSON-LD](http://json-ld.org/)
    -   [Collection+JSON](http://amundsen.com/media-types/collection/)


## Caching:
Consider Cache-ability. Query-Parameters should take into account for caching rules, so this is being cached under this specific URL. At a minimum, use the following response headers:
 * **ETag** - An arbitrary string for the version of a representation. Make sure to include the media type in the hash value, because that makes a different representation. (ex: ETag: "686897696a7c876b7e")
 * **Date** - Date and time the response was returned (in `RFC1123` format). (ex: Date: Sun, 06 Nov 1994 08:49:37 GMT)
 * **Cache-Control** - The maximum number of seconds (max age) a response can be cached. However, if caching is not supported for the response, then no-cache is the value. (ex: Cache-Control: 360 or Cache-Control: no-cache)
 * **Expires** - If max age is given, contains the timestamp (in RFC1123 format) for when the response expires, which is the value of Date (e.g. now) plus max age. If caching is not supported for the response, this header is not present. (ex: Expires: Sun, 06 Nov 1994 08:49:37 GMT)
 * **Pragma** - When Cache-Control is 'no-cache' this header is also set to 'no-cache'. Otherwise, it is not present. (ex: Pragma: no-cache)
 * **Last-Modified** - The timestamp that the resource itself was modified last (in RFC1123 format). (ex: Last-Modified: Sun, 06 Nov 1994 08:49:37 GMT)

## Response Data Options
* **Partial Responses:** 
	* Options for partial response of properties.
	* `GET /accounts/123?fields=name,surname,directory(name)`
* **Searching / Filtering:** 
	* Options for searching and filtering of the data set.
	* `GET /accounts?filter=param1=value1&param=value2&age=gte:30&age=lte:40`
	* `GET /items?q=title:red chair AND price:[10 TO 100]` (using lucene)
	* `GET /search?q=running+paid` (google style)
* **Sorting:**
	* Options for sorting of the response data set
	* `GET /users?sort_by=desc(last_modified),asc(email)`  
	* `GET /users?sort_by=-last_modified,+email`
* **Pagination:** 
	* Options to paginate. Pagination is mandatory. You may use a offset and limit query parameter along with the `GET` request. A default pagination has to be defined, for example : `limit=25`. Note that pagination may cause some unexpected behavior if many resources are added.
	* `/applications?offset=25&limit=25`
	* `/applications?limit=20`
	 * Example: `GET /accounts?offset=0&limit=25` 
	 * Response Headers
		* `206 Partial Content`
		* `Content-Range: 0-25/971`
		* `Accept-Range: order 25`
		* `Content-Type: application/json`
  ```json
  {
    "href": "https://api.foo.com/v1/accounts",
    "offset": 0,
    "limit": 25,
    "first": { "href": "https://api.foo.com/v1/accounts?offset=0&limit=25" },
    "previous": null,
    "next": { "href": "https://api.foo.com/v1/accounts?offset=25&limit=25" },
    "last": { "href": "..." },
    "items": [
    	{ "href": "https://api.foo.com/v1/accounts/1" },
        { "href": "https://api.foo.com/v1/accounts/2" },
        { "href": "https://api.foo.com/v1/accounts/3" },
        ...
    ]
  }
  ```

## Authentication
`curl -is https://$TOKEN@api.service.com/`

## Behavior
* `POST`, `GET`, `PUT`, `DELETE` != CRUD
* `GET` = READ (Idempotent)
* `HEAD` = Headers no BODY (Idempotent)
* `DELETE` = DELETE (NOT Idempotent)
* `POST` (NOT Idempotent) & `PUT` (Idempotent) can both be used for both CREATE and UPDATE

_PS: Idempotent - You can call it multiple times and expect the resource to be identical after each request_

Ensure that your GET, PUT, and DELETE operations are all  [idempotent](http://www.restapitutorial.com/lessons/idempotency.html). i.e. There should be no adverse side affects from these operations.

### Response Body: 
* `GET` should return a response body always. 
* `POST` should contain it when its feasable to have the most recent version of that object which might be slightly different than the request object: Return it by default and the client always gets the newest version of that object 

	(or as an alternative: give the power to the client with: `?\_body=false` to let him decide whether or not he wants the object back)

### API Behavior

* **POST** - For CREATE (NOT Idempotent). The request goes to a parent resource eg. `/applications` and returns a key or id for the newly created resource back in the response.
	 * Request: `POST api.service.com/applications` 
		 ```json
		 {	
		 "name": "a1b2c3",
		 "instances" : 5,
		 "cluster" : 1,
		 "location": "SG"
		 }
		 ```
	 * Response: 
		 ```json
		 {
		 "status": 201,
		 "location": "https://api.service.com/applications/a1b2c3" 
		 }
		 ```
* **PUT** For CREATE - (Idempotent). Can be used  where client knows the location already, and the Client has the ability to create an identifier for the Resource himself. But it has to contain a full replacement of the dataset.
	 * Request: `PUT https://api.service.com/applications/a1b2c3` 
		 ```json
		 {	
		 "name": "a1b2c3",
		 "instances" : 5,
		 "cluster" : 1,
		 "location": "SG"
		 }
		 ```
	 * Response:
		```json
		 {
		 "status": 201,
		 "location": "https://api.service.com/applications/a1b2c3" 
		 }
		 ```
* **POST** (NOT Idempotent) *or* `PUT` (Idempotent) for UPDATE - Has to contain the a full replacement of the dataset (Full Update Operation)
	 * Request: `POST or PUT https://api.service.com/applications/a1b2c3` 
		 ```json
		 {	
		 "name": "a1b2c3",
		 "instances" : 5,
		 "cluster" : 1,
		 "location": "SG"
		 }
		 ```
	 * Response:
		```json
		 {
		 "status": 200
		 }
		 ```
* **PATCH** For UPDATE (NOT Idempotent) - Update only part of the resource. (Partial Update)
	* Request: `PATCH https://api.service.com/applications/a1b2c3` 
		```json
		{	
		"instances" : 5,
		}
		 ```
	* Response:
		```json
		{
		"status": 200
		}
		```

* **GET** is used to Read a collection.
	* Request: `GET https://api.service.com/applications` 
	* Response:
		```json
		 {{	
		 "name": "a1b2c3",
		 "instances" : 5,
		 "cluster" : 1,
		 "location": "SG"
		 },
		 {	
		 "name": "e4f5g6",
		 "instances" : 10,
		 "cluster" : 2,
		 "location": "AU"
		 }}
		```

* **GET** is used to Read an instance.
	* Request: `GET https://api.service.com/applications/a1b2c3` 
	* Response:
		```json
		 {	
		 "name": "a1b2c3",
		 "instances" : 5,
		 "cluster" : 1,
		 "location": "SG"
		 }
		```

## “Non Resources” scenarios 
In a few use cases we have to consider operations or services rather than resources. 

You may use a POST request with a verb at the end of the URI 
`POST /emails/42/send` 
`POST /calculator/sum [1,2,3,5,8,13,21]`
`POST /convert?from=EUR&to=USD&amount=42`

## HATEOAS 
Your API should propose Hypermedia links in order to be completely discoverable. 

But keep in mind that a majority of users wont probably use those hyperlinks (for now), and will read the API documentation and copy/paste call examples. 

So, each call to the API should return in the Link Header every possible state of the application from the current state, plus self. 

You may use RFC5988 Link notaTon to implement HATEOAS :
```
GET /users/007
200 Ok
{ "id":"007", "firstname":"James",...}
Link : <https://api.fakecompany.com/v1/users>; rel="self"; method:"GET",
<https://api.fakecompany.com/v1/addresses/42>; rel="addresses"; method:"GET",
<https://api.fakecompany.com/v1/orders/1234>; rel="orders"; method:"GET"
```

## Idempotence

Simply put, an operation is idempotent if it produces the same result when called over and over. An identical request should return an identical result when done twice, two thousand, or two million times. The source of most confusion around this concept comes with the idea of identical results, however. What we expect to see is identical results in the _return form_ rather than in the _return value_.

Eg. GET should query the current state without side effects. PUT should set the state idempotently. POST/PATCH could be used for updating fully or partially. DELETE should delete if it exists.

|HTTP Method|Idempotence|Safety| 
|--|--|--|
|GET|YES|YES|
|HEAD|YES|YES|
|PUT|YES|NO|
|DELETE|YES|NO|
|POST|NO|NO|
|PATCH|NO|NO|

## Sources: 
1. <https://devhints.io/rest-api>
1. <https://www.youtube.com/watch?v=5WXYw4J4QOU>
1. <https://nordicapis.com/understanding-idempotency-and-safety-in-api-design/>
1. <https://github.com/RestCheatSheet/api-cheat-sheet>
1. <https://www.moesif.com/blog/technical/api-design/REST-API-Design-Filtering-Sorting-and-Pagination/>
1. <https://blog.octo.com/wp-content/uploads/2014/10/RESTful-API-design-OCTO-Quick-Reference-Card-2.2.pdf>_
