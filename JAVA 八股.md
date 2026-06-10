# JAVA 八股

## Java 基础

###### StringBuilder 和 StringBuffer 

首先 String 是不可变类型，每次修改都是创造一个新的 String 对象，因为 String 的底层维护的是一个不可变 char 型数组。（JDK9 之前用的是 char 类型数组，之后为了节省空间引入了 Compact Strings（紧凑字符串），使用 byte\[\]）

```Java
@Stable
private final byte[] value;
```

说回来 StringBuilder 和 StringBuffer 是**可变类型字符数组**（初始容量为 16）和长度，当长度长度超出时会触发动态扩容。它们两个唯一的不同在于 StringBuffer 是线程安全的，里面的方法都使用** Synchronized** 修饰，而 StringBuilder 是线程不安全的。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NzBmZjE0MzVkN2VmZWRkODdjZmZhZDRhNzQ5ZTQ4MDhfNjNhMzg3YTczMzM5ZjYzNTY5YzgwYjM4Yjc3ZGJhZjVfSUQ6NzYyODgxNzM1NTMwMDg2NzI3OV8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

StringBuffer，StringBuilder 的扩容

**计算新长度：** **newCapacity = oldCapacity \* 2 \+ 2 **（加 2 是为了处理初始情况为 0，因为单纯\*2 还是 0，而且方便运算）。

**创建新数组：** 使用 **new byte\[newCapacity\]** 开辟一段新的**连续**内存空间。

**数据拷贝：** 调用底层 **System\.arraycopy（）**，将旧数组的内容物理拷贝到新数组中。

**丢弃旧数组：** 原来的旧数组失去引用，等待 GC 回收。

---

###### 反射

反射允许程序在**运行时**可以动态的**获取类的信息**，并且能够在运行时创建对象，调用方法，访问修改属性。

**反射的应用**

1️⃣ Spring IOC 通过反射创建 Bean， 注入依赖。

2️⃣ MyBatis 的数据库字段和类中对象属性的映射。

3️⃣ JDK 的动态代理。

4️⃣ BeanUtils 这种工具类和 JSONs 序列化时对字段进行赋值。

---

###### 动态代理

静态代理的缺点：在编译时就要确定好代理关系，然后手动为其编写代理类。

**动态代理的做作用就是对类在运行时生成代理对象，然后可以在不改变原方法的情况下，对其进行增强。**

1️⃣ **JDK 的动态代理**

JDK 的动态代理是基于接口的，要求被代理类要实现的有接口。通过反射包下的那个** Proxy **可以创建一个代理对象，需要传入被代理对象的类加载器，所有实现接口列表和一个** InvocationHandler（）**的实现类，在** InvocationHandler（）**里面重写** invoke（）方法，**可以调用被代理对象的原始方法，并在前后对被代理对象的原始方法做增强。

```Java
UserService target = new UserServiceImpl();

UserService proxy = (UserService) Proxy.newProxyInstance(
    target.getClass().getClassLoader(),
    target.getClass().getInterfaces(),
    new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

            System.out.println("before");

            Object result = method.invoke(target, args); // 调用原方法

            System.out.println("after");

            return result;
        }
    }
);
```

然后，当代理对象被调用方法时实际上调用的是** InvocationHandler **实现类里面被增强后的** invoke **方法。

2️⃣** CGLIB 动态代理**

对于没有实现接口的类要想实现动态代理，可以基于 CGLIB，CGLIB 是基于继承然后对方法进行重写的。直接继承被代理类，然后生成子类，在子类里面重写需要增强的方法。

---

###### JDK8 新特性

1️⃣** lambda 表达式简化函数式接口**

函数式接口：接口中只有一个抽象方法（）可以通过**@FunctionalInterface **注解标注。lambda 表达式用来简化创建**匿名内部类**的过程，简化为（参数） \-\>  \{方法体\} 的格式。

2️⃣ **stream 流**，提供链式操作去处理集合或者数组。

3️⃣ **Optional 类**，对可能为 null 的对象包装，避免空指针的出现。

4️⃣ 接口的默认方法和静态方法。

5️⃣ **CompletableFuture**

###### 策略模式



###### I/O 模型

Java 的 I/O 模型主要包括 BIO、NIO 和 AIO。

BIO 是阻塞模型，一个连接对应一个线程，编程简单但不适合高并发；

NIO 是非阻塞模型，通过 Selector 实现多路复用，一个线程可以处理多个连接，是目前主流方案；

AIO 是异步模型，由操作系统完成 I/O 操作并通过回调通知应用，但在实际开发中使用较少。

在底层，NIO 依赖操作系统的**多路复用机制**（如 epoll），是高性能网络编程的基础。

---

## 集合

---

### 常用集合

###### 介绍一下常用集合

ArrayList， LinkedList， HashMap， ArrayDeque（双端队列）， PriorityQueue（优先级队列）等。

注：这里有关 AraryDeque 和 PriorityQueue 的相关使用（[LeetCode347](https://leetcode.cn/problems/top-k-frequent-elements/?envType=study-plan-v2&envId=top-100-liked)）

###### ArrayDeque

**ArrayDeque 双端队列**的底层是一个**可变长的对象数组**（`Object[] elements`）。它的核心逻辑在于通过两个指针 **`head`** 和 **`tail`** 实现循环利用。

作为**栈（Stack**）使用的常用操作

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZGRkMWJkZThiMmY5MjM5OWY0YTU4ZGQwMTk0NGUzZTFfZDEwMjUyNzFlYWMyZmUzMThlNmVjNjY1YzZhZTM4MmRfSUQ6NzYyNDc2MTIwNTQ0MTM1MDYyNF8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

作为**队列（Queue**）使用的常用操作

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OGViYTg0NDE1YzVkY2RkYjE2Nzg1NWZkZmEwNmRkYzNfMDZmMzM2M2NjNWZiNTc2ZDYxMzExODQwYjkzMDU4NzBfSUQ6NzYyNDc2MTA3NDc1NTE5MzgxN18xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

###### PriorityQueue

**PriorityQueue** 的底层是一个**动态数组**（`Object[] queue`），但它在逻辑上维护的是一棵 **完全二叉树。**

TODO 优先级队列的底层实现 

在使用时可以自定义排序规则（默认是从小到大排序），在构造函数中传入一个 **Comparator **的匿名实现类并重写里面的 **compare（） **方法。

然后每次 offer（入队）会自动调整整个堆，poll（出队）会按照优先级最高（堆顶元素）的出队。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YTQyZDgxNTgwZGRjNmEyZWU0NTc0MWU3YThkYzUwZmFfMjhmMTQwODMwZWVkNTY2ZGNjMmNhNWVhOGVjZDBkNWFfSUQ6NzYyNDc2NTI4NjI3MjU4NDYzNF8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

---

### List

###### 讲一下 java 里面 list 的几种实现，几种实现有什么不同？

**ArrayList:** 底层实现是**动态数组**，线程不安全，支持根据索引**随机访问**，增删要移动整体数组。

**LinkedList: **底层实现是**双向链表**，线程不安全，不支持随机访问，增删只需要修改 node 节点的指针。

**Vector:** 实现与 ArrayList 类似，但是 add， remove， get 等方法进行了 synchronized 修饰，是线程安全的。

**CopyOnWriteArrayList: **实现与 ArrayList 类似，使用 volatile \+ ReentrantLock 实现线程安全。

一些细节：

LinkedList 的底层数据结构是**双向链表**，

```Java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

LinkedList 源码中不仅仅维护了指向**头节点的指针 first**，还专门维护了一个指向**尾节点的指针 last**，直接访问头节点和尾节点的时间复杂度是 O（1）。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWEyM2M4MWE1YmZhNWYwZDE4YzcxMDg2ODg3MzI5NmRfZTczNzU2YzJmYWJhMjU0Y2Q3YzAxMDQ2YjAwNGRlOThfSUQ6NzYyNDc4NzM0NTU2ODUzMzcyM18xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

---

###### list 可以一边遍历一边修改元素吗？

首先，在 ArrayList 等集合内部，维护了一个名为** modCount**（修改计数器）的变量。当调用 list\.iterator（） 时，迭代器会记录当时的 modCount 的值为 **expectedModCount**。下一次迭代器执行 **next（）** 时，会检查 `modCount == expectedModCount`。如果不相等，说明集合被改过了，立刻抛出 **ConcurrentModificationException**

这里的 **修改** 是指**结构性修改操作**，指改变了集合的 `size`。例如 `add()`、`remove()`、`clear()`。这些操作会改变 `modCount`。

因此，使用迭代器或者 foreach 都会有可能造成** ConcurrentModificationException，推荐方法是：**

使用迭代器自身的 **add**， **remove，set **等方法，

使用 JAVA8 之后的 removeIf（）。

---

###### ArrayList 变成线程安全有哪些方法？

使用 **Collections** 包下的 **Collections\.synchronizedList（）** 包装，

使用并发包下的 **CopyOnWriteArrayList，**

使用 vector 。

---

###### ArrayList 中的线程安全问题？

首先，根据 ArrayList 中 add（）的源码

1️⃣ add\(\)整个过程都不是原子操作，包含容量判断，size\+\+，复制。 

2️⃣ 扩容过程（数组复制）没有同步控制，多线程下可能导致数据不一致

3️⃣ `size` 和 `elementData` 之间缺乏可见性和一致性保障，可能出现数组越界异常

4️⃣ `modCount` 不是线程安全的，会影响 fail\-fast 机制的准确性

---

###### ArrayList 扩容机制？

ArrayList 中的扩容主要是 **grow（） **方法，参数 minCapacity 是要求的最小容量，然后判断如果是 list 已经有值就根据 ArraysSupport\.newLength 获取新的容量。如果还是默认空数组 → 用默认容量初始化（**初始的容量是 10**）。

**ArraysSupport\.newLength（）：**并不是每次扩容一定是原来的 1\.5 倍，如果 **addAll（）**一个非常大的数组，导致扩容 1\.5 倍也不够用，会取 oldLength \+ Math\.*max*（minGrowth， prefGrowth）。

```TypeScript
// 扩容函数
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != *DEFAULTCAPACITY_EMPTY_ELEMENTDATA*) {
        int newCapacity = ArraysSupport.*newLength*(oldCapacity,
                minCapacity - oldCapacity,  // 至少还需要的容量
                oldCapacity >> 1           // 进行位运算扩容1.5倍的容量
                );
        
        // 复制到新数组中
        return elementData = Arrays.*copyOf*(elementData, newCapacity);
    } else {
        // 第一次添加值，初始化数组容量为10
        return elementData = new Object[Math.*max*(*DEFAULT_CAPACITY*, minCapacity)];
    }
}

// 计算扩容后新数组的容量
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {

    int prefLength = oldLength + Math.*max*(minGrowth, prefGrowth); // might overflow、
    // 新容量大于0并且小于数组容量上限
    if (0 < prefLength && prefLength <= *SOFT_MAX_ARRAY_LENGTH*) {
        return prefLength;
    } else {
         // 溢出兜底
        return *hugeLength*(oldLength, minGrowth);
    }
}
```

**为什么是 1\.5 倍？**

首先要限制扩容的次数每次进行数组复制的开销很大，其次，在计算新数组容量时，1\.5 刚好在原容量的基础上加 1/2，可以利用位运算提高速度。1\.5 倍扩容是在“减少扩容次数”和“节省内存空间”之间的最优平衡点。

---

###### CopyOnWriteArrayList 如何实现线程安全？

通过**共享读**，**复制写**的操作实现线程安全。

**写操作**：以 add（）操作时为例（set（） 和 remove（） 也类似）

① 加锁

② 复制原数组**（长度＋1）**

③ 修改

④ 修改原数组的指向

⑤ 释放锁

```Java
*/***
* * Appends the specified element to the end of this list.*
* **
* * @param e element to be appended to this list*
* * @return {@code true} (as specified by {@link Collection#add})*
* */*
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.*copyOf*(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}
```

**读操作：**

```Java
*/***
* * The lock protecting all mutators.  (We have a mild preference*
* * for builtin monitors over ReentrantLock when either will do.)*
* */*
final transient Object lock = new Object();

*/** The array, accessed only via getArray/setArray. */*
private transient volatile Object[] array;
```

正常在原数组上读，但是通过** volatile **修饰，防止指令重排, 保证线程间的可见性。


**注：**在 Java 8 中，CopyOnWriteArrayList 使用的是 ReentrantLock。**但在 Java 11\+ 中，它改为了使用内置的 synchronized**。这是因为随着 JVM 的优化，synchronized 的性能在很多场景下已经不亚于甚至优于 ReentrantLock，且能减少内存开销。

---

###### **List 与数组的互转？**

这里我全部都使用 stream 流的写法。

List \-\> 数组， **显式拆箱 mapToInt\(\) \+ toArray（）**

```Java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);

// 在stream流中使用mapToInt显式拆箱
int[] array = list.stream().mapToInt(Integer::intValue).toArray();
```

数组 \-\> List，**显式装箱 boxed\(\) \+  toList（）**

```JavaScript
// String[] 对象数组可以直接Arrays.List()
 String[] strArr = new String[]{"1", "2", "3"};
 List<String> list1 = Arrays.*asList*(strArr);
 
 // 基本数据类型要转为包装类，集合中的泛型不支持基本数据类型
 int[] arr = new int[]{1, 2, 3, 4};
// List<int[]> list1 = Arrays.asList(arr);
 List<Integer> arr2List = Arrays.*stream*(arr).boxed().toList();
```

---

### Map 

###### 如何对 map 进行遍历？

使用增强 for 对 map\.entrySet（）遍历，获取到 entry，通过 getKey（），getValue（）获取 k 和 v。

```Go
HashMap<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);
map.put("c", 3);

for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.*out*.println("key: " + entry.getKey() + ", value: " + entry.getValue());
}
```

对 map\.KeySet（）得到 key，再通过 get（）得到 value

```Java
for (String key : map.keySet()) {
    System.*out*.println("key: " + key + ", value: " + map.get(key));
}
```

使用 map\.entry（）的迭代器 iterator

```JavaScript
Iterator<Map.Entry<String, Integer>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry<String, Integer> entry = iterator.next();
    System.*out*.println("key: " + entry.getKey() + ", value: " + entry.getValue());
}
```

使用 Lamda 表达式和 foreach（）方法

```Java
map.forEach((key, value) -> System.*out*.println("key: " + key + ", value: " + value));
```

使用 stream 流

```Java
map.entrySet().stream()
              .forEach(entry -> System.*out*.println("key: " + entry.getKey() + ", value: " + entry.getValue()));
