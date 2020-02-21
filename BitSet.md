



## **适用场景：整数，无重复**

## ![img](https://img-blog.csdn.net/20161219103322771) 

## 一. Bitset 基础

Bitset，也就是位图，由于可以用非常紧凑的格式来表示给定范围的连续数据而经常出现在各种算法设计中。上面的图来自c++库中bitset的一张图。基本原理是，用1位来表示一个数据是否出现过，0为没有出现过，1表示出现过。使用的时候根据某一个是否为0表示此数是否出现过。一个1G的空间，有 8*1024*1024*1024=8.58*10^9bit，也就是可以表示85亿个不同的数。



**常见的应用**是那些需要对海量数据进行一些统计工作的时候，比如日志分析等。

**面试题**中也常出现，比如：统计40亿个数据中没有出现的数据，将40亿个不同数据进行排序等。

又如：现在有1千万个随机数，随机数的范围在1到1亿之间。现在要求写出一种算法，将1到1亿之间没有在随机数中的数求出来(百度)。

programming pearls上也有一个关于使用bitset来查找电话号码的题目。

Bitmap的常见扩展，是用2位或者更多为来表示此数字的更多信息，比如出现了多少次等。





## 二. Java中BitSet的实现



​        BitSet实现了一个按需增长的位向量。BitSet的每个组件都有一个boolean值。用非负的整数将BitSet的位编入索引。可以对每个编入索引的位进行测试、设置或者清除。通过逻辑与、逻辑或和逻辑异或操作，可以使用一个BitSet修改另一个BitSet的内容。默认情况下，set 中所有位的初始值都是false。每个BitSet都有一个当前大小，也就是该BitSet当前所用空间的位数。注意，这个大小与BitSet的实现有关，所以它可能随实现的不同而更改。BitSet的长度与BitSet的逻辑长度有关，并且是与实现无关而定义的。

> 1.除非另行说明，否则将 null 参数传递给BitSet中的任何方法都将导致NullPointerException。
> 2.在没有外部同步的情况下，多个线程操作一个BitSet是不安全的.

BitSet是位操作的对象，值只有0或1即false和true，内部维护了一个long数组，初始只有一个long，所以BitSet最小的size是64，当随着存储的元素越来越多，BitSet内部会动态扩充，最终内部是由N个long来存储，这些针对操作都是透明的。用1位来表示一个数据是否出现过，0为没有出现过，1表示出现过。使用用的时候既可根据某一个是否为0表示，此数是否出现过。一个1G的空间，有 8*1024*1024*1024=8.58*10^9bit，也就是可以表示85亿个不同的数。

Bitset这种结构虽然简单，实现的时候也有一些细节需要注意。其中关键是一些位操作，比如如何将指定位进行反转、设置、查询指定位的状态（0或者1）等。

本文，分析一下java中BitSet的实现，抛砖引玉，希望给那些需要自己设计位图结构的程序员有所启发。

**Bitset的基本操作有：**

1. 初始化一个bitset，指定大小。
2. 清空bitset。
3. 反转某一指定位。
4. 设置某一指定位。
5. 获取某一位的状态。
6. 当前bitset的bit总位数。

> 在java中，bitset的实现，位于java.util这个包中，从jdk 1.0就引入了这个数据结构。在多个jdk的演变中，bitset也不断演变。本文参照的是jdk 7.0 源代码中的实现。

### 1.类声明

```java
public class BitSet implements Cloneable, java.io.Serializable {
    /*
     * BitSets are packed into arrays of "words."  Currently a word is
     * a long, which consists of 64 bits, requiring 6 address bits.
     * The choice of word size is determined purely by performance concerns.
     */
    private final static int ADDRESS_BITS_PER_WORD = 6;
    private final static int BITS_PER_WORD = 1 << ADDRESS_BITS_PER_WORD;
    private final static int BIT_INDEX_MASK = BITS_PER_WORD - 1;
 
    /* Used to shift left or right for a partial word mask */
    private static final long WORD_MASK = 0xffffffffffffffffL;
 
    /**
     * The internal field corresponding to the serialField "bits".
     */
    private long[] words;

```

