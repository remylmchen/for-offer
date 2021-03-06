# 数据流中的中位数

## 题目：如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有值排序之后位于中间的数值。如果数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

### 解题思路

由于数据是从一个数据流中读出来的，数据的数目随着时间的变化而增加。如果用一个数据容器来保存从流中读出来的数据，当有新的数据流中读出来时，这些数据就插入到数据容器中。这个数据容器用什么数据结构定义更合适呢？ 

数组是最简单的容器。如果数组没有排序，可以用 Partition 函数找出数组中的中位数。在没有排序的数组中插入一个数字和找出中位数的时间复杂度是 O(1)和 O(n)。
 
我们还可以往数组里插入新数据时让数组保持排序，这是由于可能要移动 O(n)个数，因此需要 O(n)时间才能完成插入操作。在已经排好序的数组中找出中位数是一个简单的操作，只需要 O(1)时间即可完成。 

排序的链表时另外一个选择。我们需要 O(n)时间才能在链表中找到合适的位置插入新的数据。如果定义两个指针指向链表的中间结点（如果链表的结点数目是奇数，那么这两个指针指向同一个结点），那么可以在 O（1）时间得出中位数。此时时间效率与及基于排序的数组的时间效率一样。 

二叉搜索树可以把插入新数据的平均时间降低到 O(logn)。但是，当二叉搜索树极度不平衡从而看起来像一个排序的链表时，插入新数据的时间仍然是 O(n)。为了得到中位数，可以在二叉树结点中添加一个表示子树结点数目的字段。有了这个字段，可以在平均 O(logn)时间得到中位数，但差情况仍然是 O(n)。

为了避免二叉搜索树的最差情况，还可以利用平衡的二叉搜索树，即 AVL 树。通常 AVL 树的平衡因子是左右子树的高度差。可以稍作修改，把 AVL 的平衡因子改为左右子树结点数目只差。有了这个改动，可以用 O(logn)时间往 AVL 树中添加一个新结点，同时用 O(1)时间得到所有结点的中位数。
 
AVL 树的时间效率很高，但大部分编程语言的函数库中都没有实现这个数据结构。应聘者在短短几十分钟内实现 AVL 的插入操作是非常困难的。于是我们不得不再分析还有没有其它的方法。 

如果能够保证数据容器左边的数据都小于右边的数据，这样即使左、右两边内部的数据没有排序，也可以根据左边最大的数及右边最小的数得到中位数。如何快速从一个容器中找出最大数？用最大堆实现这个数据容器，因为位于堆顶的就是最大的数据。同样，也可以快速从最小堆中找出最小数。　　因此可以用如下思路来解决这个问题：用一个最大堆实现左边的数据容器，用最小堆实现右边的数据容器。往堆中插入一个数据的时间效率是 O(logn)。由于只需 O(1)时间就可以得到位于堆顶的数据，因此得到中位数的时间效率是 O(1)。

接下来考虑用最大堆和最小堆实现的一些细节。首先要保证数据平均分配到两个堆中，因此两个堆中数据的数目之差不能超过 1（为了实现平均分配，可以在数据的总数目是偶数时把新数据插入到最小堆中，否则插入到最大堆中）。 

还要保证最大堆中里的所有数据都要小于最小堆中的数据。当数据的总数目是偶数时，按照前面分配的规则会把新的数据插入到最小堆中。如果此时新的数据比最大堆中的一些数据要小，怎么办呢？ 

可以先把新的数据插入到最大堆中，接着把最大堆中的最大的数字拿出来插入到最小堆中。由于最终插入到最小堆的数字是原最大堆中最大的数字，这样就保证了最小堆中的所有数字都大于最大堆中的数字。 
当需要把一个数据插入到最大堆中，但这个数据小于最小堆里的一些数据时，这个情形和前面类似。

### 辅助类实现

#### 堆实现

