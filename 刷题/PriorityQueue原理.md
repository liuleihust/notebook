> PriorityQueue是一个 优先队列，它与先进先出的队列的区别在于：优先队列每次出队的元素都是优先级最高的元素。如果确定一个元素的优先级最高呢？JDK 中使用**堆**这么一种数据结构，通过堆使得每次出队的元素总是队列里面最小的，而元素的大小比较方法可以由用户指定，这里相当于指定优先级。

### 二叉堆介绍

堆的特点:

- 堆中某个节点的值总是不大于或不小于其父节点的值
- 堆总是一颗完全树。

二叉堆有两种：最大堆和最小堆。

**最大堆：父结点的键值总是大于或等于任何一个子节点的键值；最小堆：父结点的键值总是小于或等于任何一个子节点的键值。**

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112230832772-926096882.jpg)

上图就是一颗完全二叉树（二叉堆），在第n层深度被填满之前，不会开始填 n+1层深度，而且元素插入是从左往右填满。

基于这个特点，二叉树又可以用数组来表示而不是用链表。

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112230911382-1962173843.jpg)

基于数组实现的二叉堆，**对于数组中任意位置的n上元素，其左孩子在**[2n+1]**位置上，右孩子[2(n+1)]位置，它的父亲则在[(n-1)/2]上，而根的位置则是[0]**。

### PriorityQueue的底层实现

```java
private static final int DEFAULT_INITIAL_CAPACITY = 11;
transient Object[] queue;
```

可以看到 PriorityQueue 底层用动态数组实现一个二叉堆。

##### 二叉堆的添加原理及PriorityQueue 的入队实现

为了维护二叉堆的特点（根节点小于或等于任何一个子节点的键值，以小根堆为例），在添加元素的时候，需要一个 *上移* 动作，如图所示：

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231131850-1172280189.jpg)

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231149850-1681339059.jpg)

结合上面的图解，我们来说明一下二叉堆的添加元素过程：

1. **将元素2添加在最后一个位置（队尾）**。
2. **由于2比其父亲6要下，所以将元素 2 上移，交换 2 和 6 的位置**
3. **然后由于2比5小，继续将2上移，交换2和5的位置，此时2大于其父亲1，结束。**

*PriorityQueue* 实现二叉堆的添加

```java
/**
     * 添加一个元素
     */
    public boolean add(E e) {
        return offer(e);
    }
 
    /**
     * 入队
     */
    public boolean offer(E e) {
        // 如果元素e为空，则抛出空指针异常
        if (e == null)
            throw new NullPointerException();
        // 修改版本+1
        modCount++;
        // 记录当前队列中元素的个数
        int i = size ;
        // 如果当前元素个数大于等于队列底层数组的长度，则进行扩容
        if (i >= queue .length)
            grow(i + 1);
        // 元素个数+1
        size = i + 1;
        // 如果队列中没有元素，则将元素e直接添加至根（数组小标0的位置）
        if (i == 0)
            queue[0] = e;
        // 否则调用siftUp方法，将元素添加到尾部，进行上移判断
        else
            siftUp(i, e);
        return true;
    }
```

下面是扩容的具体实现：

```java
/**
     * 数组扩容
     */
    private void grow(int minCapacity) {
        // 如果最小需要的容量大小minCapacity小于0，则说明此时已经超出int的范围，则抛出OutOfMemoryError异常
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        // 记录当前队列的长度
        int oldCapacity = queue .length;
        // Double size if small; else grow by 50%
        // 如果当前队列长度小于64则扩容2倍，否则扩容1.5倍
        int newCapacity = ((oldCapacity < 64)?
                           ((oldCapacity + 1) * 2):
                           ((oldCapacity / 2) * 3));
        // 如果扩容后newCapacity超出int的范围，则将newCapacity赋值为Integer.Max_VALUE
        if (newCapacity < 0) // overflow
            newCapacity = Integer. MAX_VALUE;
        // 如果扩容后，newCapacity小于最小需要的容量大小minCapacity，则按找minCapacity长度进行扩容
        if (newCapacity < minCapacity)
            newCapacity = minCapacity;
        // 数组copy，进行扩容
        queue = Arrays.copyOf( queue, newCapacity);
    }
```