同时我们也看到使用long数组来作为内部存储结构。这个决定了，Bitset至少为一个long的大小。下面的构造函数中也会有所体现。

### 2.构造函数

```java
/**
     * Given a bit index, return word index containing it.
     */
    private static int wordIndex(int bitIndex) {
        return bitIndex >> ADDRESS_BITS_PER_WORD;
    }
 
    /**
     * Every public method must preserve these invariants.
     */
    private void checkInvariants() {
        assert(wordsInUse == 0 || words[wordsInUse - 1] != 0);
        assert(wordsInUse >= 0 && wordsInUse <= words.length);
        assert(wordsInUse == words.length || words[wordsInUse] == 0);
    }
 
    /**
     * Sets the field wordsInUse to the logical size in words of the bit set.
     * WARNING:This method assumes that the number of words actually in use is
     * less than or equal to the current value of wordsInUse!
     */
    private void recalculateWordsInUse() {
        // Traverse the bitset until a used word is found
        int i;
        for (i = wordsInUse-1; i >= 0; i--)
            if (words[i] != 0)
                break;
 
        wordsInUse = i+1; // The new logical size
    }
 
    /**
     * Creates a new bit set. All bits are initially {@code false}.
     */
    public BitSet() {
        initWords(BITS_PER_WORD);
        sizeIsSticky = false;
    }
 
    /**
     * Creates a bit set whose initial size is large enough to explicitly
     * represent bits with indices in the range {@code 0} through
     * {@code nbits-1}. All bits are initially {@code false}.
     *
     * @param  nbits the initial size of the bit set
     * @throws NegativeArraySizeException if the specified initial size
     *         is negative
     */
    public BitSet(int nbits) {
        // nbits can't be negative; size 0 is OK
        if (nbits < 0)
            throw new NegativeArraySizeException("nbits < 0: " + nbits);
 
        initWords(nbits);
        sizeIsSticky = true;
    }
 
    private void initWords(int nbits) {
        words = new long[wordIndex(nbits-1) + 1];
    }

```

 我们可以看到BitSet有两个构造方法：不带参数和带参数，不带参数的构造函数默认的初始大小为, 2^6 = 64-1=63 bit. 我们知道java中long的大小就是8个字节，也就是8*8=64bit。也就是说，bitset默认的是一个long整形的大小。带参数的构造方法指定了总共的bit位数，此方法会将其bit位数规整到一个大于或者等于这个数字的64的整倍数。比如64位，BitSet的大小是1个long，而65位时，则BitSet大小是2个long，即128位。做这么一个规定，主要是为了内存对齐，同时避免考虑到不要处理特殊情况。

### 3. Clear方法

1. 清空所有的bit位，即全部置0。通过循环方式来以此以此置0。 

> 如果是c语言，使用memset会不会快点？

```java
public void clear() {  
    while (wordsInUse > 0)  
        words[--wordsInUse] = 0;  
}  
```

2. 清空某一位

```java
public void clear(int bitIndex) {  
     if (bitIndex < 0)  
         throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);  
  
     int wordIndex = wordIndex(bitIndex);  
     if (wordIndex >= wordsInUse)  
         return;  
  
     words[wordIndex] &= ~(1L << bitIndex);  
  
     recalculateWordsInUse();  
     checkInvariants();  
 } 
```

第一行是参数检查，如果bitIndex小于0，则抛参数非法异常。 

跟之前的操作类似，先找到对应的long，然后执行逻辑运算。

a. 找到对应的long。 这行语句是 ` int wordIndex = wordIndex(bitIndex);  `

b. 操作对应的位。常见的位操作是通过与特定的mask进行逻辑运算来实现的。 

首先获取 mask(掩码)，对于 clear某一位来说，它需要的掩码是指定位为0，其余位为1，然后与对应的long进行&运算。
`   ~(1L << bitIndex); // 即获取mask
  words[wordIndex] &= ~(1L << bitIndex); //执行相应的运算。`

