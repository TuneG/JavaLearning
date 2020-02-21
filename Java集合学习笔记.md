

### Java集合

#### HashMap

>JDK1.8 以前HashMap的实现是 数组+链表
>
>JDK1.8 开始HashMap的实现是 数组+链表+红黑树

JDK1.8以前，HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

![image-20191203163552149](D:\sgmuserprofile\sqgdua\Desktop\image-20191203163552149.png)

从上图中可以看出，HashMap 底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个 HashMap
的时候，就会初始化一个数组。

JDK1.8后，HashMap的实现是 数组+链表+红黑树，如图所示：

![1038293-20181024100214531-160606597](D:\sgmuserprofile\sqgdua\Desktop\1038293-20181024100214531-160606597.png)



如果不了解什么是红黑树的，可以参考以下几篇博客：

[面试旧敌之红黑树（直白介绍深入理解）]( https://juejin.im/entry/58371f13a22b9d006882902d )

[漫画：什么是红黑树？]( https://zhuanlan.zhihu.com/p/31805309 )

[红黑树详细分析，看了都说好]( https://segmentfault.com/a/1190000012728513 )

红黑树的定义：

1. 每个节点要么是红色，要么是黑色；
2. 根节点永远是黑色的；
3. 所有的叶节点都是是黑色的（注意这里说叶子节点其实是上图中的 NIL 节点）；
4. 每个红色节点的两个子节点一定都是黑色；
5. 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；

>性质 3 中指定红黑树的每个叶子节点都是空节点，而且并叶子节点都是黑色。但 Java 实现的红黑树将使用 null 来代表空节点，因此遍历红黑树时将看不到黑色的叶子节点，反而看到每个叶子节点都是红色的。
>
>性质 4 的意思是：从每个根到节点的路径上不会有两个连续的红色节点，但黑色节点是可以连续的。
>因此若给定黑色节点的个数 N，最短路径的情况是连续的 N 个黑色，树的高度为 N - 1（此次树高度的基数从0开始）;最长路径的情况为节点红黑相间，树的高度为 2(N - 1) 。
>
>性质 5 是成为红黑树最主要的条件，后序的插入、删除操作都是为了遵守这个规定。红黑树并不是标准平衡二叉树，它以性质 5 作为一种平衡方法，使自己的性能得到了提升。

BST（二分查找数）存在的主要问题是，数在插入的时候会导致树倾斜，不同的插入顺序会导致树的高度不一样，而树的高度直接的影响了树的查找效率。理想的高度是logN，最坏的情况是所有的节点都在一条斜线上，这样的树的高度为N。

红黑树的定义中的这些约束确保了红黑树的关键特性：从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。结果是这个树大致上是平衡的(平衡的复杂度为O(lgN)，简略可以说红黑树也是O(lgN))，严格的所需要数学证明。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

***简单的说，也就是红黑树查找效率比二叉查找树效率高！***



#### 阈值的确定

 再来看看HashMap类中有两个常量： 

~~~java
 /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;
~~~

当链表中节点数量大于等于TREEIFY_THRESHOLD时，链表会转成红黑树。

当链表中节点数量小于等于UNTREEIFY_THRESHOLD时，红黑树会转成链表。

为什么TREEIFY_THRESHOLD的默认值被设定为8？

HashMap中有这样一段注释

```java
/*	 
	 * Because TreeNodes are about twice the size of regular nodes, we
     * use them only when bins contain enough nodes to warrant use
     * (see TREEIFY_THRESHOLD). And when they become too small (due to
     * removal or resizing) they are converted back to plain bins.  In
     * usages with well-distributed user hashCodes, tree bins are
     * rarely used.  Ideally, under random hashCodes, the frequency of
     * nodes in bins follows a Poisson distribution
     * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
     * parameter of about 0.5 on average for the default resizing
     * threshold of 0.75, although with a large variance because of
     * resizing granularity. Ignoring variance, the expected
     * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
     * factorial(k)). The first values are:
     *
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * more: less than 1 in ten million
*/
```

TreeNodes占用空间是普通Nodes的两倍（相较于链表结构，链表只有指向下一个节点的指针，二叉树则需要左右指针，分别指向左节点和右节点），所以只有当bin包含足够多的节点时才会转成TreeNodes（考虑到时间和空间的权衡），而是否足够多就是由TREEIFY_THRESHOLD的值决定的。当bin中节点数变少时，又会转成普通的bin。并且我们查看源码的时候发现，链表长度达到8就转成红黑树，当长度降到6就转成普通bin。

这样就解析了为什么不是一开始就将其转换为TreeNodes，而是需要一定节点数才转为TreeNodes，说白了就是trade-off，空间和时间的权衡： 

***但为什么阈值就是8和6呢？中间的7是有什么作用的呢？***

当hashCode离散性很好的时候，树型bin用到的概率非常小，因为数据均匀分布在每个bin中，几乎不会有bin中链表长度会达到阈值。但是在随机hashCode下，离散性可能会变差，然而JDK又不能阻止用户实现这种不好的hash算法，因此就可能导致不均匀的数据分布。不过理想情况下随机hashCode算法下所有bin中节点的分布频率会遵循**泊松分布**，我们可以看到，一个bin中链表长度达到8个元素的概率为0.00000006，几乎是不可能事件。这种不可能事件都发生了，说明bin中的节点数很多，查找起来效率不高。所以，之所以选择8，不是拍拍屁股决定的，而是根据概率统计决定的。由此可见，发展30年的Java每一项改动和优化都是非常严谨和科学的。

>  泊松分布适合于描述单位时间（或空间）内随机事件发生的次数。如某一服务设施在一定时间内到达的人数，电话交换机接到呼叫的次数，汽车站台的候客人数，机器出现的故障数，自然灾害发生的次数，一块产品上的缺陷数，显微镜下单位分区内的细菌分布数等等。如果有兴趣的，可以研究一下，概率是怎么算出来的！

***画外音***

通过搜索引擎搜索这个问题，发现很多下面这个答案（猜测也是相互转发）：

红黑树的平均查找长度是log(n)，如果长度为8，平均查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，这才有转换成树的必要；链表长度如果是小于等于6，6/2=3，而log(6)=2.6，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

笔者认为这个答案不够严谨：3相比4有转换的必要，而2.6相比3就没有转换的必要？起码我不敢苟同这个观点。



#### 死磕源码

HashMap实现了Map接口，继承AbstractMap。其中Map接口定义了键映射到值的规则，而AbstractMap类提供 Map 接口的骨干实现，以最大限度地减少实现此接口所需的工作，其实AbstractMap类已经实现了Map，这里标注Map 应该是更加清晰吧！ 

![Capture](D:\sgmuserprofile\sqgdua\Desktop\Capture.JPG)

 上面是一张HashMap中Outline的部分图，其中我们主要关注的是几个constant：`serialVersionUID`、`DEFAULT_INITIAL_CAPACITY`、`MAXIMUM_CAPACITY`、`DEFAULT_LOAD_FACTOR`、`TREEIFY_THRESHOLD`、`UNTREEIFY_THRESHOLD`、`MIN_TREEIFY_CAPACITY`

```java
	private static final long serialVersionUID = 362498820763181265L;


    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

`serialVersionUID`:版本ID号，基本不重要

`DEFAULT_INITIAL_CAPACITY`：默认初始容量，必须是2的n次幂，源码中取了2的4次幂，即16；

`MAXIMUM_CAPACITY`：最大容量，为2的30次幂；

`DEFAULT_LOAD_FACTOR`：装载因子，默认装载因子 (0.75)

`TREEIFY_THRESHOLD`：链表转成红黑树的阈值，为8

`UNTREEIFY_THRESHOLD`：红黑树转成链表的阈值，为6

`MIN_TREEIFY_CAPACITY`：最小的table容量，64，如果在一个bin中节点太多，table会先扩容，如果扩容发现大于最小容量，此时在变成红黑树



HashMap 章节提到

> 在 JDK1.8 中对 HashMap 进行了优化： 当 hash 碰撞之后写入链表的长度超过了阈值(默认为8)，链表将会转换为红黑树

最近在看源码的时候，模拟了一下 hashmap hash 碰撞的场景，直接向一个空的 hashmap 中，连续 put 了12个相同 hashcode 的 key, 发现并没有立即树形化成红黑树，到第11个才发生。

treeifyBin 函数的第二行就有相关判断语句

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else // 树形化代码
       .....
}
```

即，tab 的长度要大于这个 MIN_TREEIFY_CAPACITY 值才会进行红黑树转化。

```java
   /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
     * between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

这个碰撞后，链表的长度超过了默认阈值8也不是立马就树形化，还要判断下 table 的 length 是否大于64，这样链表才会转换为红黑树。

考虑改为

在 JDK1.8 中对 HashMap 进行了优化： 当 hash 碰撞之后写入链表的长度超过了阈值(默认为8)并且 table 的长度不小于64，链表将会转换为红黑树

**完整分析源码**

~~~java
 /**
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     */
/*
     * 如果元素数组为空或者数组长度小于树结构化的最小限制
     * MIN_TREEIFY_CAPACITY默认值64，对于这个值可以理解为：如果元素数组长度小于这个值，没有必要去进行结构转换
     * 当一个数组位置上集中了多个键值对，那是因为这些key的hash值和数组长度取模之后结果相同。（并不是因为这些key的hash值相同）
     * 因为hash值相同的概率不高，所以可以通过扩容的方式，来使得最终这些key的hash值在和新的数组长度取模之后，拆分到多个数组位置上。
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
/*
     * 先判断是否为空引用，然后在判断数组长度是不是小于最小树形化容量，为了避免空指针异常问题的出现
     */
            resize();// 扩容，可参见resize方法解析
    // 如果元素数组长度已经大于等于了 MIN_TREEIFY_CAPACITY，那么就有必要进行结构转换了
    // 根据hash值和数组长度进行取模运算后，得到链表的首节点
        else if ((e = tab[index = (n - 1) & hash]) != null) {
/*
			*index = (n - 1) & hash是什么意思呢？其实，他就是取模。Java之所有使用位运算(&)来代替取模运算(%)，最主要的考虑就是效率。位运算(&)效率要比代替取模运算(%)高很多，主要原因是位运算直接对内存数据进行操作，不需要转成十进制，因此处理速度非常快。
			*那么，为什么可以使用位运算(&)来实现取模运算(%)呢？这实现的原理如下：

				X % 2^n = X & (2^n – 1)

				2^n表示2的n次方，也就是说，一个数对2^n取模 == 一个数和(2^n – 1)做按位与运算 。

				假设n为3，则2^3 = 8，表示成2进制就是1000。2^3 -1 = 7 ，即0111。

				此时X & (2^3 – 1) 就相当于取X的2进制的最后三位数。

				从2进制角度来看，X / 8相当于 X >> 3，即把X右移3位，此时得到了X / 8的商，而被移掉的部分(后三位)，则是X % 8，也就是余数。
			*/
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
~~~











~~~java
public class HashMap<K, V> extends AbstractMap<K, V> 
    					   implements Map<K, V>,Cloneable, Serializable
~~~

> extends是继承父类，只要那个类不是声明为final或者那个类定义为abstract的就能继承，JAVA中不支持多重继承，但是可以用接口 来实现，这样就要用到implements，继承只能继承一个类，但implements可以实现多个接口，用逗号分开就行了;比如 `class A extends B implements C,D,E`; 

###### 构造函数

> HashMap提供了三个构造函数：
>
>    HashMap()：构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap。
>
>    HashMap(int initialCapacity)：构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap。
>
>    HashMap(int initialCapacity, float loadFactor)：构造一个带指定初始容量和加载因子的空 HashMap。

 在这里提到了两个参数：初始容量，加载因子。这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中桶的数量，初始容量是创建哈希表时的容量，加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为0.75，一般情况下我们是无需修改的 

1. 无参构造函数，构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap

~~~Java

~~~

2. 带有一个初始容量的构造函数，构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap

~~~java

~~~

3. 构造一个带指定初始容量和加载因子的空 HashMap

```java
  
```



在构造函数中，有一个特别有意思的函数`tableSizeFor`

> 这个函数出现在java 1.8中，最新的已修改，先分析这个函数，之后在分析最新的函数！

```java
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

一眼看过去，是无符号移位运算

