## 1、基础知识

### 1）集合和数组的区别

数组是固定长度的；集合可变长度的。

数组可以存储基本数据类型，也可以存储引用数据类型；集合只能存储引用数据类型。

数组存储的元素必须是同一个数据类型；集合存储的对象可以是不同数据类型。

### 2）集合框架的优点

容量自增长；

提供了高性能的数据结构和算法，使编码更轻松，提高了程序速度和质量；

可以方便地扩展或改写集合，提高代码复用性和可操作性。

通过使用JDK自带的集合类，可以降低代码维护和学习新API成本。

### 3）无序性的含义

无序性不等于随机性，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的。

### 4）不可重复性的含义

不可重复性是指添加的元素按照equals()判断时，返回false，需要同时重写equals()方法和HashCode()方法。

### 5）集合框架底层的数据结构

#### a) Collection

**List**

Arraylist：Object[]数组

Vector：Object[]数组

LinkedList：双向链表(JDK1.6之前为循环链表，JDK1.7取消了循环)

 **Set**

HashSet（无序唯一）：基于HashMap实现的，底层采用HashMap来保存元素

LinkedHashSet：LinkedHashSet是HashSet的子类，并且其内部是通过LinkedHashMap来实现的。有点类似于我们之前说的LinkedHashMap其内部是基于HashMap实现一样，不过还是有一点点区别的

TreeSet（有序唯一）：红黑树(自平衡的排序二叉树)

**Queue**

PriorityQueue：Object[]数组来实现二叉堆

ArrayQueue：Object[]数组+双指针

#### b)Map

 HashMap：JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）（将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间

 LinkedHashMap：LinkedHashMap其实是对HashMap进行了拓展，使用了双向链表来保证了顺序性。

 Hashtable：数组+链表组成的，数组是Hashtable的主体，链表则是主要为了解决哈希冲突而存在的

 TreeMap：红黑树（自平衡的排序二叉树）

### 6）为什么HashMap中String、Integer这样的包装类适合作为Key

String、Integer等包装类的特性能够保证Hash值的不可更改性和计算准确性，能够有效的减少Hash碰撞的几率

都是final类型，即不可变性，保证key的不可更改性，不会存在获取hash值不同的情况

内部已重写了equals()、hashCode()等方法，遵守了HashMap内部的规范，不容易出现Hash值计算错误的情况

### 7）Object作为HashMap的Key，应该注意什么

重写hashCode()和equals()方法

重写hashCode()是因为需要计算存储数据的存储位置，需要注意不要试图从散列码计算中排除掉一个对象的关键部分来提高性能，这样虽然能更快但可能会导致更多的Hash碰撞

重写equals()方法，需要遵守自反性、对称性、传递性、一致性以及对于任何非null的引用值x，x.equals(null)必须返回false的这几个特性，目的是为了保证key在哈希表中的唯一性

详解

当你使用Object类作为map的key时，如果你不从写hashcode方法，那么他会继承父类，对整个类进行hashcode计算，如果碰到两个对象中的属性都是一样的，实际生活场景我们希望比较的是它的内容是否相同，而不是它存储的地址是否相同，并且两个不同对象的地址也不可能相同，而要比较他们是否相同，得先让他们放到同一个map中的bucket位置。重写hashcode方法后，如果我们不重写它的equals方法，那么当发生两个相同对象（指的是属性相同，但是两个不同对象）hash碰撞时（此时两个对象的hashcode相同），当使用equals方法判断时，还是会去比较两个key的地址是否相同，如果不重写equals方法，map将以链表（或二叉树）的形式将这两个对象存储在一个bucker的位置。

## 2、区别

### 1）comparable和comparator的区别

comparable接口出自java.lang包，它有一个compareTo(Objectobj)方法用来排序

comparator接口出自java.util包，它有一个compare(Objectobj1,Objectobj2)方法用来排序

一般我们需要对一个集合使用自定义排序时，我们就要重写compareTo()方法或compare()方法

### 2）Collection和Collections有什么区别？

java.util.Collection是一个集合接口（集合类的一个顶级接口）。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式，其直接继承接口有List与Set。

Collections则是集合类的一个工具类，其中提供了一系列静态方法，用于对集合中元素进行排序、搜索以及线程安全等各种操作。

### 3）Iterator和ListIterator有什么区别？

Iterator可以遍历Set和List集合，而ListIterator只能遍历List。

