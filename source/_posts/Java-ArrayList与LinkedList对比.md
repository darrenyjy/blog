---
title: Java ArrayList与LinkedList对比
date: 2016-05-28 15:35:04
tags:
- 面试
categories:
- Java

---

## ArrayList
 * 底层是个默认大小为10的数组，所以当size需求大于10时，初始化size的时候大一点。
 * 当ArrayList的写满之后，ArrayList会生成一个更大的数组，大小变为原来的1.5倍。然后将原来数组的数据拷贝过去，因此会占用一定的时间和内存。
 ``` java
  public void ensureCapacity(int var1) {
        ++this.modCount;
        int var2 = this.elementData.length;
        if(var1 > var2) {
            Object[] var3 = this.elementData;
            int var4 = var2 * 3 / 2 + 1;
            if(var4 < var1) {
                var4 = var1;
            }
     this.elementData = Arrays.copyOf(this.elementData, var4);
        }

    }
 ```
 
 * 适合随机访问
 
## LinkedList
 * 底层是List,适用于频繁添加删除元素的情况
 