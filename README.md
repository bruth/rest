# The REST & HTTP Collection

### General References

- [Steps Toward the Glory of REST](http://martinfowler.com/articles/richardsonMaturityModel.html)
- [Phil Karlton Quote](http://martinfowler.com/bliki/TwoHardThings.html)

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
