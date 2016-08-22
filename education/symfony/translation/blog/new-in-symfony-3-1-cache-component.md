http://symfony.com/blog/new-in-symfony-3-1-cache-component

# New in Symfony 3.1: Cache component

# Symfony 3.1新特性：cache组件

This is the last article in the "New in Symfony 3.1" series and it introduces the most important new feature of Symfony 3.1: the **Cache component**.

这是"Symfony 3.1新特性"系列文章的最后一篇，我们将介绍Symfony 3.1新特性中最重要的一个特性：cache组件。

This new component is a strict implementation of the PSR-6: Caching Interface standard. You can use it to cache arbitrary content in your application and some Symfony components use it internally to improve their performance.

这个组件严格遵循并实现了"PSR-6 Caching接口"标准。你可以在你的应用里用它来缓存任意的内容，一些symfony组件内部也用它来提升性能。

The following example shows how to create, save and delete information in a filesystem-based cache:

下面的例子将会演示如何在一个给予文件系统的cache中，创建，保存，删除信息。

```php
use Symfony\Component\Cache\Adapter\FilesystemAdapter;

// available adapters: filesystem, apcu, redis, array, doctrine cache, etc.
$cache = new FilesystemAdapter();

// create a new item getting it from the cache
$numProducts = $cache->getItem('stats.num_products');

// assign a value to the item and save it
$numProducts->set(4711);
$cache->save($numProducts);

// retrieve the cache item
$numProducts = $cache->getItem('stats.num_products');
if (!$numProducts->isHit()) {
        // ... item does not exists in the cache
}
// retrieve the value stored by the item
$total = $numProducts->get();

// remove the cache item
$cache->deleteItem('stats.num_products');
```

The Cache component provides adapters for the most common caching backends (Redis, APCu), it's compatible with every Doctrine Cache adapter (Memcache, MongoDB, Riak, SQLite, etc.) and it also provides two special adapters (Proxy and Chain) for advanced setups.

cache组件为大多数常用的cache后端(Redis, APCu)提供了适配器，它和Doctrine的每一个cache适配器(Memcache, MongoDB, Riak, SQLite, etc.)也是兼容的，并且它还提供了2个特殊的适配器(代理 和 链)用做高级用途。

For example, if your application uses Redis, the example shown above is still valid. You just need to change the instantiation of the cache adapter and leave the rest of the code unchanged:

举例，如果你的应用使用Redis，上面的那个例子依然有效。你只需要修改cache适配器初始化的部分，剩下的其他代码都不用改。

```php
use Symfony\Component\Cache\Adapter\RedisAdapter;

$cache = new RedisAdapter($redisConnection);
// ...
```

The documentation of the Cache component is already finished and it will be merged soon into the official Symfony docs.

cache组件的文档已经完成了，马上就会合并到symfony官方文档中。

**Symfony Integration**

**Symfony 集成**

The Cache component is ready to be used in any PHP application, but if you use the Symfony framework, the component is already integrated. Symfony defines two different cache pools: cache.app is where you store the information generated by your own application; cache.system is where Symfony components store their contents (e.g. the Serializer and Validator metadata).

cache组件已经可以在任何php应用中使用。但是如果你使用symfony框架，这个组件已经被集成了。symfony定义了2个不同的cache池：cache.app是用来存储你应用生成的信息；cache.system是symfony组件存储内容的地方（举例：序列化和验证的元数据）。

```php
// src/AppBundle/Controller/BlogController.php
class BlogController extends Controller
{
    public function indexAction()
    {
        $cachedCategories = $this->get('cache.app')->getItem('categories');
        if (!$cachedCategories->isHit()) {
            $categories = ... // fetch categories from the database
            $cachedCategories->set($categories);
            $this->get('cache.app')->save($cachedCategories);
        } else {
            $categories = $cachedCategories->get();
        }

        // ...
    }
}
```

If your server has APCu installed, the cache.system pool uses it. Otherwise, it falls back to the filesystem cache. For the cache.app pool is recommended to use a proper cache backend such as Redis:

如果你的服务器安装了APCu，cache.system池就会使用它。否则，它会使用文件系统cache。cache.app池推荐用一个合适的cache后端，比如Redis：

```yml
# app/config/config_prod.yml
framework:
    cache:
        app: cache.adapter.redis
        default_redis_provider: "redis://localhost"
```

You can also create your own custom cache pools and they can even be based on the configuration of the default cache.app pool:

你也可以创建你自己的自定义池，他们甚至能基于默认的cache.app池的配置：

```yml
# app/config/config_prod.yml
framework:
    cache:
        # ...
        pools:
            app.cache.customer:
                adapter: cache.app
                default_lifetime: 600
```