```
private static class Heap<T> {
    // 堆中元素存放的集合
    private List<T> data;
    // 比较器
    private Comparator<T> cmp;
    /**
     * 构造函数
     *
     * @param cmp 比较器对象
     */
    public Heap(Comparator<T> cmp) {
        this.cmp = cmp;
        this.data = new ArrayList<>(64);
    }
    /**
     * 向上调整堆
     *
     * @param idx 被上移元素的起始位置
     */
    public void shiftUp(int idx) {
        // 检查是位置是否正确
        if (idx < 0 || idx >= data.size()) {
            throw new IllegalArgumentException(idx + "");
        }
        // 获取开始调整的元素对象
        T intent = data.get(idx);
        // 如果不是根元素，则需要上移
        while (idx > 0) {
            // 找父元素对象的位置
            int parentIdx = (idx - 1) / 2;
            // 获取父元素对象
            T parent = data.get(parentIdx);
            //上移的条件，子节点比父节点大，此处定义的大是以比较器返回值为准
            if (cmp.compare(intent, parent) > 0) {
                // 将父节点向下放
                data.set(idx, parent);
                idx = parentIdx;
                // 记录父节点下放的位置
            }
            // 子节点不比父节点大，说明父子路径已经按从大到小排好顺序了，不需要调整了
            else {
                break;
            }
        }
        // index此时记录是的最后一个被下放的父节点的位置（也可能是自身），
        // 所以将最开始的调整的元素值放入index位置即可
        data.set(idx, intent);
    }
    /**
     * 向下调整堆
     *
     * @param idx 被下移的元素的起始位置
     */
    public void shiftDown(int idx) {
        // 检查是位置是否正确
        if (idx < 0 || idx >= data.size()) {
            throw new IllegalArgumentException(idx + "");
        }
        // 获取开始调整的元素对象
        T intent = data.get(idx);
        // 获取开始调整的元素对象的左子结点的元素位置
        int leftIdx = idx * 2 + 1;
        // 如果有左子结点
        while (leftIdx < data.size()) {
            // 取左子结点的元素对象，并且假定其为两个子结点中最大的
            T maxChild = data.get(leftIdx);
            // 两个子节点中最大节点元素的位置，假定开始时为左子结点的位置
            int maxIdx = leftIdx;
            // 获取右子结点的位置
            int rightIdx = leftIdx + 1;
            // 如果有右子结点
            if (rightIdx < data.size()) {
                T rightChild = data.get(rightIdx);
                // 找出两个子节点中的最大子结点
                if (cmp.compare(rightChild, maxChild) > 0) {
                    maxChild = rightChild;
                    maxIdx = rightIdx;
                }
            }
            // 如果最大子节点比父节点大，则需要向下调整
            if (cmp.compare(maxChild, intent) > 0) {
                // 将较大的子节点向上移
                data.set(idx, maxChild);
                // 记录上移节点的位置
                idx = maxIdx;
                // 找到上移节点的左子节点的位置
                leftIdx = 2 * idx + 1;
            }
            // 最大子节点不比父节点大，说明父子路径已经按从大到小排好顺序了，不需要调整了
            else {
                break;
            }
        }
        // index此时记录是的最后一个被上移的子节点的位置（也可能是自身），
        // 所以将最开始的调整的元素值放入index位置即可
        data.set(idx, intent);
    }
    /**
     * 添加一个元素
     *
     * @param item 添加的元素
     */
    public void add(T item) {
        // 将元素添加到最后
        data.add(item);
        // 上移，以完成重构
        shiftUp(data.size() - 1);
    }
    /**
     * 删除堆顶结点
     *
     * @return 堆顶结点
     */
    public T deleteTop() {
        // 如果堆已经为空，就抛出异常
        if (data.isEmpty()) {
            throw new RuntimeException("The heap is empty.");
        }
        // 获取堆顶元素
        T first = data.get(0);
        // 删除最后一个元素
        T last = data.remove(data.size() - 1);
        // 删除元素后，如果堆为空的情况，说明删除的元素也是堆顶元素
        if (data.size() == 0) {
            return last;
        } else {
            // 将删除的元素放入堆顶
            data.set(0, last);
            // 自上向下调整堆
            shiftDown(0);
            // 返回堆顶元素
            return first;
        }
    }
    /**
     * 获取堆顶元素，但不删除
     *
     * @return 堆顶元素
     */
    public T getTop() {
        // 如果堆已经为空，就抛出异常
        if (data.isEmpty()) {
            throw new RuntimeException("The heap is empty.");
        }
        return data.get(0);
    }
    /**
     * 获取堆的大小
     *
     * @return 堆的大小
     */
    public int size() {
        return data.size();
    }
    /**
     * 判断堆是否为空
     *
     * @return 堆是否为空
     */
    public boolean isEmpty() {
        return data.isEmpty();
    }
    /**
     * 清空堆
     */
    public void clear() {
        data.clear();
    }
    /**
     * 获取堆中所有的数据
     *
     * @return 堆中所在的数据
     */
    public List<T> getData() {
        return data;
    }
}
```

