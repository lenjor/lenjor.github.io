---
layout: post
title: 10种基本排序算法
tags: Algorithm
---
<!-- TOC -->

- [排序算法分类](#排序算法分类)
- [各排序算法复杂度分析](#各排序算法复杂度分析)
- [（一）冒泡排序（Bubble Sort）](#一冒泡排序bubble-sort)
    - [算法描述](#算法描述)
- [（二）快速排序(Quick Sort)](#二快速排序quick-sort)
    - [算法描述](#算法描述-1)
- [（三）、简单插入排序（Insertion Sort）](#三简单插入排序insertion-sort)
    - [算法描述](#算法描述-2)
- [（四）、希尔排序(Shell Sort)](#四希尔排序shell-sort)
    - [算法描述](#算法描述-3)
- [（五）、简单选择排序（Selection Sort）](#五简单选择排序selection-sort)
    - [算法描述](#算法描述-4)
- [（六）、堆排序(Heap Sort)](#六堆排序heap-sort)
- [（七）、归并排序(Merge Sort)](#七归并排序merge-sort)
- [算法描述](#算法描述-5)
- [（八）、计数排序（Counting Sort）](#八计数排序counting-sort)
    - [算法描述](#算法描述-6)
- [（九）、桶排序(Bucket sort)](#九桶排序bucket-sort)
- [（十）、基数排序（Radix Sort）](#十基数排序radix-sort)
    - [基数排序 vs 计数排序 vs 桶排序](#基数排序-vs-计数排序-vs-桶排序)
- [参考文章](#参考文章)

<!-- /TOC -->

# 排序算法分类

十种常见排序算法可以分为两大类：
- 比较类排序：**通过比较**来决定元素间的相对次序，由于其时间复杂度不能突破O(nlogn)，因此也称为**非线性时间比较类排序**。
- 非比较类排序：**不通过比较**来决定元素间的相对次序，它可以突破基于比较排序的时间下界，以线性时间运行，因此也称为**线性时间非比较类排序**。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-11.png)
# 各排序算法复杂度分析
![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-01.png)

# （一）冒泡排序（Bubble Sort）

冒泡排序是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果它们的顺序错误就把它们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。 

## 算法描述

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素做同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 重复步骤1~3，，直到没有任何一对数字需要比较。


![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-02.gif)

代码实现

``` java
    public static void bubbleSort(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = 0; j < arr.length - i - 1; j++) {
                if (arr[j] >= arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
    }
```

存在问题：数据的顺序排好之后，冒泡算法仍然会继续进行下一轮的比较，直到arr.length-1次，后面的比较没有意义的
优化方案：设置标志位flag，如果发生了交换flag设置为true；如果没有交换就设置为false。

``` java
    public static void bubbleSort(int[] arr) {
        boolean flag; //标志位
        for (int i = 0; i < arr.length - 1; i++) {
            flag = false;
            for (int j = 0; j < arr.length - 1 - i; j++) {
                if (arr[j] >= arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                    // 发生了交换，标志位置为true
                    flag = true;
                }
            }
            // 如果没有发生交换，则剩下的元素已经是排好序了
            if (!flag) {
                break;
            }
        }
    }
```

# （二）快速排序(Quick Sort)

快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。

## 算法描述
1. 首先设定一个分界值，通过该分界值将数组分成左右两部分。
2. 将大于或等于分界值的数据集中到数组右边，小于分界值的数据集中到数组的左边。此时，左边部分中各元素都小于或等于分界值，而右边部分中各元素都大于或等于分界值。
3. 然后，左边和右边的数据可以独立排序。对于左侧的数组数据，又可以取一个分界值，将该部分数据分成左右两部分，同样在左边放置较小值，右边放置较大值。右侧的数组数据也可以做类似处理。
4. 重复上述过程，可以看出，这是一个递归定义。通过递归将左侧部分排好序后，再递归排好右侧部分的顺序。当左、右两个部分各数据排序完成后，整个数组的排序也就完成了

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-03.gif)

快速排序的实现有很多种，常见的有：（1）挖坑法  （2）左右指针法  （3）快慢指针法。  我个人最喜欢快慢指针法，以下是快慢指针的实现


```java
    /**
     * 快速排序：取一个元素作为参考值，将数组分为左右两个数组
     * 快慢指针实现法
     */
    public static void quickSort(int[] arr, int left, int right) {
        if (left >= right) {
            return;
        }
        // 参考值默认都去数组最后一个值
        int pivot = arr[right];
        int slow = left - 1;
        int fast = left;
        // 将数组分为左右个分区
        for (; fast <= right; fast++) {
            // 如果快指针
            if (arr[fast] <= pivot) {
                // 交换
                slow++;
                int temp = arr[slow];
                arr[slow] = arr[fast];
                arr[fast] = temp;
            }
        }
        int subLeft = slow - 1 < 0 ? 0 : slow - 1;
        int subRight = slow + 1;

        // 小数组
        quickSort(arr, 0, subLeft);
        // 大数组
        quickSort(arr, subRight, right);
    }
```

# （三）、简单插入排序（Insertion Sort）

插入排序（Insertion-Sort）的算法描述是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

## 算法描述
一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

1. 从第一个元素开始，该元素可以认为已经被排序；
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描；
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置；
4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
5. 将新元素插入到该位置后；
6. 重复步骤2~5。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-04.gif)

```java
    /**
     * 插入排序
     * 
     * @param arr
     */
    public static void insertionSort(int[] arr) {
        int preIndex;
        int current;
        for (int i = 1; i < arr.length; i++) {
            current = arr[i];
            preIndex = i - 1;
            while (preIndex >= 0 && current < arr[preIndex]) {
                arr[preIndex + 1] = arr[preIndex];
                preIndex--;
            }
            arr[preIndex + 1] = current;

        }
    }
```    


# （四）、希尔排序(Shell Sort)
1959年Shell发明，第一个突破O(n2)的排序算法，是简单插入排序的改进版。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫**缩小增量排序**。

## 算法描述
先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：
1. 选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
2. 按增量序列个数k，对序列进行k 趟排序；
3. 每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-05.gif)

```java
    public static void shellSort(int[] arr) {
        for (int gap = arr.length / 2; gap > 0; gap /= 2) {
            for (int i = gap; i < arr.length; i++) {
                int current = arr[i];
                int preIndex = i - gap;
                
                // 插入排序
                while (preIndex >= 0 && current < arr[preIndex]) {
                    arr[preIndex + gap] = arr[preIndex];
                    preIndex -= gap;
                }
                arr[preIndex + gap] = current;
            }
        }
    }
```    


# （五）、简单选择排序（Selection Sort）

选择排序(Selection-sort)是一种简单直观的排序算法。它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。 

## 算法描述
n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：

1. 初始状态：无序区为R[1..n]，有序区为空；
2. 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，
    将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
3. n-1趟结束，数组有序化了。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-06.gif)


```java
    /**
     * 每次选取最小值
     *
     * @param arr
     */
    private static void selectSort(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            int minIndex = i;
            for (int j = i; j < arr.length; j++) {
                if (arr[minIndex] > arr[j]) {
                    minIndex = j;
                }
            }
            if (minIndex != i) {
                int temp = arr[i];
                arr[i] = arr[minIndex];
                arr[minIndex] = temp;
            }
        }
    }
```    

# （六）、堆排序(Heap Sort)

堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点
1. 将初始待排序关键字序列(R1,R2….Rn)构建成大顶堆，此堆为初始的无序区；
2. 将堆顶元素R[1]与最后一个元素R[n]交换，此时得到新的无序区(R1,R2,……Rn-1)和新的有序区(Rn),且满足R[1,2…n-1]<=R[n]；
3. 由于交换后新的堆顶R[1]可能违反堆的性质，因此需要对当前无序区(R1,R2,……Rn-1)调整为新堆，然后再次将R[1]与无序区最后一个元素交换，得到新的无序区(R1,R2….Rn-2)和新的有序区(Rn-1,Rn)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-07.gif)

```java
public void heapSort(int[] arr) {
        //1.构建大顶堆
        for (int i = arr.length / 2 - 1; i >= 0; i--) {
            //从第一个非叶子结点从下至上，从右至左调整结构
            adjustHeap(arr, i, arr.length);
        }
        //2.调整堆结构+交换堆顶元素与末尾元素
        for (int j = arr.length - 1; j > 0; j--) {
            //将堆顶元素与末尾元素进行交换
            swap(arr, 0, j);
            //重新对堆进行调整
            adjustHeap(arr, 0, j);
        }
    }

    /**
     * 调整大顶堆（仅是调整过程，建立在大顶堆已构建的基础上）
     *
     * @param arr
     * @param i
     * @param length
     */
    public static void adjustHeap(int[] arr, int i, int length) {
        int temp = arr[i];//先取出当前元素i
        //从i结点的左子结点开始，也就是2i+1处开始
        for (int k = i * 2 + 1; k < length; k = k * 2 + 1) {
            //如果左子结点小于右子结点，k指向右子结点
            if (k + 1 < length && arr[k] < arr[k + 1]) {
                k++;
            }
            //如果子节点大于父节点，将子节点值赋给父节点（不用进行交换）
            if (arr[k] > temp) {
                arr[i] = arr[k];
                i = k;
            } else {
                break;
            }
        }
        arr[i] = temp;
    }

    /**
     * 交换元素
     *
     * @param arr
     * @param a
     * @param b
     */
    public static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
```

# （七）、归并排序(Merge Sort)
归并排序（Merge sort）是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。

作为一种典型的分而治之思想的算法应用，归并排序的实现由两种方法：

- 自上而下的递归（所有递归的方法都可以用迭代重写，所以就有了第 2 种方法）；
- 自下而上的迭代；

# 算法描述
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置；
4. 重复步骤 3 直到某一指针达到序列尾；
5. 将另一序列剩下的所有元素直接复制到合并序列尾。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-08.gif)


``` java
    /**
     * 归并排序，不断将序列分成左右两个子序列，再将子序列合并成结果
     */
    private int[] mergeSort(int[] arr) {
        // 新建一个和原数组一样大小的数组，避免递归的时候要开辟空间
        int[] temp = new int[arr.length];
        sort(arr, 0, arr.length - 1, temp);
        return arr;
    }

    private void sort(int[] arr, int left, int right, int[] temp) {
        if (left >= right) {
            return;
        }
        int mid = (left + right) / 2;
        // 递归左边的子序列
        sort(arr, left, mid, temp);
        // 递归右边的子序列
        sort(arr, mid + 1, right, temp);
        // 将两个子序列合并
        merge(arr, left, mid, right, temp);
    }

    /**
     * 对left~right的子序列进行合并
     *
     * @param arr
     * @param left
     * @param mid
     * @param right
     * @param temp
     */
    private void merge(int[] arr, int left, int mid, int right, int[] temp) {
        int l = left;   //左序列指针
        int r = mid + 1;  //右序列指针
        int i = 0;     //当前填充临时数组指针
        while (l <= mid && r <= right) {
            if (arr[l] > arr[r]) {
                temp[i++] = arr[r++];
            } else {
                temp[i++] = arr[l++];
            }
        }
        while (l <= mid) {
            temp[i++] = arr[l++];
        }
        while (r <= right) {
            temp[i++] = arr[r++];
        }

        i = 0;
        // 数组拷贝
        while (left <= right) {
            arr[left++] = temp[i++];
        }
    }
```    

# （八）、计数排序（Counting Sort）
计数排序不是基于比较的排序算法，其核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。 作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数（知道最小值和最大值来建立数组映射）。

## 算法描述
1. 找出待排序的数组中最大和最小的元素；
2. 统计数组中每个值为i的元素出现的次数，存入数组C的第i项；
3. 对所有的计数累加（从C中的第一个元素开始，每一项和前一项相加）；
4. 反向填充目标数组：将每个元素i放在新数组的第C(i)项，每放一个元素就将C(i)减去1。

![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-09.gif)


```java
    /**
     * 计数排序
     *
     * @param arr 排序数组
     * @param min 数据最小值
     * @param max 数据最大值
     */
    private static void countingSort(int[] arr, int min, int max) {
        int size = max - min;
        int[] bucket = new int[size + 1];
        for (int i = 0; i < arr.length; i++) {
            bucket[arr[i] - min]++;
        }
        for (int j = 0, k = 0; j < bucket.length; j++) {
            while (bucket[j] > 0) {
                arr[k++] = min + j;
                bucket[j]--;
            }
        }
    }
```

计数排序是一个稳定的排序算法。当输入的元素是 n 个 [min,min+m] 之间的 `m` 个整数时，时间复杂度是O(n+m)，空间复杂度也是O(n+m)，其排序速度快于任何比较排序算法。当m不是很大并且序列比较集中时，计数排序是一个很有效的排序算法。当数据的范围不大且有大量重复数据时，可以考虑采用计数排序。


# （九）、桶排序(Bucket sort)

桶排序是计数排序的升级版。它利用了函数的映射关系，高效与否的关键就在于这个映射函数的确定。桶排序 (Bucket sort)的工作的原理：假设输入数据服从均匀分布，将数据分到有限数量的桶里，每个桶再分别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）

```java
    /**
     * hash算法 index = (int) Math.floor((arr[i] - min) / bucketSize);
     *
     * @param arr
     * @param bucketSize
     */
    public static int[] bucketSort(int[] arr, int bucketSize) {
        int min = Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;
        for (int i : arr) {
            min = Math.min(min, i);
            max = Math.max(max, i);
        }
        final int DEFAULT_BUCKET_SIZE = 5;
        bucketSize = Math.max(bucketSize, DEFAULT_BUCKET_SIZE);
        int bucketCount = (int) Math.floor((max - min) / bucketSize) + 1;
        int[][] bucket = new int[bucketCount][0];

        int index;
        //将数据放入各个桶中
        for (int i = 0; i < arr.length; i++) {
            index = (int) Math.floor((arr[i] - min) / bucketSize);
            // 把元素增加到子数组中，这利用的是数组拷贝，比较耗时，待优化
            bucket[index] = arrAppend(bucket[index], arr[i]);

        }
        int arrIndex = 0;
        for (int i = 0; i < bucket.length; i++) {
            int[] bucketArr = bucket[i];
            if (bucketArr.length > 0) {
                //对每个桶进行排序
                bucketArr = InsertionSort.insertionSort(bucketArr);
                //重组覆盖原数组
                for (int value : bucketArr) {
                    arr[arrIndex++] = value;
                }
            }
        }
        return arr;
    }

    /**
     * 自动扩容，并保存数据
     *
     * @param arr
     * @param value
     */
    private static int[] arrAppend(int[] arr, int value) {
        arr = Arrays.copyOf(arr, arr.length + 1);
        arr[arr.length - 1] = value;
        return arr;
    }
```

# （十）、基数排序（Radix Sort）
基数排序是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

## 基数排序 vs 计数排序 vs 桶排序
这三种排序算法都利用了桶的概念，但对桶的使用方法上有明显差异：
- 基数排序：根据键值的每位数字来分配桶；
- 计数排序：每个桶只存储单一键值；
- 桶排序：每个桶存储一定范围的数值；


LSD 基数排序动图演示：
![](/images/posts/myBlog/2021-02-25-Basic-Sort-Algorithm-10.gif)

```java
    /**
     * 获取最高位数
     */
    private int getMaxDigit(int[] arr) {
        int maxValue = Integer.MIN_VALUE;
        for (int value: arr){
            maxValue = Math.max(maxValue, value);
        }
        return getNumLength(maxValue);
    }

    private int getNumLength(long num) {
        if (num == 0) {
            return 1;
        }
        int length = 0;
        for (long temp = num; temp != 0; temp /= 10) {
            length++;
        }
        return length;
    }

    private int[] radixSort(int[] arr, int maxDigit) {
        int mod = 10;
        int dev = 1;

        for (int i = 0; i < maxDigit; i++, dev *= 10, mod *= 10) {
            // 考虑负数的情况，这里扩展一倍队列数，其中 [0-9]对应负数，[10-19]对应正数 (bucket + 10)
            int[][] counter = new int[mod * 2][0];

            for (int j = 0; j < arr.length; j++) {
                int bucket = ((arr[j] % mod) / dev) + mod;
                counter[bucket] = arrayAppend(counter[bucket], arr[j]);
            }
            
            int pos = 0;
            for (int[] bucket : counter) {
                for (int value : bucket) {
                    arr[pos++] = value;
                }
            }
        }
        return arr;
    }

    /**
     * 自动扩容，并保存数据
     */
    private int[] arrayAppend(int[] arr, int value) {
        arr = Arrays.copyOf(arr, arr.length + 1);
        arr[arr.length - 1] = value;
        return arr;
    }

```



# 参考文章
- [图解希尔排序](https://www.cnblogs.com/chengxiao/p/6104371.html)
- [图解排序算法(三)之堆排序](https://www.cnblogs.com/chengxiao/p/6129630.html)
- [十大经典排序算法(Java版本)](https://mp.weixin.qq.com/s?__biz=MzU4NDU0MjUyNg==&mid=2247484782&idx=1&sn=6f873cd893519058546c874e5c1e7ba0&chksm=fd99753fcaeefc29c56eb6b21c1031f33bb80327e2f9f83c122a707553c912cd73cd492648b7&scene=0&xtrack=1&key=68f6322da1df34d5483f596aef6ce3fd5e31442fbe402fab93e7d423f0879963387986771a3086ad4b1f4c5feefef65b95fa3230424d7f38abc281aebef7b389dd9f0a82adb6030f90e35da2e3c015a5&ascene=14&uin=MjE3MjAyMjgwNw%3D%3D&devicetype=Windows+7&version=62060833&lang=zh_CN&pass_ticket=4SanyOLPyssQpsITDwLUxt%2FdrxV8gRGYK63Er%2F%2BPTCCympAfljmBm7YjjdsGzz1W)
