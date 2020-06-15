# Josephus problem

> There are *n* people standing in a circle waiting to be executed. The counting out begins at the first people in the circle and proceeds around the circle in the counterclockwise direction. In each step, a certain number of people are skipped and the next person is executed. The elimination proceeds around the circle (which is becoming smaller and smaller as the executed people are removed), until only the last person remains, who is given freedom.
>
> INPUT as follows:
>
> ```
> N K
> ```
>
> N is stand for the number of people
>
> K is stand for the step of executing
>
> *INPUT HAS MULTIPLE LINES*
>
> 
>
> OUTPUT as follows:
>
> ```
> F
> ```
>
> F is stand for the freedom one

翻译：人们站成一个圈，等待被杀死。计数从圆中的指定点开始，并沿圆的指定方向进行。跳过指定数量的人员后，将执行下一个人员。对其余人员重复该过程，从下一个活人开始，朝相同的方向前进并跳过相同的人数，直到只有一个人留下，并被释放。

题意分析：这是著名的约瑟夫问题。

我们可以尝试模拟一下该过程

```
# N = 10, K = 4
[1 2 3 4 5 6 7 8 9 10]
[1 2 3 X 5 6 7 8 9 10]
[1 2 3 X 5 6 7 X 9 10]
[1 X 3 X 5 6 7 X 9 10]
[1 X 3 X 5 6 X X 9 10]
[1 X X X 5 6 X X 9 10]
[1 X X X 5 6 X X 9 X]
[1 X X X 5 6 X X X X]
[X X X X 5 6 X X X X]
[X X X X 5 X X X X X]
```

仔细观察可发现每当我们从中移走一个人的时候，我们的问题便下降了一个层面。

我们解决`N = a, K = <any>`这样的问题的时候移走了一个人，那么问题就转化为了寻找`N = a - 1, K = <any>`这样的问题，同时后面的人的索引值是不变的。

```
# 删除 k
1 2 3 4 5 ... k-2 k-1 k k+1 k+2 ... N-2 N-1 N
# 剩下的序列（注意此时的第一个数变成了k+1）
k+1 k+2   ... N-2 N-1 N 1 2 3  ... k-1
```

看到了规律没有？

所有的数都加上了`k`！但是由于索引无法越过`N`，所以`N`以后的值又变回去了......

那此时的状态转移方程应该如下：

```
J(N, K) = (J(N - 1, K) + K - 1) % n + 1
```

如果索引是从0开始的呢？

那么我们就有了一个更好看的公式

```
J(N, K) = (J(N - 1, K) + K) % n
```

终止条件不难分析，可以得出（索引从0开始）

```
J(1, K) = 0
```

因此代码如下

```c
int josephus(int n, int k)
{
    return n > 1 ? (josephus(n - 1, k) + k) % n : 0;
}
```

时间复杂度为O(n)​

当然我们还有复杂度为O(k *log* n)​的方法。

显然，我们不需要每次只杀死一个人，我们可以杀死一个循环内的所有满足条件的人，那么我们的索引更新就要变成加上之前删除的个数。

因此代码如下

```c
int josephus(int n, int k)
{
    if (n == 1)
        return 0;
    if (k == 1)
        return n - 1;
    if (k > n)
        return (josephus(n - 1, k) + k) % n;
    int cnt = n / k;
    int res = josephus(n - cnt, k);
    res -= n % k;
    if (res < 0)
        res += n;
    else
        res += res / (k - 1);
    return res;
}
```