>对于整数，有四种表示方式：
>
> 二进制：0,1 ，满2进1.以0b或0B开头。也常在结尾加B
>
> 十进制：0-9 ，满10进1.
>
> 八进制：0-7 ，满8进1. 以数字0开头表示。
>
> 十六进制：0-9及A-F，满16进1. 以0x或0X开头表示。此处的A-F不区分大小写



---

例如：二进制数10101010B，运用clear()函数，将第四位清零

~~~java
import java.util.*;

public class Test_clear{
    public static void main(String[] args){
        BitSet bitset_test=new BitSet(8);
        for(int i=1;i<8;i+=2)
            bitset_test.set(i);
        System.out.println(bitset_test);
        bitset_test.clear(3);
        System.out.println(bitset_test);
    }
}
~~~

~~~java
clear(3) {  
    //Index从0开始算，第四位的index就是3
     wordIndex = wordIndex(3); //8位二进制，根据构造函数，存放在第一个long数组位
     wordIndex = 0;
       
  
     words[0] =10101010B& ~(1L << 3);  //~(00000001B)<<3=~(00001000B)=11110111B
    								   //10101010B&11110111B=10100010
    								   //第四位已置零
     recalculateWordsInUse();  
     checkInvariants();  
 } 
~~~

上述结果为：

~~~java
{1, 3, 5, 7}
{1, 5, 7}
~~~

和分析的结果相同！！！

---

注意：这里的参数检查，对负数index跑出异常，对超出大小的index，不做任何操作，直接返回。具体的原因，有待进一步思考。

3. 清空指定范围的bit位

```java
 /**
     * Sets the bits from the specified <tt>fromIndex</tt> (inclusive) to the
     * specified <tt>toIndex</tt> (exclusive) to <code>false</code>.
     *
     * @param     fromIndex   index of the first bit to be cleared.
     * @param     toIndex index after the last bit to be cleared.
     * @exception IndexOutOfBoundsException if <tt>fromIndex</tt> is negative,
     *            or <tt>toIndex</tt> is negative, or <tt>fromIndex</tt> is
     *            larger than <tt>toIndex</tt>.
     * @since     1.4
     */
    public void clear(int fromIndex, int toIndex) {
	checkRange(fromIndex, toIndex);

	if (fromIndex == toIndex)
	    return;

        int startWordIndex = wordIndex(fromIndex);
	if (startWordIndex >= wordsInUse)
	    return;

        int endWordIndex = wordIndex(toIndex - 1);
	if (endWordIndex >= wordsInUse) {
	    toIndex = length();
	    endWordIndex = wordsInUse - 1;
	}

	long firstWordMask = WORD_MASK << fromIndex;
	long lastWordMask  = WORD_MASK >>> -toIndex;
        if (startWordIndex == endWordIndex) {
            // Case 1: One word
            words[startWordIndex] &= ~(firstWordMask & lastWordMask);
        } else {
	    // Case 2: Multiple words
	    // Handle first word
	    words[startWordIndex] &= ~firstWordMask;

	    // Handle intermediate words, if any
	    for (int i = startWordIndex+1; i < endWordIndex; i++)
		words[i] = 0;

	    // Handle last word
	    words[endWordIndex] &= ~lastWordMask;
	}

	recalculateWordsInUse();
	checkInvariants();
    }
```

方法是将这个范围分成三块，startword; interval words; stopword。
其中startword，只要将从start位到该word结束位全部置0；interval words则是这些long的所有bits全部置0；而stopword这是从起始位置到指定的结束位全部置0。
而特殊情形则是没有startword和stopword是同一个long。
具体的实现，参照代码，是分别作出两个mask，对startword和stopword进行操作。



### 4. 重要的两个内部检查函数

从上面的代码，可以看到每个函授结尾都会有两个函数,如下：

` recalculateWordsInUse();
 checkInvariants();`

这两个函数，是对bitset的内部状态进行维护和检查的函数。细看实现既可明白其中原理：