下面看二叉堆的 *上移* 是如何实现的

```java
	/**
     * 上移，x表示新插入元素，k表示新插入元素在数组的位置
     */
    private void siftUp(int k, E x) {
        // 如果比较器comparator不为空，则调用siftUpUsingComparator方法进行上移操作
        if (comparator != null)
            siftUpUsingComparator(k, x);
        // 如果比较器comparator为空，则调用siftUpComparable方法进行上移操作
        else
            siftUpComparable(k, x);
    }

    private void siftUpComparable(int k, E x) {
        // 比较器comparator为空，需要插入的元素实现Comparable接口，用于比较大小
        Comparable<? super E> key = (Comparable<? super E>) x;
        // k>0表示判断k不是根的情况下，也就是元素x有父节点
        while (k > 0) {
            // 计算元素x的父节点位置[(n-1)/2],此处使用 无符号左移来 代替 /2 
            int parent = (k - 1) >>> 1;
            // 取出x的父亲e
            Object e = queue[parent];
            // 如果新增的元素k比其父亲e大，则不需要"上移"，跳出循环结束
            if (key.compareTo((E) e) >= 0)
                break;
            // x比父亲小，则需要进行"上移"
            // 交换元素x和父亲e的位置
            queue[k] = e;
            // 将新插入元素的位置k指向父亲的位置，进行下一层循环
            k = parent;
        }
        // 找到新增元素x的合适位置k之后进行赋值
        queue[k] = key;
    }

    // 这个方法和上面的操作一样，不多说了
    private void siftUpUsingComparator(int k, E x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (comparator .compare(x, (E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }
```

二叉堆添加元素的过程是，现将 元素插到数组末尾，然后不断与其父节点比较，如果符合堆的规则，就退出循环，如果不符合，就交换，继续向上查找，整个过程就是 自下向上调整堆。

#### 二叉堆的删除根原理及PriorityQueue 的出队实现

对于二叉堆的出队操作，**出队永远是要删除根元素**，也就是最下的元素，要删除根元素，就要找一个替代者移动到根位置，相对于被删除的元素来说就是"下移"。

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231344272-1677448008.jpg)

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231358272-1805184581.jpg)



![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231406772-172480369.jpg)

二叉堆的出队过程：

1. 将找出队尾的元素8，并将它在队尾位置上删除（图2）;

2. 此时队尾元素8比根元素1的**最小孩子**3要大，所以将元素1下移，交换1和3的位置（图3）；

3. 然后此时队尾元素8比元素1的最小孩子4要大，继续将1下移，交换1和4的位置（图4）；

4. 然后此时根元素8比元素1的最小孩子9要小，不需要下移，直接将根元素8赋值给此时元素1的位置，**1被覆盖则相当于删除（图5）**，结束。

*PriorityQueue*出队实现：

