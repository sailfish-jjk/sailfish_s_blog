---
layout: post
title: Openfire Cache
category: notes
---

Openfire作为一个聊天服务器，已经把调用频繁的接口进行了缓存。例如userManager中的userCache，rosterManager 中的rosterCache，曾经让我很郁闷的vcardCache等。

想吃掉这个总是捣乱的Openfire，唯一的办法就是搞懂他，迭代空挡，整理一下Openfire的缓存机制。

	interface Cache<K,V> extends java.util.Map<K,V>  

提供了基本的缓存接口，默认是通过DefaultCache类实现的。该类提供了cache操作的基本方法：put、get、remove、getCacheSize等。

Openfire通过CacheFactory管理cache创建，提供了一个统一的创建和使用Cache的工厂。

	/**
     * Storage for all caches that get created.
     */
	private static Map<String, Cache> caches = new ConcurrentHashMap<String, Cache>();
	   /**
     * This map contains property names which were used to store cache configuration data
     * in local xml properties in previous versions.
     */
    private static final Map<String, String> cacheNames = new HashMap<String, String>();
    /**
     * Default properties to use for local caches. Default properties can be overridden
     * by setting the corresponding system properties.
     */
    private static final Map<String, Long> cacheProps = new HashMap<String, Long>();

以上三个变量分别存储所有已创建的cache，已创建的cache的名称，已创建cache的属性。需要注意的是对Cache的操作需要考虑线程的同步和互斥，Openfire采用了ConcurrentHashMap来保证线程安全。

放入Cache的对象要实现Cacheable接口，可以看出，这是一个序列化的接口。并实现getCachedSize()方法，返回放入对象的size。

	public interface Cacheable extends java.io.Serializable {

	    /**
	     * Returns the approximate size of the Object in bytes. The size should be
	     * considered to be a best estimate of how much memory the Object occupies
	     * and may be based on empirical trials or dynamic calculations.<p>
	     *
	     * @return the size of the Object in bytes.
	     */
	    public int getCachedSize() throws CannotCalculateSizeException;
	}

例如rosterCache中的缓存对象Roster，其计算cachedSize方法如下：

    public int getCachedSize() throws CannotCalculateSizeException {
        // Approximate the size of the object in bytes by calculating the size
        // of the content of each field, if that content is likely to be eligable for
        // garbage collection if the Roster instance is dereferenced.
        int size = 0;
        size += CacheSizes.sizeOfObject();                           // overhead of object
        size += CacheSizes.sizeOfCollection(rosterItems.values());   // roster item cache
        size += CacheSizes.sizeOfString(username);                   // username

        // implicitFrom
        for (Map.Entry<String, Set<String>> entry : implicitFrom.entrySet()) {
            size += CacheSizes.sizeOfString(entry.getKey());
            size += CacheSizes.sizeOfCollection(entry.getValue());
        }

        return size;
    }

Roster对象的size用CacheSizes.sizeOfObject()、CacheSizes.sizeOfCollection(rosterItems.values())、CacheSizes.sizeOfString(username)等来计算。


CacheFactoryStrategy是Openfire缓存策略的接口，默认实现是DefaultLocalCacheStrategy，该实现不支持集群。另外一个实现类：ClusteredCacheFactory则是缓存集群的解决方案。

-EOF-
