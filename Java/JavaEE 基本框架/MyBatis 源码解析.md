# MyBatis 源码解析

记录mybatis源码解析。



## MyBatis的Cache接口

简介：MyBatis 的 Cache接口是作为mybatis引用外部缓存中间件（如Redis等）所必须要实现的接口。

源码及解释：

```java
package org.apache.ibatis.cache;

import java.util.concurrent.locks.ReadWriteLock;

/**
 * SPI for cache providers.
 翻译：这是作为缓存提供商的外置接口
 * One instance of cache will be created for each namespace.
 翻译：每个命名空间创建一个缓存示例（一个命名空间映射一个dao类）
 * The cache implementation must have a constructor that receives the cache id as an String parameter.
 翻译：缓存的实现类必须有一个可以接受形参为 String 类型，变量名为 id 的构造器。
 * MyBatis will pass the namespace as id to the constructor.
 翻译：MyBatis 将会通过命名空间作为 id 传入构造器
 * 
 *
 * <pre>
 解释：自定义构造器应该如下所示：
 * public MyCache(final String id) {
 *   if (id == null) {
 *     throw new IllegalArgumentException("Cache instances require an ID");
 *   }
 *   this.id = id;
 *   initialize();
 * }
 * </pre>
 *
 * @author Clinton Begin
 */

public interface Cache {

    /**
   * @return The identifier of this cache	
   外部调用，返回key即可
    */
    String getId();

    /**
   * @param key		Can be any object but usually it is a {@link CacheKey}	// 
   * @param value	The result of a select.
   将键值对放到自定义的 cache 中即可
   */
    void putObject(Object key, Object value);

    /**
   * @param key	The key
   * @return 	The object stored in the cache.
   通过key 从自定义的cache中获取value
   */
    Object getObject(Object key);

    /**
   * As of 3.3.0 this method is only called during a rollback
   * for any previous value that was missing in the cache.
   * This lets any blocking cache to release the lock that
   * may have previously put on the key.
   * A blocking cache puts a lock when a value is null
   * and releases it when the value is back again.
   * This way other threads will wait for the value to be
   * available instead of hitting the database.
   *
   *
   * @param key		The key
   * @return 		Not used
   */
    Object removeObject(Object key);

    /**
   * Clears this cache instance.
   清空自定义缓存，默认情况下在增删改时调用 （<insert><delete><update>），在每个Mapper 映射文件中可以指定
   */
    void clear();

    /**
   * Optional. This method is not called by the core.
   *
   * @return The number of elements stored in the cache (not its capacity).
   返回缓存中元素的个数，一般返回缓存中实际生效的value的个数
   */
    int getSize();

    /**
   * Optional. As of 3.2.6 this method is no longer called by the core.
   * <p>
   * Any locking needed by the cache must be provided internally by the cache provider.
   *
   * @return A ReadWriteLock
   */
    default ReadWriteLock getReadWriteLock() {
        return null;
    }

}
```

**引用自定义cache**：在mapper映射文件中，使用 `<cache type="自定义cache全类名 ">` 指定自定义的 cache，该cache需要实现ibatis包中的Cache接口；