> &emsp;Hashtable、HashMap、TreeMap都是常见的map实现，是以键值对的形式存储的容器类型。Hashtable是早期Java类库提供的一个同步的哈希表实现，不支持null键和值。HashMap是应用较为广泛的哈希表实现，功能上与Hashtable类似，主要区别在于HashMap不是同步的，支持null的键和值。TreeMap则是基于红黑树的一种提供顺序访问的Map，具体顺序可以由指定的Comparator来决定，或者根据键的默认顺序。

### 一、HashMap
#### 1.1 内部结构

> &emsp;HashMap的内部结构可以看成是数组(Node<K,V>[])和链表组成的结构，数组被分为一个个bucket，通过哈希值决定了键值对在数组中的位置；哈希值相同的键值对则以链表形式存储，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019032822220665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzUwMjY1OA==,size_16,color_FFFFFF,t_70)

> &emsp;如上图，左侧是一个table即Entry数组，对应的每一节点则是Entry，在Jdk1.8中如果链表的大小超过了阀值则会被改造为树形结构，所以可以看到图中HashMap的表现为数组+链表+红黑树的形式。树的引入也是为了避免hash冲突元素过多时，链表结构造成的效率问题。

#### 1.2 源码理解

##### 1.2.1 重要属性
```java
	//默认容量
	static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
	//最大容量
	static final int MAXIMUM_CAPACITY = 1 << 30;
	//默认负载因子，用来计算扩容临界值
	static final float DEFAULT_LOAD_FACTOR = 0.75f;
	//当链表数量大于这个数会将链表转为树
	static final int TREEIFY_THRESHOLD = 8;
	//当链表数量小于这个数会将树转为链表
	static final int UNTREEIFY_THRESHOLD = 6;
	//桶中结构转化为红黑树对应的table的最小大小
	static final int MIN_TREEIFY_CAPACITY = 64;
	//存放元素的数组，是2的幂次
	transient Node<K,V>[] table;
	//存放元素具体的值
	transient Set<Map.Entry<K,V>> entrySet;
	//元素数量
	transient int size;
	//临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
	//更改map结构的计数器
	transient int modCount;
	//负载因子
	final float loadFactor;
```
##### 1.2.2 构造器

 1. public HashMap(int initialCapacity, float loadFactor)

```java
	//initialCapacity:初始化容量；loadFactor:负载因子
	public HashMap(int initialCapacity, float loadFactor) {
		//初始容量小于0则会报错
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //初始容量大于MAXIMUM_CAPACITY最大容量则默认为MAXIMUM_CAPACITY
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //如果loadFactor小于等于o 或者 loadFactor不是一个number则会报错
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        //参数校验完成后赋值，
        this.loadFactor = loadFactor;
        //初始化threshold大小
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    //返回大于输入参数且最近的2的整数次幂的数
    static final int tableSizeFor(int cap) {
	    int n = cap - 1;
	    n |= n >>> 1;
	    n |= n >>> 2;
	    n |= n >>> 4;
	    n |= n >>> 8;
	    n |= n >>> 16;
	    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
	}
```

 2. public HashMap(int initialCapacity)

```java
	public HashMap(int initialCapacity) {
	    // 调用HashMap(int, float)型构造函数
	    this(initialCapacity, DEFAULT_LOAD_FACTOR);
	}
```

 3. public HashMap()

```java
 	public HashMap() {
 		//初始化负载因子
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
    }
```
 4. public HashMap(Map<? extends K, ? extends V> m)

