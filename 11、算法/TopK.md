



参考文章：

- https://blog.csdn.net/bcmbcmbcm/article/details/72773252
- https://blog.csdn.net/L969621426/article/details/62217226?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

> 问题：
>
> 在n个数中找到最大的K个数
>
> 比如：在arr = ｛1,5,13,99,25,64,75,32,22｝中找到最大的5个数

```java
import java.util.Arrays;

public class SuccessTopK {

    public static void main(String[] args) {
        int[] arr = {12,78,99,2,100,56,97,3,500,45,15,66,88,77,55,19,20,30};
        TopK t = new TopK(arr,10);
        int[] top = t.getTop();
        System.out.println(Arrays.toString(top));
    }
}

class TopK{
    private int[] arr;//存放整个数据
    private int[] top;//二叉堆，最后返回这个二叉堆就是5个最大的数
    private int num;//查找的数量

    /**
	 * 初始化数组，和查找最大个数
	 * @param arr
	 * @param k
	 */
    public TopK(int[] arr,int num) {
        super();
        this.arr = arr;
        this.num = num;
        top = new int[num];
        //初始化最小堆
        initHead(num);
    }


    //初始化化二叉堆
    public void initHead(int k) {
        for(int i = 0;i<k;i++) {
            top[i] = arr[i];
        }
        //将最小数设置到堆顶(排序)
        for(int i = top.length/2-1;i>=0;i--) {//i为最后开始的根节点
            sort(i);
        }
    }

    //二叉堆的排序，数组的最小数排在二叉堆顶端
    public void sort(int i) {
        int left = (i<<1)+1;
        int right = (i<<1)+2;
        int temp,min;//temp:用来交换数组数据；min:用来记录当前的节点下标

        if(left>=top.length) return;
        if((right<top.length) && top[right]<top[left]) {
            min = right;
        }else {
            min = left;
        }

        //交换
        if(top[i]>top[min]) {
            temp = top[min];
            top[min] = top[i];
            top[i] = temp;
            sort(min);
        }
    }

    //返回前5个最大数
    public int[] getTop() {

        for(int i = top.length;i<arr.length;i++) {
            if(top[0]<arr[i]) {
                top[0] = arr[i];
                sort(0);
            }
        }

        return top;
    }

}

```

![](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/TOPK思路1.png)



# 步骤

> 在n个数据中把前K个数拿出来，最为二叉堆，然后对二叉堆进行排序，使最小的数为二叉堆的根节点

![](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/TOPK思路2.png)

> 用剩余的数据依次与二叉堆的根节点元素进行比较，如果比根节点大就替换，重新排序。
>
> 循环此步骤，直到比较完整个数据，即可得到前K个大的数据 

![](https://gitee.com/sxhDrk/images/raw/master/imgs-2021-04-27/TOPK问题.png)