```java
   /**
     * Set the field wordsInUse with the logical size in words of the bit
     * set.  WARNING:This method assumes that the number of words actually
     * in use is less than or equal to the current value of wordsInUse!
     */
    private void recalculateWordsInUse() {
        // Traverse the bitset until a used word is found
        int i;
        for (i = wordsInUse-1; i >= 0; i--)
	    if (words[i] != 0)
		break;

        wordsInUse = i+1; // The new logical size
    }
```

wordsInUse 是检查当前的long数组中，实际使用的long的个数，即long[wordsInUse-1]是当前最后一个存储有有效bit的long。这个值是用于保存bitset有效大小的。

```java
    /**
     * Every public method must preserve these invariants.
     */
    private void checkInvariants() {
	assert(wordsInUse == 0 || words[wordsInUse - 1] != 0);
	assert(wordsInUse >= 0 && wordsInUse <= words.length);
	assert(wordsInUse == words.length || words[wordsInUse] == 0);
    }
```

checkInvariants 可以看出是检查内部状态，尤其是wordsInUse是否合法的函数。

### 5. 反转某一个指定位

反转，就是1变成0,0变成1，是一个与1的xor操作。

```java
  /**
     * Sets the bit at the specified index to the complement of its
     * current value.
     *
     * @param   bitIndex the index of the bit to flip.
     * @exception IndexOutOfBoundsException if the specified index is negative.
     * @since   1.4
     */
    public void flip(int bitIndex) {
	if (bitIndex < 0)
	    throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

	int wordIndex = wordIndex(bitIndex);
	expandTo(wordIndex);

	words[wordIndex] ^= (1L << bitIndex);

	recalculateWordsInUse();
	checkInvariants();
    }
```

反转的基本操作也是两步，找到对应的long， 获取mask并与指定的位进行xor操作。
` int wordIndex = wordIndex(bitIndex);
 words[wordIndex] ^= (1L << bitIndex);`
我们注意到在进行操作之前，执行了一个函数` expandTo(wordIndex);` 这个函数是确保bitset中有对应的这个long。如果没有的话，就对bitset中的long数组进行扩展。扩展的策略，是将当前的空间翻一倍。
代码如下：

```java
    /**
     * Ensures that the BitSet can accommodate a given wordIndex,
     * temporarily violating the invariants.  The caller must
     * restore the invariants before returning to the user,
     * possibly using recalculateWordsInUse().
     * @param	wordIndex the index to be accommodated.
     */
    private void expandTo(int wordIndex) {
	int wordsRequired = wordIndex+1;
	if (wordsInUse < wordsRequired) {
	    ensureCapacity(wordsRequired);
	    wordsInUse = wordsRequired;
	    }
    }

    /**
     * Ensures that the BitSet can hold enough words.
     * @param wordsRequired the minimum acceptable number of words.
     */
    private void ensureCapacity(int wordsRequired) {
	if (words.length < wordsRequired) {
	    // Allocate larger of doubled size or required size
	    int request = Math.max(2 * words.length, wordsRequired);
            words = Arrays.copyOf(words, request);
            sizeIsSticky = false;
        }
    }
```

同样，也提供了一个指定区间的反转，实现方案与clear基本相同。代码如下：

```java
    /**
     * Sets each bit from the specified <tt>fromIndex</tt> (inclusive) to the
     * specified <tt>toIndex</tt> (exclusive) to the complement of its current
     * value.
     *
     * @param     fromIndex   index of the first bit to flip.
     * @param     toIndex index after the last bit to flip.
     * @exception IndexOutOfBoundsException if <tt>fromIndex</tt> is negative,
     *            or <tt>toIndex</tt> is negative, or <tt>fromIndex</tt> is
     *            larger than <tt>toIndex</tt>.
     * @since   1.4
     */
    public void flip(int fromIndex, int toIndex) {
	checkRange(fromIndex, toIndex);

	if (fromIndex == toIndex)
	    return;

        int startWordIndex = wordIndex(fromIndex);
        int endWordIndex   = wordIndex(toIndex - 1);
	expandTo(endWordIndex);

	long firstWordMask = WORD_MASK << fromIndex;
	long lastWordMask  = WORD_MASK >>> -toIndex;
        if (startWordIndex == endWordIndex) {
            // Case 1: One word
            words[startWordIndex] ^= (firstWordMask & lastWordMask);
        } else {
	    // Case 2: Multiple words
	    // Handle first word
	    words[startWordIndex] ^= firstWordMask;

	    // Handle intermediate words, if any
	    for (int i = startWordIndex+1; i < endWordIndex; i++)
		words[i] ^= WORD_MASK;

	    // Handle last word
	    words[endWordIndex] ^= lastWordMask;
	}

	recalculateWordsInUse();
	checkInvariants();
    }
```



