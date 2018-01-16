---
layout: post
title: "ArrayList集合理解"
date: 2018-01-16
description: "java集合， ArrayList集合"
tag: java学习
---

### 创建ArrayList对象过程

    List<String> list = new ArrayList<String>();
在这个过程中，java源码的过程：


    // ArrayList内部初始化一个数组，这里使用缓存，每次创建对象都会指向这个数组对象
    // transient关键字意味着elementData将不会序列化，那么ArrayList又将如何序列化？
    transient Object[] elementData;

    // 空list
    private static final Object[] EMPTY_ELEMENTDATA = {};

      public ArrayList()
      {
        this.elementData = EMPTY_ELEMENTDATA;
      }
 调用构造函数，创建一个空的数组。

 当集合对象调用add()方法时：

        public boolean add(E paramE)
      {
        ensureCapacityInternal(this.size + 1);
        this.elementData[(this.size++)] = paramE;
        return true;
      }
这个时候，集合会初始化底层数组的大小：

    private void ensureCapacityInternal(int paramInt)
      {
        // paramInt为：1
        // 判断创建的对象和缓存的对象是否是一样的，是的话，初始化数组的长度，也就是ArrayList()对象的初始大小。
        if (this.elementData == EMPTY_ELEMENTDATA) {
          paramInt = Math.max(10, paramInt);
        }
        ensureExplicitCapacity(paramInt);
      }

      private void ensureExplicitCapacity(int paramInt)
      {
        this.modCount += 1;
        //超出了数组可容纳的长度，需要进行动态扩展
        if (paramInt - this.elementData.length > 0) {
          grow(paramInt);
        }
      }

          //这才是动态扩展的精髓，看到这个方法，ArrayList瞬间被打回原形
     79     private void grow(int minCapacity) {
     80         int oldCapacity = elementData.length;
     81         //设置新数组的容量扩展为原来数组的1.5倍
     82         int newCapacity = oldCapacity + (oldCapacity >> 1);
     83         //再判断一下新数组的容量够不够，够了就直接使用这个长度创建新数组，
     84         //不够就将数组长度设置为需要的长度
     85         if (newCapacity - minCapacity < 0)
     86             newCapacity = minCapacity;
     87         //判断有没超过最大限制
     88         if (newCapacity - MAX_ARRAY_SIZE > 0)
     89             newCapacity = hugeCapacity(minCapacity);
     90         //将原来数组的值copy新数组中去， ArrayList的引用指向新数组
     91         //这儿会新创建数组，如果数据量很大，重复的创建的数组，那么还是会影响效率，
     92         //因此鼓励在合适的时候通过构造方法指定默认的capaticy大小
     93         elementData = Arrays.copyOf(elementData, newCapacity);
     94     }


    public static <T> T[] copyOf(T[] paramArrayOfT, int paramInt)
      {
        return (Object[])copyOf(paramArrayOfT, paramInt, paramArrayOfT.getClass());
      }


      // 最后调用底层方法，返回一个固定大小得数组
      public static <T, U> T[] copyOf(U[] paramArrayOfU, int paramInt, Class<? extends T[]> paramClass)
      {
        Object[] arrayOfObject = paramClass == [Ljava.lang.Object.class ? (Object[])new Object[paramInt] : (Object[])Array.newInstance(paramClass.getComponentType(), paramInt);
        System.arraycopy(paramArrayOfU, 0, arrayOfObject, 0, Math.min(paramArrayOfU.length, paramInt));
        return arrayOfObject;
      }

到这里，AarryList()对象的初始化就完成了。**初始化长度为：10， 如果超出初始化大小会自动进行扩容，原来的1.5倍**

### 总结

1、ArrayList内部如何实现？适合什么样的操作场景？

数组实现，适合随机存取、不适合非尾部的增删操作。

2、new ArrayList<>()方法调用后所提供的ArrayList容量是多大？

0

3、未提供容量值，但是调用add方法后ArrayList容量值是多大？

至少是10，或者实际需要值（大于10）

4、ArrayList什么时候扩容？如何扩容？扩多大？

当前需要的容量要比当前ArrayList的capacity大时进行扩容；扩容的操作是重新分配数组；至少会扩容 1/2 oldCapacity（向下取整），如果newCapacity小于最少需要的容量minCapacity，那将扩大至最少需要容量。

5、ArrayList是否线程安全？

不是，没有任何synchronized方法。

6、ArrayList如何序列化？

通过readObject和writeObject，详见集合序列化

7、ArrayList最大容量是多大？

Integer.MAX_VALUE - 8，部分虚拟机在数组中预留了8位存储头部信息。

