---
title: 
   DP：最长上升子序列模型
tags: 
  [算法]
categories:
	[算法]
date: 2020/11/14
---

> 现在打算写一些短点的文章了，LeetCode系列不会再append了，如果写lc题会单独开一篇文章，然后写题解

## _最长上升子序列模型_

[300. 最长上升子序列](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#300-%E6%9C%80%E9%95%BF%E4%B8%8A%E5%8D%87%E5%AD%90%E5%BA%8F%E5%88%97)

[673. 最长递增子序列的个数](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#673-%E6%9C%80%E9%95%BF%E9%80%92%E5%A2%9E%E5%AD%90%E5%BA%8F%E5%88%97%E7%9A%84%E4%B8%AA%E6%95%B0)

[1016. 使序列递增的最小交换次数（LintCode）](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#1016-%E4%BD%BF%E5%BA%8F%E5%88%97%E9%80%92%E5%A2%9E%E7%9A%84%E6%9C%80%E5%B0%8F%E4%BA%A4%E6%8D%A2%E6%AC%A1%E6%95%B0%EF%BC%88LintCode%EF%BC%89)

[354. 俄罗斯套娃信封问题](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#354-%E4%BF%84%E7%BD%97%E6%96%AF%E5%A5%97%E5%A8%83%E4%BF%A1%E5%B0%81%E9%97%AE%E9%A2%98)

LIS有N^2的DP解法，也有NlogN的贪心的解法，具体用那种取决于数据范围

## [1017. 怪盗基德的滑翔翼](https://www.acwing.com/problem/content/1019/)

怪盗基德是一个充满传奇色彩的怪盗，专门以珠宝为目标的超级盗窃犯。

而他最为突出的地方，就是他每次都能逃脱中村警部的重重围堵，而这也很大程度上是多亏了他随身携带的便于操作的滑翔翼。

有一天，怪盗基德像往常一样偷走了一颗珍贵的钻石，不料却被柯南小朋友识破了伪装，而他的滑翔翼的动力装置也被柯南踢出的足球破坏了。

不得已，怪盗基德只能操作受损的滑翔翼逃脱。

假设城市中一共有N幢建筑排成一条线，每幢建筑的高度各不相同。

初始时，怪盗基德可以在任何一幢建筑的顶端。

他可以选择一个方向逃跑，但是不能中途改变方向（因为中森警部会在后面追击）。

因为滑翔翼动力装置受损，他只能往下滑行（即：只能从较高的建筑滑翔到较低的建筑）。

他希望尽可能多地经过不同建筑的顶部，这样可以减缓下降时的冲击力，减少受伤的可能性。

请问，他最多可以经过多少幢不同建筑的顶部(包含初始时的建筑)？

**输入格式**

输入数据第一行是一个整数K，代表有K组测试数据。

每组测试数据包含两行：第一行是一个整数N，代表有N幢建筑。第二行包含N个不同的整数，每一个对应一幢建筑的高度h，按照建筑的排列顺序给出。

**输出格式**

对于每一组测试数据，输出一行，包含一个整数，代表怪盗基德最多可以经过的建筑数量。

**数据范围**

1≤K≤100, 1≤N≤100, 0<h<10000

**输入样例：**
```c
3
8
300 207 155 299 298 170 158 65
8
65 158 170 298 299 155 207 300
10
2 1 3 4 5 6 7 8 9 10
```
**输出样例：**
```c
6
6
9
```

**解法一**

常规N^2动态规划的方式 & NlogN贪心解法（NlogN的解法回忆了一会儿才推出来，如果数据范围不是很大还是N^2的dp好写）
```java
import java.io.*;
import java.util.*;
//AcWing 1017. 怪盗基德的滑翔翼 https://www.acwing.com/problem/content/1019/
class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int T = sc.nextInt();
        while (T-- > 0) {
            int n = sc.nextInt();
            int[] w = new int[n];
            for (int i = 0; i < n; i++) {
                w[i] = sc.nextInt(); 
            }
            System.out.println(Math.max(solve(w, n), solve(reverse(w), n))); 
        }
    }

    //DP(N^2)的解法
    public static int solve(int[] w, int n) {
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int res = 1;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (w[j] > w[i]) {
                    dp[i] = Math.max(dp[j]+1, dp[i]);  
                }
            }
            res = Math.max(res, dp[i]);
        }
        return res; 
    }

    //tail[i]: 长度为i的递减子序列最大结尾
    public static int solve2(int[] w, int n) {
        int[] tail = new int[n];
        int len = 0;
        for (int i = 0; i < n; i++) {
            int idx = search(tail, len, w[i]);
            if (idx == -1) {
                tail[len++] = w[i];
            } else {
                tail[idx] = w[i];
            }
        }
        return len;
    }

    //从左到右找第一个小于target的元素下标，替换它，使得结尾更大，长度更长
    public static int search(int[] tail, int len, int target){
        int left = 0, right = len-1;
        int res = -1;
        while (left <= right) {
            int mid = left + (right - left)/2;
            if (tail[mid] <= target) {
                res = mid;
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return res;
    }

    public static int[] reverse(int[] w) {
        for (int i = 0, j = w.length-1; i < j; i++, j--) {
            int temp = w[i];
            w[i] = w[j];
            w[j] = temp;
        }
        return w;
    }
}
```

## [1014. 登山](https://www.acwing.com/problem/content/1016/)

五一到了，ACM队组织大家去登山观光，队员们发现山上一个有N个景点，并且决定按照顺序来浏览这些景点，即每次所浏览景点的编号都要大于前一个浏览景点的编号。

同时队员们还有另一个登山习惯，就是不连续浏览海拔相同的两个景点，并且一旦开始下山，就不再向上走了。

队员们希望在满足上面条件的同时，尽可能多的浏览景点，你能帮他们找出最多可能浏览的景点数么？

**输入格式**

第一行包含整数N，表示景点数量。

第二行包含N个整数，表示每个景点的海拔。

**输出格式**

输出一个整数，表示最多能浏览的景点数。

**数据范围**
2≤N≤1000
**输入样例：**
```c
8
186 186 150 200 160 130 197 220
```
**输出样例：**
```c
4
```

**解法一**

LIS转换了一下而已，求每个点从**左到右** 和 从**右到左**的最长上升子序列最大和就ok了
```java
import java.util.*;
import java.io.*;
class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int N = sc.nextInt();
        int[] w = new int[N];
        for (int i = 0; i < N; i++) {
            w[i] = sc.nextInt();
        }
        System.out.println(solve(w, N));
    }

    //186 186 150 200 160 130 197 220  := 4
    public static int solve(int[] w, int N) {
        int[] dp1 = new int[N];
        int[] dp2 = new int[N];
        Arrays.fill(dp1, 1);
        Arrays.fill(dp2, 1);
        int res = 1;
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < i; j++) {
                if (w[j] < w[i]) {
                    dp1[i] = Math.max(dp1[j]+1, dp1[i]);          
                }
            }
        }
        for (int i = N-1; i >= 0; i--) {
            for (int j = N-1; j > i; j--) {
                if (w[j] < w[i]) {
                    dp2[i] = Math.max(dp2[j]+1, dp2[i]);          
                }
            }
            res = Math.max(res, dp1[i]+dp2[i]-1);
        }
        return res;
    }
}
```
> [482. 合唱队形](https://www.acwing.com/problem/content/484/)和这一题一样，不重复写了

## [1012. 友好城市](https://www.acwing.com/problem/content/1014/)

Palmia国有一条横贯东西的大河，河有笔直的南北两岸，岸上各有位置各不相同的N个城市。

北岸的每个城市有且仅有一个友好城市在南岸，而且不同城市的友好城市不相同。

每对友好城市都向政府申请在河上开辟一条直线航道连接两个城市，但是由于河上雾太大，政府决定避免任意两条航道交叉，以避免事故。

编程帮助政府做出一些批准和拒绝申请的决定，使得在保证任意两条航线不相交的情况下，被批准的申请尽量多。

**输入格式**

第1行，一个整数N，表示城市数。

第2行到第n+1行，每行两个整数，中间用1个空格隔开，分别表示南岸和北岸的一对友好城市的坐标。

**输出格式**

仅一行，输出一个整数，表示政府所能批准的最多申请数。

**数据范围**

1≤N≤5000,
0≤xi≤10000

**输入样例：**
```c
7
22 4
2 6
10 3
15 12
9 8
17 17
4 2
```
**输出样例：**
```c
4
```

**解法一**

和之前的[354. 俄罗斯套娃信封问题](http://imlgw.top/2019/09/01/leetcode-dong-tai-gui-hua/#354-%E4%BF%84%E7%BD%97%E6%96%AF%E5%A5%97%E5%A8%83%E4%BF%A1%E5%B0%81%E9%97%AE%E9%A2%98)一样，还简单一点，坐标都是唯一的，不需要考虑坐标重合的问题，具体可以去看看俄罗斯套娃的这个题

```java
import java.util.*;
import java.io.*;
class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int N = sc.nextInt();
        int[][] w = new int[N][2];
        for (int i = 0; i < N; i++) {
            w[i][0] = sc.nextInt();
            w[i][1] = sc.nextInt();
        }
        System.out.println(solve2(w, N));
    }

    //5000 * 5000 = 2500 0000，能过但是有点慢
    public static int solve(int[][] w, int N) {
        Arrays.sort(w, (w1, w2)->w1[0]-w2[0]);
        int res = 1;
        int[] dp = new int[N];
        for (int i = 0; i < N; i++) {
            dp[i] = 1;
            for (int j = 0; j < i; j++) {
                if (w[j][0] < w[i][0] && w[j][1] < w[i][1]) {
                    dp[i] = Math.max(dp[j]+1, dp[i]);
                }
            }
            res = Math.max(res, dp[i]);
        }
        return res;
    }

    //NlogN贪心二分的做法
    public static int solve2(int[][] w, int N) {
        Arrays.sort(w, (w1, w2)->w1[0]-w2[0]);
        int[] tail = new int[N];
        int len = 0;
        for (int i = 0; i < N; i++) {
            int idx = search(tail, len, w[i][1]);
            if (idx == -1) {
                tail[len++] = w[i][1];
            } else {
                tail[idx] = w[i][1];
            }
        }
        return len;
    }

    //从左到右找第一个大于target的，后面替换它，使得结尾更小
    public static int search(int[] tail, int len, int target) {
        int left = 0, right = len-1;
        int res = -1;
        while (left <= right) {
            int mid = left + (right - left)/2;
            if (tail[mid] > target) {
                res = mid;
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return res;
    }
}
```

## [1016. 最大上升子序列和](https://www.acwing.com/problem/content/1018/)

一个数的序列 bi，当 b1<b2<…<bS 的时候，我们称这个序列是上升的。

对于给定的一个序列(a1,a2,…,aN)，我们可以得到一些上升的子序列(ai1,ai2,…,aiK)，这里1≤i1<i2<…<iK≤N。

比如，对于序列(1,7,3,5,9,4,8)，有它的一些上升子序列，如(1,7),(3,4,8)等等。

这些子序列中和最大为18，为子序列(1,3,5,9)的和。

你的任务，就是对于给定的序列，求出最大上升子序列和。

注意，最长的上升子序列的和不一定是最大的，比如序列(100,1,2,3)的最大上升子序列和为100，而最长上升子序列为(1,2,3)。

**输入格式**

输入的第一行是序列的长度N。

第二行给出序列中的N个整数，这些整数的取值范围都在0到10000(可能重复)。

**输出格式**

输出一个整数，表示最大上升子序列和。

**数据范围**

1≤N≤1000

**输入样例：**
```c
7
1 7 3 5 9 4 8
```
**输出样例：**
```
18
```

**解法一**

没啥好说的，比较简单
```java
import java.util.*;
import java.io.*;
class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int N = sc.nextInt();
        int[] w = new int[N];
        for (int i = 0; i < N; i++) {
            w[i] = sc.nextInt();
        }
        System.out.println(solve(w, N));
    }

    public static int solve(int[] w, int N) {
        int[] dp = new int[N];
        int res = 0;
        for (int i = 0; i < N; i++) {
            dp[i] = w[i];
            for (int j = 0; j < i; j++) {
                if (w[j] < w[i]) {
                    dp[i] = Math.max(dp[i], dp[j]+w[i]);
                }
            }
            res = Math.max(res, dp[i]);
        }
        return res;
    }
}
```
## [1010. 拦截导弹](https://www.acwing.com/problem/content/description/1012/)

某国为了防御敌国的导弹袭击，发展出一种导弹拦截系统。

但是这种导弹拦截系统有一个缺陷：虽然它的第一发炮弹能够到达任意的高度，但是以后每一发炮弹都不能高于前一发的高度。

某天，雷达捕捉到敌国的导弹来袭。

由于该系统还在试用阶段，所以只有一套系统，因此有可能不能拦截所有的导弹。

输入导弹依次飞来的高度（雷达给出的高度数据是不大于30000的正整数，导弹数不超过1000），计算这套系统最多能拦截多少导弹，如果要拦截所有导弹最少要配备多少套这种导弹拦截系统。

**输入格式**: 

共一行，输入导弹依次飞来的高度。

**输出格式**: 

第一行包含一个整数，表示最多能拦截的导弹数。

第二行包含一个整数，表示要拦截所有导弹最少要配备的系统数。

**输入样例：**
```c
389 207 155 300 299 170 158 65
```
**输出样例：**
```c
6
2
```

**解法一**
```java
import java.util.*;
import java.io.*;
class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        List<Integer> lis = new ArrayList<>();
        while (sc.hasNext()) {
            lis.add(sc.nextInt());
        }
        int[] w = new int[lis.size()];
        for (int i = 0; i < lis.size(); i++) {
            w[i] = lis.get(i);
        }
        int[] res = solve(w, lis.size());
        System.out.println(res[0]);
        System.out.println(res[1]);
    }

    //300 250 275 252 200 138 245 := 5,2
    //最多击落多少就不多说了
    //最少需要的系统数其实就是最长上升子序列，因为这个子序列是必不可能在同一套系统中被击落
    //当遇到这个子序列中的元素的时候，就需要增加一套系统，所以最少就是这个子序列的长度
    public static int[] solve(int[] w, int N){
        int[] down = new int[N];
        int[] up = new int[N];
        int max = 0, count = 1;
        for (int i = 0; i < N; i++) {
            up[i] = down[i] = 1;
            for (int j = 0; j < i; j++) {
                if (w[j] >= w[i]) {
                    down[i] = Math.max(down[j]+1, down[i]);
                } else {
                    up[i] = Math.max(up[j]+1, up[i]);
                }
            }
            max = Math.max(max, down[i]);
            count = Math.max(count, up[i]);
        }
        return new int[]{max, count};
    }
}
```

最多击落多少就不多说了，最少需要的系统数其实就是最长上升子序列，因为这个子序列是**必不可能在同一套系统中被击落**，当遇到这个子序列中的元素的时候，就需要增加一套系统，所以最少就是这个子序列的长度