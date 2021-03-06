# 滑动窗口的最大值

## 题目：给定一个数组和滑动窗口的大小，请找出所有滑动窗口里的最大值。

### 举例说明

例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小，那么一共存在 6 个滑动窗口，它们的最大值分别为{4,4,6,6,6,5}。

### 解题思路

如果采用蛮力法，这个问题似乎不难解决：可以扫描每一个滑动窗口的所有数字并找出其中的最大值。如果滑动窗口的大小为 k，需要 O(k)时间才能找出滑动窗口里的最大值。对于长度为 n 的输入数组，这个算法总的时间复杂度是 O(nk)。 

实际上一个滑动窗口可以看成是一个队列。当窗口滑动时，处于窗口的第一个数字被删除，同时在窗口的末尾添加一个新的数字。这符合队列的先进先出特性。如果能从队列中找出它的最大数，这个问题也就解决了。 

在面试题 21 中。我们实现了一个可以用 O(1)时间得到最小值的栈。同样，也可以用 O(1)时间得到栈的最大值。同时在面试题 7 中，我们讨论了如何用两个栈实现一个队列。综合这两个问题的解决方法，我们发现如果把队列用两个栈实现，由于可以用 O(1)时间得到栈中的最大值，那么也就可以用 O(1)时间得到队列的最大值，因此总的时间复杂度也就降到了 O(n)。 

我们可以用这个方法来解决问题。不过这样就相当于在一轮面试的时间内要做两个面试题，时间未必够用。再来看看有没有其它的方法。 

下面换一种思路。我们并不把滑动窗口的每个数值都存入队列中，而只把有可能成为滑动窗口最大值的数值存入到一个两端开口的队列。接着以输入数字{2,3,4,2,6,2,5,1}为例一步分析。
 
数组的第一个数字是 2，把它存入队列中。第二个数字是3.由于它比前一个数字 2 大，因此 2不可能成为滑动窗口中的最大值。2 先从队列里删除，再把3存入到队列中。此时队列中只有一个数字 3。针对第三个数字 4 的步骤类似，最终在队列中只剩下一个数字 4。此时滑动窗口中已经有 3 个数字，而它的最大值 4 位于队列的头部。 

接下来处理第四个数字 2。2 比队列中的数字 4 小。当 4 滑出窗口之后 2 还是有可能成为滑动窗口的最大值，因此把 2 存入队列的尾部。现在队列中有两个数字 4 和 2，其中最大值 4 仍然位于队列的头部。 

下一个数字是 6。由于它比队列中已有的数字 4 和 2 都大，因此这时 4 和 2 已经不可能成为滑动窗口中的最大值。先把 4 和 2 从队列中删除，再把数字 6 存入队列。这个时候最大值 6 仍然位于队列的头部。 

第六个数字是 2。由于它比队列中已有的数字 6 小，所以 2 也存入队列的尾部。此时队列中有两个数字，其中最大值 6 位于队列的头部。 

接下来的数字是 5。在队列中已有的两个数字 6 和 2 里，2 小于 5，因此 2 不可能是一个滑动窗口的最大值，可以把它从队列的尾部删除。删除数字 2 之后，再把数字 5 存入队列。此时队列里剩下两个数字 6 和 5，其中位于队列头部的是最大值 6。

数组最后一个数字是 1，把 1 存入队列的尾部。注意到位于队列头部的数字 6 是数组的第 5 个数字，此时的滑动窗口已经不包括这个数字了，因此应该把数字 6 从队列删除。那么怎么知道滑动窗口是否包括一个数字？应该在队列里存入数字在数组里的下标，而不是数值。当一个数字的下标与当前处理的数字的下标之差大于或者等于滑动窗口的大小时，这个数字已经从滑动窗口中滑出，可以从队列中删除了。

### 代码实现

```
import java.util.*;
public class Test65 {
    private static List<Integer> maxInWindows(List<Integer> data, int size) {
        List<Integer> windowMax = new LinkedList<>();
        // 条件检查
        if (data == null || size < 1 || data.size() < 1) {
            return windowMax;
        }
        Deque<Integer> idx = new LinkedList<>();
        // 窗口还没有被填满时，找最大值的索引
        for (int i = 0; i < size && i < data.size(); i++) {
            // 如果索引对应的值比之前存储的索引值对应的值大或者相等，就删除之前存储的值
            while (!idx.isEmpty() && data.get(i) >= data.get(idx.getLast())) {
                idx.removeLast();
            }
            //  添加索引
            idx.addLast(i);
        }
        // 窗口已经被填满了
        for (int i = size; i < data.size(); i++) {
            // 第一个窗口的最大值保存
            windowMax.add(data.get(idx.getFirst()));
            // 如果索引对应的值比之前存储的索引值对应的值大或者相等，就删除之前存储的值
            while (!idx.isEmpty() && data.get(i) >= data.get(idx.getLast())) {
                idx.removeLast();
            }
            // 删除已经滑出窗口的数据对应的下标
            if (!idx.isEmpty() && idx.getFirst() <= (i - size)) {
                idx.removeFirst();
            }
            // 可能的最大的下标索引入队
            idx.addLast(i);
        }
        // 最后一个窗口最大值入队
        windowMax.add(data.get(idx.getFirst()));
        return windowMax;
    }
    private static List<Integer> arrayToCollection(int[] array) {
        List<Integer> result = new LinkedList<>();
        if (array != null) {
            for (int i : array) {
                result.add(i);
            }
        }
        return result;
    }
    public static void main(String[] args) {
        // expected {7};
        List<Integer> data1 = arrayToCollection(new int[]{1, 3, -1, -3, 5, 3, 6, 7});
        System.out.println(data1 + "," + maxInWindows(data1, 10));
        // expected {3, 3, 5, 5, 6, 7};
        List<Integer> data2 = arrayToCollection(new int[]{1, 3, -1, -3, 5, 3, 6, 7});
        System.out.println(data2 + "," + maxInWindows(data2, 3));
        // expected {7, 9, 11, 13, 15};
        List<Integer> data3 = arrayToCollection(new int[]{1, 3, 5, 7, 9, 11, 13, 15});
        System.out.println(data3 + "," + maxInWindows(data3, 4));
        // expected  {16, 14, 12};
        List<Integer> data5 = arrayToCollection(new int[]{16, 14, 12, 10, 8, 6, 4});
        System.out.println(data5 + "," + maxInWindows(data5, 5));
        // expected  {10, 14, 12, 11};
        List<Integer> data6 = arrayToCollection(new int[]{10, 14, 12, 11});
        System.out.println(data6 + "," + maxInWindows(data6, 1));
        // expected  {14};
        List<Integer> data7 = arrayToCollection(new int[]{10, 14, 12, 11});
        System.out.println(data7 + "," + maxInWindows(data7, 4));
    }
}
```

### 运行结果

![](images/83.png)