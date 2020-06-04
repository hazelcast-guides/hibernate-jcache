# Using Jcache SPI for using Hazelcast as L2C for Hibernate

This is an alternative approach of using `Hazelcast-hibernate` plugin to use Hazelcast as the caching provider for Hibernate.
The plugin can be found [here](https://www.github.com/hazelcast/hazelcast-hibernate5). Note that the local region caches supported by
the plugin are not available here. Unlike plugin - which stores query results locally, this way supports query results to be stored in
distributed map. 

## About Jcache

[JCache (JSR 107)](https://www.javadoc.io/doc/javax.cache/cache-api/1.1.1) is a standard caching API for Java. 
It provides an API for applications to be able to create and work with in-memory cache of objects. Hibernate has 
started to support Jcache implementations for L2C with version 5.2. Using only a CachingProvider implementation,
L2C can be used for SessionFactory objects in Hibernate. 

## Dependencies

- Include `hibernate-jcache` dependency. The version must be the same as 
`hibernate-core` version: 

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jcache</artifactId>
    <version>${version.hibernate-core}</version>
</dependency>
```

- Use `hazelcast-client` or `hazelcast` dependency to use client or server mode respectively. Note that starting from version 4.0,
 `hazelcast-client` is not a separate package but placed in `hazelcast`.
```xml
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <!-- artifactId>hazelcast-client</artifactId -->
    <version>${version.hazelcast}</version>
</dependency>

```  

### Hibernate JCache Region Factory 

In Hibernate configuration:

For Hibernate 5.2.x:
```xml
<property name="hibernate.cache.region.factory_class">org.hibernate.cache.jcache.JCacheRegionFactory</property>
```

For Hibernate 5.3.0 and higher:
```xml
<property name="hibernate.cache.region.factory_class">jcache</property>
```
### Hazelcast Provider Type


`CLIENT`: You can connect to an existing Hazelcast cluster and store the cache data there. Then you can scale the cluster
up or down independent from your app. Data resides in L2C can be available even a SessionFactory is closed
or application is shut down. So this makes L2C available across SessionFactories and Hibernate applications. 

To try client version with the code sample, run `StartCluster` first to create a cluster.
            
`EMBEDDED`: A new Hazelcast instance is created along with your application. This instance cannot be scaled up or 
down unless you run or shutdown another Hibernate application with the same server config. Latency of communicating to 
Hazelcast Cluster will be less than the client mode.
          
You can learn more about client and embedded version [here](https://hazelcast.zendesk.com/hc/en-us/articles/115004441586-What-s-the-difference-between-client-server-vs-embedded-topologies-).

In Hibernate configuration:

- Using Native Client
```xml
<property name="hibernate.javax.cache.provider">com.hazelcast.client.cache.impl.HazelcastClientCachingProvider</property>
<property name="hibernate.javax.cache.uri">hazelcast-client.xml</property>
```

- Using Embedded Hazelcast Instance
```xml
<property name="hibernate.javax.cache.provider">com.hazelcast.cache.impl.HazelcastServerCachingProvider</property>
<property name="hibernate.javax.cache.uri">hazelcast.xml</property>
```

### Cache Configuration

To see your cache statistics on Hazelcast Management Center, you have to enable statistics 
(disabled by default) for the cache configuration of Hazelcast instance. 
In `hazelcast.xml, see ` <cache>` part:

```xml
<cache name="custom.cache.region.name">
    <statistics-enabled>true</statistics-enabled>
    <management-enabled>true</management-enabled>
</cache> 
```

***NOTE: If client mode is used, map configurations must be set on the server configuration, not on the client side.***

If you do not define a specific region name, the default cache configuration is going to be
used for this region, hence will not be visible on Management Center. To apply
a config to all regions, you can use wildcards for `name` property (e.g. `name="*"`). 

For other cache configurtion properties, see `cache` configuration section on document 
[here](https://docs.hazelcast.org/docs/latest/manual/html-single/index.html#icache-configuration).
Eviction size and policy properties might be useful for this case.
