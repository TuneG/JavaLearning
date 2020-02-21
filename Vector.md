~~~java
/**
 * Vector is a variable size contiguous indexable array of Objects. The size of
 * the Vector is the number of Objects it contains. The capacity of the Vector
 * is the number of Objects it can hold.
 * <p>
 * Objects may be inserted at any position up to the size of the Vector,
 * increasing the size of the Vector. Objects at any position in the Vector may
 * be removed, shrinking the size of the Vector. Objects at any position in the
 * Vector may be replaced, which does not affect the Vector size.
 * <p>
 * The capacity of a Vector may be specified when the Vector is created. If the
 * capacity of the Vector is exceeded, the capacity is increased, doubling by
 * default.
 * 
 * @see java.lang.StringBuffer
 */
~~~

### Vector的介绍

Vector 可实现自动增长的对象数组。
java.util.vector提供了向量类(vector)以实现类似动态数组的功能。在Java语言中没有指针的概念，但如果正确灵活地使用指针又确实可以大大提高程序的质量。比如在c,c++中所谓的“动态数组”一般都由指针来实现。为了弥补这个缺点，Java提供了丰富的类库来方便编程者使用，vector类便是其中之一。事实上，灵活使用数组也可以完成向量类的功能，但向量类中提供大量的方法大大方便了用户的使用。 
    创建了一个向量类的对象后，可以往其中随意插入不同类的对象，即不需顾及类型也不需预先选定向量的容量，并可以方便地进行查找。对于预先不知或者不愿预先定义数组大小，并且需要频繁地进行查找，插入，删除工作的情况。可以考虑使用向量类。 

Vector 类实现了一个动态数组。和 ArrayList 很相似，但是两者是不同的：

- Vector 是同步访问的。
- Vector 包含了许多传统的方法，这些方法不属于集合框架。

Vector 主要用在事先不知道数组的大小，或者只是需要一个可以改变大小的数组的情况。

### 构造函数

Vector 类支持 4 种构造方法。

第一种构造方法创建一个默认的向量，默认大小为 10：

```java
    private static final int DEFAULT_SIZE = 10;
 /**
     * Constructs a new Vector using the default capacity.
     */
    public Vector() {
        this(DEFAULT_SIZE, 0);
    }
```

第二种构造方法创建指定大小的向量。

```java
  /**
     * Constructs a new Vector using the specified capacity.
     * 
     * @param capacity
     *            the initial capacity of the new vector
     */
    public Vector(int capacity) {
        this(capacity, 0);
    }

```

第三种构造方法创建指定大小的向量，并且增量用 capacityIncrement 指定。增量表示向量每次增加的元素数目。

```java
    /**
     * Constructs a new Vector using the specified capacity and capacity
     * increment.
     * 
     * @param capacity
     *            the initial capacity of the new Vector
     * @param capacityIncrement
     *            the amount to increase the capacity when this Vector is full
     */
    public Vector(int capacity, int capacityIncrement) {
        elementCount = 0;
        try {
            elementData = newElementArray(capacity);
        } catch (NegativeArraySizeException e) {
            throw new IllegalArgumentException();
        }
        this.capacityIncrement = capacityIncrement;
    }

```

第四种构造方法创建一个包含集合 c 元素的向量：

```java
  /**
     * Constructs a new instance of <code>Vector</code> containing the
     * elements in <code>collection</code>. The order of the elements in the
     * new <code>Vector</code> is dependent on the iteration order of the seed
     * collection.
     * 
     * @param collection
     *            the collection of elements to add
     */
    public Vector(Collection<? extends E> collection) {
        this(collection.size(), 0);
        Iterator<? extends E> it = collection.iterator();
        while (it.hasNext()) {
            elementData[elementCount++] = it.next();
        }
    }
```

### Vector的方法

除了从父类继承的方法外 Vector 还定义了以下方法：