### 6. 设置某一指定位 （or 操作）

```java
   /**
     * Sets the bit at the specified index to <code>true</code>.
     *
     * @param     bitIndex   a bit index.
     * @exception IndexOutOfBoundsException if the specified index is negative.
     * @since     JDK1.0
     */
    public void set(int bitIndex) {
	if (bitIndex < 0)
	    throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

        int wordIndex = wordIndex(bitIndex);
	expandTo(wordIndex);

	words[wordIndex] |= (1L << bitIndex); // Restores invariants

	checkInvariants();
    }
```

```java
    /**
     * Sets the bit at the specified index to the specified value.
     *
     * @param     bitIndex   a bit index.
     * @param     value a boolean value to set.
     * @exception IndexOutOfBoundsException if the specified index is negative.
     * @since     1.4
     */
    public void set(int bitIndex, boolean value) {
        if (value)
            set(bitIndex);
        else
            clear(bitIndex);
    }

```

```java
    /**
     * Sets the bits from the specified <tt>fromIndex</tt> (inclusive) to the
     * specified <tt>toIndex</tt> (exclusive) to <code>true</code>.
     *
     * @param     fromIndex   index of the first bit to be set.
     * @param     toIndex index after the last bit to be set.
     * @exception IndexOutOfBoundsException if <tt>fromIndex</tt> is negative,
     *            or <tt>toIndex</tt> is negative, or <tt>fromIndex</tt> is
     *            larger than <tt>toIndex</tt>.
     * @since     1.4
     */
    public void set(int fromIndex, int toIndex) {
	checkRange(fromIndex, toIndex);

	if (fromIndex == toIndex)
	    return;

        // Increase capacity if necessary
        int startWordIndex = wordIndex(fromIndex);
        int endWordIndex   = wordIndex(toIndex - 1);
	expandTo(endWordIndex);

	long firstWordMask = WORD_MASK << fromIndex;
	long lastWordMask  = WORD_MASK >>> -toIndex;
        if (startWordIndex == endWordIndex) {
            // Case 1: One word
	    words[startWordIndex] |= (firstWordMask & lastWordMask);
        } else {
	    // Case 2: Multiple words
	    // Handle first word
	    words[startWordIndex] |= firstWordMask;

	    // Handle intermediate words, if any
	    for (int i = startWordIndex+1; i < endWordIndex; i++)
		words[i] = WORD_MASK;

	    // Handle last word (restores invariants)
	    words[endWordIndex] |= lastWordMask;
	}

	checkInvariants();
    }

```

```java
   /**
     * Sets the bits from the specified <tt>fromIndex</tt> (inclusive) to the
     * specified <tt>toIndex</tt> (exclusive) to the specified value.
     *
     * @param     fromIndex   index of the first bit to be set.
     * @param     toIndex index after the last bit to be set
     * @param     value value to set the selected bits to
     * @exception IndexOutOfBoundsException if <tt>fromIndex</tt> is negative,
     *            or <tt>toIndex</tt> is negative, or <tt>fromIndex</tt> is
     *            larger than <tt>toIndex</tt>.
     * @since     1.4
     */
    public void set(int fromIndex, int toIndex, boolean value) {
	if (value)
            set(fromIndex, toIndex);
        else
            clear(fromIndex, toIndex);
    }
```

思路与flip是一样的，只是执行的是与1的or操作。
同时jdk中提供了，具体设置成0或1的操作，以及设置某一区间的操作。