Iterator只能单向遍历，而ListIterator可以双向遍历（向前/后遍历）。

ListIterator实现Iterator接口，然后添加了一些额外的功能，比如添加一个元素、替换一个元素、获取前面或后面元素的索引位置。

### 4）List

#### a) ArrayList和Vector的区别

线程安全

Vector使用了Synchronized来实现线程同步，是线程安全的，而ArrayList是非线程安全的。

性能

ArrayList在性能方面要优于Vector。

扩容机制

ArrayList和Vector都会根据实际的需要动态的调整容量，只不过在Vector扩容每次会增加1倍，而ArrayList只会增加50%。

#### b) ArrayList和LinkedList的区别

数据结构实现：

ArrayList是动态数组的数据结构实现，而LinkedList是双向链表的数据结构实现。

随机访问效率：

ArrayList比LinkedList在随机访问的时候效率要高，因为LinkedList是线性的数据存储方式，所以需要移动指针从前往后依次查找。

增加和删除效率：

在非首尾的增加和删除操作，LinkedList要比ArrayList效率要高，因为ArrayList增删操作要影响数组内的其他数据的下标。

内存空间占用：

LinkedList比ArrayList更占内存，因为LinkedList的节点除了存储数据，还存储了两个引用，一个指向前一个元素，一个指向后一个元素。

线程安全：

ArrayList和LinkedList都是不同步的，也就是不保证线程安全。

### 5）Set

#### hashset、linkedhashset和treeset三者的异同

HashSet是Set接口的主要实现类，HashSet的底层是HashMap，线程不安全的，可以存储null值；

LinkedHashSet是HashSet的子类，能够按照添加的顺序遍历；

TreeSet底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。

### 6）Queue

#### a) Queue与Deque的区别

Queue是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循先进先出（FIFO）规则。

Queue扩展了Collection的接口，根据因为容量问题而导致操作失败后处理方式的不同可以分为两类方法:一种在操作失败后会抛出异常（add、remove、element），另一种则会返回特殊值（offer、poll、peek）。

Deque是双端队列，在队列的两端均可以插入或删除元素。

Deque扩展了Queue的接口,增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：

抛出异常

addFirst(Ee)、addLast(Ee)、removeFirst()、removeLast()、getFirst()、getLast()

返回特殊值

offerFirst(Ee)、offerLast(Ee)、pollFirst()、pollLast()、peekFirst()、peekLast()

#### b) ArrayDeque与LinkedList的区别

ArrayDeque和LinkedList都实现了Deque接口，两者都具有队列的功能

ArrayDeque是基于可变长的数组和双指针来实现，而LinkedList则通过链表来实现。

ArrayDeque不支持存储NULL数据，但LinkedList支持。

ArrayDeque是在JDK1.6才被引入的，而LinkedList早在JDK1.2时就已经存在。

ArrayDeque插入时可能存在扩容过程,不过均摊后的插入操作依然为O(1)。虽然LinkedList不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。

从性能的角度上，选用ArrayDeque来实现队列要比LinkedList更好。

#### c) PriorityQueue

PriorityQueue是在JDK1.5中被引入的,其与Queue的区别在于元素出队顺序是与优先级相关的，即总是优先级最高的元素先出队。

PriorityQueue利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据

PriorityQueue通过堆元素的上浮和下沉，实现了在O(logn)的时间复杂度内插入元素和删除堆顶元素

PriorityQueue是非线程安全的，且不支持存储NULL和non-comparable的对象。

PriorityQueue默认是小顶堆，但可以接收一个Comparator作为构造参数，从而来自定义元素优先级的先后。

### 7）Map

#### a) HashMap和Hashtable的区别

线程安全：

HashMap是非线程安全的，HashTable是线程安全的,因为HashTable内部的方法基本都经过synchronized修饰。（如果你要保证线程安全的话就使用ConcurrentHashMap吧！）

效率：

因为线程安全的问题，HashMap要比HashTable效率高一点。另外，HashTable基本被淘汰，不要在代码中使用它；

对Nullkey和Nullvalue的支持：

HashMap可以存储null的key和value，但null作为键只能有一个，null作为值可以有多个；HashTable不允许有null键和null值，否则会抛出NullPointerException

初始容量大小和每次扩充容量大小的不同：

创建时如果不指定容量初始值，Hashtable默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。

创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小（HashMap中的tableSizeFor()方法保证

底层数据结构：

JDK1.8以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）（将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable没有这样的机制。