```

---

###### HashMap 的实现原理和底层数据结构？

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NjM3OGI4NDdhNDgzYjkxN2E5OGUzZDY1NDExYmVmNmNfOGM1NDg4ZjIxY2JlNWUxODkzNmYwYjIzNmNjNmYzMzhfSUQ6NzYyNTA1NDU4MDU2MDEzNzE3Ml8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

HashMap 的底层数据结构为**数组，链表，红黑树。**

HashMap 底层使用一个 Node\<K，V\> \[\] 数组（初始大小为 16），每个位置（bucket）存放键值对，通过哈希函数将键值对映射到数组对应的位置。如果多个键值对映射到同一个位置，会在同一个 bucket 上以链表的形式存储，当链表长度大于 8 且数组长度大于 64 时，为了提高 HashMap 的查询效率会对链表优化为**红黑树（Olog（n））**。

---

###### HashMap 底层为什么要用红黑树而不用平衡二叉树？

红黑树追求的是一种近似的平衡，整棵树的最大高度和最小高度不能相差 2 倍。平衡二叉树要求的是绝对的平衡，即每个节点的左右子树高度之差不能超过 1。

理论上来说使用平衡二叉树的查询效率比红黑树要好，但是在 map 中还存在 put，remove 操作，对于平衡二叉树而言每次插入删除时都要满足完全平衡的条件去旋转调整树的结构**，红黑树在“插入、删除”导致的重平衡开销与“查询效率”之间取得了更好的权衡。**

注：为什么链表\-\>红黑树的阈值为 8？

这其实跟泊松分布（Poisson Distribution）有关。在 Hash 算法正常的情况下，同一个桶位能积压 8 个元素的概率极低（约为千万分之六）。选择 8 是为了在绝大多数正常情况下保持链表的高效，只在极少数异常情况下才动用红黑树。

---

###### HashMap 的扩容机制介绍一下？

**触发扩容的条件：**当 map 中的元素个数超出了负载因子（默认为 0\.75）时会触发扩容。

**扩容大小：**将创建一个新的，大小为原数组大小**两倍**的数组。

将原数组中的 node 移动到新的数组中，这个过程不需要重新计算在新数组中的索引。原因如下，计算在 HashMap 中的过程：

1️⃣由 key 计算出 hashcode 

2️⃣index = hashcode \& \(cap \- 1\)

由于每次的扩容都是将原 cap 变为原大小的 2 倍，因为容量翻倍，二进制表示中只是在高位多了一个 `1`。我们只需要看原哈希值在那一个新增位上是 0 还是 1 即可，如果是 0，1\&0 = 0，在新的哈希表中的位置保持不变，反之，在新的哈希表中的位置 = oldCap \+ oldIndex。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ODIwZGE4Yzc2ZTFiNjY5YzAzNjkzMjIwNWMxNTRmNDNfMWU3YjQ1ZmI3NzFkZGFiODRkMjM4OGEzMWFlODU5YmRfSUQ6NzYyNTA2OTMzMjU3MzM0MjY0OV8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

---

###### 为什么 HashMap 的大小为 2 的幂？

**1️⃣**用 与`&` 运算代替 取模`%`。

**2️⃣**利用 `n-1` 全是 1 的特性，让 Hash 分布更均匀，减少碰撞。

**3️⃣**扩容时数据迁移更高效。

---

###### 解决哈希冲突的方法？

1️⃣**链接法**，当发生哈希冲突时使用链表等数据结构将发生冲突的节点放在同一个 bucket 下。

2️⃣**开放寻址法**，发生冲突后向后找下一个可用的 bucket。

3️⃣**多重哈希法**，使用多个哈希函数避免冲突。

4️⃣哈希桶扩容：当哈希冲突过多时，可以动态地扩大哈希桶的数量。

---

###### 在 Java 的 hashmap 中 get 和 put 的过程 ？

put\(\)

1️⃣** 确定位置。**根据 key 去 hash（key）计算 hashcode，与数组长度做位运算得到映射的数组下标。

2️⃣ **判断初始化。**如果是第一次 put，这时数组为 null 会对数组大小进行初始化操作，初始化大小为 16。

3️⃣ **插入元素处理冲突。**对应数组 bucket 位置为空，new 一个 entry 直接插入，如果数组对应位置不为空发生了冲突，则需要遍历对应的链表或者红黑树通过比较是否有相同的 key。有相同的 key 直接覆盖旧的 value，key 不同就插入对应的位置。

4️⃣** 转红黑树。**如果是链表插入后还要判断如果满足链表长大于 8 且数组长度大于 64 时将链表转为红黑树。

5️⃣** 检查扩容。**而对方当达到负载因子要求大小时，触发 hashmap 的扩容机制。



get\( \) 

1️⃣ **计算 Hash 值**：对 Key 进行同样的 hash（key） 计算，找到对应的数组下标。

2️⃣ **首节点匹配**：直接检查桶位的第一个节点。如果第一个节点的 hash 相等且 equals 返回 true，直接返回。

3️⃣ **遍历查找**：如果首节点不匹配，判断后续结构：红黑树：调用 getTreeNode 在 $O(\log n)$ 时间内找到节点。链表：通过 next 指针循环遍历，直到找到 equals 为 true 的节点。

4️⃣ **返回结果**：找到则返回 value，没找到则返回 null。



注：无论是 get 操作还是 put 操作都会发生线程安全问题。

---

###### 为啥 String 适合做 Map 的 Key 呢？

因为 String 是不可变的，每次修改 String 的值，实际上是 new 了一个新的 String 对象。

一旦一个 String 对象被创建，它的内容就不会改变。这意味着它的 hashCode 在整个生命周期内也是恒定的。

而且由于 `String` 是不可变的，在 String 内部做了一个非常聪明的优化：**缓存 ****`hash`**** 值**。

```Java
public final class String implements ... {
    private int hash; // 缓存 hash 码，默认为 0
    
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            // 只有第一次调用时才计算，之后直接返回 h
            hash = h = isLatin1() ? StringLatin1.hashCode(value) : StringUTF16.hashCode(value);
        }
        return h;
    }
}
```

注：为什么数组做不了 Key？[LeetCode 字母异位词](https://leetcode.cn/problems/group-anagrams/?envType=study-plan-v2&envId=top-100-liked)

- **`equals()`**** 的问题**：数组没有重写 `Object` 类的 `equals()` 方法。这意味着它比较的是**内存地址**（引用），而不是数组里的内容。

    - 即便两个 `char[]` 都存着 `['a', 'b', 'c']`，只要它们是不同的对象，`a1.equals(a2)` 就会返回 `false`。

- **`hashCode()`**** 的问题**：数组的 `hashCode` 是基于对象地址生成的，而不是基于内容。

    - 内容完全一样的两个数组，它们的 `hashCode` 大概率不同。

**结果**：用一个数组存进去，下次用一个内容一模一样的数组去取，结果是 `null`。

---

###### 重写 HashMap 的 equals 和 hashcode 方法需要注意什么？

**要保证 equals（）相同时，hashcode（）一定相同，hashcode 相同时，equals 可以不相同。**

```TypeScript
public class TicketKey {
    String trainId; // 车次
    String seatNo;  // 座位号

    public TicketKey(String trainId, String seatNo) {
        this.trainId = trainId;
        this.seatNo = seatNo;
    }
    // 假设这里没有重写 equals 和 hashCode
}

Map<TicketKey, String> orderMap = new HashMap<>();
// 1. 存入一张 G1024 次列车 01A 座位的订单
TicketKey key1 = new TicketKey("G1024", "01A");
orderMap.put(key1, "张三的订单");

// 2. 尝试用同样的信息去查询
TicketKey key2 = new TicketKey("G1024", "01A");
System.out.println(orderMap.get(key2)); // 输出：null ！！
```

如果不重写 hashCode：会导致 HashMap 里的内容相同的对象分散在不同的桶中，无法通过 Key 取值。

如果不重写 equals：即使两个对象落到了同一个桶里，HashMap 也会觉得它们是两个不同的对象（因为默认 == 比较地址），依然取不到值。

---

###### ConcurrentHashMap 怎么实现的？

JDK1\.8 开始，主要通过** Node（volatile） \+ CAS \+ synchroized**

数组的节点（Node）内部的 val 和 next 指针都加了 volatile，确保了多线程间的可见性。

```Java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;   // 使用volatile修饰 node的value
    volatile Node<K,V> next; // // 使用volatile修饰node的next指针
```

在插入元素是通过 CAS 或者 synchroized 保证线程安全，这里的锁是 bucket 级别的。

put 过程：

1️⃣ **计算 Hash**：对 Key 进行哈希扰动（ 计算 hashcode 时混合高低位，增加随机性 减少冲突 ）。

2️⃣ **死循环自旋 （for）**：不断尝试插入，直到成功。

3️⃣ **如果槽位为空**：**使用 CAS 尝试插入新节点**。如果成功，则直接返回；如果失败（说明有其他线程在抢占），则进入下一次循环。

4️⃣ **如果正在扩容** （MOVED 状态）：当前线程会去**协助扩容**（这也是它高性能的秘诀之一）。

5️⃣ **如果发生哈希冲突**：使用 synchronized 锁住当前的首节点（桶位头）。此时锁的粒度非常细：只锁住这一个桶，其他桶位的读写完全不受影响。

6️⃣ 在锁内进行链表或红黑树的**插入/覆盖**。

7️⃣ **检查树化**：如果链表过长，转为红黑树（链表长大于 8 且数组长度大于 64）。

8️⃣ 根据负载因子，检查是否应该**扩容**。

JDK1\.7 的实现方式

在 JDK1\.7 中，ConcurrentHashMap 采用分段锁机制实现并发控制。

它将整个 Map 分成多个 Segment，每个 Segment 继承自 ReentrantLock，相当于一个独立的小 HashMap。

在写操作时，只需要锁住对应的 Segment，从而允许多个线程同时操作不同的 Segment，提高并发性能。

在读操作时，不需要加锁，因为 value 使用 volatile 修饰，保证了可见性。

---

###### 已经用了 synchronized，为什么还要用 CAS 呢？

ConcurrentHashMap 使用这两种手段来保证线程安全主要是一种权衡的考虑，在某些操作中使用 synchronized，还是使用 CAS，主要是根据锁竞争程度来判断的。

**桶位为空时（使用**** CAS****）：**

操作：直接用 CAS（ **CPU 提供的一条原子指令**） 尝试把新节点放进去。

此时没有哈希冲突，不需要“锁”住任何人。如果多个线程同时发现这个位置是空的，只有一个能 CAS 成功，其他的会失败并进入下一次循环（自旋）。完全无锁化。在高并发初始化桶位时，CAS 直接加锁快得多。

**桶位不为空/发生碰撞时（使用 synchronized）**

操作：通过 synchronized（f） 锁住这个桶的首节点 `f`。

逻辑复杂性：在链表中寻找位置、对比 equals、插入尾部、或者在红黑树中平衡旋转**，这些操作包含多个步骤，无法通过单一的 CPU CAS 指令完成。**

---

###### hashtable 和 concurrentHashMap 有什么区别

Hashtable 使用**全表加锁**，线程安全但并发性能差；ConcurrentHashMap 采用更细粒度的并发控制（JDK1\.7 分段锁，JDK1\.8 CAS \+ synchronized），在保证线程安全的同时大幅提升了并发性能。

---

###### Guava 和 Caffeine

Guava 和 Caffeine 都是本地缓存，两者的底层数据结构和实现原理不同：

Guava

数据结构：哈希表 \+ 双向链表（实现 LRU）

并发控制：使用类似于 jdk1\.7 之前 ConcurrentHashMap 的分段锁

淘汰算法：LRU \+ LFU

Caffeine

数据结构：ConcurrentHashMap \+ 多队列

并发控制：CAS \+ synchronized

淘汰算法：**W\-TinyLFU**

LRU（Least Recently Used）淘汰最久没用的，LFU（Least Frequently Used）淘汰最不经常用的。

Guava Cache 基于分段锁和 LRU 算法实现，在高并发场景下存在锁竞争问题，且命中率相对较低。C**affeine 是 Guava Cache 的升级版本，采用 CAS 和无锁结构优化并发性能，并引入 W\-TinyLFU 算法，结合访问频率和时间进行缓存淘汰，显著提升了命中率和性能。**

---

### Set

###### Set 集合有什么特点？如何实现 key 无重复的？

**HashSet** 的底层其实就是一个 **HashMap****，**对应** TreeSet **的底层是 **TreeMap（有序的）。**

HashSet：利用 HashMap 的 Key 唯一性（hashCode \+ equals）。

TreeSet：利用 TreeMap 的 Key 唯一性（Comparable / Comparator）。

TreeSet compareTo（自然排序）或 compare（定制排序）方法的返回值来判定。判定标准：如果 compare（a， b） == 0，则认为这两个元素是重复的，不会存入。

---

###### 有序的 Set 是什么？记录插入顺序的集合是什么？

Set 结合本身无序，唯一的。有序的 Set 是** TreeSet **和** LinkedHashSet**。

TreeSet 是基于红黑树实现，TreeSet 默认按照元素的“自然顺序”（实现 `Comparable` 接口，如数字升序、字符串字典序）排序，也可以在构造时传入自定义的 `Comparator`。

记录插入顺序的集合通常指的是 LinkedHashSet，由于多维护了一个双向链表，它不仅保证元素的唯一性，还可以保持元素的插入顺序。

---

## JUC

### 线程

###### Java 的内存模型（JMM）介绍一下？

JMM 的作用是定义多线程环境下，变量如何在内存中存储、读取，以及线程之间如何通信的规范，保证在多线程环境下**原子性**、**可见性**和**有序性**。

JMM 把内存分为两部分：**主内存**和**工作内存，**主内存是线程之间共享，工作内存是每个线程私有的，线程修改变量的过程：

1️⃣ 线程从主内存拷贝变量副本到工作内存。

2️⃣ 线程在工作内存中执行代码，修改副本。

3️⃣ 线程将修改后的值刷新回主内存。

这样做会有问题，线程 A 修改了变量其他线程 B 看不到，多个线程同时修改时结果会错乱，（编译器和处理器）会在**不影响单线程结果的情况下**做指令重排也会影响结果，这其实就是 JMM 的三大特性： **可见性**，**原子性**，**有序性**。

---

###### JMM 的三大特性？

1️⃣** 可见性**

一个线程修改了共享变量的值，其他线程能够立刻感知到值的变化。

**实现方式**：`volatile` 关键字、`synchronized`、`final`。

2️⃣ **原子性**

一个操作要么全部执行不能中断，要么不执行。

**实现方式**：`synchronized`、`Lock`、Atomic 原子类（基于 CAS）。

3️⃣ **有序性**

程序执行顺序可能被优化器重排（指令重排）

**实现方式**：`volatile`（禁止指令重排）、`synchronized`。

---

###### 什么是 Happens\-Before 原则？

Happens\-Before 是 JMM 规定的一套规则，如果操作 A **Happens\-Before** 操作 B，那么 **A 的结果对 B 是可见的**。具体在 JMM 执行中大概有这么几种：

1️⃣ **程序顺序执行**，同一线程，代码按顺序执行。

2️⃣** 获取锁和释放锁**，解锁 happens\-before 加锁。

3️⃣ **volatile 变量规则，**对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

4️⃣ **线程规则**，线程开启先于线程执行，线程结束后主线程才可见。

---

###### java 里面的线程和操作系统的线程一样吗？

Java 线程与操作系统的原生线程是**一一对应**的。

实现原理：当调用 thread\.start（） 时，JVM 会调用操作系统的 API（如 Linux 下的 pthread\_create）来创建一个真正的系统级线程。

**优点是利用多核 CPU 的并行处理能力**，**缺点是创建销毁线程涉及到 OS 线程的用户态与内核态的转换，消耗资源。**

虚拟线程

虚拟线程是 JDK 21 引入的一种轻量级线程模型，它由 JVM 调度，而不是直接映射到操作系统线程。相比传统线程，一个虚拟线程的创建和切换成本非常低，可以支持成千上万甚至百万级的并发。

它的核心是 **M:N 调度模型**，也就是多个虚拟线程复用少量的操作系统线程。当虚拟线程发生阻塞（比如 IO 操作）时，JVM 会自动挂起这个虚拟线程，并释放底层线程去执行其他任务，从而避免线程资源浪费。

在使用上，它和普通线程几乎一致，可以用同步的编码方式写出高并发程序，开发体验比基于回调的异步模型更简单。

不过它主要适用于 **IO 密集型场景**，对于 CPU 密集型任务提升不明显。

---

###### 线程创建的方式有哪些？

1️⃣ **继承 Thread 类**

定义子类 \-\> 重写 **run（）** \-\> 创建对象 \-\> 调用** start（） **开启线程。

2️⃣ **实现 Runable 接口**

第一种继承 Thread 类的方式有个缺点，由于单继承的问题不能再去继承其他类。

```Java
@FunctionalInterface
public interface Runnable {
    
*    *public abstract void run();
}
```

实现接口 \-\> 编写 run（） \-\> 将实例传入 Thread 构造私有对象 \-\> 调用 start（）。

3️⃣ **实现 Callable 接口（结合 FutureTask（））**

前两种方式都有一个共同缺点：run（） 方法没有返回值，且不能抛出检查异常。Callable 解决了这两个问题。

实现 **Callable **\-\> 重写 **call（）** \-\> 用** FutureTask** 包装 \-\> 交给 Thread 执行。

```Java
@FunctionalInterface  // 表示这是一个函数式接口
public interface Callable<V> {
   
