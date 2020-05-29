# HashMap

https://blog.csdn.net/v123411739/article/details/78996181



# ConcurrentHashMap

jdk1.7 使用segment分段锁

### jdk1.8

get()

> 使用volatile修饰变量，直接get，不会获取到旧值。
>
> ```java
> public V get(Object key) {
>         Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
>         int h = spread(key.hashCode());//散列 得到key在数组中的位置
>         if ((tab = table) != null && (n = tab.length) > 0 &&
>             (e = tabAt(tab, (n - 1) & h)) != null) {
>             if ((eh = e.hash) == h) {
>                 if ((ek = e.key) == key || (ek != null && key.equals(ek)))
>                     return e.val;
>             }
>             else if (eh < 0)
>                 return (p = e.find(h, key)) != null ? p.val : null;
>             while ((e = e.next) != null) {
>                 if (e.hash == h &&
>                     ((ek = e.key) == key || (ek != null && key.equals(ek))))
>                     return e.val;
>             }
>         }
>         return null;
>     }
> @SuppressWarnings("unchecked")
> static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
>     return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
> }
> ```
>
> 

put()

> 使用cas和synchronized：
>
> ```java
> public V put(K key, V value) {
>     return putVal(key, value, false);
> }
> 
> /** Implementation for put and putIfAbsent */
> final V putVal(K key, V value, boolean onlyIfAbsent) {
>     if (key == null || value == null) throw new NullPointerException();
>     int hash = spread(key.hashCode());
>     int binCount = 0;
>     for (Node<K,V>[] tab = table;;) {
>         Node<K,V> f; int n, i, fh;
>         if (tab == null || (n = tab.length) == 0)
>             tab = initTable();
>         else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
>             // put到空容器中 不需要锁 cas即可
>             if (casTabAt(tab, i, null,
>                          new Node<K,V>(hash, key, value, null)))
>                 break;   // no lock when adding to empty bin
>         }
>         else if ((fh = f.hash) == MOVED)
>             // 如果正在调整大小，则帮助转移
>             tab = helpTransfer(tab, f);
>         else {
>             V oldVal = null;
>             // 否则就是添加新元素。需要synchronized 获取当前node的锁
>             synchronized (f) {
>                 if (tabAt(tab, i) == f) {
>                     if (fh >= 0) {
>                         binCount = 1;
>                         for (Node<K,V> e = f;; ++binCount) {
>                             K ek;
>                             if (e.hash == hash &&
>                                 ((ek = e.key) == key ||
>                                  (ek != null && key.equals(ek)))) {
>                                 oldVal = e.val;
>                                 if (!onlyIfAbsent)
>                                     e.val = value;
>                                 break;
>                             }
>                             Node<K,V> pred = e;
>                             if ((e = e.next) == null) {
>                                 pred.next = new Node<K,V>(hash, key,
>                                                           value, null);
>                                 break;
>                             }
>                         }
>                     }
>                     else if (f instanceof TreeBin) {
>                         Node<K,V> p;
>                         binCount = 2;
>                         if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
>                                                               value)) != null) {
>                             oldVal = p.val;
>                             if (!onlyIfAbsent)
>                                 p.val = value;
>                         }
>                     }
>                 }
>             }
>             if (binCount != 0) {
>                 if (binCount >= TREEIFY_THRESHOLD)
>                     treeifyBin(tab, i);
>                 if (oldVal != null)
>                     return oldVal;
>                 break;
>             }
>         }
>     }
>     addCount(1L, binCount);
>     return null;
> }
> 
> @SuppressWarnings("unchecked")
> static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
>     return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
> }
> 
> // 使用cas操作
> static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
>                                     Node<K,V> c, Node<K,V> v) {
>     return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
> }
> ```
>
> 