```java
   /**
     * Returns the value of the bit with the specified index. The value
     * is <code>true</code> if the bit with the index <code>bitIndex</code>
     * is currently set in this <code>BitSet</code>; otherwise, the result
     * is <code>false</code>.
     *
     * @param     bitIndex   the bit index.
     * @return    the value of the bit with the specified index.
     * @exception IndexOutOfBoundsException if the specified index is negative.
     */
    public boolean get(int bitIndex) {
	if (bitIndex < 0)
	    throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

	checkInvariants();

	int wordIndex = wordIndex(bitIndex);
	return (wordIndex < wordsInUse)
	    && ((words[wordIndex] & (1L << bitIndex)) != 0);
    }
```

同样的两步走，这里的位操作时&。可以看到，如果指定的bit不存在的话，返回的是false，即没有设置。
jdk同时提供了一个获取指定区间的bitset的方法。当然这里的返回值会是一个bitset，是一个仅仅包含需要查询位的bitset。注意这里的大小也仅仅是刚刚能够容纳必须的位（当然，规整到long的整数倍）。代码如下：

```java
   /**
     * Returns a new <tt>BitSet</tt> composed of bits from this <tt>BitSet</tt>
     * from <tt>fromIndex</tt> (inclusive) to <tt>toIndex</tt> (exclusive).
     *
     * @param     fromIndex   index of the first bit to include.
     * @param     toIndex     index after the last bit to include.
     * @return    a new <tt>BitSet</tt> from a range of this <tt>BitSet</tt>.
     * @exception IndexOutOfBoundsException if <tt>fromIndex</tt> is negative,
     *            or <tt>toIndex</tt> is negative, or <tt>fromIndex</tt> is
     *            larger than <tt>toIndex</tt>.
     * @since   1.4
     */
    public BitSet get(int fromIndex, int toIndex) {
	checkRange(fromIndex, toIndex);

	checkInvariants();

	int len = length();

        // If no set bits in range return empty bitset
        if (len <= fromIndex || fromIndex == toIndex)
            return new BitSet(0);

        // An optimization
        if (toIndex > len)
            toIndex = len;

        BitSet result = new BitSet(toIndex - fromIndex);
        int targetWords = wordIndex(toIndex - fromIndex - 1) + 1;
        int sourceIndex = wordIndex(fromIndex);
	boolean wordAligned = ((fromIndex & BIT_INDEX_MASK) == 0);

        // Process all words but the last word
        for (int i = 0; i < targetWords - 1; i++, sourceIndex++)
            result.words[i] = wordAligned ? words[sourceIndex] :
		(words[sourceIndex] >>> fromIndex) |
		(words[sourceIndex+1] << -fromIndex);

        // Process the last word
	long lastWordMask = WORD_MASK >>> -toIndex;
        result.words[targetWords - 1] =
	    ((toIndex-1) & BIT_INDEX_MASK) < (fromIndex & BIT_INDEX_MASK)
	    ? /* straddles source words */
	    ((words[sourceIndex] >>> fromIndex) |
	     (words[sourceIndex+1] & lastWordMask) << -fromIndex)
	    :
	    ((words[sourceIndex] & lastWordMask) >>> fromIndex);

        // Set wordsInUse correctly
        result.wordsInUse = targetWords;
        result.recalculateWordsInUse();
	result.checkInvariants();

	return result;
    }
```

这里有一个tricky的操作，即fromIndex的那个bit会存在返回bitset的第0个位置，以此类推。如果fromIndex不是word对齐的话，那么返回的bitset的第一个word将会包含fromIndex所在word的从fromIndex开始的到fromIndex+1开始的的那几位（总共加起来是一个word的大小）。

> 其中>>>是无符号位想右边移位的操作符。

### 8. 获取当前bitset总bit的大小

