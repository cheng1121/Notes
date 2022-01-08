### Java集合类
Java集合大致可以分为Set、List、Queue和Map四种体系
其中Set代表无序、不可重复的集合；List代表有序、重复的集合；Map代表具有映射关系的集合；Queue体系集合，代表一种队列集合实现

Java集合就像一种容器，可以把多个对象(实际上是对象的引用)放入该容器中。从Java5增加了泛型以后，Java集合就可以记住容器中对象的数据类型，使得编码更加简洁、健壮

##### Java集合和数组的区别
1. 数组长度在初始化时指定，意味着只能保存定长的数据，而集合可以保存数量不确定的数据。同时可以保存具有映射关系的数据（即关联数组，键值对key-value）
2. 数组元素即可以是基本数据类型的值，也可以是对象。集合里只能保存对象(实际上只是保存对象的引用变量)，基本数据类型的变量要转换成对应的包装类才能放入集合类中

##### Java集合类之间的继承关系
![image.png](https://upload-images.jianshu.io/upload_images/11142016-993487761cea40b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中，ArrayList,HashSet,LinkedList,TreeSet是我们经常会用到的已实现的集合类

Map实现类用于保存具有映射关系的数据。Map保存的每项数据都是key-value对。key是不可重复的，用于标识集合里的每项数据
![image.png](https://upload-images.jianshu.io/upload_images/11142016-26727622f8b956ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图中，HashMap，TreeMap是我们经常会用到的集合类

##### 使用Iterator遍历集合
```
        List<Integer> list = createList(10);
        Iterator<Integer> iterator = list.iterator();

        while (iterator.hasNext()){

            int i = iterator.next();
            i = 10;
        }
```

##### Set集合
Set集合不允许包含相同的元素，如果试图把两个相同的元素加入同一个Set集合中，则添加操作失败，add()方法返回false，且新元素不回被加入

##### List集合
List集合代表一个元素有序、可重复的集合，集合中每个元素都有其对应的顺序索引。List集合允许使用重复元素，可以通过索引来访问指定位置的重复元素。索引值为元素的添加顺序

##### Queue集合
队列，是指先进先出(FIFO,first in first out)的容器，队列的头部是在队列中存放时间最长的元素，队列的尾部是保存在队列中存放时间最短的元素。新元素插入(offer)到队列的尾部，访问元素(poll)会返回队列头部的元素。通常队列不允许随机访问

##### Map集合
保存具有映射关系的数据，以键值对形式(key-value)保存。二者可以是任何引用类型的数据。Key不允许重复，即同一个Map对象的任何两个Key通过equals方法比较总是返回false
   
1. Map集合与Set集合的关系，如果把Map里边所有key放在一起看，它们就组成了一个Set集合，通过keySet()方法返回一个由所有key组成的Set集合
2. Map集合与List集合的关系，把所有value放在一起看，它们又非常类似一个List：元素与元素之间可以重复，每个元素可以根据索引来查找，只是Map中索引不再使用整数值，而是以另外一个对象作为索引


##### ArrayList
是一种相对来说比较简单的数据，可以自动扩容，内部是由数组实现的，也可以认为是“动态数组”
特点：
1. 以数组实现，默认大小为10，如果超出则增加50%容量，如果还不足，则大小设置为需求值
2. 具有数组的优点：随机访问元素性能很高，在数组末尾添加元素性能也很高
3. 具有数组的缺点：非末尾添加元素，修改，移除某一元素，的性能很差。需要使用System.arrayCopy()方法来移动部分受影响的元素

##### LinkedList
基于双向链表实现，容量无限制，但双向链表本身使用了更多空间，也需要额外的链表指针操作

1. 具有链表的优点：删除、添加的性能很高
2.具有链表的确定：不能随机访问，只能顺序访问，所以读取的性能很差，LinkedList做了优化，如果要访问的元素的下表大于链表所有元素的一半则从末尾开始移起。所以时间复杂度从O(n)变为O(n/2)

##### HashMap
底层基于散列表来实现，基于Map接口实现、允许null键值、非同步、不保证有序、也不保证顺序不随时间变化
HashMap有两个重要参数：容量(Capacity)和负载因子(Load factor)。容量是HashMap内部的数组长度。负载因子是决定数组是否扩容的最大比例；当前已存数据数量与总容量的比值 > 负载因子 那么数组就需要扩容，反之不需要

 - put函数的实现:
 1. 对key的hashCode()做hash，然后再计算index (该计算过程散列函数的计算过程)
2. 如果没有碰撞(冲突),直接放到桶(bucket,散列表中的数组)内
3. 如果碰撞了，以链表形式存在buckets中
4. 如果碰撞导致链表过长，就把链表转为红黑树(红黑树暂时还不懂)
5. 如果节点已存在，就替换对应的value(保证key的唯一性)
6. 如果bucket满了(当前已存数据数量与总容量的比值 > 负载因子)，就需要resize(扩容)
```
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, I;
       //tab为空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //计算index 并对null处理，为null直接存key对应的value
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //index对应节点已存在，则直接给e赋值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果index对应节点为树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
             //如果index对应节点为链表
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
           //如果e不为空，表示存在key对应的映射
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
         //判断是否需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

- get函数的实现：
1. 没有冲突，直接返回对应的value
2. 有冲突：
       (1). 是树，则在树中通过key.equals(k)查找，时间复杂度O(log n)
       (2). 是链表，则在链表中通过key.equals(k)查找，时间复杂度O(n)
```
   public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

  final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //检查第一个元素直接命中
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
             //未命中
            if ((e = first.next) != null) {
               //如果为树
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //在链表中查找
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

##### LinkedHashMap
是Hash表(散列表)和链表的实现，并且依靠双向链表保证了迭代顺序是插入的顺序。继承于HashMap，重新实现了afterNodeAccess、afterNodeInsertion、afterNodeRemoval三个函数，都是为了保证双向链表中的节点次序或者双向链表容量所做的一些额外的事情，目的就是保持双向链表中节点的顺序要从eldest到youngest,其他的操作也基本上是为了维护好具有访问顺序的双向链表