```java
/**
     * 删除并返回队头的元素，如果队列为空则抛出NoSuchElementException异常（该方法在AbstractQueue中）
     */
    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    /**
     * 删除并返回队头的元素，如果队列为空则返回null
     */
   public E poll() {
        // 队列为空，返回null
        if (size == 0)
            return null;
        // 队列元素个数-1
        int s = --size ;
        // 修改版本+1
        modCount++;
        // 队头的元素
        E result = (E) queue[0];
        // 队尾的元素
        E x = (E) queue[s];
        // 先将队尾赋值为null
        queue[s] = null;
        // 如果队列中不止队尾一个元素，则调用siftDown方法进行"下移"操作
        if (s != 0)
            siftDown(0, x);
        return result;
    }

    /**
     * 上移，x表示队尾的元素，k表示被删除元素在数组的位置
     */
    private void siftDown(int k, E x) {
        // 如果比较器comparator不为空，则调用siftDownUsingComparator方法进行下移操作
        if (comparator != null)
            siftDownUsingComparator(k, x);
        // 比较器comparator为空，则调用siftDownComparable方法进行下移操作
        else
            siftDownComparable(k, x);
    }

    private void siftDownComparable(int k, E x) {
        // 比较器comparator为空，需要插入的元素实现Comparable接口，用于比较大小
        Comparable<? super E> key = (Comparable<? super E>)x;
        // 通过size/2找到一个没有叶子节点的元素,小于 half 的节点都不是叶子节点
        int half = size >>> 1;        // loop while a non-leaf
        // 比较位置k和half，如果k小于half，则k位置的元素就不是叶子节点
        while (k < half) {
             // 找到根元素的左孩子的位置[2n+1]
            int child = (k << 1) + 1; // assume left child is least
             // 左孩子的元素
            Object c = queue[child];
             // 找到根元素的右孩子的位置[2(n+1)]
            int right = child + 1;
            // 如果左孩子大于右孩子，则将c复制为右孩子的值，这里也就是找出左右孩子哪个最小
            if (right < size &&
                ((Comparable<? super E>) c).compareTo((E) queue [right]) > 0)
                c = queue[child = right];
            // 如果队尾元素比根元素孩子都要小，则不需"下移"，结束
            if (key.compareTo((E) c) <= 0)
                break; 
            // 队尾元素比根元素孩子都大，则需要"下移"
            // 交换跟元素和孩子c的位置
            queue[k] = c;
            // 将根元素位置k指向最小孩子的位置，进入下层循环
            k = child;
        }
        // 找到队尾元素x的合适位置k之后进行赋值
        queue[k] = key;
    }

    // 这个方法和上面的操作一样，不多说了
    private void siftDownUsingComparator(int k, E x) {
        int half = size >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((E) c, (E) queue [right]) > 0)
                c = queue[child = right];
            if (comparator .compare(x, (E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = x;
    }
```

在 删除根元素时，不是直接将根元素删除，而是找出队尾的元素，并在队尾的位置上删除，然后通过根元素的下移，给队尾元素找到一个合适的位置，最终覆盖掉根元素，从而达到删除根元素的目的。

#### 堆的构造过程

  假设有一个无序的数组，要求我们将这个数组建成一个二叉堆，你会怎么做呢？最简单的办法当然是将数组的数据一个个取出来，调用入队方法。但是这样做，每次入队都有可能会伴随着元素的移动，这么做是十分低效的。那么有没有更加高效的方法呢，我们来看下。

 为了方便，我们将上面我们图解中的数组去掉几个元素，只留下7、6、5、12、10、3、1、11、15、4（顺序已经随机打乱）。ok、那么接下来，我们就按照当前的顺序建立一个二叉堆，暂时不用管它是否符合标准。 

​     int a = [7, 6, 5, 12, 10, 3, 1, 11, 15, 4 ];



![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231554069-320425957.jpg)

　我们观察下用数组a建成的二叉堆，很明显，对于叶子节点4、15、11、1、3来说，它们已经是一个合法的堆。所以只要最后一个节点的父节点，也就是最后一个非叶子节点a[4]=10开始调整，然后依次调整a[3]=12，a[2]=5，a[1]=6，a[0]=7，分别对这几个节点做一次"下移"操作就可以完成了堆的构造。ok，我们还是用图解来分析下这个过程。

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231621710-356715415.jpg)



![](C:\Users\Administrator\Documents\笔记\img\681047-20160112231647428-1137275577.jpg)

![](C:\Users\Administrator\Documents\笔记\img\681047-20160112232539069-543299823.jpg)

我们参照图解分别来解释下这几个步奏：

1. 对于节点a[4]=10的调整（图1），只需要交换元素10和其子节点4的位置（图2）。

2. 对于节点a[3]=12的调整，只需要交换元素12和其最小子节点11的位置（图3）。

3. 对于节点a[2]=5的调整，只需要交换元素5和其最小子节点1的位置（图4）。

4. 对于节点a[1]=6的调整，只需要交换元素6和其最小子节点4的位置（图5）。

5. 对于节点a[0]=7的调整，只需要交换元素7和其最小子节点1的位置，然后交换7和其最小自己点3的位置（图6）。

*PriorityQueue*  的建堆代码:

```java
private void heapify() {
        for (int i = (size >>> 1) - 1; i >= 0; i--)
            siftDown(i, (E) queue[i]);
    }
```

**int** i = (size >>> 1) - 1，这行代码是为了找寻最后一个非叶子节点，然后倒序进行"下移"siftDown操作，是不是很显然了。