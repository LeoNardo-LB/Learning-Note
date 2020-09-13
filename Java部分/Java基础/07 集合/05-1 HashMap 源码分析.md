# HashMap 源码分析

注意：请参照JDK1.8的HashMap源码

### HashMap流程图

![hashmap1](_images/hashmap1.png)

## HashMap的几个变量与常量

### 变量

```java
/**
 * The load factor for the hash table.
 * 加载因子，
 * @serial
 */
final float loadFactor;

/**
     * The next size value at which to resize (capacity * load factor).
     * 扩容阈值，当数组长度达到扩容阈值时，即开始扩容操作。（加载因子 * 数组长度 = 扩容阈值）
     * @serial
     */
// (The javadoc description is true upon serialization.
// Additionally, if the table array has not been allocated, this
// field holds the initial array capacity, or zero signifying
// DEFAULT_INITIAL_CAPACITY.)
int threshold;

/**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     * 修改次数，每进行对HashMap进行以此结构性修改时，修改次数+1；要求在迭代器迭代过程中，
     * 不能改变修改次数（即为不可进行结构性修改）。否则将抛出ConcurrentModificationException
     */
transient int modCount;

/**
     * The number of key-value mappings contained in this map.
     * 该Map中键值对的个数
     */
transient int size;

/**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     * 是一个单向链表的数组。该变量时HashMap实际上存储数据的结构，底层实现了 Map.Entry<K,V> 接口，存放键值对。
     */
transient Node<K,V>[] table;

/**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     * 用于存放键值对的结构，在第一次调用 HashMap对象的entrySet()方法时进行初始化。
     */
transient Set<Map.Entry<K,V>> entrySet;
```

### 常量

```java
/**
     * The default initial capacity - MUST be a power of two.
     * HashMap默认容量
     */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     * HashMap最大容量
     */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
     * The load factor used when none specified in constructor.
     * 默认加载因子
     */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     * 数组转成红黑树的最小链表长度的阈值
     */
static final int TREEIFY_THRESHOLD = 8;

/**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     * 红黑树转化位数组的最小红黑树节点个数的阈值
     */
static final int UNTREEIFY_THRESHOLD = 6;

/**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     * 存储结构由链表转化为红黑树的最小键值对个数：单个链表在长度达到8个之后，该链表会转化为红黑树存储
     */
static final int MIN_TREEIFY_CAPACITY = 64;

```



## HashMap 源码（常用方法）

#### 无参构造器：public HashMap()

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 * 构造一个空的HashMap对象，使用默认容量与默认加载因子。
 *
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

说明：此方法只是给加载因子赋初始值（0.75），没有其他操作。

#### 带参构造器：public HashMap(int initialCapacity)

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and the default load factor (0.75).
 * 使用指定容量构造一个空的HashMap，使用默认加载因子0.75
 *
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

进入`this(initialCapacity, DEFAULT_LOAD_FACTOR);`

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 * 使用指定的容量与加载因子初始化一个空的HashMap
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)	// 如果传入的容量小于0，则抛出异常
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)	//若传入值大于最大容量
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))	// 加载因子校验：小于0或非Float类型，则抛出异常
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // 初始化加载因子
    this.loadFactor = loadFactor;
    // 初始化扩容阈值，下面有说明
    this.threshold = tableSizeFor(initialCapacity);
}
```

`tableSizeFor(initialCapacity)`方法说明：该方法会返回一个2的n次方的值，该值的取值是大于传入值且范围内最小的值，比如传入值位13，返回16（2的4次方），传入50，返回64（2的6次方）

#### 添加方法：public V put(K key, V value)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

首先调用`hash(key)`方法，计算出key对应的哈希值

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

接下来调用`putVal(hash(key), key, value, false, true)`方法，该方法将元素放入HashMap中（代码有格式的修改）

```java
/**
 * Implements Map.put and related methods.
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; 	// 保存当前 Node数组的临时变量
    Node<K,V> p; 	// 用于链表遍历的节点
    int n, i;	// n用于保存tab数组的长度；i用于指定tab数组的索引
    // 给刚刚定义的tab变量赋值，并且判断其是否为空（或长度位0），若为空，则调用resize()方法扩容；
    // resize()方法后面有详解
    if ((tab = table) == null || (n = tab.length) == 0)	
        n = (tab = resize()).length;
    // 计算节点放入的索引（通过与Hash算法），然后赋值给P，判断P是否为空。若为空，则新建一个节点，然后放在tab数组索引指向的位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 否则，就说明该索引位置有Node元素，接下来进行如下操作。
    else {
        Node<K,V> e; 
        K k;
        // 若哈希值相等，且内容或引用相等，则覆盖
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

