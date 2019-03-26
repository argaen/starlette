
For caching we recommend using [aiocache](https://github.com/argaen/aiocache). It is a caching framework that supports multiple backends and other nice features like serializers, plugings, locking. For more info have a look at [aiocache docs](https://aiocache.readthedocs.io/en/stable/).

Let's start with a simple example where you want to cache an expensive function:

```
import asyncio
from aiocache import cached

from starlette.applications import Starlette
from starlette.responses import JSONResponse
import uvicorn


app = Starlette(debug=True)


@cached(ttl=5)
async def expensive():
    await asyncio.sleep(5)
    return {'hello': 'world'}


@app.route('/')
async def homepage(request):
    result = await expensive()
    return JSONResponse(result)

if __name__ == '__main__':
    uvicorn.run(app, host='0.0.0.0', port=8000)
```

In the example above, the function called `expensive` is being cached for 5 seconds. You can specify different settings to control the cache behavior like:

- Backend to use (memory is default but you can choose redis and memcached too)
- ttl, serializer, namespace and other attributes to use

For more configuration options, check the [cached decorator docs](https://aiocache.readthedocs.io/en/latest/decorators.html#cached)


You also have the possibility of creating a new cache instance explicitly and use it as you please:

```
import asyncio
from aiocache import Cache

from starlette.applications import Starlette
from starlette.responses import JSONResponse
import uvicorn


app = Starlette(debug=True)
cache = Cache(Cache.MEMORY, ttl=5)


async def expensive():
    await asyncio.sleep(5)
    return {'hello': 'world'}


@app.route('/')
async def homepage(request):
    cached_result = await cache.get('key')
    if cached_result:
        return JSONResponse(cached_result)

    result = await expensive()
    await cache.set('key', result)
    return JSONResponse(result)

if __name__ == '__main__':
    uvicorn.run(app, host='0.0.0.0', port=8000)
```

Here we are using the [cache](https://aiocache.readthedocs.io/en/latest/caches.html#cache) class to instantiate a memory cache at startup and in the `homepage` function we've added the cache behavior. To avoid having hardcoded configuration for the cache in code, we can use the config module:

```

import asyncio
from aiocache import Cache

from starlette.applications import Starlette
from starlette.config import Config
from starlette.responses import JSONResponse
import uvicorn

config = Config(".env")
DEBUG = config("DEBUG", cast=bool)
CACHE_URL = config("CACHE_URL")

app = Starlette(debug=DEBUG)
cache = Cache.from_url(CACHE_URL)


async def expensive():
    await asyncio.sleep(5)
    return {'hello': 'world'}


@app.route('/')
async def homepage(request):
    cached_result = await cache.get('key')
    if cached_result:
        return JSONResponse(cached_result)

    result = await expensive()
    await cache.set('key', result)
    return JSONResponse(result)

if __name__ == '__main__':
    uvicorn.run(app, host='0.0.0.0', port=8000)
```

being the `.env` config file:

```
DEBUG=true
CACHE_URL="memory://?ttl=5"
```

Given you can use cache instances as you wish, one interesting use case is having a generic http cache middleware for Starlette:

```
TODO
```