### 升序比较器

```
/**
 * 升序比较器
 */
private static class IncComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1 - o2;
    }
}
```

### 降序比较器

```
/**
 * 降序比较器
 */
private static class DescComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o2 - o1;
    }
}
```

####代码实现

```
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
public class Test64 {
    private static class Heap<T> {
        // 堆中元素存放的集合
        private List<T> data;
        // 比较器
        private Comparator<T> cmp;
        /**
         * 构造函数
         *
         * @param cmp 比较器对象
         */
        public Heap(Comparator<T> cmp) {
            this.cmp = cmp;
            this.data = new ArrayList<>(64);
        }
        /**
         * 向上调整堆
         *
         * @param idx 被上移元素的起始位置
         */
        public void shiftUp(int idx) {
            // 检查是位置是否正确
            if (idx < 0 || idx >= data.size()) {
                throw new IllegalArgumentException(idx + "");
            }
            // 获取开始调整的元素对象
            T intent = data.get(idx);
            // 如果不是根元素，则需要上移
            while (idx > 0) {
                // 找父元素对象的位置
                int parentIdx = (idx - 1) / 2;
                // 获取父元素对象
                T parent = data.get(parentIdx);
                //上移的条件，子节点比父节点大，此处定义的大是以比较器返回值为准
                if (cmp.compare(intent, parent) > 0) {
                    // 将父节点向下放
                    data.set(idx, parent);
                    idx = parentIdx;
                    // 记录父节点下放的位置
                }
                // 子节点不比父节点大，说明父子路径已经按从大到小排好顺序了，不需要调整了
                else {
                    break;
                }
            }
            // index此时记录是的最后一个被下放的父节点的位置（也可能是自身），
            // 所以将最开始的调整的元素值放入index位置即可
            data.set(idx, intent);
        }
        /**
         * 向下调整堆
         *
         * @param idx 被下移的元素的起始位置
         */
        public void shiftDown(int idx) {
            // 检查是位置是否正确
            if (idx < 0 || idx >= data.size()) {
                throw new IllegalArgumentException(idx + "");
            }
            // 获取开始调整的元素对象
            T intent = data.get(idx);
            // 获取开始调整的元素对象的左子结点的元素位置
            int leftIdx = idx * 2 + 1;
            // 如果有左子结点
            while (leftIdx < data.size()) {
                // 取左子结点的元素对象，并且假定其为两个子结点中最大的
                T maxChild = data.get(leftIdx);
                // 两个子节点中最大节点元素的位置，假定开始时为左子结点的位置
                int maxIdx = leftIdx;
                // 获取右子结点的位置
                int rightIdx = leftIdx + 1;
                // 如果有右子结点
                if (rightIdx < data.size()) {
                    T rightChild = data.get(rightIdx);
                    // 找出两个子节点中的最大子结点
                    if (cmp.compare(rightChild, maxChild) > 0) {
                        maxChild = rightChild;
                        maxIdx = rightIdx;
                    }
                }
                // 如果最大子节点比父节点大，则需要向下调整
                if (cmp.compare(maxChild, intent) > 0) {
                    // 将较大的子节点向上移
                    data.set(idx, maxChild);
                    // 记录上移节点的位置
                    idx = maxIdx;
                    // 找到上移节点的左子节点的位置
                    leftIdx = 2 * idx + 1;
                }
                // 最大子节点不比父节点大，说明父子路径已经按从大到小排好顺序了，不需要调整了
                else {
                    break;
                }
            }
            // index此时记录是的最后一个被上移的子节点的位置（也可能是自身），
            // 所以将最开始的调整的元素值放入index位置即可
            data.set(idx, intent);
        }
        /**
         * 添加一个元素
         *
         * @param item 添加的元素
         */
        public void add(T item) {
            // 将元素添加到最后
            data.add(item);
            // 上移，以完成重构
            shiftUp(data.size() - 1);
        }
        /**
         * 删除堆顶结点
         *
         * @return 堆顶结点
         */
        public T deleteTop() {
            // 如果堆已经为空，就抛出异常
            if (data.isEmpty()) {
                throw new RuntimeException("The heap is empty.");
            }
            // 获取堆顶元素
            T first = data.get(0);
            // 删除最后一个元素
            T last = data.remove(data.size() - 1);
            // 删除元素后，如果堆为空的情况，说明删除的元素也是堆顶元素
            if (data.size() == 0) {
                return last;
            } else {
                // 将删除的元素放入堆顶
                data.set(0, last);
                // 自上向下调整堆
                shiftDown(0);
                // 返回堆顶元素
                return first;
            }
        }
        /**
         * 获取堆顶元素，但不删除
         *
         * @return 堆顶元素
         */
        public T getTop() {
            // 如果堆已经为空，就抛出异常
            if (data.isEmpty()) {
                throw new RuntimeException("The heap is empty.");
            }
            return data.get(0);
        }
        /**
         * 获取堆的大小
         *
         * @return 堆的大小
         */
        public int size() {
            return data.size();
        }
        /**
         * 判断堆是否为空
         *
         * @return 堆是否为空
         */
        public boolean isEmpty() {
            return data.isEmpty();
        }
        /**
         * 清空堆
         */
        public void clear() {
            data.clear();
        }
        /**
         * 获取堆中所有的数据
         *
         * @return 堆中所在的数据
         */
        public List<T> getData() {
            return data;
        }
    }
    /**
     * 升序比较器
     */
    private static class IncComparator implements Comparator<Integer> {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o1 - o2;
        }
    }
    /**
     * 降序比较器
     */
    private static class DescComparator implements Comparator<Integer> {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o2 - o1;
        }
    }
    private static class DynamicArray {
        private Heap<Integer> max;
        private Heap<Integer> min;
        public DynamicArray() {
            max = new Heap<>(new IncComparator());
            min = new Heap<>(new DescComparator());
        }
        /**
         * 插入数据
         *
         * @param num 待插入的数据
         */
        public void insert(Integer num) {
            // 已经有偶数个数据了（可能没有数据）
            // 数据总数是偶数个时把新数据插入到小堆中
            if ((min.size() + max.size()) % 2 == 0) {
                // 大堆中有数据，并且插入的元素比大堆中的元素小
                if (max.size() > 0 && num < max.getTop()) {
                    // 将num加入的大堆中去
                    max.add(num);
                    // 删除堆顶元素，大堆中的最大元素
                    num = max.deleteTop();
                }
                // num插入到小堆中，当num小于大堆中的最大值进，
                // num就会变成大堆中的最大值，见上面的if操作
                // 如果num不小于大堆中的最大值，num就是自身
                min.add(num);
            }
            // 数据总数是奇数个时把新数据插入到大堆中
            else {
                // 小堆中有数据，并且插入的元素比小堆中的元素大
                if (min.size() > 0 && num > min.size()) {
                    // 将num加入的小堆中去
                    min.add(num);
                    // 删除堆顶元素，小堆中的最小元素
                    num = min.deleteTop();
                }
                // num插入到大堆中，当num大于小堆中的最小值进，
                // num就会变成小堆中的最小值，见上面的if操作
                // 如果num不大于大堆中的最小值，num就是自身
                max.add(num);
            }
        }
        public double getMedian() {
            int size = max.size() + min.size();
            if (size == 0) {
                throw new RuntimeException("No numbers are available");
            }
            if ((size & 1) == 1) {
                return min.getTop();
            } else {
                return (max.getTop() + min.getTop()) / 2.0;
            }
        }
    }
    public static void main(String[] args) {
        DynamicArray array = new DynamicArray();
        array.insert(5);
        System.out.println(array.getMedian()); // 5
        array.insert(2);
        System.out.println(array.getMedian()); // 3.5
        array.insert(3);
        System.out.println(array.getMedian()); // 3
        array.insert(4);
        System.out.println(array.getMedian()); // 3.5
        array.insert(1);
        System.out.println(array.getMedian()); // 3
        array.insert(6);
        System.out.println(array.getMedian()); // 3.5
        array.insert(7);
        System.out.println(array.getMedian()); // 4
        array.insert(0);
        System.out.println(array.getMedian()); // 3.5
        array.insert(8);
        System.out.println(array.getMedian()); // 4
    }
}
```

### 运行结果

![](images/82.png)