# ArrayList 相关面试题以及底层源码分析

### ArrayList 和 LinkedList 异同

主要从结构，时间，和空间上去回答。

- `ArrayList` 底层使用 `Object` 数组；`LinkedList` 底层使用双向链表。
- 插入和删除：
    - 对于 `add(E e)` 方法，`ArrayList` 默认将元素添加在列表末尾，时间复杂度为 `O(1)`。但如果指定了添加位置 （`add(int index, E e)`），时间复杂度为 `O(n)`，因为此添加位置后的每个元素都要向后移一位。
    - 对于插入，删除，`LinkedList` 时间复杂度为 `O(1)`。如果在指定位置添加和删除，时间复杂度为 `O(n)`，因为需要先移动到指定位置再进行操作。
- `ArrayList` 支持快速访问 (底层是数组；继承了 `RandomAccess` 接口)；`LinkedList` 不支持。
- `ArrayList` 的空间浪费主要体现在列表的结尾会预留一定的容量空间；`LinkedList` 的空话花费主要体现在它的每一个元素需要消耗更多空间，因为要存放直接后继和直接前驱以及数据。
- 线程都不安全；

### ArrayList 扩容机制

以下用 JDK 11 源码分析。

```java
transient Object[] elementData;
private int size;
// 此列表已被结构修改的次数
protected transient int modCount = 0;
private static final int DEFAULT_CAPACITY = 10;

private int newCapacity(int minCapacity) {
    // overflow-conscious code
    // elementData 的 length 可能是大于存储的元素数量
    int oldCapacity = elementData.length;
    // newCapacity 是 oldCapacity 的 ~1.5 倍
    // old: 10 -> new: 15
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        // 第一次添加元素时，capacity 直接扩展为 10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}

private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                        newCapacity(minCapacity));
}

private Object[] grow() {
    return grow(size + 1);
}

private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}

public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);
    modCount++;
    final int s;
    Object[] elementData;
    if ((s = size) == (elementData = this.elementData).length)
        elementData = grow();
    System.arraycopy(elementData, index,
                        elementData, index + 1,
                        s - index);
    elementData[index] = element;
    size = s + 1;
}
```

__扩容过程__：

- 当 `add` 第 1 个元素时，`size == elementData.length` 为 `true`（都为 0），所以进入 `grow()`。`oldCapacity` 为 0，`newCapacity()` 返回 `DEFAULT_CAPACITY`，也就是 10。`elemengData.length` 变为 10，`size` 变为 1。
- 当 `add` 第 2 个元素时，`size = 2`，`elemengData.length = 10`, 所以 `size != elementData.length`，不会进入 `grow()`。所以 `elementData.length` 还是 10, `size` 变为 2。
- 当 `add` 第 11 个元素时，`size == elementData.length` 为 `true`（都为 10），所以进入 `grow()`。`oldCapacity` 为 10，`newCapacity` 被扩展成为 10 的 1.5 倍，也就是 15。所以 `elementData.length` 变为 15, `size` 变为 11。

### ensureCapacity 方法

```java
public void ensureCapacity(int minCapacity) {
    if (minCapacity > elementData.length
        && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
                && minCapacity <= DEFAULT_CAPACITY)) {
        modCount++;
        grow(minCapacity);
    }
}
```

`ensureCapacity(int minCapacity)` 可以确保 `elementData` 的长度至少为 `minCapacity`。

最好在 add 大量元素之前用 `ensureCapacity` 方法，以减少增量重新分配的次数。