#### b) HashMap和HashSet的区别

HashSet底层就是基于HashMap实现的

HashMap实现了Map接口，HashSet实现Set接口

HashMap用来存储键值对，HashSet仅存储对象

HashMap调用put()向map中添加元素，HashSet调用add()方法向Set中添加元素

HashMap使用键（Key）计算hashcode，HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性

#### c) HashMap和TreeMap的区别

TreeMap和HashMap都继承自AbstractMap，但是需要注意的是TreeMap它还实现了NavigableMap接口和SortedMap接口。

实现NavigableMap接口让TreeMap有了对集合内元素的搜索的能力。

实现SortMap接口让TreeMap有了对集合中的元素根据键排序的能力。默认是按key的升序排序，不过我们也可以指定排序的比较器。

综上，相比于HashMap来说TreeMap主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。

#### d) ConcurrentHashMap和Hashtable的区别

ConcurrentHashMap和Hashtable的区别主要体现在实现线程安全的方式上不同。

底层数据结构

JDK1.7的ConcurrentHashMap底层采用分段的数组+链表实现；

JDK1.8采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。

Hashtable和JDK1.8之前的HashMap的底层数据结构类似都是采用数组+链表的形式，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的。

实现线程安全的方式

在JDK1.7的时候，ConcurrentHashMap（分段锁）对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。到了JDK1.8的时候已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized和CAS来操作。（JDK1.6以后对synchronized锁做了很多优化）整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本；

Hashtable(同一把锁):使用synchronized来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用put添加元素，另一个线程不能使用put添加元素，也不能使用get，竞争会越来越激烈效率越低。

## 3、源码分析

### 1）ArrayList

#### a)  基础知识

 优缺点

优点

ArrayList底层以数组实现，是一种随机访问模式。ArrayList实现了RandomAccess接口，因此查找的时候非常快。

ArrayList在顺序添加一个元素的时候非常方便。

缺点

删除元素的时候，需要做一次元素复制操作。如果要复制的元素很多，那么就会比较耗费性能。

插入元素的时候，也需要做一次元素复制操作，缺点同上。

 适用场景

顺序添加、随机访问的场景。

#### b)   为什么ArrayList的elementData加上transient修饰？

ArrayList 实现了 Serializable 接口，这意味着 ArrayList 支持序列化。transient 的作用是说不希望 elementData 数组被序列化，重写了 writeObject 实现。每次序列化时，先调用 defaultWriteObject() 方法序列化 ArrayList 中的非 transient 元素，然后遍历 elementData，只序列化已存入的元素，这样既加快了序列化的速度，又减小了序列化之后的文件大小。

### 2）HashMap

#### a)  数据结构

1.8：数组+链表/红黑树

1.7：数组+链表

#### b)   HashMap的插入原理

   i.  判断数组是否为空，为空进行初始化;

  ii.  不为空，计算k的hash值，通过(n-1)&hash计算应当存放在数组中的下标index;

  iii.  查看table[index]是否存在数据，没有数据就构造一个Node节点存放在table[index]中；

  iv.  存在数据，说明发生了hash冲突(存在二个节点key的hash值一样),继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；

  v.  如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)

  vi.  如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于8并且数组长度大于64，大于的话链表转换为红黑树；

 vii.  插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。

#### c)   说说Hashmap的初始值

一般如果newHashMap()不传值，默认大小是16，负载因子是0.75，如果自己传入初始大小k，初始化大小为大于k的2的整数次方，例如如果传10，大小为16。

#### d)   HashMap的工作原理

HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，然后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。HashMap在每个链表节点中储存键值对对象。

当两个不同的键对象的hashcode相同时会发生什么？它们会储存在同一个bucket位置的链表中。键对象的equals()方法用来找到键值对。

#### e)   Hashmap是如何扩容的

扩容：创建一个新的Entry空数组，长度是原数组的2倍。

ReHash：遍历原Entry数组，把所有的Entry重新Hash到新数组。

#### f)   为什么1.7使用的是尾插法，而1.8改用头插法？

在多线程下可能出现环形链表

#### g)   Hashmap初始长度为什么是2的幂？

因为在使用不是2的幂的数字的时候，Length-1的值是所有二进制位全为1，这种情况下，index的结果等同于HashCode后几位的值。

#### h)   一般在多线程的场景，如何使用map集合

 Hashtable

 ConcurrentHashMap

 使用Collections.synchronizedMap(Map)创建线程安全的map集合；

