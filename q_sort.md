# 快速排序

[TOC]

## 简介

快速排序是经典的排序方式之一，在大学算法课上是必学的内容。不过工作了这么久，中间也没有怎么写算法，东西也忘得差不多了。最近找工作的时候居然被问到快排，有点儿懵了。所以再来回顾一下，自己写一个快排。

## 实现

不说这么多，直接上代码：

~~~java
public class QSort {

    public static <T extends Comparable> void sort(T[] elements) {
        quickSortHelper(elements, 0, elements.length - 1);
    }

    private static <T extends Comparable> void quickSortHelper(T[] elements, int left, int right) {
        if (left < right) {
            int pivot = partition(elements, left, right);
            quickSortHelper(elements, left, pivot - 1);
            quickSortHelper(elements, pivot + 1, right);
        }
    }

    private static <T extends Comparable> int partition(T[] elements, int left, int right) {
        // 下文
    }
}
~~~

对于partition的实现，我参考了两种方式：

方式1：

~~~ java
    private static <T extends Comparable> int partition(T[] elements, int left, int right) {
        T pivot = elements[left];
        while (left < right) {
            while (left < right && elements[right].compareTo(pivot) >= 0) {
                right--;
            }
            elements[left] = elements[right];

            while (left < right && elements[left].compareTo(pivot) <= 0) {
                left++;
            }
            elements[right] = elements[left];
        }

        elements[left] = pivot;

        return left;
    }
~~~

方式2：

~~~java
    private static <T extends Comparable> int partition(T[] elements, int left, int right) {
        T pivot = elements[left];
        int i = left + 1;

        for (int j = i; j <= right; j++) {
            if (elements[j].compareTo(pivot) < 0) {
                swap(elements, i, j);
                i ++;
            }
        }


        swap(elements, left, i - 1);

        return i - 1;
    }
~~~

然后我使用了一个方法简单地作也对比：

~~~java
        List<Integer> sample = new ArrayList<>();
        Random random = new Random();
        for (int i = 0; i < 1000000; i++) {
            sample.add(random.nextInt(10000));
        }
        long start, end;

        Integer[] array = sample.toArray(new Integer[0]);
        start = System.nanoTime();
        Arrays.sort(array); //使用Tim Sort
        end = System.nanoTime();
        System.out.println("Tim Time Used: " + (end - start) / 1000);

        Integer[] array3 = sample.toArray(new Integer[0]);
        start = System.nanoTime();
        QSort.sort2(array3); //使用方法2Partition
        end = System.nanoTime();
        System.out.println("P2 Time Used: " + (end - start) / 1000);

        Integer[] array2 = sample.toArray(new Integer[0]);
        start = System.nanoTime();
        QSort.sort(array2); //使用方法1Partition
        end = System.nanoTime();
        System.out.println("P1 Time Used: " + (end - start) / 1000);
~~~

## 结论

发现以下的现象：

1. 在数据已经是顺序或逆序，并且数据量大（100w）的情况下，方法1和方法2都会有StackOverflowException，这是因为递归调用的原因；
2. 在数据量小（1000）,并且数据被充分打乱的情况下，快排可以比Tim Sort更快；
3. 在数据量大的时候, Tim Sort明显快于快排。

通过查资料，可以知道快排的算法复杂度平均为nlogn。并且在以下情况下算法复杂度最高，为n^2：

1. 顺序;
2. 逆序；
3. 所有元素都相等（1和2的特殊情况）。