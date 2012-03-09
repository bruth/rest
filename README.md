# The REST & HTTP Collection

## General Musings

### Don't Call It REST

..unless you [know][1] it conforms to the architecture. Web APIs sounds just
as cool...

### Hypermedia As The Engine of Application State (HATEOAS)

This is a constraint of the REST architecture. Thinking of a typical Web page,
majority of the page is made up of informational text as well as links, form
elements and buttons. A Web browser renders these elements in such a way that
enables the user to interact with them.

As an example, a request made to http://www.google.com will
result in a response containing a body of text with a `Content-Type` of `text/html`
which is a representation of the current state of the resource. Since the browser understands
the media type `text/html`, it parses the response body and renders elements
into the common Web page we all know and love.

This Web page enables us, the end users, to proceed traversing this HTML document.
The thing to remember is that there is no _need_ to know where to go or what to do
next on the Web page. This hypertext document drives the application state with it's
representation via the links, forms and buttons provided. All the user needs to do
is click on a link or fill out and submit a form to navigate the Web site.

Thinking about this in the context of other media types and other user agents (humans
are less frequently the consumers of Web APIs), it begins to get complicated. Why?
Most media types are not inherently descriptive or there are less standards or conformity
around them. A popular media type in today's Web APIs is JavaScript Object Notation
(JSON). The issue faced by many Web APIs is interoperability and consumability across
clients.

In HTML, for example, the anchor element `<a />` is the standardized element for
representing a hyperlink. Since it is a standard, it makes it easy to consume.
Even if a computer agent was consuming the HTML, it could parse the text and extract
all of the anchor `href` attributes which represents a list of subsequent locations
in the document that can be requested.

This is not the case with JSON (or XML or YAML). There is no standard attribute
on a JSON object that guarantees a hyperlink. Is it `url`, `uri`, `link`,
`location`, `href` or `mothereffingURL`? No one knows. Well, the client does, whos
developer also wrote the Web API =)

[1]: http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven

- [Steps Toward the Glory of REST](http://martinfowler.com/articles/richardsonMaturityModel.html)
- [Phil Karlton Quote](http://martinfowler.com/bliki/TwoHardThings.html)
- [How to GET a Cup of Coffee](http://www.infoq.com/articles/webber-rest-workflow)

## Validation Caching: ETag & Last-Modified
_Features: cache validation, conditional requests, optimistic concurrency_

### Usage

_These headers provide a way to compare the client and server state of a resource._

**Reduce bandwith.** An initial client request will get the data, while subsequent
requests should simply return a `304 Not Modified` with no response body.

**Validation of client data.** `PUT` and `PATCH` requests alters a resource's state,
but if the resource's state changes right before the request is received, the action
would be performed on stale data. To prevent resource corruption and the
[lost update problem](http://www.w3.org/1999/04/Editing/), always add one of these
headers to validate the state.

### Caution

**Last-Modified might not be good enough.** HTTP dates only have resolution down to a
second, thus if a resource's state changes in less than a second after being
requested, the client could be using stale data until the next state change.

**Last-Modified dates are not unique.** An ETag can easily be generated to be unique,
thus when a client sends the `If-None-Match` or `If-Match` header, the server can
rely on the `Etag` for the existential lookup without ever touching the database.
This is not the case with the `Last-Modified` date, since the date _uniqueness_ is
relative to the resource. If diligent effort is put into ensuring the resource URI
truly represents a unique resource, then the combination of URI and the modified
date may be suit the needs of cache invalidation.

### Strategies

**Reduce server processing.** Generate an ETag and use it as a key in a hash or
key-value data store (memcache, redis) for fast existential lookup.

**Invalidate the key by unsetting it when the resource state changes.** Many ETag
implementations use MD5 or SHA1 hashes, but for cache invalidation, their
non-determinism makes it cumbersome to know what the ETag was before. 

### Examples

- ETag "ready" Django model using a `version` number - https://gist.github.com/2001416
- ETag "ready" Django model using a `modified` datetime - https://gist.github.com/2001429
    - It is very common for database tables to have a `modified` timestamp column. We
    can use that instead of adding an extra `version` column.

### References

- [Precedence of ETag vs. Last-Modified](http://stackoverflow.com/a/1560098)
- [Optimistic Concurrency](http://en.wikipedia.org/wiki/Concurrency_control)
- [Lost Update Problem](http://www.w3.org/1999/04/Editing/)
- [REST Better HTTP Cache](http://www.odino.org/301/rest-better-http-cache)