| 序号 | 方法描述                                                     |
| ---- | ------------------------------------------------------------ |
| 1    | void add(int index, Object element)   在此向量的指定位置插入指定的元素。 |
| 2    | boolean add(Object o)   将指定元素添加到此向量的末尾。       |
| 3    | boolean addAll(Collection c)  将指定 Collection 中的所有元素添加到此向量的末尾，按照指定 collection 的迭代器所返回的顺序添加这些元素。 |
| 4    | boolean addAll(int index, Collection c)  在指定位置将指定 Collection 中的所有元素插入到此向量中。 |
| 5    | void addElement(Object obj)   将指定的组件添加到此向量的末尾，将其大小增加 1。 |
| 6    | int capacity()  返回此向量的当前容量。                       |
| 7    | void clear()  从此向量中移除所有元素。                       |
| 8    | Object clone()  返回向量的一个副本。                         |
| 9    | boolean contains(Object elem)  如果此向量包含指定的元素，则返回 true。 |
| 10   | boolean containsAll(Collection c)  如果此向量包含指定 Collection 中的所有元素，则返回 true。 |
| 11   | void copyInto(Object[] anArray)   将此向量的组件复制到指定的数组中。 |
| 12   | Object elementAt(int index)  返回指定索引处的组件。          |
| 13   | Enumeration elements()  返回此向量的组件的枚举。             |
| 14   | void ensureCapacity(int minCapacity)  增加此向量的容量（如有必要），以确保其至少能够保存最小容量参数指定的组件数。 |
| 15   | boolean equals(Object o)  比较指定对象与此向量的相等性。     |
| 16   | Object firstElement()  返回此向量的第一个组件（位于索引 0) 处的项）。 |
| 17   | Object get(int index)  返回向量中指定位置的元素。            |
| 18   | int hashCode()  返回此向量的哈希码值。                       |
| 19   | int indexOf(Object elem)   返回此向量中第一次出现的指定元素的索引，如果此向量不包含该元素，则返回 -1。 |
| 20   | int indexOf(Object elem, int index)   返回此向量中第一次出现的指定元素的索引，从 index 处正向搜索，如果未找到该元素，则返回 -1。 |
| 21   | void insertElementAt(Object obj, int index)  将指定对象作为此向量中的组件插入到指定的 index 处。 |
| 22   | boolean isEmpty()  测试此向量是否不包含组件。                |
| 23   | Object lastElement()  返回此向量的最后一个组件。             |
| 24   | int lastIndexOf(Object elem)   返回此向量中最后一次出现的指定元素的索引；如果此向量不包含该元素，则返回 -1。 |
| 25   | int lastIndexOf(Object elem, int index)  返回此向量中最后一次出现的指定元素的索引，从 index 处逆向搜索，如果未找到该元素，则返回 -1。 |
| 26   | Object remove(int index)   移除此向量中指定位置的元素。      |
| 27   | boolean remove(Object o)  移除此向量中指定元素的第一个匹配项，如果向量不包含该元素，则元素保持不变。 |
| 28   | boolean removeAll(Collection c)  从此向量中移除包含在指定 Collection 中的所有元素。 |
| 29   | void removeAllElements()  从此向量中移除全部组件，并将其大小设置为零。 |
| 30   | boolean removeElement(Object obj)  从此向量中移除变量的第一个（索引最小的）匹配项。 |
| 31   | void removeElementAt(int index)  删除指定索引处的组件。      |
| 32   | protected void removeRange(int fromIndex, int toIndex) 从此 List 中移除其索引位于 fromIndex（包括）与 toIndex（不包括）之间的所有元素。 |
| 33   | boolean retainAll(Collection c)  在此向量中仅保留包含在指定 Collection 中的元素。 |
| 34   | Object set(int index, Object element)  用指定的元素替换此向量中指定位置处的元素。 |
| 35   | void setElementAt(Object obj, int index)  将此向量指定 index 处的组件设置为指定的对象。 |
| 36   | void setSize(int newSize)   设置此向量的大小。               |
| 37   | int size()   返回此向量中的组件数。                          |
| 38   | List subList(int fromIndex, int toIndex)  返回此 List 的部分视图，元素范围为从 fromIndex（包括）到 toIndex（不包括）。 |
| 39   | Object[] toArray()  返回一个数组，包含此向量中以恰当顺序存放的所有元素。 |
| 40   | Object[] toArray(Object[] a)  返回一个数组，包含此向量中以恰当顺序存放的所有元素；返回数组的运行时类型为指定数组的类型。 |
| 41   | String toString()  返回此向量的字符串表示形式，其中包含每个元素的 String 表示形式。 |
| 42   | void trimToSize()   对此向量的容量进行微调，使其等于向量的当前大小。 |





### 实例

下面的程序说明这个集合所支持的几种方法：

```java
import java.util.*;

public class VectorDemo {

   public static void main(String args[]) {
      // initial size is 3, increment is 2
      Vector v = new Vector(3, 2);
      System.out.println("Initial size: " + v.size());
      System.out.println("Initial capacity: " +
      v.capacity());
      v.addElement(new Integer(1));
      v.addElement(new Integer(2));
      v.addElement(new Integer(3));
      v.addElement(new Integer(4));
      System.out.println("Capacity after four additions: " +
          v.capacity());

      v.addElement(new Double(5.45));
      System.out.println("Current capacity: " +
      v.capacity());
      v.addElement(new Double(6.08));
      v.addElement(new Integer(7));
      System.out.println("Current capacity: " +
      v.capacity());
      v.addElement(new Float(9.4));
      v.addElement(new Integer(10));
      System.out.println("Current capacity: " +
      v.capacity());
      v.addElement(new Integer(11));
      v.addElement(new Integer(12));
      System.out.println("First element: " +
         (Integer)v.firstElement());
      System.out.println("Last element: " +
         (Integer)v.lastElement());
      if(v.contains(new Integer(3)))
         System.out.println("Vector contains 3.");
      // enumerate the elements in the vector.
      Enumeration vEnum = v.elements();
      System.out.println("\nElements in vector:");
      while(vEnum.hasMoreElements())
         System.out.print(vEnum.nextElement() + " ");
      System.out.println();
   }
}
```

以上实例编译运行结果如下：

```java
Initial size: 0
Initial capacity: 3
Capacity after four additions: 5
Current capacity: 5
Current capacity: 7
Current capacity: 9
First element: 1
Last element: 12
Vector contains 3.

Elements in vector:
1 2 3 4 5.45 6.08 7 9.4 10 11 12
```