```java
	public HashMap(Map<? extends K, ? extends V> m) {
	    // 初始化填充因子
	    this.loadFactor = DEFAULT_LOAD_FACTOR;
	    // 将m中的所有元素添加至HashMap中
	    putMapEntries(m, false);
	}
	
	final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
        	//判断table是否已经初始化
            if (table == null) {
            	//计算初始化的阈值t
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                //计算得到的t大于阈值，则初始化阈值
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            //如果table已经初始化，并且元素个数大于阈值则进行扩容
            else if (s > threshold)
                resize();
            //将m中所有元素全部添加到HashMap中
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

##### 1.2.3 put() / putVal()

```java
	//HashMap对外提供的放入元素的方法是put()方法，put()方法实际是调用putVal()来实现放入元素的操作
	public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
	
	 /**
     * @param hash key的hash值
     * @param key 键
     * @param value 键对应的值
     * @param onlyIfAbsent 如果是true将不改变已存在的value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //步骤1:如果table还未被初始化，或者table的长度==0则调用resize()方法
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //步骤2:(n - 1) & hash确定元素在哪个桶中，桶如果为空，新生成Node结点放入桶中
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //如果桶中已经存在该元素
        else {
            Node<K,V> e; K k;
            //步骤3：如果key存在则直接覆盖value
            //比较桶中第一个元素(数组中的结点)的hash值相等，key相等
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //步骤4：如果该链是红黑树，将节点放入树中
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //步骤5：如果该链表是链表结构
            else {
                for (int binCount = 0; ; ++binCount) {
                    //到达链表的尾部
                    if ((e = p.next) == null) {
                    	//尾部插入新节点
                        p.next = newNode(hash, key, value, null);
                        //节点树达到阀值则将链表转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //判断链表中结点的key值与插入的元素的key值是否相等
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                    p = e;
                }
            }
            //找到key值、hash值与插入元素相等的结点
            if (e != null) {
            	//记录value值
                V oldValue = e.value;
                //onlyIfAbsent为false或者旧的value等于null，则用新值替换旧值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                //回调
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //结构性修改
        ++modCount;
        //步骤6：超过阀值则扩容
        if (++size > threshold)
            resize();
        //回调
        afterNodeInsertion(evict);
        return null;
    }
```
存储流程：

 1. 根据key计算得到key.hash = (h = k.hashCode()) ^ (h >>> 16)；
 2. 根据key.hash计算得到桶数组的索引index = key.hash & (table.length - 1)
 3. ① 如果该位置没有数据，用该数据新生成一个节点保存新数据，返回null
 4. ② 如果该位置有数据且是一个红黑树，那么执行相应的插入 / 更新操作
 5. ③ 如果该位置有数据且是一个链表，分两种情况一是该链表没有这个节点，另一个是该链表上有这个节点
 6. 如果该链表没有这个节点，那么采用尾插法新增节点保存新数据，返回null；如果该链表已经有这个节点了，那么找到该节点并更新新数据，返回老数据。

##### 1.2.4 get()

```java
	//get value by key
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
	
	 /**
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //table不等于null 并且 table数组长度大于0 并且 根据hash对应的桶的位置不能于null
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //与桶中第一项相等则直接返回第一个Node
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //桶中不至一个节点
            if ((e = first.next) != null) {
            	//如果是红黑树，则在红黑树中查找
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //如果是链表，则遍历链表查找
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
    
```

##### 1.2.5 resize

> resize方法是在hashmap中的键值对大于阀值时或者初始化时，就调用resize方法进行扩容
> 每次扩展的时候，都是扩展原大小2倍
> 扩展后Node对象的位置要么在原位置，要么移动到原偏移量两倍的位置

```java
	final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {//如果table不为空
            if (oldCap >= MAXIMUM_CAPACITY) {//如果原数组容量大于MAXIMUM_CAPACITY，则赋值为整数的最大阀值
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //如果原数组扩容两倍小于MAXIMUM_CAPACITY 并且 原数组大于等于初始化容量
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; //双倍扩容阀值
        }
        else if (oldThr > 0) //table为空 初始化阀值不为空
            newCap = oldThr;
        else { 
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //计算阀值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//根据newCap新建table
        table = newTab;
        if (oldTab != null) { //原table不等于空，则将原元素移到新的table
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {//如果原table j位置节点不为空
                    oldTab[j] = null;
                    if (e.next == null)//如果e的后面没有Node节点，则直接对e的hash值对新的数组长度求模获得存储位置
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)//如果e节点是红黑树的类型，那么添加到红黑树中
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
	                        next = e.next;
	                        if ((e.hash & oldCap) == 0) {//如果结点e的hash值与原hash桶数组的长度作与运算为0
	                            if (loTail == null)//如果loTail为null
	                                loHead = e;//将e结点赋值给loHead
	                            else
	                                loTail.next = e;//否则将e赋值给loTail.next
	                            loTail = e;//然后将e复制给loTail
	                        } else {
	                            if (hiTail == null)//如果hiTail为null
	                                hiHead = e;//将e赋值给hiHead
	                            else
	                                hiTail.next = e;//如果hiTail不为空，将e复制给hiTail.next
	                            hiTail = e;//将e复制个hiTail
	                        }
                    	} while ((e = next) != null);//直到e为空
                    if (loTail != null) {//如果loTail不为空
                        loTail.next = null;//将loTail.next设置为空
                        newTab[j] = loHead;//将loHead赋值给新的hash桶数组[j]处
                    }
                    if (hiTail != null) {//如果hiTail不为空
                        hiTail.next = null;//将hiTail.next赋值为空
                        newTab[j + oldCap] = hiHead;//将hiHead赋值给新的hash桶数组[j+旧hash桶数组长度]
                    }
                }
            }
        }
        return newTab;
    }
```
### 二、总结
1. HashMap的数据结构
&emsp;数组+链表+红黑树结构，数组是一个定长的Entry数组，put时会通过Hash算法进行计算元素的数组下标，在同一个位置有多个元素即Hash冲突时，会将元素加在链表的尾部，当链表大于8时，会将链表转为红黑树以提升效率。
2. 你知道hash的实现吗？为什么要这样实现？
&emsp;在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

3. 如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？
&emsp;如果超过了负载因子(默认0.75)，则会重新resize一个原来长度两倍的HashMap，并且重新调用hash方法。



