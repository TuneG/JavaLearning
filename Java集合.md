Java集合

* 为了保存数量不确定的数据，以及保存具有映射关系的数据（也被称为关联数组）。
* Java提供集合类，集合类主要负责保存、盛装其他数据，因此集合类也被称为容器类。
* 所有集合类都位于***java.util***包下。
* Java的集合类主要由两个接口派生而出：Collection和Map，Collection和Map是Java集合框架的根接口，这两个接口又包含了一些子接口或实现类。 

### Collection集合



![image-20191028090639677](D:\sgmuserprofile\sqgdua\AppData\Roaming\Typora\typora-user-images\image-20191028090639677.png)

* ***Set***        无序集合，元素不可重复；
* ***Queue***    队列；
* ***List***          有序集合，元素可重复。



### Map集合

![image-20191028090656462](D:\sgmuserprofile\sqgdua\AppData\Roaming\Typora\typora-user-images\image-20191028090656462.png)

### Collection接口

* Collection接口是List、Set和Queue接口的父接口，该接口里定义的方法既可用于操作Set集合，也可用于操作List和Queue集合。
* Collection提供了大量添加、删除、访问的方法来访问集合元素。

### Iterator接口

* Iterator接口也是Java集合框架的成员，但它与Collection系列、Map系列的集合不一样：Collection系列集合、Map系列集合主要用于盛装其他对象，而Iterator则主要用于遍历（即迭代访问）Collection集合中的元素，Iterator对象也被称为迭代器。
* Iterator接口里定义了如下4个方法：
  * ***boolean hasNext()***：如果被迭代的集合还元素没有被遍历，则返回true。
  * ***Object next()***：返回集合里下一个元素。
  * ***void remove()*** ：删除集合里上一次next方法返回的元素 。
  * ***void forEachRemaining(Consumer action)***，这是Java 8为Iterator新增的默认方法，该方法可使用Lambda表达式来遍历集合元素。

### foreach遍历

* 使用JDK1.5提供的foreach循环来迭代访问集合元素更加便捷
* 循环迭代访问集合元素时，该集合也不能被改变，否则将引发***ConcurrentModificationException***异常  

### HashSet类

* HashSet是Set接口的典型实现，大多时候使用Set集合时就是使用这个实现类。HashSet按Hash算法来存储集合中的元素，因此具有很好的存取和查找性能。当向HashSet集合中存入一个元素时，HashSet会调用该对象的***hashCode()***方法来得到该对象的hashCode值，然后根据该HashCode值来决定该对象在HashSet中存储位置。如果有两个元素通过equals方法比较返回true，但它们的***hashCode()***方法返回值不相等，HashSet将会把它们存储在不同位置，也就可以添加成功。 
  1. 不能保证元素的排列顺序，顺序可能与元素的添加顺序不同，元素的顺序可能变化。
  2. HashSet不是同步的，如果多个线程同时访问一个HashSet，如果有2条或者2条以上线程同时修改了HashSet集合时，必须通过代码来保证其同步。
  3. 集合元素值可以是null。

### LinkedHashSet

1. LinkedHashSet集合也是根据元素hashCode值来决定元素存储位置，但它同时使用链表维护元素的次序，这样使得元素看起来是以插入的顺序保存的。也就是说，当遍历LinkedHashSet集合里元素时，HashSet将会按元素的添加顺序来访问集合里的元素。
2. LinkedHashSet需要维护元素的插入顺序，因此性能略低于HashSet的性能，但在迭代访问Set里的全部元素时将有很好的性能，因为它以链表来维护内部顺序。 

### TreeSet

* TreeSet是SortedSet接口的唯一实现，正如SortedSet名字所暗示的，TreeSet可以确保集合元素处于排序状态。与前面HashSet集合相比，TreeSet还提供了如下几个额外的方法：
  1. ***Object first()***：返回集合中的第一个元素。 
  2. ***Object last()***：返回集合中的最末一个元素。 
  3. ***Object lower(Object e)***：返回集合中位于指定元素之前的元素（即小于指定元素的最大元素，参考元素不需要是TreeSet的元素）。
  4. ***Object higher(Object e)***：返回集合中位于指定元素之后的元素（即大于指定元素的最小元素，参考元素不需要是TreeSet的元素）。 
  5. ***SortedSet subSet(fromElement, toElement)***：返回此Set的子集合，范围从fromElement（包含）到toElement（不包含）。 
  6. ***SortedSet headSet(toElement)***：返回此Set的子集，由小于toElement的元素组成。 
  7. ***SortedSet tailSet(fromElement)***：返回此Set的子集，由大于或等于fromElement的元素组成。

### EnumSet

* EnumSet是一个专为枚举类设计的集合类，EnumSet中所有元素都必须是指定枚举类型的枚举值，该枚举类型在创建EnumSet时显式或隐式地指定。EnumSet的集合元素也是有序的，EnumSet以枚举值在Enum类的定义顺序来决定集合元素的顺序
* EnumSet在内部以位向量的形式存储，这种存储形式非常紧凑、高效，因此EnumSet对象占用内存很小，而且运行效率很好。尤其是当进行批量操作（如调用containsAll 和 retainAll方法）时，如其参数也是EnumSet集合，则该批量操作的执行速度也非常快。 
* EnumSet集合不允许加入null元素。如果试图插入null元素，EnumSet将抛出 NullPointerException异常。如果仅仅只是试图测试是否出现null元素、或删除null元素都不会抛出异常，只是删除操作将返回false，因为没有任何null元素被删除。

