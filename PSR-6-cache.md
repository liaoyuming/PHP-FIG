# 缓存接口

缓存是提升项目性能的一种常见方式，也是许多框架和库最常见的功能之一。这导致，许多库都有自己的，功能各异的缓存库。这种差异会使开发人员不得不学习多种系统，而且系统可能没有提供他们所需要的功能。此外，缓存库的开发者同样面临着一个窘境，是只支持有限数量的几个框架还是创建一堆庞 大的适配器类。

一个通用的缓存系统接口应该解决掉这些问题。库和框架的开发人员能够知道缓存系统会按照他们所 预期的方式工作，缓存系统的开发人员只需要实现单一的接口，而不是各种各样的适配器。

关键词 “必须”("MUST")、“一定不可/一定不能”("MUST NOT")、“需要”("REQUIRED")、 “将会”("SHALL")、“不会”("SHALL NOT")、“应该”("SHOULD")、“不该”("SHOULD NOT")、 “推荐”("RECOMMENDED")、“可以”("MAY")和”可选“("OPTIONAL")的详细描述可参见 [RFC 2119]( http://tools.ietf.org/html/rfc2119)。

## 目标

本 PSR 的目标是创建一套通用的接口规范，能够让开发人员整合到现有框架和系统，而不需要特定的开发。

## 定义

* **调用类库 (Calling Library)** - 实际需要缓存服务的库或代码，该库将调用实现本标准接口的缓存服务，而不需要知道这些缓存服务的具体实现。

* **实现类库 (Implementing Library)** - 该库负责实现此标准，供调用类库 (Calling Library)使用。实现类库 (Implementing Library)必须提供实现 Cache\CacheItemPoolInterface 和Cache\CacheItemInterface 接口的类。 实现类库 (Implementing Library)必须支持下面描述的秒级别的粒度的最小 TTL 的功能。

* **生存时间值 (TTL - Time To Live)** - 项目的生存时间值 (TTL)是存储该项目到被认为失效的时间量。TTL 通常被定义为以秒为单位的整数或 DateInterval 对象。

* **过期时间 (Expiration)** - 项目被设置为失效的实际时间。一般通过将 TTL 加上存储对象的时间来计算，也可以使用 DataTime 对象来显式设置。


    假如一个设置为 300s TTL 缓存项，保存于 1:30:00 ，那么它的过期时间为 1:35:00。

    实现库**可以**在要求的到期时间之间过期缓存项，但是在到达过期时间后，**必须**将该项目设置为过期。如果一个调用类库要求保存某缓存项，但没有指定到期时间，或者指定了一个空的到期时间或 TTL，则实现类库**可以**使用配置的默认时间。如果没有设置默认时间，则实现类库**必须**将其按永久或是底层实现支持的最长时间来缓存项目

* **键 (KEY)** 至少一个字符的字符串，用于唯一标识缓存的项目。实现类库**必须**支持 UTF-8 编码的，由“A-Z”，“a-z”，“0-9”，“_"和“。”字符任何顺序组成的，长度最多为64个字符的键。实现类库**可以**支持额外的字符和编码，以及更长的长度，但必须至少支持前面指定的最小值。类库适合自己转义键字符串，但**必须**能够返回原始的未修改的键字符串。

* **命中 (Hit)**

* **未命中 (Miss)**

* **延迟 (Deferred)**

## 数据

## 接口

### CacheItemInterface

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemInterface 定义了一个用于与缓存对象进行交互的接口
 */
interface CacheItemInterface
{
    /**
     * 返回当前缓存项的键（key）
     *
     * 键（key）由实现类库来加载，但是在需要的时候，应该提供给更高级别的调用者
     *
     * @return string
     *   缓存项的键（key）字符串
     */
    public function getKey();

    /**
     * Retrieves the value of the item from the cache associated with this object's key.
     *
     * The value returned must be identical to the value originally stored by set().
     *
     * If isHit() returns false, this method MUST return null. Note that null
     * is a legitimate cached value, so the isHit() method SHOULD be used to
     * differentiate between "null value was found" and "no value was found."
     *
     * @return mixed
     *   The value corresponding to this cache item's key, or null if not found.
     */
    public function get();

    /**
     * Confirms if the cache item lookup resulted in a cache hit.
     *
     * Note: This method MUST NOT have a race condition between calling isHit()
     * and calling get().
     *
     * @return bool
     *   True if the request resulted in a cache hit. False otherwise.
     */
    public function isHit();

    /**
     * Sets the value represented by this cache item.
     *
     * The $value argument may be any item that can be serialized by PHP,
     * although the method of serialization is left up to the Implementing
     * Library.
     *
     * @param mixed $value
     *   The serializable value to be stored.
     *
     * @return static
     *   The invoked object.
     */
    public function set($value);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param \DateTimeInterface|null $expiration
     *   The point in time after which the item MUST be considered expired.
     *   If null is passed explicitly, a default value MAY be used. If none is set,
     *   the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAt($expiration);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param int|\DateInterval|null $time
     *   The period of time from the present after which the item MUST be considered
     *   expired. An integer parameter is understood to be the time in seconds until
     *   expiration. If null is passed explicitly, a default value MAY be used.
     *   If none is set, the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAfter($time);

}
~~~

### CacheItemPoolInterface

Cache\CacheItemPoolInterface 的主要目的是接受调用类库(Calling Library)中的一个键值并返回相关联的 Cache\CacheItemInterface 对象。它也是整个缓存集合的主要交互点。池的所有配置和初始化都由实现类库(Implementing Library)决定。

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemPoolInterface generates CacheItemInterface objects.
 */
interface CacheItemPoolInterface
{
    /**
     * Returns a Cache Item representing the specified key.
     *
     * This method must always return a CacheItemInterface object, even in case of
     * a cache miss. It MUST NOT return null.
     *
     * @param string $key
     *   The key for which to return the corresponding Cache Item.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return CacheItemInterface
     *   The corresponding Cache Item.
     */
    public function getItem($key);

    /**
     * Returns a traversable set of cache items.
     *
     * @param string[] $keys
     *   An indexed array of keys of items to retrieve.
     *
     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return array|\Traversable
     *   A traversable collection of Cache Items keyed by the cache keys of
     *   each item. A Cache item will be returned for each key, even if that
     *   key is not found. However, if no keys are specified then an empty
     *   traversable MUST be returned instead.
     */
    public function getItems(array $keys = array());

    /**
     * Confirms if the cache contains specified cache item.
     *
     * Note: This method MAY avoid retrieving the cached value for performance reasons.
     * This could result in a race condition with CacheItemInterface::get(). To avoid
     * such situation use CacheItemInterface::isHit() instead.
     *
     * @param string $key
     *   The key for which to check existence.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if item exists in the cache, false otherwise.
     */
    public function hasItem($key);

    /**
     * Deletes all items in the pool.
     *
     * @return bool
     *   True if the pool was successfully cleared. False if there was an error.
     */
    public function clear();

    /**
     * Removes the item from the pool.
     *
     * @param string $key
     *   The key to delete.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the item was successfully removed. False if there was an error.
     */
    public function deleteItem($key);

    /**
     * Removes multiple items from the pool.
     *
     * @param string[] $keys
     *   An array of keys that should be removed from the pool.

     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the items were successfully removed. False if there was an error.
     */
    public function deleteItems(array $keys);

    /**
     * Persists a cache item immediately.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   True if the item was successfully persisted. False if there was an error.
     */
    public function save(CacheItemInterface $item);

    /**
     * Sets a cache item to be persisted later.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   False if the item could not be queued or if a commit was attempted and failed. True otherwise.
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Persists any deferred cache items.
     *
     * @return bool
     *   True if all not-yet-saved items were successfully saved or there were none. False otherwise.
     */
    public function commit();
}
~~~


### CacheException

此异常接口旨在当发生严重错误时使用，包括但不限于**缓存设置**，例如连接到缓存服务器或提供的无效凭据。

实现类库抛出的任何异常，必须实现此接口。

~~~php
<?php

namespace Psr\Cache;

/**
 * 由实现类库 (Implementing Library)抛出的所有异常的异常接口
 */
interface CacheException
{
}
~~~

### InvalidArgumentException

~~~php
<?php

namespace Psr\Cache;

/**
 * Exception interface for invalid cache arguments.
 *
 * Any time an invalid argument is passed into a method it must throw an
 * exception class which implements Psr\Cache\InvalidArgumentException.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~