     // 允许抛出异常，返回结果
*    *V call() throws Exception;
}
```

**FutureTask\( \) **

FutureTask 实现了** RunnableFuture** 接口，而 RunnableFuture 同时继承了** Runnable** 和 **Future。**

因为实现了 Runnable，所以它可以被 Thread 执行。因为实现了** Future（异步回调）**，所以它可以保存 Callable 运行后的结果，并提供手动获取结果的方法** futureTask\.get（）**。

```Java
public class FutureTask<V> implements RunnableFuture<V> {
    /*
     * Revision notes: This differs from previous versions of this
     * class that relied on AbstractQueuedSynchronizer, mainly to
     * 
```

```Java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    */***
*     * Sets this Future to the result of its computation*
*     * unless it has been cancelled.*
*     */*
*    *void run();
}
```

缺点？

**futureTask\.get（） 是阻塞操作。在生产环境下，直接调用 get（） 可能会导致接口超时。**通常有以下应对方案：

isDone（）：轮询检查任务是否完成（不推荐，浪费 CPU）。

get（long timeout， TimeUnit unit）：带超时的获取，规定时间内没结果就抛出异常。

CompletableFuture（JDK 8\+）：这是更现代的做法，支持回调（Callback），任务完成后自动触发后续逻辑，彻底告别手动阻塞。

4️⃣ 通过线程池创建线程

---

### 线程池

###### 介绍一下 JAVA 中的线程池？

线程池是一种利用池化技术管理线程的资源池。它的核心思想是：**复用**已创建的线程，而不是为每个任务都创建和销毁新线程。通过复用已存在的线程，减少线程创建和销毁造成的消耗（**创建线程需要分配栈内存、与 操作系统线程的状态切换**）。

---

###### 线程池的核心参数？

**corePoolSize：**核心线程数，**长期存活**。

**maximumPoolSize：**最大线程数，包括**核心线程 \+ 非核心线程**。

**keepAliveTime：**非核心线程空闲时的存活时间。

**unit：**keepAliveTime 的时间单位。

**workQueue：**工作队列。当没有空闲的线程执行新任务时，该任务就会被放入工作队列中，等待执行。

**threadFactory：**创建线程池的工厂（可以设置自定义命名线程）。

**handler：**当队列满且达到最大线程数时，新来的任务执行的拒绝策略。

```Java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
```

---

###### 如何使用（创建线程池 ）？

1️⃣ **new 一个 ThreadPoolExecutor **，指定线程池参数（推荐）。

这样做的好处是让开发人员对线程池的参数允许有更加精细的控制，从而避免资源耗尽的风险。

通过** execute（）**或者** submit（）**提交任务：

```Java
// 没有返回值
executor.execute(() -> {
    System.out.println(Thread.currentThread().getName() + " 执行任务");
});

// 有返回值
Future<String> future = executor.submit(() -> "result");
```

2️⃣ 通过 Executors 提供的工具类创建（不推荐）

Executors 是静态工厂类，提供了几种常用的线程池配置方案：

newFixedThreadPool（int n）：固定线程数量。

newCachedThreadPool（）：可缓存线程池，有多少任务开多少线程，空闲时回收。

newSingleThreadExecutor（）：只有一个线程的线程池，保证任务按顺序执行。

newScheduledThreadPool（int n）：支持定时及周期性任务执行。

**Executors 提供的工具类创建线程的弊端：**

1️⃣ 容易导致 OOM （内存溢出）

跟使用的队列有关，可以看作无界队列（Integer 的最大值），可能堆积大量的请求，造成 OOM。

2️⃣ 屏蔽了核心细节，线程数不可控，如 newCachedThreadPool。

---

###### 如何设置核心线程数？

CPU 密集型（如加密、排序、复杂计算）：线程数不宜过多，通常设置 CPU 核数＋1。

IO 密集型（如读写数据库、调用第三方接口、读写文件）：线程会经常阻塞等待，CPU 核数 \* 2。

---

###### 工作队列有哪些拒绝策略？

**ThreadPoolExecutor\.AbortPolicy：**抛出 RejectedExecutionException 来拒绝新任务的处理（默认的拒绝策略）。

**ThreadPoolExecutor\.CallerRunsPolicy：**调用执行者自己的线程运行任务，也就是直接在调用 execute 方法的线程中运行（run）被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。。

**ThreadPoolExecutor\.DiscardPolicy：**不处理新任务，直接丢弃掉。

**ThreadPoolExecutor\.DiscardOldestPolicy：**此策略将丢弃最早的未处理的任务请求。

可以通过实现 **RejectedExecutionHandler **自定义拒绝策略。

---

###### 线程池的工作流程？

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NTc5NmZlMTJmMzAzMTFjMmQyZjVhNzc0YzFlMjM3ZjlfN2I0ODM2NjBkYTAyMmJkZGVmY2NjYmMyMDQ2ZmFhN2JfSUQ6NzYyNTgyMTAyMTM3NTM1MjAwNV8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

**核心线程在执行完任务后并不会销毁**。它们底层其实是在运行一个 **while** 循环不断的轮询接受新的任务。

---

###### 线程池用到了那些设计模式？

> ThreadPoolExecutor
> 
> ├── workQueue → 生产者消费者模式
> 
> ├── ThreadFactory → 工厂模式
> 
> ├── RejectedExecutionHandler → 策略模式
> 
> ├── runWorker → 模板方法模式
> 
> ├── 线程复用 → 享元模式
> 
> 

---

###### 提交给线程池中的任务可以被撤回吗？

可以的，当执行** ****service\.submit（\.\.\.）** 时，线程池内部其实做了一层包装：

包装：传的是 Runnable 还是 Callable，线程池都会把它包装成一个 **FutureTask**。入队时把这个 FutureTask 丢进阻塞队列。返回时立刻把这个 FutureTask 对象作为 Future 接口返回，通过 Future 提供的

**Future\.cancel****（boolean mayInterruptIfRunning）**通过参数可以选择如果任务在执行是否发送一个中断信号，方法撤销提交的任务。

撤回的三种场景：

**任务还在队列中排队**：任务会被直接标记为“取消”，等轮到它时，线程池会直接跳过它，撤回成功。

**任务已经开始运行：**

如果传入 false：任务会继续执行完，但 Future\.get（） 会抛出异常，撤回失败。

如果传入 true：JVM 会尝试给执行该任务的线程发送一个 中断信号 （interrupt）。

**任务已经执行完：**返回 false，撤回失败。

Future 的介绍见 12[JAVA 八股](https://xcnm79jcno1y.feishu.cn/docx/Pk4sdT3ZVof7CrxAxSXc0SB2nYb#share-TaKqduoa2o7IUHxvZVjcHDHpnce)

---

###### 线程池中 shutdown （），shutdownNow（）这两个方法有什么作用？

shutdown（） 是有序关闭，**不再接收新任务，但会执行完已提交任务**；

shutdownNow（） 是“立即关闭”，**尝试中断正在执行的任务，并返回未执行的任务列表。**

---

###### 如何给线程池中的线程命名？

1️⃣ 使用 Guava 提供的创建线程池工厂时携带命名规则，将这个线程池工厂作为参数传到 ThreadPoolExcutors 构造器中。创建线程时就会按照规则命名。

```Java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
                        .setNameFormat(threadNamePrefix + "-%d")
                        .setDaemon(true).build();
ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, 
                                    maximumPoolSize,
                                    keepAliveTime, 
                                    TimeUnit.MINUTES, workQueue, threadFactory);
```

2️⃣ 通过实现** ThreadFactory **自定义线程工厂，增加一个 name 字段。

---

###### execute（） vs submit（）的区别？

1️⃣ execute（） 和 submit（） 都可以向线程池提交任务。

2️⃣ execute（） 只能提交 Runnable 任务，**没有返回值，异常会直接抛出**；

3️⃣ submit（） 可以提交 Runnable 或 Callable 任务，**并返回 Future 对象，可以获取执行结果，同时异常会被封装在 Future 中，****需要通过 get（） 方法获取结果抛出异常。**

4️⃣ submit（） 的**底层实际上是将任务包装成 FutureTask**，**再调用 execute（） 执行**。

submit（runnable），通过 get 能获得返回值吗？

能，只不过是 null。

---

###### Future



---

###### CompletableFuture





---

### 并发和锁

###### 怎么保证多线程并发安全？

保证多线程并发安全的核心是控制多线程对共享资源的访问。

1️⃣ 加显示锁 synchronized 或 Lock 保证线程间的互斥访问。

2️⃣ 使用 CAS，无锁化操作数据（低并发时使用）。

3️⃣ 使用线程安全的集合，ConcurrentHashMap，CopyOnWriteArrayList。

4️⃣ 通过 ThreadLocal 避免线程间共享数据。

5️⃣ 对于一些要进行原子操作的变量采用原子类型，对变量进行原子操作（基于 CAS）。

---

###### JUC 中的并发工具类？

1️⃣ **ConutDownLatch **通过 wait（）使当前线程等待并且维护一个计数器（设一个初始值），在其他线程中调用 countDown 会使计数器减一，当计数器减为 0 时，会唤醒阻塞的线程。可以达到让一个线程等待其他线程执行完再执行的效果。

2️⃣** CyclicBarrier **用于让一组线程相互等待，直到所有线程都到达某个屏障点后，才一起继续执行，并且它可以重复使用（cyclic）。

3️⃣ **Semaphore** 可以**规定一个数据被并发访问的线程数量**，线程通过 acquire（） 获取许可，获取不到就阻塞；通过 release（） 归还许可，从而唤醒等待线程。

4️⃣ 线程安全的集合 ConcurrentHashMap，CopyOnWriteArrayList。

5️⃣ Future 和 Callable。

Callable 和 Runnable 的区别。

Future 的解释

---

###### volatile

通过 volatile 修饰变量，两个作用：**一是保存数据修改时的线程间可见**，**二是防止指令重排**。

1️⃣ 保持线程间可见性

**当变量被 volatile 修饰后，会在一个线程修改后强制刷回主内存，并且对该变量的读操作会从主内存中获取。**

2️⃣ 防止指令重排

什么是指令重排

指令重排是编译器或 CPU 在不改变单线程执行结果的前提下，对代码执行顺序进行优化调整的过程。

禁止指令重排的原理

---

###### CAS

CAS（Compare And Swap）是一种基于 CPU 原子指令实现的无锁并发机制，是实现乐观锁的基础。它在更新变量时，会先比较当前值是否等于预期值 A，如果相等则更新为新值 B，否则更新失败，可以选择重试。

实现乐观锁的方式

1️⃣** 使用 CAS**，java 中提供了一系列的支持原子操作的变量类型可以实现乐观锁的操作，如 AtomicInteger、AtomicLong。

2️⃣ **版本号法，**在修改数据时增加一个版本号的字段。每次修改时判断版本号是否和更新前获取的版本号一致，不一致就不修改。并且每次修改后递增版本号。

CAS 缺点

1️⃣** ABA 问题**

**ABA 问题是说，使用 CAS 时对数据预期值等于 A 时才修改，但是肯能这个数据先变为 B 又修改为 A，修改了两次，这是再修改不能保证原子性。**

**AtomicStampedReference **通过在 CAS 操作中引入版本号（stamp），在比较时同时校验值和版本号，从而解决 ABA 问题。每次更新时版本号都会递增，即使值恢复为原值，版本号也会发生变化，从而避免误判。

2️⃣ 如果长时间不成功，CAS 会一直自旋浪费 CPU。

3️⃣ 只能保证单个变量的原子性，不能保证多个变量的原子性。

---

###### Synchronized

Synchronized 是 JAVA 提供的内置锁，Synchronized 的本质是对象锁，通过 Synchronized（Obj）可以修饰方法或者代码块。

**修饰实列方法时，锁的是 this 对象；修饰类方法（静态方法）时锁的是类对象。**

原理

synchronized（obj） 是基于对象的 Monitor（监视器锁）实现的。当线程进入同步代码块时，会尝试获取该对象的 Monitor，如果获取成功，则进入临界区，并将锁的持有计数加 1；如果获取失败，则线程会进入阻塞状态，加入等待队列。当线程退出同步代码块时，会释放锁（计数减 1），当计数为 0 时真正释放锁，并唤醒等待队列中的线程重新竞争锁。

**Synchronized 的锁升级过程**

jdk 对此进行了优化，synchronized 的锁会根据竞争情况，重量级从低成本到高成本逐步升级，以提高性能。

1️⃣ **无锁（初始状态）**，对象刚创建，没有线程获取锁监视器。

2️⃣ **偏向锁**，当有一个线程进入时，在对象头（Mark Word）记录这个线程 ID，下次访问时会直接获取锁对象连 CAS 都不用。

3️⃣ **轻量级锁**，当第二个线程来的时候偏向锁会失效，此时是有线程竞争的，每个线程会使用 CAS 去检查对象头中的线程 ID 获取锁监视器。

4️⃣ **重量级锁**，当线程竞争激烈，会导致大量线程的 CAS 一直失败（自旋 10 次失败），一直自旋操作时，锁会升级会重量级锁。使用操作系统互斥锁（Mutex），其他线程会阻塞减少 CPU 的消耗。

---

###### ReentrantLock

ReentrantLock 是 Lock 包下的锁，基于 AQS 实现。

AQS 实现 ReentrantLock（公平锁）

1️⃣ 继承 AQS 这个抽象类

2️⃣ 重写 tryAcquire 和 tryRelease 方法

**tryAcquire:** 尝试获取锁，如果是当前线程持有就将 state 值＋1，返回获取成功（保证可重入的特性）。如果队列中有线程节点，就不能获取锁（公平性，保证每次都从队列队头获取线程）。

**tryRelease: **释放锁时，将 state 减 1，如果减为 0 说明没有线程持有锁，成功释放。

AQS 会自动处理线程的排队、阻塞和唤醒。

3️⃣AQS 会帮我们做到，线程入队 自旋判断 park 阻塞 unpark 唤醒 FIFO 调度。

```Java
import java.util.concurrent.locks.AbstractQueuedSynchronizer;

public class FairReentrantLock {

    private static class Sync extends AbstractQueuedSynchronizer {

        // 判断锁是否被当前线程持有
        protected boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        // 尝试获取锁
        protected boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 公平性检查：检查队列中是否有前驱节点, 如果有, 则当前线程不能获取锁
                if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (current == getExclusiveOwnerThread()) {
                // 可重入逻辑：如果是当前线程持有锁, 则增加持有次数
                int nextc = c + acquires;
                if (nextc < 0) {
                    throw new Error("Maximum lock count exceeded");
                }
                setState(nextc);
                return true;
            }
            return false;
        }

        // 尝试释放锁
        protected boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread()!= getExclusiveOwnerThread()) {
                throw new IllegalMonitorStateException();
            }
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        // 提供一个条件变量, 用于实现更复杂的同步需求, 这里只是简单实现
        ConditionObject newCondition() {
            return new ConditionObject();
        }
    }

    private final Sync sync = new Sync();

    // 加锁方法
    public void lock() {
        sync.acquire(1);
    }

    // 解锁方法
    public void unlock() {
        sync.release(1);
    }

    // 判断当前线程是否持有锁
    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    // 提供一个条件变量, 用于实现更复杂的同步需求, 这里只是简单实现
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

---

###### synchronized 和 reentrantlock 区别？

1️⃣ **实现方式**，synchronized 是基于 JVM 的对象的锁监视器实现，reentrantlock 是基于 AQS 实现。

2️⃣ **公平性**，synchronized 是不公平的，reentranlock 可以设置公平性。

3️⃣ **获取锁解锁**，synchronized 通过直接修饰方法或者代码块，线程进入获取锁，结束释放锁。reentranlock 需要手动的定义锁，释放锁。

4️⃣ **是否能响应中断（interrupt）**，synchronized 不能响应中断（JVM 层面的 Monitor 锁没有中断感知）中断会进入 blocked 状态，reentranlock 可以感知中断抛出异常。

**在简单场景下推荐使用 synchronized，而在需要更高灵活性的场景下使用 ReentrantLock。在简单场景下推荐使用 synchronized，而在需要更高灵活性的场景下使用 ReentrantLock。**

---

###### AQS

AQS（Abstract Queued Synchronizer）是一个用来构建锁和同步器的抽象类，核心是通过 CAS \+ 状态变量 \+ 等待队列去提供一个构建同步工具的框架。

AQS 我的理解就是一个队列排队管理器，有 4 个重要的组成：

1️⃣ 使用一个** volatile 修饰的 state** 表示同步状态；

2️⃣ **通过 CAS 操作 state**，实现无锁方式尝试获取资源；

3️⃣ **维护一个 CLH 变体的双向队列**，将获取资源失败的线程封装为节点加入队列排队；

4️⃣ 当线程获取失败后，会先进入队列并自旋判断前驱节点状态，必要时通过 LockSupport 的 park 挂起线程，在资源释放时通过 unpark 唤醒后继线程继续竞争。

---

###### 公平锁和非公平锁

在 AQS 实现的锁中，公平锁是指线程在获取锁时，会先检查同步队列中是否存在前驱节点，如果有则必须排队，按照 FIFO 顺序获取锁；非公平锁则在尝试获取锁时可以直接通过 CAS 抢占锁，即使队列中已有等待线程，也允许插队。

---

###### 介绍一下乐观锁和悲观锁？

悲观锁假设并发冲突一定会发生，因此在操作前加锁，保证线程安全，但会导致阻塞；乐观锁假设冲突较少，不加锁，在更新时通过 CAS 或版本号机制进行校验，如果失败则重试。悲观锁适用于高冲突场景，乐观锁适用于低冲突、高并发场景。

---

###### ThreadLocal

ThreadLocal 的主要作用是提供了一种**线程本地的变量机制**，允许线程去创建线程本地变量，让变量在**不同线程之间相互隔离**。

ThreaLocal 实现线程隔离

每个线程都有自己的 ThreadLocalMap，这个 map 的 Key 是 ThreadLocal 实例，value 可以存放要进行线程中保存到值。当调用 set 方法时，会去自己的线程中的 ThreadLocalMap 中调用 set 方法，保存到当前线程，同样的调用 get 方法时也是从自己的线程 ThreadLocalMap 中获取数据。不同的线程即使使用同一个 ThreadLocal 对象不同的线程拿到的值也不同。

内存泄漏

在 ThreadLocal 中，key 是弱引用，当发生 GC 时，如果 ThreadLocal 没有被外部强引用指向，就会被回收，此时 ThreadLocalMap 中对应的 key 会变成 null。但 value 仍然是强引用，会被 ThreadLocalMap 持有，从而形成无效的 entry。如果不进行清理，这些 entry 可能长期存在，造成内存泄漏。因此需要在使用完 ThreadLocal 后手动调用 remove（） 方法进行清理。

为什 key 要设计成弱引用，value 要设计成强引用？

ThreadLocalMap 中 key 使用弱引用，是为了让 ThreadLocal 对象在不再使用时能够被 GC 回收；value 使用强引用，是为了保证业务数据在使用期间不会被意外回收，从而保证程序的正确性。

跨线程访问 ThreadLocal 

如果需要跨线程传递数据，可以通过手动传递、**InheritableThreadLocal **或 **TransmittableThreadLocal** 实现。

**InheritableThreadLocal 只在创建线程时生效，因此在线程池中会失效**，因为线程池中的线程是复用的，不会重复创建。

一般使用** TransmittableThreadLocal**，它会在子线程执行前，捕获主线程的 ThreadLocal 然后执行完再恢复。

---

###### 死锁

多个线程互相持有对方需要的资源，导致都不能获取到资源而不能释放锁，导致都无法继续执行。

死锁是指多个线程在执行过程中因争夺资源而造成的一种互相等待的现象。死锁的产生需要满足四个条件：**互斥、请求**与**保持**、**不可剥夺**和**循环等待**。解决死锁的方法包括统一加锁顺序、一次性申请资源、使用超时机制以及死锁检测等

---

## JVM



## Spring

### Spring

###### IOC

**IOC （Inversion of Control） 控制反转**，对象的创建权由程序代码转移到 Spring 容器。由容器来负责对象的生命周期和依赖关系，开发时只需通过声明依赖，而不需要自己去 new 对象，降低了模块之间的耦合。

---

###### IOC 的底层实现原理？

Spring IOC 主要依赖 Java 的**反射机制 \+ 工厂模式 \+ 缓存（Map 结构）**来实现 Bean 的创建和管理。

**1️⃣ 存储 BeanDefinition**

IOC 容器并不是直接存储实例化的对象，而是先使用 ConcurrentHashMap 存储 BeanDefinition（类的元数据），key 是 beanName（默认是类名首字母小写），value 是对应的 BeanDefinition。

BeanDefinition 中包含： 

类的**全限定名**、**作用域**（singleton / prototype）、**是否懒加载**、**依赖关系**、**初始化方法**等。

**2️⃣ 注册 Bean**

容器启动时，会通过 @ComponentScan、@Bean 或 XML 配置进行扫描，将解析后的 BeanDefinition 注册到容器中，本质就是 put 到 beanDefinitionMap 中。

**3️⃣ 创建 Bean**

在需要创建 Bean 时（如 singleton 在启动阶段），通过反射调用构造方法实例化对象。

**4️⃣ 依赖注入（DI）**

Spring 会在实例化之后，通过反射完成依赖注入，比如 @Autowired 会根据类型或名称从容器中找到对应 Bean 并注入，底层是通过字段赋值、setter 方法或构造器完成。

**5️⃣ Bean 缓存（单例池）**

对于 singleton Bean，会创建完成后放入单例池（singletonObjects，本质是一个 Map），后续再获取时直接从缓存中拿，避免重复创建。

**6️⃣ 生命周期扩展**

Spring 通过 BeanPostProcessor 在 Bean 初始化前后进行扩展，例如 AOP 就是在这里生成代理对象并替换原始对象。

---

###### Spring IOC 什么时候创建 Bean（实例化） ？

这个取决于容器和 Bean 的作用域（**@Scope**），ApplicationContext 是 Spring 和 SpringMVC 默认使用的容器类型。

1️⃣ 单例的 Bean**@Scope（"singleton"）**， 会在容器启动时就创建好所有的单例和非懒加载的 Bean。

2️⃣ 多例的 Bean**@Scope（"prototype"）**，会在每当程序调用一次 getBean（），容器都会重新执行一遍完整的生命周期逻辑，创建一个全新的实例。

3️⃣ 懒加载（bean 上有**@Lazy 注解**），容器启动时**不创建**，只有当程序第一次通过 **context\.getBean（）** 获取该 Bean，或者该 Bean 被注入到其他正在初始化的 Bean 中时，才会触发实例化。

---

###### Spring IOC 中存放的是什么？

Spring 容器中主要存储两类内容：一类是 BeanDefinition，用来描述 Bean 的定义信息，比如类名、作用域和依赖关系；另一类是 Bean 实例对象，通常存放在单例池中。底层本质上是通过多个 Map 结构来管理这些信息，比如 singletonObjects 用于存放单例 Bean。

---

###### 依赖注入的三种方式？

1️⃣** 构造器注入**

构造器注入是 Spring 官方推荐的注入方式，**可以利用 final 修饰保证这个对象在整个生命周期内不可变**，使用构造器注入强制在创造对象时就要完成依赖注入。

**2️⃣ Setter（）方法注入**

Spring 的三级缓存可以解决 Setter 方法注入单例 Bean 的循环依赖问题问题。

3️⃣** 字段注入**

代码简洁但隐藏依赖关系，不推荐生产代码。

@Autowired 和 @Resource 的区别

**@AutoWired** 优先按照类型注入，支持 @Qualifier，再按照名称（Spring 的注解）。

**@Resource** 优先按照名称注入、再按照类型（JDK 的注解）。

---

###### AOP

AOP 通过动态代理技术将一些通用逻辑（比如**事务**，**日志记录**，**权限校验**等）从业务代码中分离出来，实现横向抽取。

AOP 实现一个业务抽取

1️⃣ 写一个切面类（**@Aspect **注解标注）。

2️⃣ 在**@Pointcut（）**里面写切入点表达式指定要增强的方法，方法是切入点的单位。

3️⃣ 写通知，在类中编写方法做增强功能，通知的类型有：

@Before  方法执行前

@After   方法执行后

@Around  环绕通知，需要手动调用连接点的注解标注的原始业务方法** joinPoint\.proceed（）；**

@AfterReturning 正常返回

@AfterThrowing 异常

---

###### SpringAOP 的底层实现？

Spring Aop 的实现依赖于动态代理技术，在运行时生成目标类的代理对象，在不修改原始代码的基础上对方法进行增强。

Spring AOP 支持两种方式的动态代理：

1️⃣ 如果目标类有实现接口时，会通过反射包下的 Proxy 和 InvocationHandle 接口实现。

2️⃣ 当目标类没有实现接口时，会通过 CGLIB 生成一个被代理类的子类作为代理。

---

###### Spring 怎么解决循环依赖？

Spring 通过三级缓存将未经依赖注入的 bean 暴露出来，从而打破循环依赖。（通过 setter 或者字段注入的单例 bean）

一级缓存：存放的是已经实例化，属性填充完毕的，初始化方法执行完毕的最终形态的 Bean。**getBean（）**就是从一级缓存中获取的。

二级缓存：存放的是提前暴露出来的，实例化但是还没有属性填充的 bean。

三级缓存：存放的是 Bean 的**工厂对象**，用来生成半成品或者代理对象的 Bean。

**A\-\>B\-\>A 的 Bean 的创建流程：**

1️⃣ 去实例化 A 并将 A 的 Bean 工厂对象放到第三级缓存。

2️⃣ 属性注入时发现 A 要依赖 B，依次从一二三级缓存中获取失败。去实例化 B，流程和 A 一样。

4️⃣ 在对 B 进行依赖注入时发现要依赖 A，这时可以从第三级缓存中获取 A 的工厂对象，创建半成品 A 进行依赖注入。将半成品的 A 存放到二级缓存的同时删除第三级缓存中的 A。B 实例化初始化后将 B 放到一级缓存，同时删除 B 在二级、三级缓存中的记录。

4️⃣ 回到 A 的依赖注入，这时一级缓存中已经有 B 的 Bean，A 可以完成依赖注入初始化后，放到一级缓存，并删除二级缓存中的 A。

为什么要使用第三级缓存

第三级缓存主要是解决动态代理或者 AOP 的问题。第三级缓存中的 ObjectFactory 可以根据情况生成原始对象或者代理对象，如果只用二级缓存依赖注入时注入的是原始对象，但是后续**初始化**后如果要代理又会重新生成代理对象。这样一个 Bean 就会有两个实例。

为什么不能解决通过构造器注入的循环依赖

**因为构造器注入要求在实例化前就要获取到类的所有依赖信息，仍然会导致死锁。**

---

###### Spring 框架用到了那些设计模式？

1️⃣ **工厂设计模式，**BeanFactory 和 ApplicationContext。

2️⃣ **单例设计模式，**Spring 中的 Bean 默认都是 **Singleton** 作用域。Spring 底层维护了一个单例注册表（缓存），确保对于同一个 Bean 定义，容器中只存在一个共享的实例。

3️⃣ **模板方法设计模式，**各种** xxxTemplate**，JdbcTemplate、RestTemplate、RedisTemplate。

4️⃣ **代理设计模式，**Spring AOP。

5️⃣ **适配器设计模式，**SpringMVC 中的 HandlerAdapter。

6️⃣ **观察者设计模式，**事件 （ApplicationEvent）发布者 （ApplicationEventPublisher）监听者 （ApplicationListener）。

---

###### Spring 事务的失效场景？

Spring 事务是基于 AOP 和代理对象实现的，失效的原因就是没有走代理。

1️⃣ **自调用，**在同一个类中，被另一个方法调用这个事务方法。相当于是 this 调用，不会走代理。

2️⃣ **非 public 方法的事务**，默认情况下 SpringAOP 只会拦截 public 修饰的方法。

3️⃣ **在 try\-catch 中没有抛出异常，**Spring 事务默认只有抛出异常才回滚。

4️⃣ **非 Spring 管理的对象，**会绕过 IOC 容器没有代理对象。

5️⃣ 在**多线程**中，事务是绑定在当前线程的，开辟一个新线程，新线程的部分拿不到原来事务的上下文。因为 Spring 事务是通过 **ThreadLocal **绑定在当前线程上的。

6️⃣ **事务传播属性设置不当。**

事务的传播行为

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NWMwNjk0NGQ4ZDMwYjY0NWNjMWFiODI5YjVkM2ExNWFfOTU4MjY1NGIwZDZhOGY2YzcwMDYwYjFmYTMxMTg3NDlfSUQ6NzYzMDY1OTAzMTI2NzI2NTQ4OV8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

---

###### Bean 的生命周期

1️⃣ **实例化**

反射调用构造方法或者工厂方法创建 bean（此时对象内部的的字段还是 null）。

2️⃣ **依赖注入**

通过反射给对象的字段赋值，（Spring 的三级缓存在这里生效）。

3️⃣** 触发 Aware 回调**

如果实现了 Aware 接口，会触发 Aware 回调，让 Bean 感知容器上下文信息。

4️⃣** ****BeanPostProcessor 前置方法**

允许修改 Bean，准备 AOP / 注解解析。

5️⃣** 初始化**

三种初始化方法顺序：@PostConstruct → afterPropertiesSet → init\-method（初始化资源 建立连接 启动组件）

6️⃣ **BeanPostProcessor 后置方法（****重要）**

Spring 会在这个阶段生成** AOP 代理对象**，和**触发类上的注解**。

7️⃣ **使用**

Bean 放入单例池，供外部通过 getBean（）可以获取。

8️⃣ **销毁**

@PreDestroy → DisposableBean → destroy\-method，关闭连接清理缓存。

---

###### Bean 都是单例的吗？

**Spring 中的 Bean 默认都是单例的**。可以通过**@Scope **注解指定作用范围。

@Scope 注解

---

###### SpringBean 生命周期中的扩展点

**BeanPostProcessor **作用在初始化前后；
**@PostConstruct** 和 **init\-method** 作用在初始化阶段，其中 @PostConstruct 先执行，init\-method 后执行；
**@PreDestroy **和 **destroy\-method **作用在销毁阶段，其中 @PreDestroy 先执行，destroy\-method 后执行。

只有单例的 bean 才会执行销毁流程，多例的 Bean 的销毁不会归 spring 管理。

---

### SpringMVC

###### 介绍一下 SpringMVC 分层

**MVC 是一种软件设计模式（Model\-View\-Controller）**，将开发中业务逻辑，数据管理和页面展示分离解耦，从而提高代码的扩展性和可维护性。M 是模型负责业务逻辑处理，V 是视图负责页面展示，C 是控制器负责接收请求，调用相关业务逻辑并响应请求。

###### Spring MVC 处理一个请求的流程？

SpringMVC 处理一个请求的大致过程：

Spring MVC 的核心处理流程是围绕 DispatcherServlet 展开的。请求首先进入**前端控制器 DispatcherServlet**，然后通过 **HandlerMapping **找到对应的 Controller 方法，再通过** HandlerAdapter 进行方法调用**。Controller 执行业务逻辑后返回 M**odelAndView 或 JSON，**如果是页面请求则通过** ViewResolver 解析视图**并渲染页面，最终**返回响应**结果给客户端。**JSON 等数据响应，则通过****消息转换器****直接写回响应体**

---

###### 拦截器和过滤器介绍一下？

拦截器是 SpringMVC 提供的，过滤器是 Servlet 包下的。

1️⃣ 拦截器（Intercepor）可以作用在 DispatcherServlet 之后和 Controller 请求前后，只拦截 MVC 的请求拦截器更适合**做业务相关的处理，如登录校验和权限控制。**

2️⃣ 过滤器过滤器是基于 Servlet 规范实现的，作用于 DispatcherServlet 之前，可以拦截所有请求和静态资源。过滤器适合做通用处理，如编码、跨域等，但是拿不到业务层相关的信息。

定义一个拦截器的步骤

1️⃣ 实现 **HandlerInterceptor **接口，重写里面的三个拦截方法

```Java
public class MyInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // Controller 执行前
        return true; // true放行，false拦截
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        // Controller 执行后，视图渲染前
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // 请求完成后（常用于资源清理）
    }
}
```

2️⃣ 将写好的拦截器注册到 MVC 中，在 WebMvcConfigurer 的实现类中的 addInterceptors 方法中注册

```Java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/**")       // 拦截路径
                .excludePathPatterns("/login"); // 放行路径
    }
}
```

---

### SpringBoot

###### SpringBoot 相比 Spring 的优势？

1️⃣ SpringBoot 提供了**自动化配置**，

2️⃣ SpringBoot 提供了**快速的项目启动器**，

3️⃣ SpringBoot 默认集成了**多种内嵌服务器**，

---

###### 怎么理解 SpringBoot 中的约定大于配置？

1️⃣ Spring Boot 的约定大于配置指的是框架通过**提供大量默认配置**和**自动装配机制**，使开发者只需要关注核心业务配置，而不需要进行繁琐的显式配置。

2️⃣ 它主要通过自动配置、默认配置以及条件装配实现，例如通过 @EnableAutoConfiguration 加载配置类，并结合 @ConditionalOnClass 和 @ConditionalOnMissingBean 等条件注解，根据环境自动创建 Bean。

3️⃣ 当默认配置不满足需求时，开发者可以通过**自定义配置**或**覆盖 Bean** 来进行扩展。

---

###### SpringBoot 自动装配的原理？

通过**@EnableAutoConfiguration 注解**和**条件化配置**，在项目中引入相关依赖后，SpringBoot 就能将这些相关的依赖注入到程序上下文中的功能。

原理

1️⃣ 在 SpringBoot 启动类中，@SpringBootApplication 注解内部的**@EnableAutoConfiguration **是自动装配的核心注解。

2️⃣ @EnableAutoConfiguration 通过**@Import **注解导入** AutoConfigurationImportSelector **这个核心类。在启动时负责加载自动配置类。

3️⃣ 从所有 jar 包的 META\-INF/spring/org\.springframework\.boot\.autoconfigure\.AutoConfiguration\.imports 文件中读取候选配置类，然后通过 @Conditional 注解进行过滤，最终将符合条件的配置类导入到 Spring 容器中，从而完成自动配置。

---

###### 怎么自定义一个 Start 启动器？

这里是比较规范的做法，编写组件实现**起步依赖**和**自动装配，**每个 starter 本质上就是一个 Maven 模块。

1️⃣ **创建自动装配模块**

在这个模块中定义需要的类，编写自动配置类（@Configura 可以标也可以不用加）注册 Bean，并通过 @Conditional 注解控制生效条件。在资源目录文件下以。imports 结尾的文件中添加这个配置类的全类名，

2️⃣ **创建 starter 模块**

再创建一个 starter 模块引入该 autoconfigure 依赖，供外部使用。用户只需引入 starter 依赖即可自动获得相关功能，无需额外配置。

为什么还要加一层 Starter 包装，直接使用自动装配模块不行吗？

是可以的，可以不写 starter 模块，直接引入 autoconfigure 模块也能完成自动配置。只是一个 starter 可能会引入多个自动装配模块，用来统一的引入依赖和版本管理。

---

###### Springboot 怎么做到导入就可以直接使用的？

主要得益于**起步依赖**、**自动配置**和**条件注解**。

1️⃣ 首先通过 Starter 实现 Maven 依赖聚合和传递，自动引入相关 jar 包；

2️⃣ 其次通过** @EnableAutoConfiguration** 启动自动配置机制，利用 **AutoConfigurationImportSelector** 从 classpath 下的 **AutoConfiguration\.imports **文件**加载所有自动配置类**；

3️⃣ 最后结合 @Conditional 条件注解进行筛选，按需创建 Bean 并注入 Spring 容器，从而实现开箱即用。

---

## MySQL

### 数据库基础

###### SQL 和 NOSQL 的区别？

SQL 数据库强调结构化、强一致性和事务**（ACID）**，适合复杂查询，数据隔离，安全存储；**NoSQL 强调高性能、可扩展性和灵活结构（BASE）**，适合海量数据和高并发场景。

SQL，如 MySQL 和 Oracle（A 原子性 、C 一致性、I 隔离性、持久性）。

NoSQL，如 Redis 和 MongoDB（BA 基本可以、S 软状态、E 最终一致性）。

---

###### 数据库三大范式？

1️⃣ 每一列（属性）都是**不可再分的最小单元**。

2️⃣ 在第一范式的基础上，非主键必须**完全依赖**于主键，不能只依赖一部分（针对联合主键）。

3️⃣ 在第二范式的基础上，非主键必须直接依赖于主键，不能**存在依赖传递**。

---

###### MySQL 中如何避免重复插入数据？

1️⃣ 给唯一的字段加唯一 unique 约束， 确保这个值在该列中唯一。

2️⃣ 在执行插入语句中** on duplicate key update**， 指定当插入数据时如果遇到主键或唯一键冲突，就会转为执行更新操作，常用于实现幂等写入或**存在则更新，不存在则插入**的业务场景。

```SQL
--加锁（把插入 + 更新合并成一个原子操作）
INSERT INTO users (email, name)
 VALUES ('example@example.com', 'John Doe') 
 ON DUPLICATE KEY UPDATE name = VALUES(name);
```

意思是如果数据库中有重复的（‘example@example\.comm‘， 'John Doe'） 发生冲突了，不会执行插入操作，会执行修改，set name = "John Doe"。

3️⃣ 使用 INSERT IGNORE 在插入数据时当有重复数据会忽略这条插入语句。

---

###### char 和 varchar 的区别？

char 是固定长度字符串类型，定义时需要指定固定的长度，存储时会在末尾补空格。

varchar 是可变长度字符串类型，定义时需要指定最大长度**，存储时会根据实际的长度占用存储空间。**

---

###### int（1）和 int（10）在 mysql 中有什么不同？

INT（1） 和 INT（10） 的区别主要在于** 显示宽度**，而不是存储范围或数据类型本身的大小。

---

###### 怎么存储 ip 地址？

1️⃣ 以**字符串类型**存储原始的点分十进制。这样做**优点**是直接查询，插入比较直观，不需要做额外的转换。**缺点**是不利于进行范围查询并且字符串占用空间大。

2️⃣ 整数类型存储，将 ip 地址转为 32 位无符号数字。优点是占用内存小只需要 int 类型，便于范围查询，缺点是还需要额外的转换操作，不够直观。

---

###### 讲一下数据库约束？

1️⃣ 主键约束：PRIMARY KEY 标识表中的主键。

2️⃣ 唯一约束：UNIQUE 约束字段唯一。

3️⃣ 非空约束：NOT NULL 该字段不能为空

4️⃣ 默认约束：DEFAULT 插入没有赋值时，给一个默认值。

5️⃣ 外键约束：FOREIGN KEY （user\_id） REFERENCES users（id） 用来建立表之间的关系

6️⃣ 检查约束：CHECK （age \>= 0） 做默认的检查（MySQL8\.0\+）。

---

###### in 和 exists 关键字的区别？

首先， 在 **MySQL** 中，IN 是**值匹配**，EXISTS 是**是否存在**；IN 适合小结果集，EXISTS 适合大结果集或关联查询。

in

```Java
SELECT * FROM user
WHERE id IN (1, 2, 3);
// 或者在子查询中
SELECT * FROM user
WHERE id IN (SELECT user_id FROM orders);
```

in 会把子查询结果全部查出来，然后做匹配。

exists

只要子查询**有一条满足条件**就返回 true（立刻停止）。

---

###### 介绍一下 mysql 中的一下基本函数？

1️⃣ 字符串类型，CONCAT（拼接）、SUBSTRING（截取）

2️⃣ 数值类型，如 ROUND、ABS

3️⃣ 时间日期，NOW、DATE\_ADD

4️⃣ 聚合函数，COUNT、SUM、AVG

---

###### 一条 SQL 查询语句的执行顺序？

**确定要重新的表**

1️⃣** FROM：**确定数据来源（表）

2️⃣ **JOIN / ON：**进行多表连接

**过滤**

3️⃣** WHERE：**对原始数据进行行过滤

4️⃣** GROUP BY：**对数据进行分组

5️⃣** HAVING：**对分组后的结果过滤

**查询**

**6️⃣ SELECT：**选择需要返回的列

**对查询结果进行处理**

7️⃣ **DISTINCT：**对结果去重

8️⃣ **ORDER BY：**排序

9️⃣ **LIMIT：**限制返回行数

注： WHERE 和 HAVING 的区别

where 在 group by 分组前执行，having 在分组后执行；where 不能使用聚合函数，having 可以使用聚合函数（分组之后）。

---

### 存储引擎

###### 介绍一下 MySQL 的存储引擎？

1️⃣ **InnoDB，**是 MySQL 默认的存储引擎，具有 ACID 事务支持，行级锁，外键等，适合高并发对数据一致性要求较高的场景。

2️⃣ MyISAM，**MyISAM 不支持事务和行锁**，但读性能高，适合读多写少场景。

3️⃣ Memory，将数据存储到内存中，适合高性能的读操作，但是在服务器重启或崩溃时数据丢失。

---

###### 为什么 InnoDB 是 MySQL 的默认存储引擎？

1️⃣ 支持事务，具有 ACID 特性，保证数据的一致性和安全性。

2️⃣ 并发性高，InnoDB 使用颗粒度更小的行级锁，具有更高的并发性能。

3️⃣ 能够崩溃恢复，通过 redolog 日志实现断电数据恢复，保证持久性。

---

###### innodb 与 MyISAM 的区别

InnoDB 和 MyISAM 是 MySQL 的两种存储引擎。InnoDB 支持**事务**、**行级锁**和**外键**，并且具备崩溃恢复能力，适合**高并发和数据一致性要求高的场景**；MyISAM 不支持事务和行锁，采用**表锁机制**，**查询性能较高但并发写能力较差**。目前大多数业务系统都使用 InnoDB。

在索引实现上，**虽然 MyISAM 引擎和 InnoDB 引擎都是使用 B\+Tree 作为索引结构**，但是两者的实现方式不太一样。InnoDB 引擎中，其数据文件本身就是索引文件。相比 **MyISAM，****索引文件和数据文件是分离的**，其表数据文件本身就是按 B\+Tree 组织的一个索引结构，树的叶节点 data 域保存了完整的数据记录。

---

### 索引

###### 介绍一下 MySQL 的索引？

在 MySQL 中，索引是一种用于提高查询效率的数据结构，**本质上是通过特定的数据结构对数据进行有序存储**（通常基于 B\+ 树），从而避免全表扫描。

优点：提高查询速度，加快排序和分组，通过创建唯一索引确保数据唯一性。

缺点：首先存储索引本身要消耗一定的**空间**其次在执行增删改操作时可能要**动态的维护**索引的结构。

---

###### 索引的分类？

1️⃣ 按数据结构分类

B\+树索引、Hash 索引、Full\-text 索引（全文索引）

2️⃣ 实际存储分类

**聚簇索引，**主键索引的 B\+树的叶子节点存放的是实际数据。在创建表时 InnoDB 会默认使用主键作为聚簇索引创建表的物理存储，如果没有主键，会选择第一个不包含 null 值的唯一索引作为聚簇索引，以上都没有，InnoDB 会生成一个隐式自增 id 作为索引键。

**二级索引，**二级索引的 B\+树的叶子节点存放的是主键值。

回表查询

当查询时，使用到二级索引，但是查询的结果不在二级索引中。需要根据查到的主键值再次进行检索主键索引，这个过程就是回表查询。

覆盖索引

反之，当查询的结果可以直接在二级索引中查到，不用再回到主键索引中回表查询，这个过程就是覆盖索引。

3️⃣ 按字段类型分类

主键索引，唯一索引，前缀索引

4️⃣ 按字段个数分类

单列索引和联合索引

最左匹配原则

在使用联合索引时，建立索引树的时候会按照从左到右的的方式进行建立。举个例子，就是说如果我给字段（a，b，c）建立联合索引后，会优先按照 a 的顺序排序，当最左字段值相同时再从左到右按照联合索引中下一个字段的值排序。这样做，在联合索引中，最左列是全局有序的，而后续列是在前缀列相同的情况下局部有序的。

因此查询条件必须满足最左前缀原则，索引才会生效。

---

###### MySQL 中的索引是怎么实现的？

1️⃣ InnoDB 存储引擎使用 B\+ 树作为索引结构，B\+ 树的每个节点对应一个磁盘页（Page），默认大小为 16KB，用于高效地进行磁盘 IO 和范围查询。

2️⃣ B\+树存储时，非叶子节点只存放索引，叶子节点存放数据，并且每个节点的数据会按照索引顺序存储（非聚簇索引叶子节点存放的是主键值）。

3️⃣ 而且每层的父节点的索引都会出现在子节点中，这样所有的叶子节点会包含所有的索引信息。

4️⃣ 为了方便**倒序遍历、范围查询**和**排序**，B\+树叶子节点之间形成**双向链表**的结构。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTAzZWNjY2ZhYzE3NDM3Y2FmODJiNDUzNzU4NTQzZGRfZDk1NzkwYjVkYTBiMTUzYzFhMmNiN2RmZGYyMGFlZjRfSUQ6NzYyNjI5NTEzNzA3Mjc5NDgyNF8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

---

###### B\+树的特性？

1️⃣ 叶子节点存放数据，非叶子节点只存放索引。

2️⃣ 非叶子节点间以双向链表连接。

3️⃣ 非叶子节点都在同一层，确保了所有数据项的检索都具有相同的 I/O 延迟，提高了搜索效率。

---

###### B\+树和 B 树的区别？

1️⃣ 在 B\+树中，数据都存储在叶子节点上，而非叶子节点只存储索引信息；而 B 树的非叶子节点既存储索引信息也存储部分数据。 

2️⃣ B\+树的叶子节点使用链表相连，便于范围查询和顺序访问；B 树的叶子节点没有链表连接。

3️⃣ B\+树的查找性能更稳定，每次查找都需要查找到叶子节点；而 B 树的查找可能会在非叶子节点找到数据，性能相对不稳定。

---

###### MySqL 为什么要使用 B\+树？

1️⃣ B\+树的非叶子节点是只存放索引，叶子节点才存放数据。这样做就使得每个非叶子节点能够存放更多的索引值，相同的数据量对比 B 树和二叉树会高度显得更加小，平均访问数据时的磁盘 I/O 次数会更少。

2️⃣ 另外，B\+树的叶子节点之间采用双向链表，适合基于范围的顺序查找。

其他数据几个数据结构的缺点：

**Hash 表：**在等值查询的时候效率贼快，但是不支持范围查询和排序。而且 InnoDB 也实现了自适应 hash，**对频繁访问的 B\+ 树索引页在内存中自动构建哈希结构，从而加速等值查询。**

**B 树**：查询可能在根节点就命中了（快），也可能在叶子节点才命中（慢）。这种波动对高性能系统来说是不友好的。而 B\+树所有的数据都在叶子节点。

---

###### 创建联合索引时要注意什么？

尽量把区分度高的字段放到最左边，尽量的满足最左前缀匹配。（将区分度高、使用频率高的字段放在前面；同时需要避免索引失效的情况，比如范围查询会截断索引、函数操作或类型不匹配等；另外可以通过覆盖索引减少回表，提高查询效率。\) 

---

###### 什么字段适合做索引？

1️⃣ 适合建立索引

经常 where 查询的字段，如果查询条件不是一个，考虑建立联合索引。

经常用于 Order by 和 group by 的字段，这样查询的时候就不需要再做一次排序，因为索引建立后在 B\+树中都是排序好的。

2️⃣ 不适合建立索引

表数据较少时，经常需要更新的字段，不会频繁查询，区分度较低（如性别）的字段。

###### 常见的索引失效的原因？

索引失效的本质是在 MySQL 中，索引失效通常是因为查询条件不符合索引的有序结构（B\+树），或者对索引列进行了计算、转换，导致无法利用索引进行快速定位。

1️⃣ 使用**左模糊匹配**时。

2️⃣ 对**索引列进行计算**，包括函数、类型转换等。

4️⃣ 使用联合索引时没有遵循最左匹配原则。

5️⃣ 在 where 条件中 OR 前**是索引列，OR 后面不是索引列。**

6️⃣ 索引设置不合理数据**区分度太低**，**优化器放弃索引走全表扫描**（例如使用性别作为索引）。

---

###### 索引优化的方法？

1️⃣ 使用覆盖索引减少回表查询。

2️⃣ 建立联合索引时，将区分度最大的字段放到最左边，以尽量满足最左前缀法则。

3️⃣ 对长字符串建立索引时，考虑使用前缀索引，避免索引字段占据空间。

前缀索引是指为了减少索引字段的大小，增加一个索引页专门记录完整的索引字段而只取前面的几位建立索引。

4️⃣  避免索引失效。[JAVA 八股](https://xcnm79jcno1y.feishu.cn/docx/Pk4sdT3ZVof7CrxAxSXc0SB2nYb#share-EPSjd6AbAoqAeNxVRPscKUCGnof)

5️⃣  索引下推。[JAVA 八股](https://xcnm79jcno1y.feishu.cn/docx/Pk4sdT3ZVof7CrxAxSXc0SB2nYb#share-QVecds0CjoNtTLxFnagcv1m9nEf)

6️⃣  使用自增 id。[JAVA 八股](https://xcnm79jcno1y.feishu.cn/docx/Pk4sdT3ZVof7CrxAxSXc0SB2nYb#share-PXTvdenjVoa8qhxsE0ScEAs6nYc)

---

###### 什么是索引下推（ICP）？

**将本该在 Server 层进行的过滤操作，下推到存储引擎层（InnoDB）执行，减少回表的次数。**

如有索引（name， age）

```Java
SELECT * FROM users WHERE name LIKE '张%' AND age = 25;
```

**age 的处境**：在 （name， age） 索引中，虽然 age 就在索引树里，但由于 name 这一列是模糊匹配，MySQL 无法直接利用索引顺序直接“跳”到 age=25 的位置。

**传统模式 （Using where）**：

1️⃣ 存储引擎通过索引找到第一个姓“张”的人。

2️⃣ 回表，查出整行数据传给 Server 层。

3️⃣ Server 层看他是不是 25 岁。

4️⃣ 重复这个过程，直到找完所有姓“张”的人。

**索引下推模式 （Using index condition）**：

1️⃣ 存储引擎通过索引找到第一个姓“张”的人。

**2️⃣ 不回表**，直接在索引里看这个人的 `age` 是不是 25。

**3️⃣ 如果是 25 岁，才执行回表**拿到整行数据；如果不是，直接在索引里找下一个姓“张”的人。

查看是否使用了索引下推

在 EXPLAIN 执行计划中看到** Using index condition**，说明使用了索引下推。索引下推是在存储引擎层（InnoDB）执行的优化，由 MySQL 优化器决定是否使用。

---

###### 为什么要使用自增主键？

因为在 InnoDB 中，自增主键能保证 B\+ 树索引的**顺序写入**，极大减少了页分裂和磁盘碎片。

自增主键 vsUUID

更详细的说，如果使用自增主键，那么每次插入的新数据就会按顺序添加到当前索引节点的位置，不需要移动已有的数据，当页面写满，就会自动开辟一个新页面。因为每次**插入一条新记录，都是追加操作，不需要重新移动数据**，因此这种插入数据的方法效率非常高。

如果我们使用非自增主键（如 UUID），由于每次插入主键的索引值都是随机的，因此每次插入新的数据时，就可能会插入到现有数据页中间的某个位置**，这将不得不移动其它数据来满足新数据的插入，甚至需要从一个页面复制数据到另外一个页面**，我们通常将这种情况称为**页分裂**。页分裂还有可能会造成大量的**内存碎片**，导致索引结构不紧凑，从而影响查询效率。

---

### 事务

###### 事务四大特性？

1️⃣ **原子性（A）**一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间环节。

2️⃣** 一致性（C）** 事务执行前后，数据库的完整性约束没有被破坏，数据从一个一致性状态转移到另一个一致性状态。

3️⃣ **隔离性（I）**多个并发事务之间相互隔离，不互相干扰。

4️⃣ **持久性（D）** 事务一旦提交，其结果就是永久性的，即便系统崩溃数据也不会丢失。

---

###### InnoDB 是如何实现事务的四大特性的？

1️⃣ 原子性（A）undolog 回滚日志。

2️⃣ 一致性（C）它依赖于其他三个特性（A、I、D），数据一致性是事务的最终目标。

3️⃣ 隔离性（I）MVCC 多版本并发控制或者锁机制。

4️⃣ 持久性（D）redolog 重做日志。

---

###### MySQL 有那些并发事务问题？

1️⃣** 脏读，**一个事务读到另一个事务还未提交的数据。

2️⃣** 不可重复读，**同一个事务内两次读到同一个数据，但是两次的值不一样。

3️⃣ **幻读，**一个事务去查询一个数据，两次查询到的记录数量不一样（或者第一次没查到，第二次又查到了）。

---

###### MySQL 是怎么解决这些并发事务问题？

1️⃣ 锁机制

2️⃣ 设置事务隔离级别

3️⃣ MVCC 并发版本控制

---

###### 事务的隔离级别？

隔离级别从低到高：

1️⃣** 读未提交（RU）**，读到其他事务还没提交的数据。

2️⃣ **读已提交（RC）**，一个事务只有提交后，它的修改才能被其他的事务看到。

3️⃣ **可重复读（RR）**，在一个事务中，多次查询同一个数据是一样的，不受其他事务修改的影响（MySQL 的默认隔离级别）。

4️⃣ **串行化（serializable）**，这是最高的隔离级别。它通过强制事务**排序执行**来避免冲突。

实现

读未提交， 直接读取最新的内存数据，**会出现脏读、不可重复读和幻读**。

串行化，使用**读写锁**实现，不会有并发安全问题。

读已提交和可重复读都是通过**（MVCC）**read view 和 undo log 日志实现。

在读已提交的隔离级别下，**不会出现脏读、可能会发生不可重复读和幻读**，可重复读可能会发生幻读现象。

---

###### 可重复读隔离级别下，A 事务提交的数据，在 B 事务能看见吗？

这个得分情况：

1️⃣ B 事务正常进行是读不到的，在可重复读这个隔离级别下，事务开启时 MVCC 会生成 read view（读视图），在以后得读取中都会复用这个读视图，察觉不到这个数据变化。

2️⃣ 但是如果使用当前读去查询，**SELECT \.\.\. FOR UPDATE**（当前读）、**SELECT \.\.\. LOCK IN SHARE MODE** （共享锁）或者 **UPDATE/DELETE** 语句（写操作，当前读）时，会从数据的最新版本，无视 MVCC 的快照，获取 A 事务提交后的最新数据。

---

###### Select \.\.\. For update 

Select \.\.\. For update 是当前读并且加锁。这行查询的底层 InnoDB 会加上行锁（索引命中时，如果索引没有命中会给整张表加上锁）。

注意   在** RR **隔离级别下，普通的 select 是不会产生幻读的，因为读取的是 MVCC 的 read view 为快照读。而使用 select \.\.\. For update 读取的是最新数据，会产生幻读（前后两次读取到的记录数不一致）加上**间隙锁**能避免幻读。 这也是为什么在 RR 隔离级别避免产生幻读的一个方法。

---

###### 串行化隔离级别是怎么实现的？

是通过加行级锁实现的，主要就是对所有的读操作也加锁，禁止并发写入。

在 serializable 下普通的 select 语句会加上** lock in share mode（共享锁），**写操作还是自动加排他锁。

---

###### MVCC 的实现原理？

MVCC 是通过 InonoDB 每一行的隐藏字段 **trx\_id** 和 **roll\_pointer ，undo log **版本链和 **Read view** 读视图实现的。

**trx\_id：**每行数据记录的最后一次修改这一行数据的事务 id。

**roll\_pointer：**回滚指针，指向 undo log 版本链。

**undo log 版本链：**每次修改这行数据时，在新数据覆盖同时，将旧数据写到 undo 日志中，用 roll\_pointer 串起来形成版本链。

**Read view** 读视图：作用是决定你能看到哪一个版本的数据。

**Read View 有四个重要的字段：**

**Read view 的作用就是要判断要从哪一个 undolog 版本链中获取数据。**

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWNmZjQ0Mjg5MGRkZDlhNGYwZjNhZmIzNjdmYWFjMjVfOTFkMzkyYjU4MWI3NGEzYzk0ODg5ODEwYTJkMWVmNGVfSUQ6NzYyNjcwODcwOTUwNjQzNjI5OV8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

**RR \& RC**

在 RR（可重复读）隔离级别下，开启事务后的第一个 select 快照读会创建读视图，以后都会复用这个 read view。

在 RC（读已提交）隔离级别下，事务每次 select 都会生成新的读视图，保证每次读取到的是**最新已提交的数据版本**。

---

###### RR 隔离级别下的幻读问题？

RR，可重复读的隔离级别下虽然很大程度上避免了幻读，但是并没有完全解决幻读问题。 首先 RR 隔离级别下在快照读是通过 MVCC 防止幻读，在当前读通过加锁（间隙锁）防止幻读。

出现幻读的情况：

1️⃣ 当前读：当使用当前读比如 select for update 时加临间锁（Next\-Key Lock）来避免幻读。

2️⃣ 快照读：在可重复读隔离级别下，事务 A 第一次执行普通的 select 语句时生成了一个 ReadView，之后事务 B 向表中新插入了一条 id = 5 的记录并提交。接着，事务 A 对 id = 5 这条记录进行了更新操作，在这个时刻，这条新记录的 trx\_id 隐藏列的值就变成了事务 A 的事务 id，之后事务 A 再使用普通 select 语句去查询这条记录时就可以看到这条记录了，于是就发生了幻读。

对于幻读的发生

---

### 锁

###### 讲一下 MySQL 中都有那些锁？

通过 flush tables with read lock 开启全局锁，开启后整个数据库变为只读状态，这时其他线程只能读，用于全库的**复制备份**中数据的一致性。

---

###### 行级锁

InnoDB 的行锁不是加在记录上的，而是**加在索引上的**，因此所有**行锁都要走索引为前提**。

1️⃣ 记录锁，锁定单条索引记录

2️⃣ 间隙锁，在可重复读（RR）隔离级别下的**范围查询**中触发。锁定索引记录之间的间隙，防止幻读现象。

3️⃣ 临键锁，**记录锁 \+ 间隙锁**的组合。它既锁定记录本身，也锁定记录之前的间隙，**它是 InnoDB 在 RR 隔离级别下默认的行锁类型。**

分析下面的 SQL：

```SQL
-- 事务 
SELECT * FROM users WHERE id BETWEEN 10 AND 20 FOR UPDATE;
```

情况 1：id 是唯一索引，会给（10， 20\]整个区间加上锁（10 到 20 存在），同时给间隙加间隙锁。

情况 2：id 是普通索引，说明会有重复的，它还会锁定 **10 之前的第一个间隙** 和 **20 之后的第一个间隙**。

情况 3：id 没有索引，如果 id 上没有索引，MySQL 必须进行全表扫描来定位数据，行锁会退化为表锁。

---

###### 表级锁

1️⃣ 表锁，通过 Lock TABLES 加锁，表共享读锁 （Table Read Lock）：所有事务都可以读该表，但任何事务都不能写。表独占写锁 （Table Write Lock）：只有持有锁的事务可以读写该表，其他事务的读写都会被阻塞。

2️⃣ 元数据锁，自动加锁，保证事务执行期间表结构不会改变。

3️⃣ 意向锁，自动加锁，如果没有意向锁，当你尝试给表加“表写锁”时，必须逐行检查有没有行锁，效率极低。有了意向锁，只需检查表上是否有 IX（意向排他锁） 锁，就能瞬间判断是否能加表锁。

**意向共享锁 （IS）**：表示事务准备给某些行加行级共享锁。

**意向排他锁 （IX）**：表示事务准备给某些行加行级排他锁。

---

### 日志

###### redo log

Redo log 重做日志是存储引擎层的，记录的是数据修改的物理日志（**某页的某条记录被改成了什么**），用于保证事务的**持久性**，在数据库宕机时恢复已提交的数据。

WAL \( Write\-Ahead Logging\) 

WAL 是指数据库在持久化到磁盘时先写日志，再写数据。

redolog 的作用

1️⃣ 实现事务的持久性。

2️⃣ 将随机写变为顺序写。

虽然数据页最终刷盘仍然是随机写，但 redo log 的作用是将原本每次更新都触发的随机写，转变为先顺序写日志，再由后台线程批量、延迟地刷脏页，从而大幅减少随机 IO 的次数和对用户请求的影响。

redolog 是怎么保证持久性的

1️⃣ **WAL**，在事务提交之前会先把数据的修改记录到 redo log 中并将 redo log 写到磁盘，就算在数据写到磁盘前发生宕机了，也可以根据 redo log 的内容重新刷盘。

2️⃣ **随机写变顺序写**，redo log 是一个环形缓冲区，对 redo log 的写操作是顺序写入，避免了频繁的随机写数据到磁盘的 IO 操作。

3️⃣ **checkpoint 机制**，标记哪些日志可以删了，推动刷盘。在恢复数据时，系统会根据标记来确定从哪个位置开始应用 redo log。

---

###### undo log

**Undo Log（回滚日志）**是存储引擎层的日志记录的是**数据修改前的旧值，**用于**保证事务的原子性以及支持 MVCC 多版本并发控制。**
 当对数据执行增删改操作时，InnoDB 会在修改前记录对应的 undo log，保存数据的历史版本信息，并通过数据行中的隐藏字段 roll\_pointer 将不同版本串联成版本链。
 在事务回滚时，可以通过 undo log 执行反向操作恢复数据；在快照读中，InnoDB 会结合 undo log 构建的版本链和 Read View，实现数据的**多版本可见性控制**。

什么时候删除 undo log

Undo log 分为 **insert undo log** 和** update undo log**，其中 update undo log 不能在事务提交后立即删除，因为 MVCC 可能仍然需要访问这些历史版本。

---

###### binlog

在 MySQL 中，binlog **是 Server 层的逻辑日志**，用于**记录数据库的所有修改操作**，binlog 是追加写写满一个文件后就创建一个新的文件继续写保存的是全量日志，主要用于**主从复制**和**数据恢复。**

Bin log 的三种格式：**STATEMENT（语句模式）记录的是 SQL 语句**，**ROW（行模式）记录每一行的逻辑变化**，**MIXED（混合模式）**。

---

###### Binlog 能不能取代 redolog？

**binlog 不能取代 redolog，binlog 是 server 层的日志，不能记录有那些脏页还没有刷盘。**

---

###### 事务的两阶段（2PC）提交过程？

事务提交后，**redo log** 和 **binlog** 都要持久化到磁盘，但是这两个是独立的逻辑，可能出现半成功的状态，这样就造成两份日志之间的逻辑不一致。如果这两个日志不一致，比如 Redo Log 提交了但 Binlog 没写完，**就可能导致主从数据不一致。**

**解决方法是事务分为两阶段提交方案：**

**1️⃣ prepare 阶段**：将 XID（内部 XA 事务的 ID） 写入到 redo log，同时将 redo log 对应的事务状态设置为 prepare，然后将 redo log 持久化到磁盘。必须先保证 redo log（prepare）持久化成功，才会去写 binlog，所以不会出现 binlog 成功但 redo log 失败的情况

**2️⃣ commit 阶段**：把 XID 写入到 binlog，然后将 binlog 持久化到磁盘（`sync_binlog = 1 的作用`），接着调用引擎的提交事务接口，将 redo log 状态设置为 commit，此时该状态并不需要持久化到磁盘，只需要 write 到文件系统的 page cache 中就够了，因为只要 binlog 写磁盘成功，就算 redo log 的状态还是 prepare 也没有关系，一样会被认为事务已经执行成功。**两阶段提交是以 binlog 写入成功为事务提交成功的标识。**

遇到崩溃时

在 MySQL 重启时，会按照顺序去扫喵 redo log 文件，遇到状态是 prepare 的 redo log 就拿着 redo log 的 XID（事务 id）去 binlog 中查找是否存在这个 XID，这时会出现两个情况：

① 如果没有 bin log 中存在这个 XID 说明 redo log 写入成功但是 bin log 写入失败，这时候要回滚。

② 如果 bin log 中存在 XID，说明 redo log 和 bin log 都完成刷盘，可以提交事务。

两阶段提交本质就是：先把 redo log 标记为**待提交**，再写 binlog，最后再确认提交，通过宕机恢复时判断 binlog 是否存在来决定事务是否真正提交，从而保证两者一致。

---

###### MySQL 持久化的整个过程？

一条写操作从执行到“真正落盘”的全过程：

```SQL
UPDATE user SET name = 'A' WHERE id = 1;
```

1️⃣ **修改 Buffer Pool**

会通过索引找的数据页（B\+数的节点），如果不在内存先去磁盘加载到 Buffer Pool，在内存中修改。Buffer Pool 中修改的数据页就是**脏页**。

2️⃣ **写 undo log 日志**

写入 undo log buffer（内存），后续刷到 undo tablespace（磁盘）。

3️⃣ **写 Redo log 日志**

记录这次修改的物理记录（某个数据页某个位置发生修改）。写入 redo buffer，根据策略刷到磁盘，这时的** redo log 状态是 prepare，**因为还没和 bin log 对齐。

4️⃣ **写 bin log 逻辑日志**

记录 SQL 或行变更（看具体的记录策略）。

5️⃣ **redo log commit**

这时，将 redo log 状态变为 commit，redo log 写到磁盘，bin log 写入成功作为事务提交成功。

6️⃣ **刷盘**

后台线程（flush thread）把脏页从 Buffer Pool 刷到磁盘。

7️⃣ **宕机恢复**

todo

---

### SQL 优化

###### explain

explain 是查看 sql 的执行计划，主要用来分析 sql 语句的执行过程，比如有没有走索引，有没有外部排序，有没有索引覆盖等等。

对于执行计划，参数有：

- possible\_keys 字段表示可能用到的索引；

- key 字段表示实际用的索引，如果这一项为 NULL，说明没有使用索引；

- key\_len 表示索引的长度；

- rows 表示扫描的数据行数。

- **type 表示数据扫描类型**，我们需要重点看这个。

type 字段就是描述了找到所需数据时使用的扫描方式是什么，常见扫描类型的**执行效率从高到低的顺序**：

system \> const \> eq\_ref \> ref \> range \> index \> ALL

extra

**Extra 表示 MySQL 在执行 SQL 时的额外操作或优化信息**，通常用来判断是否有性能问题。

Using index: 使用覆盖索引。

Using where: 使用索引之外的过滤条件。

Using filesort: 使用额外的排序，order by 上的字段是非索引字段。

Using temporary：使用了临时表，使用了 order by 或者其他复杂查询。

Using index condition: 使用了索引下推。

索引下推（ICP）是在存储引擎层（InnoDB）执行的优化，由 MySQL 优化器决定是否使用。

---

###### 给你张表，发现查询速度很慢，你有那些解决方案

**1️⃣ 分析查询语句**：使用 EXPLAIN 命令分析 SQL 执行计划，找出慢查询的原因，比如是否使用了全表扫描，是否存在索引未被利用的情况等，并根据相应情况对索引进行适当修改。

**2️⃣ 创建或优化索引**：根据查询条件创建合适的索引，特别是经常用于 WHERE 子句的字段、Orderby 排序的字段、Join 连表查询的字典、 group by 的字段，并且如果查询中经常涉及多个字段，考虑创建联合索引，使用联合索引要符合最左匹配原则，不然会索引失效。

**3️⃣ 避免索引失效：**比如不要用左模糊匹配、函数计算、表达式计算等等。

**4️⃣ 查询优化**：避免使用 SELECT \*，只查询真正需要的列；使用覆盖索引，即索引包含所有查询的字段；联表查询最好要以小表驱动大表，并且被驱动表的字段要有索引，当然最好通过冗余字段的设计，避免联表查询。

**5️⃣ 分页优化：**针对 limit n，y 深分页的查询优化，可以把 Limit 查询转换成某个位置的查询：select \* from tb\_sku where id\>20000 limit 10，该方案适用于主键自增的表，

深分页问题

查询时查询很靠后的位置开始分页只查询少量的数据，会导致跳过大量的无效数据：

```Java
SELECT * FROM user LIMIT 100000, 10;
```

使用子查询＋覆盖索引，先通过 id 定位到要查询的分页起点（这一步是会走索引的）再取 limit:

```SQL
SELECT * FROM user
WHERE id >= (
    -- 子查询只查询索引字段
    SELECT id FROM user ORDER BY id LIMIT 100000, 1
)
-- 确定分页位置，外层分页
LIMIT 10;
```

**6️⃣ 优化数据库表**：如果单表的数据超过了千万级别，考虑是否需要将大表拆分为小表，减轻单个表的查询压力。

**7️⃣ 使用缓存技术**：引入缓存层，如 Redis，存储热点数据和频繁查询的结果，但是要考虑缓存一致性的问题，对于读请求会选择旁路缓存策略（先写数据库再删除缓存），对于写请求会选择先更新 db，再删除缓存的策略

---

###### 如果 explain 中的索引不正确，有办法干预吗？

使用 force index \+ 索引 命令，强制走索引。

---

### 主从

###### 主从复制的过程？

1️⃣ **主库写入 bin log**

主库在事务提交时通过两阶段提交机制写入 binlog。

2️⃣ **从库接受 bin log 并写入 relay log**

从库通过 I/O 线程从主库拉取 binlog 并写入 relay log。

3️⃣ **从库执行 relay log 中的 sql**

然后由 SQL 线程读取 relay log 并执行，从而实现数据同步。默认采用异步复制，也可以通过半同步复制增强一致性。**（异步复制：主库提交事务后，不等待从库同步完成，就直接返回成功）**

异步复制的缺点

可能主库写 bin log 成功，但是从库还没来得及更新，这时去查从库查不到数据，会造成短时间的主从不一致。

要更安全可以使用半同步复制，会等到至少有一个从库收到 bin log 才返回复制成功。

relay log 的作用



---

###### 读写分离

读写分离是将**写操作路由到主库**，**读操作路由到从库**，从而提升系统的读性能和扩展性。它通常基于主从复制实现，但由于主从复制是异步的，可能会带来主从延迟和数据不一致问题。

---

###### 主从延迟



---

###### 主从一致性



---

### 分库分表

#### 什么场景下需要分库分表？

1️⃣ **分库**主要是为了解决单机数据库的**连接数瓶颈和资源压力问题**，通过将数据拆分到多个数据库实例上，实现连接压力分散和系统横向扩展。

2️⃣ **分表**主要是为了解决单表数据量过大带来的**查询性能下降问题**，通过分片键将数据拆分到多个物理表中，减少单表数据量，从而提升查询和写入效率。

---

## Redis

### 数据结构

Redis 的数据结构主要有 5 种常用的 \+4 种新增的数据结构

- **String** 类型的应用场景：缓存对象、常规计数、分布式锁、共享 session 信息等。

- **List** 类型的应用场景：消息队列（但是有两个问题：1。 生产者需要自行实现全局唯一 ID；2\. 不能以消费组形式消费数据）等。

- **Hash** 类型：缓存对象、购物车等。

- **Set** 类型：聚合计算（并集、交集、差集）场景，比如点赞、共同关注、抽奖活动等。

- **Zset** 类型：排序场景，比如排行榜、电话和姓名排序等。

Redis 后续版本又支持四种数据类型，它们的应用场景如下：

- **BitMap**（2\.2 版新增）：二值状态统计的场景，比如签到、判断用户登陆状态、布隆过滤器等；

- **HyperLogLog**（2\.8 版新增）：海量数据基数统计的场景，比如百万级网页 UV 计数等；

- **GEO**（3\.2 版新增）：存储地理位置信息的场景，比如滴滴叫车；

- **Stream**（5\.0 版新增）：消息队列，相比于基于 List 类型实现的消息队列，有这两个特有的特性：自动生成全局唯一消息 ID，支持以消费组形式消费数据。

---

###### String 

Redis 中 String 类型的底层数据结构是使用的** SDS（简单动态字符串）**，相比于 C 语言的默认字符串更快也更安全。

1️⃣ 在原来字符串的字符数组之上添加了** len **字段用来记录当前字符串的长度，C 语言获取字符串的长度是 O（N）需要遍历整个字符串。

2️⃣ **SDS 是以二进制的方式**处理数组中的数据，因此 Redis 中 String 对类型是没有限制的。

3️⃣** 安全**，不会发生缓冲区溢出的情况。引入** len **和** alloc（记录分配的空间长度）**字段，在 SDS 对字符串操作时自动判断是否会溢出。

###### Hash

Redis 在实现上采用了 **listpack（压缩链表 zipList 的升级版）** 和 **dict** （C 语言的字典）两种存储方式，根据 hash 的大小动态选择存储格式。

**hash 表的扩容，即 rehash 操作**：

1️⃣ 给「哈希表 2」 分配空间，一般会比「哈希表 1」 **大 2 倍**；

2️⃣ 将「哈希表 1 」的数据**渐进式迁移**到「哈希表 2」 中，渐进式 rehash 步骤如下：

- 在 rehash 进行期间，每次哈希表元素进行新增、删除、查找或者更新操作时，Redis 除了会执行对应的操作之外，还会顺序将「哈希表 1 」中索引位置上的所有 key\-value 迁移到「哈希表 2」 上。

- 随着处理客户端发起的哈希表操作请求数量越多，最终在某个时间点会把「哈希表 1 」的所有 key\-value 迁移到「哈希表 2」，从而完成 rehash 操作。

**3️⃣ 巧妙地把一次性大量数据迁移的开销分摊到了多次处理请求过程中，避免了一次性 rehash 的耗时操作造成 Redis 阻塞。**

4️⃣ 迁移完成后，「哈希表 1 」的空间会被释放，并把「哈希表 2」 设置为「哈希表 1」，然后在「哈希表 2」 新创建一个空白的哈希表，为下次 rehash 做准备。

哈希表扩容时有读请求怎么办？

会在原来的哈希表中查找，找不到再去扩容的那张表上查找。

###### List

quicklist = 双向链表 \+ listpack

###### Set

**无序唯一**，Set 类型的底层数据结构是由**哈希表或整数集合**实现的：

1️⃣ 如果集合中的元素都是整数且元素个数小于 512 个，Redis 会使用**整数集合**作为 Set 类型的底层数据结构；

2️⃣ 如果集合中的元素不满足上面条件，则 Redis 使用**哈希表**作为 Set 类型的底层数据结构。

###### Zset

**有序唯一**，由 **listpack **或**跳表（skipList）**实现，**跳表是在链表基础上改进过来的，实现了一种「多层」的有序链表这样的好处是能快读定位数据（O（logn）级别的查找类似于二分查找）。**

###### 介绍一下跳表 skiplist

首先，跳表在结构上是在链表的额基础上对每个节点加上层索引，比如链表 1 \-\- 2 \-\- 3 \-\- 4 \-\- 5 \-\- 6 \-\- 7，在原始的链表上会增加层级 1 可能会指向 3 再指向 5，类似向上再会增加层级。这样做避免了从第一个开始遍历到最后一个的 O（n）的时间复杂度。因为 zset 结构要频繁的对比，查找。

跳表怎么设置层高？

跳表在创建每个节点时，会生成该节点的层高，最高是 64。每次以一定概率（Redis 中为 1/4）决定是否继续向更高层扩展，直到随机条件不满足为止。这样可以使得节点在各层的分布呈现几何分布，从而保证跳表整体的时间复杂度为 O（logN）。

```C
while (random() < p) {
    level++;
}
```

为什么使用跳表而不使用 B\+树？

B\+树的设计主要是为了降低书的高度，MySQL 索引结构减少磁盘 IO 次数。Redis 是纯内存的操作，要求的是减少内存的使用，加快查询速度，而且相比于 B\+树的实现，跳表的实现更加简单，

###### 为什么要用 listpack 代替 ziplist ？

首先来说 ziplist，压缩链表是内存连续的顺序型数据结构，是 Redis 为了节省内存开发的。整体结构有点类似于数组，表头三个字段分别记录整个链表的字节数、表尾的偏移量和节点数量，表尾有一个结束标识。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjY0NjgxYzBmOGE4NGI4ZWI0YmVhMGQxOGNmZGE1NWJfMjFmNDEzNmM4OTk1OWVkZjBjZWMwYTQ4M2Q2Zjg4MmVfSUQ6NzYyOTM0NzAwNzk4MzM1NzEyNl8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

ziplist 中的节点 entry 记录前一个 node 占的长度，当前节点的类型和数据。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWY4ZGIwMDg3MzNmYjhiYmNlYTI2Y2RjOTZjN2U3ZTNfYmMxOTc4YzYzMTgzZTQ0MzJkZTkzMjQ2NjBkYjUxMjJfSUQ6NzYyOTM0NzIxNzYxNDMxMDYxMF8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

这种数据会根据不同数据类型和大小为每个 entry 分配不同的内存空间。但是由于每个节点都会记录前一个节点的长度，当中间每个 entry 的数据发生变化会导致内存的重新分配，严重的会发生**连锁更新**的问题。

**为了解决 ziplist 连锁更新的问题，后来引入了 listpack。**

在每个节点取消了记录前一个节点长度的字段，但继承了 ziplist 对数据的用一块连续的内存空间来紧凑地保存数据，并且为了节省内存的开销，**listpack 节点会采用不同的编码方式保存不同大小的数据。**

---

### Redis 线程

###### Redis 为什么快？

1️⃣ **基于内存存储**，内存访问速度远远快于磁盘访问速度，因此 Redis 能够快速读取和写入数据。

2️⃣ **采用单线程模型**，不需要进行多线程间的上下文切换，避免了多线程带来的竞争和同步开销。

3️⃣** 高效的数据结构**，Redis 专门设计了 STRING、LIST、HASH 等高效的数据结构，依赖各种数据结构提升了读写的效率。

4️⃣ I/O 多路复用。

---

###### Redis 哪里使用了多线程？

1️⃣ Redis 的单线程模型是指从 redis **接受客户端请求，进行读写操作，返回给客户端**的这个过程是由单线程（主线程）完成的。

2️⃣ 但是在 redis 启动时会额的创建 3 个后台线程，**AOF 刷盘**、**关闭文件**和**释放内存**。这些比较耗时的任务创建单独的线程来处理，如果把这些任务都放在主线程来处理，那么 Redis 主线程就很容易发生阻塞。

3️⃣ 另外在配置文件中通过显式开启 io\-thread N 会额外再创建 N\-1 个 I/O 线程（主线程也是一个 I/O 线程）去处理 i/o 请求。

---

###### Redis 怎么实现 io 多路复用？

Redis 的 I/O 主要实际两个，**与客户端进行 socket 通信进行数据读写并通过 IO 多路复用机制高效处理多个连接**，与磁盘的 I/O 进行持久化。这里的多路是指多个网路连接客户端。

以 Redis 的 I/O 多路复用程序 **epoll** 为例。多个客户端连接服务端时，Redis 会将客户端 socket 对应的 fd（文件描述符） 注册进 epoll，然后 epoll 同时监听多个文件描述符（FD）是否有数据到来，如果有数据来了就通知事件处理器处理，这样就不会存在服务端一直等待某个客户端给数据的情形。整个文件事件处理器是在单线程上运行的，但是通过 I/O 多路复用模块的引入，实现了同时对多个 FD 读写的监控，达到 IO 多路复用的效果。

解释一下 epoll（事件监听器）

**epoll （通过红黑树和一个就绪链表）是 Linux 提供的一种高性能 I/O 多路复用机制，用于在一个线程中高效监听和管理大量 socket 连接的读写事件。**对比传统的 select/poll 的方式，通过在内核中维护一个就绪事件队列，使得应用程序只需关注有事件发生的连接，从而避免遍历所有连接，提高性能。

---

###### Redis 的网络模型？

Redis 的网路模型使用** Reactor 模型**，

Redis 采用基于** IO 多路复用（如 epoll）**的 **Reactor 事件驱动模型**，在单线程中通过事件循环监听多个客户端连接。当客户端与 Redis 建立连接时 Redis 将其注册到 epoll，当某个连接有读写事件时，通过事件分发机制调用对应的处理器完成命令执行和结果返回，从而实现高并发处理。在 Redis 6 之后，引入了多线程处理网络 IO，以进一步提升性能，但命令执行仍然保持单线程。

---

### 事务

###### 如何实现 redis 的原子性？

单个的 redis 操作本来就是原子性的，多个 redis 的执行命令可以放到一个 lua 脚本中执行，保证原子性。

###### redis 事务

redis 的事务通过** MULTI **开启，**EXEC **提交。只能保证全部执行或者全部不执行，**不能保证出错回滚。**

---

### 持久化

###### AOF 日志实现持久化

每执行一条指令就以追加的方式写入到** AOF **日志文件中，然后 Redis 重启时，会读取该文件记录的命令，然后逐一执行命令的方式来进行数据恢复。

AOF 提供了三种持久化磁盘的策略：

###### RDB 快照实现持久化

RDB 通过定期生成内存数据的快照文件（某一时刻 Redis 里面的数据）实现持久化。

AOF 方法做故障恢复时，**需要全量把日志都执行一遍，一旦 AOF 日志非常多，势必会造成 Redis 的恢复操作缓慢**。而 RDB 记录的是记**录某一个瞬间的内存数据，记录的是实际数据**。Redis 提供了两个命令来生成 RDB 文件，分别是 save 和 bgsave：

- 执行了 save 命令，就会在主线程生成 RDB 文件，由于和执行操作命令在同一个线程，所以如果写入 RDB 文件的时间太长，**会阻塞主线程**；

- 执行了 bgsave 命令，会创建一个子进程来生成 RDB 文件，这样可以**避免主线程的阻塞**。

###### AOF vs RDB

**AOF: **

**优点：**首先，AOF 提供了**更好的数据安全性**，因为它默认每接收到一个写命令就会追加到文件末尾。即使 Redis 服务器宕机，也只会丢失最后一次写入前的数据。其次，AOF 支持多种同步策略（如 everysec、always 等），可以根据需要调整数据安全性和性能之间的平衡。同时，AOF 文件在 Redis 启动时可以通过重写机制优化，减少文件体积，加快恢复速度。并且，即使文件发生损坏，AOF 还提供了 redis\-check\-aof 工具来修复损坏的文件。

AOF 三种写回策略：**Always 同步写回**，每个命令执行完都持久化；**Everysec 每秒写回**，宕机时会丢失最后一秒的数据，**NO 由操作系统决定写回时机**。

**缺点**：因为记录了每一个写操作，所以 AOF 文件通常比 RDB 文件更大，**消耗更多的磁盘空间**。并且，频繁的磁盘 IO 操作（尤其是同步策略设置为 always 时）可能会对 Redis 的写入性能造成一定影响。

**RDB: **

**优点**： RDB 通过快照的形式保存某一时刻的数据状态，文件体积小，备份和恢复的速度非常快。并且，RDB 是在主线程之外通过 fork 子进程来进行的，不会阻塞服务器处理命令请求，对 Redis 服务的性能影响较小。最后，由于是定期快照，RDB 文件通常比 AOF 文件（全量写操作指令）小得多。

**缺点**： RDB 方式在两次快照之间，如果 Redis 服务器发生故障，这段时间的数据将会丢失。并且，如果在 RDB 创建快照到恢复期间有写操作，恢复后的数据可能与故障前的数据不完全一致

###### Redis 使用的是哪一种？

**使用混合持久化的方式**。生成新的 AOF 文件的过程会会将内存中的数据以 RDB 的文件写入 AOF，后续的写指令以 AOF 的格式追加。数据恢复时，大部分的前半段数据直接快速读取到内存（RDB），后半部分 AOF 执行命令，即保留了 AOF 日志的数据安全完整，又兼顾这个 RDB 读取的速度和占用空间较小的优势。

---

### 内存淘汰和过期删除

###### 内存淘汰和过期删除？

1️⃣ 内存淘汰策略是在**内存满了**的时候，redis 会触发内存淘汰策略，来淘汰一些不必要的内存资源，以腾出空间，来保存新的内容

2️⃣ 过期键删除策略是将**已过期的键值**对进行删除，Redis 采用的删除策略是**惰性删除\+定期删除**。

###### 介绍一下 redis 中有哪些内存淘汰策略？ 

###### 介绍一下 Redis 过期删除策略

Redis 的过期删除策略主要有两种，**惰性删除**和**定期删除**。

1️⃣ 惰性删除

每次从数据库访问 key 时，都检测 key 是否过期，如果过期则删除该 key。

2️⃣ 定期删除

每隔一段时间**随机**从数据库中取出一定数量的 key 进行检查，并删除其中的过期 key。 Redis 的定期删除的流程：

1️⃣ 从过期字典中随机抽取 20 个 key；

2️⃣ 检查这 20 个 key 是否过期，并删除已过期的 key；

3️⃣ 如果本轮检查的已过期 key 的数量，超过 5 个，也就是已过期 key 的数量占比随机抽取 key 的数量大于 25%，则继续重复步骤 1；如果已过期的 key 比例小于 25%，则停止继续删除过期 key，然后等待下一轮再检查。

Redis 的做法是两种删除策略配合使用，并不会立即删除，因为删除 key 也是要消费一部分 CPU 时间。

---

### 双写一致性

1️⃣ **先修改数据库，再删除缓存**

在极端情况下会造成缓存与数据库数据不一致的情况：

读线程 A，写线程 B 同时访问一个数据。此时缓存中没有这个数据，读线程回去读数据库，在读线程 A 读完数据库到重新写入缓存这段时间，写线程 B 完成了修改并更新缓存，线程 A 将刚才查到的旧值写入缓存，此时数据库和缓存的数据是不一致的。

如果删除失败了有什么解决方案吗？

将失败的消息加入消息队列（MQ），由消费者不断重试直到删除成功或者最终丢到死信队列中人工解决。

2️⃣** 延时双删**

先删除缓存，再更新数据库，延时几百毫秒再次删除缓存（**防止更新数据库期间有旧数据重新进入缓存**）。

3️⃣** canal 监听数据库配合 MQ 异步更新缓存**

解决这个问题的核心不在于杜绝所有冲突，而是保证最终一致性，利用**订阅 Binlog 异步同步缓存数据**来最大程度降低不一致的时间窗口。

---

### 场景

###### 缓存穿透

缓存穿透是指请求去访问一个不存在的数据时，因为缓存不能命中，数据库也没有查完之后不能重建缓存，造成如果有大量的这样的请求过来会直接达到数据库。

**解决方案**

**1️⃣ 缓存空值，**当发现数据库也没有时向缓存中存储一个 null，在业务层对 null 值做校验。

**2️⃣ 布隆过滤器**

布隆过滤器由**初始值都为 0 的位图数组**和** N 个哈希函数**两部分组成。当我们在写入数据库数据时，在布隆过滤器里做个标记，这样下次查询数据是否在数据库时，只需要查询布隆过滤器，如果查询到数据没有被标记，说明不在数据库中。

布隆过滤器会通过 3 个操作完成标记：

1️⃣ 使用 N 个哈希函数分别对数据做哈希计算，得到 N 个哈希值；

2️⃣ 将第一步得到的 N 个哈希值对位图数组的长度取模，得到每个哈希值在位图数组的对应位置。

3️⃣ 将每个哈希值在位图数组的对应位置的值设置为 1；

**注：**布隆过滤器由于是基于哈希函数实现查找的，高效查找的同时**存在哈希冲突的可能性**，查询布隆过滤器说数据存在，并不一定证明数据库中存在这个数据，但是查询到数据不存在，数据库中一定就不存在这个数据。

###### 缓存击穿

缓存击穿是指一个频繁访问的数据**（热点 key）失效**（TTL 到期），无法从缓存中读取，短时间内会击垮数据库。

**解决方案**

1️⃣** 加分布式锁**，使得只能有一个线程去访问数据库重建缓存，但是会使大量的线程一直处于等待锁的过程。

2️⃣ **设置逻辑过期时间**，查询时先检查过期时间的字段，时间到就开一个新的单独线程异步重建缓存。但是实际上是永不过期的，先暂时从缓存返回旧的数据，**但是牺牲了一致性**。

###### 缓存雪崩

缓存雪崩是指大量的**缓存数据同一时间到期**（失效）或者** redis 服务直接宕机**，造成大量的请求直接同时访问数据库，造成数据库压力骤增。

**解决方案**

1️⃣ 给不同的数据设置不同的过期时间，避免同一时刻失效。

2️⃣ 使用多级缓存和缓存预热提升容错能力。

3️⃣ 做请求限流和降级保护数据库，提高 Redis 抗压。

4️⃣ 以及通过 Redis 哨兵或集群模式提升系统高可用性。

---

###### 热 Key

**什么是热 key**

热 Key 是指某个 Key 在短时间内被大量访问，导致** Redis 单点压力过大**甚至成为性能瓶颈。解决思路是：分散请求、缓存兜底、限流保护。

解决方案

**1️⃣ 拆分 Key 分散请求，或者拆分 key 配合分散到不同的 redis 节点。**

如果拆分后的 Key 仍然落在同一个 Redis 节点上，只能缓解单 Key 的热点问题，但无法解决节点级的性能瓶颈。要彻底解决热 Key 问题，需要结合 Redis Cluster，将拆分后的 Key 分布到不同节点，从而实现真正的负载分散。

2️⃣ 在 redis 前加一层本地缓存如** Caffeine，**减少 redis 的频繁访问。

3️⃣ 对请求进行**限流**，常见的限流算法如漏桶，令牌桶。

4️⃣ 如果热 key 的产生来自于读请求，可以采用 redis 读写分离

###### 大 Key

todo

###### Redis 的应用场景？

**1️⃣ 缓存**： 通过将热门数据存储在内存中，可以极大地提高访问速度，减轻数据库负载。

**2️⃣ 排行榜**： Redis 的有序集合结构非常适合用于实现排行榜和排名系统，可以方便地进行数据排序和排名。

**3️⃣分布式锁**： Redis 的特性可以用来实现分布式锁，确保多个进程或服务之间的数据操作的原子性和一致性。

**4️⃣ 计数器** 由于 Redis 的原子操作和高性能，它非常适合用于实现计数器和统计数据，如网站访问量、点赞等。

**5️⃣ 消息队列**： Redis 的发布订阅功能使其成为一个轻量级的消息队列，它可以用来实现发布和订阅模式，以便实时处理消息。

**6️⃣ 布隆过滤器：**使用 Redis 的 `Bloom Filter` 模块（RedisBloom）。

###### Redis 怎么实现分布式锁？

**分布式锁**

在分布式或者集群环境中，确保同一时刻只有一个应用实例去访问公共资源。Redis 是单线程 \+ 高性能的 KV 存储，非常适合用来实现分布式锁。

实现方式

通过 setnx 和 lua 脚本实现

**获取锁：**set key value nx ex 。如果 key 存在说明已经有线程抢到锁了返回 0，获取锁失败，如果 key 不存在将 value 设置为一个标识该应用实例的唯一标识，并设置过期时间避免该获取锁的应用挂了形成死锁。

**释放锁：**解锁的过程加锁删除这个 key，但不能乱删，要保证执行操作的客户端就是加锁的客户端。所以，解锁的时候，我们要先判断锁的 unique\_value 是否为加锁客户端，是的话，才将 lock\_key 键删除。可以看到，解锁是有两个操作，这时就需要 Lua 脚本来保证解锁的原子性，因为 Redis 在执行 Lua 脚本时，可以以原子性的方式执行，保证了锁释放操作的原子性。

```Lua
// 释放锁时，先比较 unique_value 是否相等，避免锁的误释放
if redis.call("get",KEYS[1]) == ARGV[1] then
  return redis.call("del",KEYS[1])
else
  return 0
end
```

如果不保证解锁的原子性，会发生什么样的问题？

会解锁时，可能会释放掉其它应用线程的锁，比如在判断是自己的锁标识成功在删除前锁过期了，其他线程获取成功，接着删除，就删除成其他应用线程的锁。

###### 既然你提到了 **setnx** 实现分布式锁，那么如果拿到锁的线程挂了，怎么防止死锁？

给锁设置一个合理的**过期时间（Lease Time）**。同时，为了防止业务逻辑执行时间过长导致锁提前失效，可以引入 **Redisson 的看门狗（Watchdog）机制**，实现锁的自动续期，确保在业务没处理完之前锁一直有效。

### 分布式集群



---

## 消息队列

###### Rocket MQ 架构组成

1️⃣ **Topic**

Topic 可以理解为是**存储队列**的一个顶层容器，可以将同一业务逻辑的队列放到同一个 topic。Topic 是 RocketMQ 中**逻辑上的消息分类单位**。在底层，一个 Topic 会被拆分成多个 Message Queue 分布在不同的 Broker 上，从而实现高并发的读写。

Tag 是 topic 下的二级分类，可以更细致的过滤队列。

2️⃣ **Queue**

3️⃣** NameServer**

NameServer 是 Rocket MQ 的注册中心，存储所有 broker 的地址，名称和 id 以及该 broker 上所有的 topic 的队列配置。

核心功能

4️⃣** Broker**

**负责消息存储与投递**，支持主从部署。

5️⃣** Producer**

6️⃣ **Consumer**

两种消费模式

Pull（主动拉取）

Push（本质也是 Pull \+ 长轮询）

###### **NameServer**

NameServer 是 Rocket MQ 的注册中心，存储所有 broker 的地址，名称和 id 以及该 broker 上所有的 topic 的队列配置。

核心功能 

1️⃣ **Broker 管理**：接收 Broker 的注册和心跳，维护 Broker 的存活状态。
2️⃣ **路由服务**：为 Producer 和 Consumer 提供 Topic 到 Broker 的路由信息。

心跳机制   通过心跳机制管理连接的 broker，Broker 每隔 30s 发送心跳，NameServer 每隔 10s 进行心跳检测，超过 120s 没有检测到会认为该 broker 挂掉，会从路由表中删除。

不需要 NameServer 的一致性  Rocket MQ 中的 NameServer 是无状态的，彼此互不通信。**Broker** 启动时与所有的 NameServer 建立连接，会向所有的 NameServer 节点轮询发送心跳，汇报自己的 Topic 信息。每个 NameServer 最终都会存有一份完整的 Broker 列表（保证 AP，舍弃 CP 的做法）。

---

###### Broker 

broker 的作用：

1️⃣ 消息的**中转和持久化**，Broker 最基本的工作就是接收 Producer 发来的消息，把它存入磁盘，然后等待 Consumer 来取。

broker持久化消息

**CommitLog**：所有发往该 Broker 的消息，不论属于哪个 Topic，都会**顺序写入**这个大文件（磁盘文件）。

**ConsumeQueue**：为了提高读取效率，Broker 会为每个 Topic 的每个 Queue 维护一个索引文件，指向 CommitLog 中的物理偏移量。

这里 **CommitLog \+ ConsumeQueue 分离，本质是：用“顺序写”换性能，用“索引”换读取效率（索引分离）。**

2️⃣ 与 NameServer 建立连接上报路由信息。

3️⃣ 处理各种特殊消息，进行消息过滤，处理**延迟消息**和**事务消息**。

---

###### Rocket MQ 发送到消费一条消息的流程？

1️⃣ Producer 启动后，从 NameServer 获取 Topic 的路由信息（Broker 地址、队列分布）
 2️⃣ 根据负载均衡策略选择一个 MessageQueue
 3️⃣ 发送消息到对应的 Broker
 4️⃣ Broker 将消息顺序写入 CommitLog，并构建 ConsumeQueue 索引（用于快速消费），再进行刷盘
 5️⃣ Broker 返回发送结果（同步 / 异步 / 单向）
 6️⃣ Consumer 启动后，同样从 NameServer 获取路由信息，并与 Broker 建立连接
 7️⃣ Consumer 从 Broker 拉取消息（默认是 Pull 模式，业务上看起来像 Push）
 8️⃣ 消费成功后，**向 Broker 提交 offset**，记录消费进度

---

###### 延时消息实现的原理？

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NmIyOTczMzQ5NzM4ZmVjMWE2YmMwMWRmYjI5NTgyZjNfNGFlMWI1NmRlYjQxZjBjYTJlOTM0NGZiY2EzYjZhNGJfSUQ6NzYzNDczNDc5Mzk2MzQ0MTMzOF8xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

延时消息是由 **Producer 发送**的，发送时会设置一个 delayLevel。Broker 收到消息后，不会直接写入目标 Topic，而是先存入一个内部的延时 Topic（SCHEDULE\_TOPIC\_XXXX）对应的队列中。Broker 内部有一个定时任务（**ScheduleMessageService**），会不断扫描这些延时队列，根据消息的存储时间和延迟级别判断是否到期。一旦到达设定时间，就会把消息重新投递到原始 Topic 中，同时去掉延迟属性。这样消息就变成普通消息，可以被 Consumer 正常消费。

---

###### Rocket MQ 消息顺序怎么保证？

RocketMQ 的顺序性是建立在 **Queue（队列）** 之上的，因此只要能保证一组要消费的消息发到同一个队列并顺序消费，就能实现顺序性。Rocket MQ 的消息顺序性要保证**生产顺序性**和**消费顺序性**。

1️⃣ **生产者顺序发送**

在发送消息的时候传一个相同的 hashKey，可以根据业务去传比如订单号。内部会对 orderId 做 hash 运算，固定到某一个 Queue。

```Java
*/***
* * Same to {@link #syncSend(String, Message)} with send orderly with hashKey by specified.*
* **
* * @param destination formats: `topicName:tags`*
* * @param message {@link org.springframework.messaging.Message}*
* * @param hashKey use this key to select queue. for example: orderId, productId ...*
* * @return {@link SendResult}*
* */*
public SendResult syncSendOrderly(String destination, Message<?> message, String hashKey) {
    return syncSendOrderly(destination, message, hashKey, producer.getSendMsgTimeout());
}

// 例如
rocketMQTemplate.syncSendOrderly(
    "topic",
    message,
    orderId // 关键：同一个订单走同一个队列
);
```

2️⃣ **消费者顺序消费**

同一业务路由到同一队列 \+ 单线程顺序消费，从而保证局部有序

单线程消费 topic 里的消息，将消费模式设置为** ConsumeMode\.ORDERLY** 或者使用提供的 **MessageListenerOrderly** 监听器。**Rocket MQ 底层会加上队列锁，确保同一个队列只能被同一个消费者线程消费。**

```Java
@RocketMQMessageListener(
        topic = TicketRocketMQConstant.*ORDER_DELAY_CLOSE_TOPIC_KEY*,  // 监听topic
        selectorExpression = TicketRocketMQConstant.*ORDER_DELAY_CLOSE_TAG_KEY*,  // 监听的tag
        consumerGroup = TicketRocketMQConstant.*TICKET_DELAY_CLOSE_CG_KEY*,   // 消费组
        consumeMode = ConsumeMode.*ORDERLY *// 顺序消费
)
```

---

###### Rocket MQ 消息重复消费怎么解决？

RocketMQ 不保证消息不重复，但保证消息不丢失（至少消费一次）。解决重复消费的唯一方案是**业务端**实现**幂等性（Idempotency）**。

常见实现幂等的方案

1️⃣ 利用数据库唯一键保证不重复插入。

2️⃣ 使用 Redis 去重

```Plain Text
setnx messageId
```

messageId 是 Broker 给每条消息分配的唯一标识，也可以缓存业务唯一标识 id（更推荐）。

3️⃣ 有些业务的状态字段可以作为天然的幂等键，如订单状态。

4️⃣ 本地消息表。

---

###### Rocket MQ 如何处理分布式事务？

在 RocketMQ 中使用的是 **事务消息**加上**事务反查机制** 来保证只有事务 A 的本地事务提交成功，事务 B 才会收到消息：

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZWQ2NTNhZDMwNjQxYTU3ZWVmMzJkYTcwYzY4MmM0MThfMzQ2ODk4NmJjMjAwNDdmZWI0MjQwYzA0ODVlMTkwNzdfSUQ6NzYyNzEzOTQzNTczNzU4Mjc5N18xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)



在分布式转账场景中，如果 A 扣款成功但 B 加款失败，本质是分布式事务的一致性问题。

**1️⃣ RocketMQ 事务消息只保证本地事务和消息发送的一致性，无法覆盖跨服务的分布式事务问题，例如订单服务成功但库存服务失败，这种情况需要依赖消费者重试机制或业务补偿机制实现最终一致性。**

2️⃣ 一般不会使用强一致性事务，而是采用最终一致性方案。常见做法是通过消息队列实现异步调用，B 服务消费失败时通过 RocketMQ 的重试机制进行多次尝试，如果仍然失败则进入死信队列进行补偿处理。

3️⃣ 同时也可以引入业务补偿机制，例如在 B 失败后触发 A 的回滚操作，或者通过定时任务进行对账修复，从而保证最终一致性。

---

###### Rocket MQ 消息堆积了怎么办？

TODO

---

###### Rocket MQ 怎么保证消息的可靠性（消息不丢失）？

MQ 通过**生产者确认机制**、**Broker 持久化机制**以及**消费者确认机制（包括重试机制）**，从发送、存储到消费三个阶段保证消息不丢失。

1️⃣ 首先在生产者侧，通过**同步发送**、**失败重试（Rocket MQ 默认两次）**以及发送确认机制（收到 broker 的）确保消息成功发送。

```Java
SendResult sendResult = rocketMQTemplate.syncSend
```

生产者发送消息会返回** SendResult **类型的结果，通过获取发送状态可以判断是否发送成功。

2️⃣ 其次在 Broker 侧，通过消息持久化（**CommitLog**）、刷盘机制以及主从复制保证消息可靠存储。

broker 持久化消息

**异步刷盘（默认）：** 消息写入操作系统的 PageCache 后立即返回。优点是快，缺点是如果系统断电，PageCache 中的数据会丢失。

**同步刷盘：** 只有消息真正写入磁盘（CommitLog）后，Broker 才返回成功。这保证了物理存储的绝对安全，但性能会有所下降。

3️⃣ 消费端，消费者成功消费后才向 broker 发送** ACK（CONSUME\_SUCCESS）**，broker 收到成功后才提交** offset**，如果返回 **RECONSUME\_LATER **会被放回**重试队列**，如果一条消息重试了 16 次还是失败会放入**死信队列**，由人工干预做兜底。

**什么是 offset**

**offset 就是“消费进度标记”，用来记录 Consumer 已经消费到队列的哪一条消息了，相当于队列种中的游标。**

---

## 微服务

###### 常见的限流算法

限流：用于控制对某个服务、接口或功能的访问速率。它的主要目的是防止过度的请求或流量超过系统的处理能力，比如登录暴力破解，接口恶意刷请求、瞬间高并发访问等。

1️⃣ **固定窗口限流**

将时间划分为固定窗口大小，在每个时间窗口内限制请求的次数。比如每秒钟限制请求数 100，在 1 秒内第 100 个请求后的无效。但是这种做法会有边界突刺，在两个窗口之间可能请求会超出阈值。

2️⃣ **滑动窗口限流**

滑动窗口与固定窗口相比把窗口分为固定的小时间片段，每个时间片段有自己的计数器。当请求到达时，会逐渐删除过时的时间片段，每次统计的是最近的一个窗口的请求数量，这种算法可以更好地处理突发请求，但是实现复杂，存储成本高。

3️⃣ **漏桶算法**

请求以固定速率进入漏桶。如果漏桶已满，则多余的请求将被丢弃或延迟处理。这种算法不允许突发流量。

4️⃣ **令牌桶算法（最优的做法）**

模拟了一个**令牌桶**，桶中以固定速率生成令牌。每个令牌代表一个请求的许可。当请求到达时，需要先从令牌桶中获取令牌，如果桶中没有足够的令牌，则请求被限制。这种算法可以平滑地控制请求的速率。

---

### GateWay

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YWYzYjRhNWIzZWU2ZjU5NGY4NjAwNjlkMjJmMmJkN2VfMDM3OTdiZDQ5NjUxNzRhNGU0YWE2MTRiNTJiNWY3NDJfSUQ6NzYzMTAwMDM1OTA1MTQxNDczM18xNzgxMDU1NTEyOjE3ODExNDE5MTJfVjM)

###### 网关的作用？

统一管理所有微服务请求入口，可以做统一的**登录校验**，**鉴权**，**不同服务之间传递用户信息**等。

主流的网关：SpringCloudGateway，基于 Spring 的 WebFlux 技术，完全支持响应式编程，吞吐能力更强。

###### SpringCloud GateWay

引入依赖，在配置文件中配置路由，主要有三个参数，id   路由对应的目标服务和路由断言。

```YAML
cloud:
    nacos:
      server-addr: 192.168.150.101:8848
    gateway:
      routes:
        - id: item # 路由规则id，自定义，唯一
          uri: lb://item-service # 路由的目标服务，lb代表负载均衡，会从注册中心拉取服务列表
          predicates: # 路由断言，判断当前请求是否符合当前规则，符合则路由到目标服务
            - Path=/items/**,/search/** # 这里是以请求路径作为判断规则
        - id: cart
```

###### 网关在请求转发前做登录校验

通过在转发请求的过滤器 NettyRoutingFilter 前加一层登录校验的自定义过滤器就行了（设置 Ordered 优先级）。在自定义过滤器中放行登录请求路径，对其他路径请求做校验 JWT。

###### 校验后传递用户信息到下游服务

由于网关发送请求到微服务依然采用的是 Http 请求，因此可以将用户信息以请求头的方式传递到下游微服务（**这一步在网关过滤器校验成功后做**）。然后微服务可以从请求头中获取登录用户信息。在每个服务内部利用 SpringMVC 的拦截器来实现登录用户信息获取，并存入 ThreadLocal，方便后续使用。

###### 微服务之间传递用户信息（OpenFeign）

虽然在网关的过滤器中已经把 JWT 的信息保存到了请求头中，但是在微服务之间的调用时是一次新的请求，不会携带原始请求的 Header。

在微服务之间调用时需要再做一次拦截，借助 Feign 中提供的一个拦截器接口：feign\.RequestInterceptor，在 RequestTemplate 类来添加请求头，将用户信息保存到请求头中。这样以来，每次 OpenFeign 发起请求的时候都会调用该方法，传递用户信息。

在整个微服务中，用户信息的传递：

客户端（带 JWT）
 ↓
网关（解析 JWT → 写 Header）
 ↓
A 服务（从 Header 取 → ThreadLocal）
 ↓
A 调用 B（Feign 拦截器 → 再写 Header）
 ↓
B 服务（再取 Header 使用）

---

### Nacos

###### Nacos 的作用，为什么 要引入 Nacos？

Nacos 是一个用于服务注册和服务发现，并且配置管理的组件。

###### 注册中心

如果服务较多，并且进行了多实例部署，那么对于每个实例的 ip 和端口都不同。当进行跨服务调用时并不知道选择哪一个服务并且当某一个实例宕机时或者部署了新的实例后不同服务之间是感受不到的。Nacos 的作用就在于对服务的注册、发现以及健康状态管理。

1️⃣** 服务注册**

**引入 Nacos 的依赖**，在配置文件中**配置 Nacos 的地址并指明服务名称**（这个名称是在 Nacos 中进行跨服务调用的标识）。当服务启动时会自动的将自己的 ip 和端口注册到 Nacos 中，服务提供者每 30 秒发送一次心跳，90 秒没有收到该服务就被剔除（**心跳机制**）。

**2️⃣ 服务发现**

配置流程和服务注册一样，不过要加入负载均衡 LoadBalancer 依赖。这里 Nacos 的依赖于服务注册时一致，这个依赖中**同时包含了服务注册和发现**的功能。因为任何一个微服务都可以调用别人。当进行远程调用时会采用负载均衡算法选择一个实例进行调用。

###### 配置中心

可以把微服务共享的配置抽取到 Nacos 中统一管理，这样就不需要每个微服务都重复配置了。

---

### OpenFeign

OpenFeign 是一**个声明式的 HTTP 客户端**，用于实现微服务之间的远程调用。解决原来的 restTemplate 中手动发起的 HTTP 请求，把远程调用转变为类似于 SpringMVC 通过声明注解的本地调用。

**底层是通过动态代理 \+ HTTP 请求的方式。**

OpenFeign 完成了服务拉取、负载均衡、发送 http 请求的所有工作。

```Java
// 在配置文件中注册到 Nacos 中的服务名
@FeignClient("item-service")   
public interface ItemClient {
    
    @GetMapping("/items")
    List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
}
```



---

### Sentinel

###### 什么是 Sentinel？

Sentinel 是阿里巴巴开源的一款服务保护框架。



###### 请求限流



###### 线程隔离



###### 服务熔断



---

### Seata



---

## 计算机网络



## 操作系统



## 常用工具



### git



### Linux



### Docker