```java
   /**
     * Returns the "logical size" of this <code>BitSet</code>: the index of
     * the highest set bit in the <code>BitSet</code> plus one. Returns zero
     * if the <code>BitSet</code> contains no set bits.
     *
     * @return  the logical size of this <code>BitSet</code>.
     * @since   1.2
     */
    public int length() {
        if (wordsInUse == 0)
            return 0;

        return BITS_PER_WORD * (wordsInUse - 1) +
	    (BITS_PER_WORD - Long.numberOfLeadingZeros(words[wordsInUse - 1]));
    }
```

###  

### 9. hashcode

hashcode是一个非常重要的属性，可以用来表明一个数据结构的特征。bitset的hashcode是用下面的方式实现的：

```java
    /**
     * Returns a hash code value for this bit set. The hash code
     * depends only on which bits have been set within this
     * <code>BitSet</code>. The algorithm used to compute it may
     * be described as follows.<p>
     * Suppose the bits in the <code>BitSet</code> were to be stored
     * in an array of <code>long</code> integers called, say,
     * <code>words</code>, in such a manner that bit <code>k</code> is
     * set in the <code>BitSet</code> (for nonnegative values of
     * <code>k</code>) if and only if the expression
     * <pre>((k&gt;&gt;6) &lt; words.length) && ((words[k&gt;&gt;6] & (1L &lt;&lt; (bit & 0x3F))) != 0)</pre>
     * is true. Then the following definition of the <code>hashCode</code>
     * method would be a correct implementation of the actual algorithm:
     * <pre>
     * public int hashCode() {
     *      long h = 1234;
     *      for (int i = words.length; --i &gt;= 0; ) {
     *           h ^= words[i] * (i + 1);
     *      }
     *      return (int)((h &gt;&gt; 32) ^ h);
     * }</pre>
     * Note that the hash code values change if the set of bits is altered.
     * <p>Overrides the <code>hashCode</code> method of <code>Object</code>.
     *
     * @return  a hash code value for this bit set.
     */
    public int hashCode() {
	long h = 1234;
	for (int i = wordsInUse; --i >= 0; )
            h ^= words[i] * (i + 1);

	return (int)((h >> 32) ^ h);
    }
```

这个hashcode同时考虑了没给word以及word的位置。当有bit的状态发生变化时，hashcode会随之改变。  



## 三. Java中Bitset的使用

BitSet的使用非常简单，只要对需要的操作调用对应的函数即可。

