# LRU

LRU的实现比较简单，因为java中的LinkedHashMap有很多特点正好适合LRU的思想。LRU（least recently used）最近最少使用，首先淘汰最长时间未被使用的页面。使用链表实现，当一个元素被访问了，把元素移动到链表的顶端，插入新元素也放到顶端，当缓存数量到达限制，直接从链表底端移除元素。

```jade
public class LRU<k, v> extends LinkedHashMap<k, v> {

  private final int MAX_SIZE;

  public LRU(int capcity) {
    super(8, 0.75f,true);
    this.MAX_SIZE = capcity;
  }

  @Override
  public boolean removeEldestEntry(Map.Entry<k, v> eldest) {
    if (size() > MAX_SIZE) {
      System.out.println("移除的元素为：" + eldest.getValue());
    }
    return size() > MAX_SIZE;
  }

  public static void main(String[] args) {
    Map<Integer, Integer> map = new LRU<>(5);
    for (int i = 1; i <= 11; i++) {
      map.put(i, i);
      System.out.println("cache的容量为：" + map.size());
      if (i == 4) {
        map.get(1);
      }
    }

    System.out.println("=-=-=-=-=-=-=-map元素:");
    map.entrySet().forEach(integerIntegerEntry -> System.out.println(integerIntegerEntry.getValue()));

  }

}
```

> LinkedHashMap有一个 removeEldestEntry(Map.Entry eldest)方法，通过覆盖这个方法，加入一定的条件，满足条件返回true。当put进新的值方法返回true时，便移除该map中最老的键和值。



# LFU

LFU(Least Frequently Used)淘汰一定时期内被访问次数最少的元素。如果元素的一定时间内的访问次数相同时，则比较他们的最新一次的访问时间。

```java
public class LFU<k, v> {
  private final int capcity;

  private Map<k, v> cache = new HashMap<>();

  private Map<k, HitRate> count = new HashMap<>();

  public LFU(int capcity) {
    this.capcity = capcity;
  }

  public void put(k key, v value) {
    v v = cache.get(key);
    if (v == null) {
      if (cache.size() == capcity) {
        removeElement();
      }
      count.put(key, new HitRate(key, 1, System.nanoTime()));
    } else {
      addHitCount(key);
    }
    cache.put(key, value);
  }

  public v get(k key) {
    v value = cache.get(key);
    if (value != null) {
      addHitCount(key);
      return value;
    }
    return null;
  }

  //移除元素
  private void removeElement() {
    HitRate hr = Collections.min(count.values());
    cache.remove(hr.key);
    count.remove(hr.key);
  }

  //更新访问元素状态
  private void addHitCount(k key) {
    HitRate hitRate = count.get(key);
    hitRate.hitCount = hitRate.hitCount + 1;
    hitRate.lastTime = System.nanoTime();
  }

  //内部类
  class HitRate implements Comparable<HitRate> {
    private k key;
    private int hitCount;
    private long lastTime;

    private HitRate(k key, int hitCount, long lastTime) {
      this.key = key;
      this.hitCount = hitCount;
      this.lastTime = lastTime;
    }

    @Override
    public int compareTo(HitRate o) {
      int compare = Integer.compare(this.hitCount, o.hitCount);
      return compare == 0 ? Long.compare(this.lastTime, o.lastTime) : compare;
    }
  }


  public static void main(String[] args) {
    LFU<Integer, Integer> cache = new LFU<>(3);
    cache.put(2, 2);
    cache.put(1, 1);

    System.out.println(cache.get(2));
    System.out.println(cache.get(1));
    System.out.println(cache.get(2));

    cache.put(3, 3);
    cache.put(4, 4);

    //1、2元素都有访问次数，放入3后缓存满，加入4时淘汰3
    System.out.println(cache.get(3));
    System.out.println(cache.get(2));
    //System.out.println(cache.get(1));
    System.out.println(cache.get(4));

    cache.put(5, 5);
    //目前2访问2次，1访问一次，4访问一次，由于4的时间比较新，放入5的时候移除1元素。
    System.out.println("-=-=-=-");
    cache.cache.entrySet().forEach(entry -> {
      System.out.println(entry.getValue());
    });

  }
}
```

