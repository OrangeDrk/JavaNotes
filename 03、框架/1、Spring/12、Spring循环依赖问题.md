[TOC]

# 3级缓存解决循环依赖

3个map

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    ...
        // 从上至下 分表代表这“三级缓存”
        private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
    private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
    ...

        /** Names of beans that are currently in creation. */
        // 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
        // 它在Bean开始创建时放值，创建完成时会将其移出~
        private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

    /** Names of beans that have already been created at least once. */
    // 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
    // 至少被创建了一次的  都会放进这里~~~~
    private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```