#### i)   你知道HashMap的哈希函数怎么设计的吗？为什么怎么设计？

hash函数是先拿到key的hashcode，是一个32位的int值，然后让hashcode的高16位和低16位进行异或操作。

这个也叫扰动函数，这么设计有二点原因

一定要尽可能降低hash碰撞，越分散越好；

算法一定要尽可能高效，因为这是高频操作,因此采用位运算；

#### j) 为什么采用hashcode的高16位和低16位异或能降低hash碰撞？hash函数能不能直接用key的hashcode？

因为key.hashCode()函数调用的是key键值类型自带的哈希函数，返回int型散列值。int值范围为-2147483648~2147483647，前后加起来大概40亿的映射空间。只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。你想，如果HashMap数组的初始大小才16，用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标。

#### k) 1.8除了对hash函数做了优化，1.8还有别的优化吗？

数组+链表改成了数组+链表或红黑树；

链表的插入方式从头插法改成了尾插法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后；

扩容的时候1.7需要对原数组中的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小；

在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容

#### l) 你分别跟我讲讲为什么要做这几点优化？

1)    防止发生hash冲突，链表长度过长，将时间复杂度由O(n)降为O(logn);

2)    因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环；

3)    这是由于扩容是扩大为原数组大小的2倍，用于计算数组位置的掩码仅仅只是高位多了一个1

#### m) 链表转红黑树和红黑树转链表的阈值？为什么？

阈值是8，红黑树转链表阈值为6。

因为根据泊松分布，，在hash函数设计合理的情况下，发生hash碰撞8次的几率为百万分之6，概率说话。因为8够用了，至于为什么转回来是6，因为如果hash碰撞次数在8附近徘徊，会一直发生链表和红黑树的互相转化，为了预防这种情况的发生。

#### n) 为什么HashMap使用红黑树而不使用AVL树

 AVL以及红黑树是高度平衡的树数据结构。它们非常相似，真正的区别在于在任何添加/删除操作时完成的旋转操作次数。

 两种实现都缩放为a O(lg N)，其中N是叶子的数量，但实际上AVL树在查找密集型任务上更快：利用更好的平衡，树遍历平均更短。另一方面，插入和删除方面，AVL树速度较慢：需要更多的旋转次数才能在修改时正确地重新平衡数据结构。

 在AVL树中，从根到任何叶子的最短路径和最长路径之间的差异最多为1。在红黑树中，差异可以是2倍。

 两个都是O（log n）查找，但平衡AVL树可能需要O（log n）旋转，而红黑树将需要最多两次旋转使其达到平衡（尽管可能需要检查O（log n）节点以确定旋转的位置）。旋转本身是O（1）操作，因为你只是移动指针。

### 3）Hashtable

#### a)  Hashmap和Hashtable的区别

Hashtable是不允许键或值为null的，HashMap的键值则都可以为null

 实现方式不同

Hashtable继承了Dictionary类，而HashMap继承的是AbstractMap类。Dictionary是JDK1.0添加的

 初始化容量不同

HashMap的初始容量为：16，Hashtable初始容量为：11，两者的负载因子默认都是：0.75

 扩容机制不同

当现有容量大于总容量负载因子时，HashMap扩容规则为当前容量翻倍，Hashtable扩容规则为当前容量翻倍+1

 迭代器不同

HashMap中的Iterator迭代器是fail-fast的，Hashtable的Iterator迭代器也是fail-fast，但它的Enumerator是fail-safe的

#### b) 什么是fail-fast？它的原理

 含义

快速失败（fail—fast）是java集合中的一种机制，在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除），则会抛出ConcurrentModificationException。

 原理

迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个modCount变量。

集合在被遍历期间如果内容发生变化，就会改变modCount的值。

每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

#### c)  什么是fail-safe？它的原理

 含义

fail-safe:这种遍历*基于容器的一个克隆*。因此，对容器内容的修改不影响遍历。java.util.concurrent包下的容器都是安全失败的,可以在多线程下并发使用,并发修改。常见的的使用fail-safe方式遍历的容器有ConcerrentHashMap和CopyOnWriteArrayList等。

 原理

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException。

 缺点

1)    基于拷贝内容的优点是避免了ConcurrentModificationException，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

2)    需要复制集合，产生大量的无效对象，开销大

### 4）  ConcurrentHashMap

### 5）  PriorityQueue