```java

package util;
 
import java.util.Arrays;
import java.util.BitSet;
 
public class BitSetDemo {
 
	/**
	 * 求一个字符串包含的char
	 * 
	 */
	public static void containChars(String str) {
		BitSet used = new BitSet();
		for (int i = 0; i < str.length(); i++)
			used.set(str.charAt(i)); // set bit for char
 
		StringBuilder sb = new StringBuilder();
		sb.append("[");
		int size = used.size();
		System.out.println(size);
		for (int i = 0; i < size; i++) {
			if (used.get(i)) {
				sb.append((char) i);
			}
		}
		sb.append("]");
		System.out.println(sb.toString());
	}
 
	/**
	 * 求素数 有无限个。一个大于1的自然数，如果除了1和它本身外，不能被其他自然数整除(除0以外）的数称之为素数(质数） 否则称为合数
	 */
	public static void computePrime() {
		BitSet sieve = new BitSet(1024);
		int size = sieve.size();
		for (int i = 2; i < size; i++)
			sieve.set(i);
		int finalBit = (int) Math.sqrt(sieve.size());
 
		for (int i = 2; i < finalBit; i++)
			if (sieve.get(i))
				for (int j = 2 * i; j < size; j += i)
					sieve.clear(j);
 
		int counter = 0;
		for (int i = 1; i < size; i++) {
			if (sieve.get(i)) {
				System.out.printf("%5d", i);
				if (++counter % 15 == 0)
					System.out.println();
			}
		}
		System.out.println();
	}
	
	/**
	 * 进行数字排序
	 */
	public static void sortArray() {
		int[] array = new int[] { 423, 700, 9999, 2323, 356, 6400, 1,2,3,2,2,2,2 };
		BitSet bitSet = new BitSet(2 << 13);
		// 虽然可以自动扩容，但尽量在构造时指定估算大小,默认为64
		System.out.println("BitSet size: " + bitSet.size());
 
		for (int i = 0; i < array.length; i++) {
			bitSet.set(array[i]);
		}
		//剔除重复数字后的元素个数
		int bitLen=bitSet.cardinality();	
 
		//进行排序，即把bit为true的元素复制到另一个数组
		int[] orderedArray = new int[bitLen];
		int k = 0;
		for (int i = bitSet.nextSetBit(0); i >= 0; i = bitSet.nextSetBit(i + 1)) {
			orderedArray[k++] = i;
		}
 
		System.out.println("After ordering: ");
		for (int i = 0; i < bitLen; i++) {
			System.out.print(orderedArray[i] + "\t");
		}
		
		System.out.println("iterate over the true bits in a BitSet");
		//或直接迭代BitSet中bit为true的元素iterate over the true bits in a BitSet
		for (int i = bitSet.nextSetBit(0); i >= 0; i = bitSet.nextSetBit(i + 1)) {
			System.out.print(i+"\t");
		}
		System.out.println("---------------------------");
	}
	
	/**
	 * 将BitSet对象转化为ByteArray
	 * @param bitSet
	 * @return
	 */
	public static byte[] bitSet2ByteArray(BitSet bitSet) {
        byte[] bytes = new byte[bitSet.size() / 8];
        for (int i = 0; i < bitSet.size(); i++) {
            int index = i / 8;
            int offset = 7 - i % 8;
            bytes[index] |= (bitSet.get(i) ? 1 : 0) << offset;
        }
        return bytes;
    }
 
	/**
	 * 将ByteArray对象转化为BitSet
	 * @param bytes
	 * @return
	 */
    public static BitSet byteArray2BitSet(byte[] bytes) {
        BitSet bitSet = new BitSet(bytes.length * 8);
        int index = 0;
        for (int i = 0; i < bytes.length; i++) {
            for (int j = 7; j >= 0; j--) {
                bitSet.set(index++, (bytes[i] & (1 << j)) >> j == 1 ? true
                        : false);
            }
        }
        return bitSet;
    }
	
	/**
	 * 简单使用示例
	 */
	public static void simpleExample() {
		String names[] = { "Java", "Source", "and", "Support" };
		BitSet bits = new BitSet();
		for (int i = 0, n = names.length; i < n; i++) {
			if ((names[i].length() % 2) == 0) {
				bits.set(i);
			}
		}
 
		System.out.println(bits);
		System.out.println("Size : " + bits.size());
		System.out.println("Length: " + bits.length());
		for (int i = 0, n = names.length; i < n; i++) {
			if (!bits.get(i)) {
				System.out.println(names[i] + " is odd");
			}
		}
		BitSet bites = new BitSet();
		bites.set(0);
		bites.set(1);
		bites.set(2);
		bites.set(3);
		bites.andNot(bits);
		System.out.println(bites);
	}
 
	public static void main(String args[]) {
		//BitSet使用示例
		BitSetDemo.containChars("How do you do? 你好呀");
		BitSetDemo.computePrime();
		BitSetDemo.sortArray();
		BitSetDemo.simpleExample();
		
		
		//BitSet与Byte数组互转示例
		BitSet bitSet = new BitSet();
        bitSet.set(3, true);
        bitSet.set(98, true);
        System.out.println(bitSet.size()+","+bitSet.cardinality());
        //将BitSet对象转成byte数组
        byte[] bytes = BitSetDemo.bitSet2ByteArray(bitSet);
        System.out.println(Arrays.toString(bytes));
         
        //在将byte数组转回来
        bitSet = BitSetDemo.byteArray2BitSet(bytes);
        System.out.println(bitSet.size()+","+bitSet.cardinality());
        System.out.println(bitSet.get(3));
        System.out.println(bitSet.get(98));
        for (int i = bitSet.nextSetBit(0); i >= 0; i = bitSet.nextSetBit(i + 1)) {
			System.out.print(i+"\t");
		}
	}
}

```

