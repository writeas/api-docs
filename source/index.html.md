---
title: Write.as API Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - go

toc_footers:
  - <a href='https://developers.write.as'>Developers Write.as</a>
  - <a href='https://m.abunchtell.com/@writeas_dev'>@writeas_dev on Mastodon</a>
  - <a href='https://writing.exchange/@write_as'>@write_as on Mastodon</a>
  - <a href='https://twitter.com/writeas__'>@writeas__</a>
  - <a href='https://write.as/blog/'>Updates</a>
  - <a href='https://write.as'>Write.as</a>
  - <a href='https://github.com/lord/slate'>Docs by Slate</a>

search: true

code_clipboard: true
---

# Introduction

Welcome to the [Write.as](https://write.as) API! Our API gives you full access to Write.as data and lets you build your own applications or utilities on top of it.

Our API is accessible at **https://write.as/api/** (HTTPS _only_) and via our Tor hidden service at **writeas7pm7rcdqg.onion/api/**.

Backwards compatibility is important to us since we have a large set of [clients](https://write.as/apps) in the wild. As we add new features, we usually add endpoints and properties alongside existing ones, but don't remove or change finalized ones that are documented here. If a breaking API change is required in the future, we'll version new endpoints and update these docs.

This documentation represents the officially-supported, latest API. Any properties or endpoints you discover in the API that aren't documented here should be considered experimental or beta functionality, and subject to change without any notice.

## Libraries

These are our official libraries in different programming languages.

* **Go** - `go get go.code.as/writeas.v1` &middot; [GoDoc](https://godoc.org/go.code.as/writeas.v1) &middot; [GitHub](https://github.com/writeas/go-writeas)
* **Java** - [GitHub](https://github.com/writeas/java-writeas)

These libraries were created by the community &mdash; [let us know](https://write.as/contact) if you create one, too!

* **Javascript** - `npm install writeas` &middot; [npm](https://www.npmjs.com/package/writeas) &middot; [GitHub](https://github.com/devsnek/writeas.js)

## Errors

> Failed requests return with an error `code` and `error_msg`, like below.

```json
{
  "code": "400",
  "error_msg": "Bad request."
}
```

The Write.as API uses conventional HTTP response codes to indicate the status of an API request. Codes in the `2xx` range indicate success or that more information is available in the returned data, in the case of bulk actions.
Codes in the `4xx` range indicate that the request failed with the information provided. Codes in the `5xx` range indicate an error with the Write.as servers (you shouldn't see many of these).

Error Code | Meaning
---------- | -------
400 | Bad Request -- The request didn't provide the correct parameters, or JSON / form data was improperly formatted.
401 | Unauthorized -- No valid user token was given.
403 | Forbidden -- You attempted an action that you're not allowed to perform.
404 | Not Found -- The requested resource doesn't exist.
405 | Method Not Allowed -- The attempted method isn't supported.
410 | Gone -- The entity was unpublished, but may be back.
429 | Too Many Requests -- You're making too many requests, especially to the same resource.
500, 502, 503 | Server errors -- Something went wrong on our end.

## Responses

> Successful requests return with a `code` in the `2xx` range and a `data` object or array, depending on the request.

```json
{
  "code": "200",
  "data": {
  }
}
```

All responses are returned in a JSON object containing two fields:

Field | Type | Description
----- | ---- | -----------
`code` | integer | An HTTP status code in the `2xx` range.
`data` | object or array | A single object for most requests; an array for bulk requests.

This wrapper will never contain an `error_msg` property at the top level.

<aside class="success">
Most endpoints <em>accept</em> both form data or JSON, assuming form data by default unless a <code>Content-Type: application/json</code> header is sent.
</aside>


# Authentication

The API doesn't require any authentication, either for the client or end user. However, if you want to perform actions on behalf of a user, you'll need to pass a user access token with any requests:

`Authorization: Token 00000000-0000-0000-0000-000000000000`

See the [Authenticate a User](#authenticate-a-user) section for information on logging in.

<aside class="notice">
Replace <code>00000000-0000-0000-0000-000000000000</code> with a user's access token.
</aside>


# Posts

Posts are the most basic entities on Write.as. Each can have its own appearance, and isn't connected to any other Write.as entity by default. They can exist without an owner (user), or with an owner but no collection, or with both an owner and collection.

When posts are published with no owner, it's up to the client to remember the posts a user has published. By keeping this information on the client, users can write anonymously -- both according to Write.as and to anyone who reads any given post.

Users can choose between different appearances for each post, usually passed to and from the API as a `font` property:

| Argument | Appearance (Typeface) | Word Wrap? | Also on |
| -------- | --------------------- | ---------- | ------- |
| `sans` | Sans-serif (Open Sans) | Yes | _N/A_ |
| `serif` / `norm` | Serif (Lora) | Yes | _N/A_ |
| `wrap` | Monospace | Yes | _N/A_ |
| `mono` | Monospace | No | paste.as |
| `code` | Syntax-highlighted monospace | No | paste.as |

## Publish a Post

```go
c := writeas.NewClient()
p, err := c.CreatePost(&writeas.PostParams{
	Title:   "My First Post",
	Content: "This is a post.",
})
```

```shell
curl "https://write.as/api/posts" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"body": "This is a post.", "title": "My First Post"}'
```

> Example Response

```json
{
  "code": 201,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M",
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:46Z",
    "title": "My First Post",
    "body": "This is a post.",
    "tags": [
    ]
  }
}
```

This creates a new post, associating it with a user account if authenticated. If the request is successful, the post will be available all of these locations:

* write.as/`{id}`
* writeas7pm7rcdqg.onion/`{id}`
* paste.as/`{id}` -- if `font` was _code_ or _mono_

### Authentication

This can be done anonymously or while [authenticated](#authentication).

<aside class="notice">
When the request is unauthenticated, the client should store the <code>token</code> returned from this request so users can modify their post later. Otherwise it isn't necessary.
</aside>

### Definition

`POST /api/posts`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | The content of the post.
**title** | string | no | An optional title for the post. If none is given, Write.as will try to extract one for the browser's title bar, but not change any appearance in the post itself.
**font** | string | no | One of any post appearance types [listed above](#posts). If invalid, defaults to `serif`.
**lang** | string | no | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | no | Whether or not the content should be display right-to-left, for example if written in Arabic or Hebrew.
**created** | date | no | An optional _published_ date for the post, formatted `2006-01-02T15:04:05Z`.
**crosspost** | array | no | **Must be authenticated**. An array of integrations and associated usernames to post to. See [Crossposting](#crossposting).

### Returns

The newly created post.


## Retrieve a Post

```go
c := writeas.NewClient()
p, err := c.GetPost("rf3t35fkax0aw")
```

```shell
curl https://write.as/api/posts/rf3t35fkax0aw
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:05Z",
    "title": "My First Post",
    "body": "This is a post.",
    "tags": [  
    ],
    "views": 0
  }
}
```

This retrieves a post entity. It includes extra Write.as data, such as page views and extracted hashtags.

### Definition

`GET /api/posts/{POST_ID}`


## Update a Post

```go
c := writeas.NewClient()
p, err := c.UpdatePost("rf3t35fkax0aw", "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M", &writeas.PostParams{
	Content: "My post is updated.",
})
```

```shell
curl "https://write.as/api/posts/rf3t35fkax0aw" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M", "body": "My post is updated."}'
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": null,
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:05Z",
    "title": "My First Post",
    "body": "My post is updated.",
    "tags": [  
    ],
    "views": 0
  }
}
```

This updates an existing post.

### Authentication

This can be done anonymously or while [authenticated](#authentication).

<aside class="notice">
If done anonymously, it requires past knowledge of the existing post's <code>token</code>.
</aside>

### Definition

`POST /api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**token** | string | maybe* | The post's modify token. *Required if not authenticated.
**body** | string | yes | The content of the post.
**title** | string | no | An optional title for the post. Supplying a parameter but leaving it blank will remove any title currently on the post.
**font** | string | no | One of any post appearance types [listed above](#posts). If invalid, it doesn't change.
**lang** | string | no | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | no | Whether or not the content should be display right-to-left, for example if written in Arabic.

### Returns

The entire post as it stands after the update.


## Unpublish a Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/posts/rf3t35fkax0aw" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M", "body": ""}'
```

> Response

```json
{
  "code": 410,
  "error_msg": "Post unpublished by author."
}
```

<aside class="warning">
<strong>Unfinalized design</strong>.
</aside>

This unpublishes an existing post. Simply pass an empty `body` to an existing post you're authorized to update, and in the future that post will return a `410 Gone` status.

The client will likely want to maintain what was in the body before the post was unpublished so it can be restored later.

### Definition

`POST /api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | Should be an empty string (`""`).

### Returns

An error `410` with a message: _Post unpublished by author._


## Delete a Post

```go
c := writeas.NewClient()
err := c.DeletePost("rf3t35fkax0aw", "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M")
```

```shell
curl "https://write.as/api/posts/rf3t35fkax0aw?token=ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M" \
  -X DELETE
```

> A successful deletion returns a `204` with no content in the body.

This deletes a post.

### Definition

`DELETE /api/posts/{POST_ID}`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**token** | string | _if unauth'd_ | The post's modify token.

### Returns

A `204` status code and no content in the body.


## Claim Posts

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
err := c.ClaimPosts(&[]OwnedPostParams{
	{
		ID:    "rf3t35fkax0aw",
		Token: "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M",
	},
})
```

```shell
curl "https://write.as/api/posts/claim" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '[{"id": "rf3t35fkax0aw", "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M"}]'
```

> Always returns a `200`

This adds unowned posts to a Write.as user / account.

### Definition

`POST /api/posts/claim`

### Arguments

The body should contain an array of objects with these parameters:

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**id** | string | yes | ID of the post to claim
**token** | string | yes | Modify token of the post

### Returns

A `200`. Since this performs an action on multiple posts, the success/failure code are contained in each resulting post returned.


# Collections

Collections are referred to as **blogs** on most of Write.as. Each gets its own shareable URL.

Each user has one collection matching their username, but can also have more collections connected to their account as a [Casual or Pro](https://write.as/subscribe) user.

<aside class="notice">
All collection requests must be <a href="#authentication">authenticated</a>, except for retrieval when a collection is <strong>not</strong> private.
</aside>

## Create a Collection

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
coll, err := c.CreateCollection(&CollectionParams{
	Alias: "new-blog",
	Title: "The Best Blog Ever",
})
```

```shell
curl "https://write.as/api/collections" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"alias": "new-blog", "title": "The Best Blog Ever"}'
```

> Example Response

```json
{
  "code": 201,
  "data": {
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "",
    "style_sheet": "",
    "email": "new-blog-wjn6epspzjqankz41mlfvz@writeas.com",
    "views": 0,
    "total_posts": 0
  }
}
```

This creates a new collection. **Casual or Pro user required**.

### Definition

`POST /api/collections`

### Arguments

Clients must supply either a `title` or `alias` (or both). If only a `title` is given, the `alias` will be generated from it.

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**alias** | string | no | A valid collection slug / alias, containing only alphanumerics and hyphens. If none given, Write.as will generate a valid alias from the `title` value.
**title** | string | no | An optional title for the collection. If none given, Write.as will use the `alias` value.

### Returns

The newly created collection.

<aside class="success">
Clients should store the returned <code>alias</code> for future operations.
</aside>

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 403,
  "error_msg": "You must be a Pro user to do that."
}
```

Error Code | Meaning
---------- | -------
400 | Request is missing required information, or has bad form data / JSON.
401 | Incorrect information given.
403 | You're not a Pro user.
409 | Alias is taken.
412 | You've reached the maximum number of collections allowed.


## Retrieve a Collection

```go
c := writeas.NewClient()
coll, err := c.GetCollection("new-blog")
```

```shell
curl https://write.as/api/collections/new-blog
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "",
    "style_sheet": "",
    "public": true,
    "views": 9,
    "total_posts": 0
  }
}
```

This retrieves a collection and its metadata.

### Authentication

Collections can be retrieved without authentication. However, [authentication](#authentication) is required for retrieving a private collection or one with scheduled posts.

### Definition

`GET /api/collections/{COLLECTION_ALIAS}`

### Returns

The requested collection.


## Update a Collection

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/collections/new-blog" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"description": "A great blog.", "style_sheet": "body { color: blue; }"}'
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "A great blog.",
	"style_sheet": "body { color: blue; }",
	"script": "",
    "email": "new-blog-wjn6epspzjqankz41mlfvz@writeas.com",
    "views": 0
  }
}
```

This updates attributes of an existing collection.

### Definition

`POST /api/collections/{COLLECTION_ALIAS}`

### Arguments

Supply only the fields you would like to update. Any fields left out will remain unchanged.

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**title** | string | no | The title of the collection.
**description** | string | no | The collection description.
**style_sheet** | string | no | The full custom CSS to apply to the collection. **Requires Write.as Pro**
**script** | string | no | The full custom Javascript to apply to the collection. **Supported only on Write.as**
**visibility** | integer | no | The collection Publicity setting. Must be one of the valid options listed below. **Requires Write.as Pro**
**pass** | string | no | The password required to view a password-protected blog (`visibility = 4`).
**mathjax** | boolean | no | Whether or not to enable MathJax rendering on the collection.

### Valid `visibility` values

Valid options for the `visibility` field:

* `0` = Unlisted
* `1` = Public
* `2` = Private
* `4` = Password-protected; requires `pass` value

### Returns

The newly created collection.

### Errors

This endpoint will attempt to update all possible fields. It will silently skip values it cannot update, such as premium attributes on Write.as, or the Public visibility option when a WriteFreely instance isn't configured for this.

Error Code | Meaning
---------- | -------
400 | Request is missing required information, or has bad form data / JSON.
401 | Incorrect information given.
403 | You're not a Pro user.


## Delete a Collection

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl https://write.as/api/collections/new-blog \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X DELETE
```

> A successful deletion returns a `204` with no content in the body.

This permanently deletes a collection and makes any posts on it anonymous.

### Definition

`DELETE /api/collections/{COLLECTION_ALIAS}`

### Returns

A `204` status code and no content in the body.


## Retrieve a Collection Post

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl https://write.as/api/collections/new-blog/posts/my-first-post
```

> Example Response

```json
{
  "code": 200,
  "data": {
	"id": "hjb7cvwaevy9eayp",
	"slug": "my-first-post",
	"appearance": "norm",
	"language": "",
	"rtl": false,
	"created": "2016-07-09T14:29:33Z",
	"title": "My First Post",
	"body": "This is a blog post.",
	"tags": [
	],
	"views": 0
  }
}
```

This retrieves a single post from a collection by the post's slug.

### Authentication

Collection posts can be retrieved without authentication. However, [authentication](#authentication) is required for retrieving a post from a private collection.

### Definition

`GET /api/collections/{COLLECTION_ALIAS}/posts/{SLUG}`

### Returns

The requested collection post.


## Publish a Collection Post

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
p, err := c.CreatePost(&writeas.PostParams{
	Title:      "My First Post",
	Content:    "This is a post.",
	Collection: "new-blog",
})
```

```shell
curl "https://write.as/api/collections/new-blog/posts" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"body": "This is a blog post.", "title": "My First Post"}'
```

> Example Response

```json
{
  "code": 201,
  "data": {
    "id": "rf3t35fkax0aw",
    "slug": "my-first-post",
    "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M",
    "appearance": "norm",
    "language": "",
    "rtl": false,
    "created": "2016-07-09T01:43:46Z",
    "title": "My First Post",
    "body": "This is a post.",
    "tags": [
    ],
    "collection": {
      "alias": "new-blog",
      "title": "The Best Blog Ever",
      "description": "",
      "style_sheet": "",
      "public": true,
      "url": "https://write.as/new-blog/"
    }
  }
}
```

This creates a new post and adds it to the given collection. User must be authenticated.

### Definition

`POST /api/collections/{COLLECTION_ALIAS}/posts`

### Arguments

This accepts the same arguments as [anonymous posts](#publish-a-post), plus an optional `crosspost` parameter.

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | yes | The content of the post.
**title** | string | no | An optional title for the post. If none is given, Write.as will try to extract one for the browser's title bar, but not change any appearance in the post itself.
**font** | string | no | One of any post appearance types [listed above](#posts). If invalid, defaults to `serif`.
**lang** | string | no | ISO 639-1 language code, e.g. _en_ or _de_.
**rtl** | boolean | no | Whether or not the content should be display right-to-left, for example if written in Arabic or Hebrew.
**created** | date | no | An optional _published_ date for the post, formatted `2006-01-02T15:04:05Z`.
**crosspost** | array | no | An array of integrations and associated usernames to post to. See [Crossposting](#crossposting).

### Returns

The newly created post.


## Retrieve Collection Posts

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl https://write.as/api/collections/new-blog/posts
```

> Example Response

```json
{ 
  "code": 200,
  "data": {  
    "alias": "new-blog",
    "title": "The Best Blog Ever",
    "description": "",
    "style_sheet": "",
    "private": false,
    "total_posts": 1,
    "posts": [
      {
        "id": "hjb7cvwaevy9eayp",
        "slug": "my-first-post",
        "appearance": "norm",
        "language": "",
        "rtl": false,
        "created": "2016-07-09T14:29:33Z",
        "title": "My First Post",
        "body": "This is a blog post.",
        "tags": [
        ],
        "views": 0
      }
    ]
  }
}
```

This retrieves a collection's posts along with the collection's data.

### Authentication

Collection posts can be retrieved without authentication. However, [authentication](#authentication) is required for retrieving a private collection or one with scheduled posts.

### Definition

`GET /api/collections/{COLLECTION_ALIAS}/posts`

### Arguments

Query string parameters can be passed to affect post presentation.

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**body** | string | no | Desired post body format. Can be left blank for raw text, or `html` to get back HTML generated from Markdown.

### Returns

The requested collection and its posts in a `posts` array.


## Move a Post to a Collection

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/collections/new-blog/collect" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '[{"id": "rf3t35fkax0aw", "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M"}]'
```

> Example Response

```json
{
  "code": 200,
  "data": [
    {
      "code": 200,
	  "post": {
        "id": "rf3t35fkax0aw",
        "slug": "my-first-post",
        "token": "ozPEuJWYK8L1QsysBUcTUKy9za7yqQ4M",
        "appearance": "norm",
        "language": "",
        "rtl": false,
        "created": "2016-07-09T01:43:46Z",
        "title": "My First Post",
        "body": "This is a post.",
      }
	}
  ]
}
```

This adds a group of posts to a collection. This works for either posts that were created anonymously (i.e. don't belong to the user account) or posts already owned by the user account.

### Definition

`POST /api/collections/{COLLECTION_ALIAS}/collect`

### Arguments

Pass an array of objects, each containing the following parameters:

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**id** | string | yes | The ID of the post to add to the collection
**token** | string | maybe* | The post's modify token. *Required if the post doesn't belong to the requesting user.

### Returns

A `200` at the top level for all valid, authenticated requests. `data` contains an array of response envelopes: each with `code` and `post` (with full post data) on success, or `code` and `error_msg` on failure for any given post.


## Pin a Post to a Collection

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
err := c.PinPost(&writeas.PinnedPostParams{
	ID:       "rf3t35fkax0aw",
	Position: 1,
})
```

```shell
curl "https://write.as/api/collections/new-blog/pin" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '[{"id": "rf3t35fkax0aw", "position": 1}]'
```

> Example Response

```json
{
  "code": 200,
  "data": [
    {
      "id": "rf3t35fkax0aw",
      "code": 200
	}
  ]
}
```

This pins a blog post to a collection. It'll show up as a navigation item in the collection/blog home page header, instead of on the blog itself.

### Definition

`POST /api/collections/{COLLECTION_ALIAS}/pin`

### Arguments

Pass an array of objects, each containing the following parameters:

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**id** | string | yes | The ID of the post to pin to the collection
**position** | integer | no | The numeric position in which to pin the post. Will pin at end of list if no parameter is given.

### Returns

A `200` at the top level for all valid, authenticated requests. `data` contains an array of response envelopes: each with `code` and `id`, plus `error_msg` on failure for any given post.


## Unpin a Post from a Collection

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
err := c.UnpinPost(&writeas.PinnedPostParams{
	ID: "rf3t35fkax0aw",
})
```

```shell
curl "https://write.as/api/collections/new-blog/unpin" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '[{"id": "rf3t35fkax0aw"}]'
```

> Example Response

```json
{
  "code": 200,
  "data": [
    {
      "id": "rf3t35fkax0aw",
      "code": 200
    }
  ]
}
```

This unpins a blog post from a collection. It'll remove the navigation item from the collection/blog home page header and put the post back on the blog itself.

### Definition

`POST /api/collections/{COLLECTION_ALIAS}/unpin`

### Arguments

Pass an array of objects, each containing the following parameters:

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**id** | string | yes | The ID of the post to unpin from the collection

### Returns

A `200` at the top level for all valid, authenticated requests. `data` contains an array of response envelopes: each with `code` and `id`, plus `error_msg` on failure for any given post.


# Users

Users have posts and collections associated with them that can be accessed across devices and browsers.

Having an account isn't necessary to interact with Write.as, but provides useful functionality impossible or difficult to implement client-side. It does come with the trade-off of anonymity for pseudonymity.

However, Write.as is also set up to work well pseudo- and anonymously at the same time. If your client performs actions on behalf of a user, simply send the `Authorization` header for user actions and leave it off for anonymous requests, saving any published posts' `token` like normal. Posts published anonymously can still be synced at any time, or never synced to ensure an account isn't associated with posts a user doesn't want to be associated with.


## Authenticate a User

```go
c := writeas.NewClient()
u, err := c.LogIn("matt", "12345")
```

```shell
curl "https://write.as/api/auth/login" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"alias": "matt", "pass": "12345"}'
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "access_token": "00000000-0000-0000-0000-000000000000",
    "user": {
      "username": "matt",
      "email": "matt@example.com",
      "created": "2015-02-03T02:41:19Z"
    }
  }
}
```

Authenticates a user with Write.as, creating an access token for future authenticated requests.

Users can only authenticate with their primary account, i.e. the first collection/blog they created, which may or may not have multiple collections associated with it.

### Definition

`POST /api/auth/login`

### Arguments

Parameter | Type | Required | Description
--------- | ---- | -------- | -----------
**alias** | string | yes | The user's username / alias.
**pass** | string | yes | The user's password.

### Returns

A user access token and the authenticated user's full data.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 401,
  "error_msg": "Incorrect password."
}
```

Error Code | Meaning
---------- | -------
400 | Request is missing required information, or has bad form data / JSON.
401 | Incorrect information given.
404 | User doesn't exist.
429 | You're trying to log in too many times too quickly. You shouldn't see this unless you're doing something bad (in which case, please stop that).


## Log User Out

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
err := c.LogOut()
```

```shell
curl "https://write.as/api/auth/me" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -X DELETE
```
> A successful deletion returns a `204` with no content in the body.

Un-authenticates a user with Write.as, permanently invalidating the access token used with the request.

<aside class="warning">
This is important for keeping accounts secure, so that users keep access to their account limited to as few devices as possible. <em>Always</em> consider building this functionality into your Write.as clients.
</aside>

### Definition

`DELETE /api/auth/me`

### Returns

A `204` status code and no content in the body.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 404,
  "error_msg": "Token is invalid."
}
```

Error Code | Meaning
---------- | -------
400 | Request is missing an access token / Authorization header
404 | Token is invalid.


## Retrieve Authenticated User

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/me" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X GET
```

> Example Response

```json
{
  "code": 200,
  "data": {
    "username": "matt"
  }
}
```

### Definition

`GET /api/me`

### Arguments

None.

### Returns

An authenticated user's basic data.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 401,
  "error_msg": "Invalid access token."
}
```

Error Code | Meaning
---------- | -------
401 | Access token is invalid or expired.


## Retrieve User's Posts

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
p, err := c.GetUserPosts()
```

```shell
curl "https://write.as/api/me/posts" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X GET
```

> Example Response

```json
{
  "code": 200,
  "data": [
    {  
      "id": "7xe2dbojynjs1dkk",
      "slug": "cool-post",
      "appearance": "norm",
      "language": "en",
      "rtl": false,
      "created": "2017-11-12T03:49:36Z",
      "updated": "2017-11-12T03:49:36Z",
      "title": "",
      "body": "Cool post!",
      "tags": [],
      "views": 0,
      "collection": {
        "alias": "matt",
        "title": "Matt",
        "description": "My great blog!",
        "style_sheet": "",
        "public": true,
        "views": 46
      }
    },
    {
      "id": "rf3t35fkax0aw",
      "slug": null,
      "appearance": "norm",
      "language": "",
      "rtl": false,
      "created": "2016-07-09T01:43:46Z",
      "updated": "2016-07-09T01:43:46Z",
      "title": "My First Post",
      "body": "This is a post.",
      "tags": [],
      "views": 0
    }
  ]
}
```

### Definition

`GET /api/me/posts`

### Arguments

None.

### Returns

An array of the authenticated user's posts, including anonymous and collection posts.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 401,
  "error_msg": "Invalid access token."
}
```

Error Code | Meaning
---------- | -------
401 | Access token is invalid or expired.


## Retrieve User's Collections

```go
c := writeas.NewClient()
c.SetToken("00000000-0000-0000-0000-000000000000")
colls, err := c.GetUserCollections()
```

```shell
curl "https://write.as/api/me/collections" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X GET
```

> Example Response

```json
{
  "code": 200,
  "data": [
    {
      "alias": "matt",
      "title": "Matt",
      "description": "My great blog!",
      "style_sheet": "",
      "public": true,
      "views": 46,
      "email": "matt-7e7euebput9t5jr3v4csgferutf@writeas.com",
      "url": "https://write.as/matt/"
    }
  ]
}
```

### Definition

`GET /api/me/collections`

### Arguments

None.

### Returns

All of an authenticated user's owned collections.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 401,
  "error_msg": "Invalid access token."
}
```

Error Code | Meaning
---------- | -------
401 | Access token is invalid or expired.


## Retrieve User's Channels

```go
// Currently unsupported in the Go client.
// Use curl command or contribute at:
//   https://github.com/writeas/writeas-go
```

```shell
curl "https://write.as/api/me/channels" \
  -H "Authorization: Token 00000000-0000-0000-0000-000000000000" \
  -H "Content-Type: application/json" \
  -X GET
```

> Example Response

```json
{
  "code": 200,
  "data": [
    {
      "id":"twitter",
      "name":"Twitter",
      "username":"ilikebeans"
    },
    {
      "id":"mastodon",
      "url":"writing.exchange",
      "name":"writing.exchange",
      "username":"matt"
    }
  ]
}
```

### Definition

`GET /api/me/channels`

### Arguments

None.

### Returns

An array of the authenticated user's connected channels, or _integrations_.

For channels that aren't a centralized service, like Mastodon, you'll also see a `url` property of the specific instance or host that the user has connected to.

### Errors

Errors are returned with a user-friendly error message.

```json
{
  "code": 401,
  "error_msg": "Invalid access token."
}
```

Error Code | Meaning
---------- | -------
401 | Access token is invalid or expired.


# Integrations

Write.as integrates with other services on the web to provide features like crossposting upon publishing a Write.as post.

On the web, these are known as **channels**, and if you're logged in, they can be added [here](https://write.as/me/c/).

## Crossposting

> Example `crosspost` Parameter

```json
"crosspost": [
  { "twitter": "writeas__" },
  { "twitter": "ilikebeans" },
  { "medium": "someone" },
  { "tumblr": "example" }
]
```

Crossposting is easy to do when publishing a new post. It requires authentication and at least one connected integration / channel on the web.

<aside class="notice">
Soon you'll be able to retrieve a user's connected accounts, so this may only be relevant to users posting from their own account for now.
</aside>

For the `crosspost` parameter on a new post, pass a JSON array of single-property objects. The object's sole property should be the ID of the integration you want to crosspost to (see below), and its value should be the user's connected username _on that service_.

ID | Integration
--- | ----------
twitter | Twitter
tumblr | Tumblr
medium | Medium


---

Missing something? Questions? Find us on [Slack](http://slack.write.as), [IRC](irc://irc.freenode.net/writeas), or [other places](https://write.as/contact) and ask us.
