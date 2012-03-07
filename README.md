## The REST & HTTP Collection

### ETag & Last-Modified
_Features: cache validation, conditional requests, optimistic concurrency_

> These headers provide a way to compare the client and server state of
> a resource.

*Reduce bandwith.* An initial client request will get the data, while subsequent
requests should simply return a `304 Not Modified` with no response body.

*Validation of client data.* `PUT` and `PATCH` requests alters a resource's state,
but if the resource's state changes right before the request is received, the action
would be performed on stale data. To prevent resource corruption and
[lost updates][concurrency control], always add one of these headers to validate
the state.

#### Caution

*Last-Modified might not be good enough.* HTTP dates only have resolution down to a
second, thus if a resource's state changes in less than a second after being
requested, the client could be using stale data until the next state change.

#### Strategies

*Reduce server processing.* Generate an ETag and use it as a key in a hash or
key-value data store (memcache, redis) for fast existential lookup.

*Invalidate the key by unsetting it when the resource state changes.* Many ETag
implementations use MD5 or SHA1 hashes, but for cache invalidation, their
non-deterministic _feature_ makes it cumbersome to work with. 

*Django Example*

```python
from django.db import models
from django.core.cache import cache

class VersionedModel(models.Model):
    version = models.IntegerField(default=0)

    class Meta(object):
        abstract = True

    def save(self, *args, **kwargs):
        etag = self.invalidate_cache()
        self.version += 1
        try:
            super(VersionedModel, self).save(*args, **kwargs)
        except Exception, e:
            # Revert if the save failed
            self.version -= 1
            cache.set(etag, 1)
            raise e

    def invalidate_cache(self):
        etag = self.generate_etag()
        cache.delete(etag)
        return etag

    def generate_etag(self):
        kwargs = {
            'app': self._meta.app_label,
            'model': self._meta.module_name,
            'id': self.id,
            'version': self.version,
        }
        return '{app}.{model}-{id},{version}'.format(**kwargs)


# music/models.py
class Song(VersionedModel):
    title = models.CharField(max_length=100)


# interpreter
>>> song = Song(title='Here Come the Sun')
>>> song.save()
>>> cache.get('music.song-1,1)
1
>>> song.title = 'Here Comes the Sun'
>>> song.save()
>>> cache.get('music.song-1,1)
>>> cache.get('music.song-1,2)
1
```

[concurrency control]: http://en.wikipedia.org/wiki/Concurrency_control