### List接口

* List集合代表一个有序集合，集合中每个元素都有其对应的顺序索引。List集合允许使用重复元素，可以通过索引来访问指定位置的集合元素。因为List集合默认按元素的添加顺序设置元素的索引，例如第一次添加的元素索引为0，第二次添加的元素索引为1…… 
* List作为Collection接口的子接口，当然可以使用Collection接口里全部方法。而且由于List是有序集合，因此List集合里包含了根据索引来操作集合元素的方法。

###  ListIterator接口

* ListIterator与Iterator接口不同，它不仅可以向后迭代，它还可以向前迭代。

* ListIterator包含增加了如下3个方法：
  * ***boolean hasPrevious()***：返回该迭代器关联的集合是否还有上一个元素。
  * ***Object previous()***：返回该迭代器的上一个元素。
  * ***void add()***：在指定位置插入一个元素。



###      Iterator(迭代),Enumeration(枚举) 

* 迭代是取出集合中元素的一种方式。

* 因为Collection中有iterator方法，所以每一个子类集合对象都具备迭代器。

  ***用法：***

  ~~~java
  for(Iterator iter = iterator();iter.hasNext();  )
  {
  	System.out.println(iter.next());
  }
  ~~~

  ~~~java
  Iterator iter = l.iterator();
  while(iter.hasNext())
  {
  	System.out.println(iter.next());
  }
  ~~~

  ***注意事项：***

> 1. 迭代器在Collcection接口中是通用的。
> 2. 迭代器的next方法是自动向下取元素，要避免出现NoSuchElementException。
> 3. 迭代器的next方法返回值类型是Object，所以要记得类型转换。

 

### 比较Comparator

​     比较函数强行对某些对象 collection 进行*整体排序*。可以将 Comparator 传递给 sort 方法（如 Collections.sort），从而允许在排序顺序上实现精确控制。  

> 例如：

~~~java
public class User {
    int a;
    String s;
    public  User(int a,String s) {
          this.a=a;
          this.s=s;
    }
}

ArrayList<User> arr=new ArrayList();
arr.add(new User(21,"21"));
arr.add(new User(12,"12"));
arr.add(new User(3,"3"));
arr.add(new User(14,"14"));
arr.add(new User(5,"5"));
arr.add(new User(26,"26"));
Collections.sort(arr);

~~~

### ArrayList与Vector

> 相同点

* ArrayList和Vector类都是基于数组实现的List类，所以ArrayList和Vector类封装了一个动态再分配的Object[]数组。每个ArrayList或Vector对象有一个capacity属性，这个capacity表示它们所封装的Object[]数组的长度。当向ArrayList或Vector中添加元素时，它们的capacity会自动增加。

> 不同点

* rrayList和Vector的显著区别是；ArrayList是线程不安全的，当多条线程访问同一个ArrayList集合时，如果有超过一条线程修改了ArrayList集合，则程序必须手动保证该集合的同步性。但Vector集合则是线程安全的，无需程序保证该集合的同步性。 

### 理解Map

* Map里key集和Set集合里元素的存储形式也很像，Map子类和Set子类在名字也惊人地相似：如Set接口下有HashSet、LinkedHashSet、SortedSet（接口）、TreeSet、EnumSet等实现类和子接口，而Map接口下则有HashMap、LinkedHashMap、SortedMap（接口）、TreeMap、EnumMap等实现类和子接口。正如它们名字所暗示的，Map的这些实现类和子接口中key集存储形式和对应Set集合中元素的存储形式完全相同。
* 如果把Map所有value放在一起来看，它们又非常类似于一个List：元素与元素之间可以重复，每个元素可以根据索引来查找，只是Map中的索引不再使用整数值，而是以另一个对象来作为索引。如果需要从List集合中取出元素，需要提供该元素的数字索引。

### HashMap和Hashtable

​      HashMap和Hashtable都是Map接口的典型实现类，它们的区别如下：

1. Hashtable是一个线程安全的Map实现，但HashMap是线程不安全的实现，所以HashMap比Hashtable的性能高一点；但如果有多条线程访问同一个Map对象时，使用Hashtable实现类会更好。
2. Hashtable不允许使用null作为key和value，如果试图把null值放进Hashtable中，将会引发***NullPointerException***异常；但HashMap可以使用null作为key或value。

### Map接口

* Map用于保存具有映射关系的数据，因此Map集合里保存着两组值，一组值用于保存Map里的key，另外一组值用于保存Map里的value，key和value都可以是任何引用类型的数据。Map的key不允许重复，即同一个Map对象的任何两个key通过equals方法比较总是返回false。
* key和value之间存在单向一对一关系，即通过指定的key，总能找到唯一的、确定的value。从Map中取出数据时，只要给出指定的key，就可以取出对应的value。 

### HashSet和HashMap的性能

​        因为HashSet和HashMap、Hashtable都使用hash算法来决定其元素（对HashMap则是key）的存储，因此HashSet、HashMap的hash表包含如下属性：

* 容量（capacity）：hash表中桶的数量。
* 初始化容量（initial capacity）：创建hash表时桶的数量。HashMap和HashSet都允许在构造器中指定初始化容量。
* 尺寸（size）：当前散列表中记录的数量。
* 负载因子（load factor）：负载因子等于“size/capacity”。负载因子为0，表示空的hash表，0.5表示半满的散列表，依此类推。轻负载的散列表具有冲突少、适宜插入与查询的特点（但是使用Iterator迭代元素时会变慢）。