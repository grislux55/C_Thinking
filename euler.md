### 欧拉函数的学习
---
- 定义：

>
    对正整数n，欧拉函数(φ(n))是小于或等于n的正整数中与n互质的数的数目。

- 性质：

    1. φ(mn) = φ(m) * φ(n)
    2. 若n为质数，φ(2n) = φ(n)
    3. 一个数的所有质因子之和是 φ(n) * n / 2

- 通项公式：

    `φ(x)=x*(1-1/p1)*(1-1/p2)*(1-1/p3)*(1-1/p4)*…*(1-1/pn)`

    其中p1,p2……pn为x的所有素因数，且相同的因子只算一遍

- 计算技巧：

    - 假设/声明：

        n = a^x * b^y * c^z,其中a,b,c为质因子、tmp变量初始化为n。

    - 从2到floor(sqrt(n+0.5))，步长为1，开始判断是否为n的因子（因为每次都会将因子除尽，所以第一次找到的因子一定是质因子）。

    - 当找到一个因子i后，tmp减去tmp/i。

    - 由于每个质因子计算一次，所以将n中的i除尽。

    - n如果大于1，则tmp减去tmp/n。

    - 得出的tmp即是欧拉函数的值。

```
    注意：由于只考虑到了根号n，还存在可能出现有大于根号n的因子的情况。
    n的值为1时已经除尽没有了多余的因子。
    不为1则说明n是大于根号n的一个质因子（因为n不可能有两个大于n的质因子）。
```

```
    注意：tmp-=tmp/i === tmp=tmp-tmp/i === tmp=tmp*(1-1/i)
```

```
    注意：还有另一种算法就是每次操作tmp的时候令tmp等于(tmp/i)*(i-1)，判断n时令tmp等于(tmp/n)*(n-1)。
```

#### 两种算法的比较：

- 程序：
```
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll euler(ll n) {
    ll tmp = n;
    ll half = (ll)sqrt(n + 0.5);
    for (ll i = 2; i <= half; i++) {
        if (n % i == 0) {
            tmp = (tmp / i) * (i - 1);
            while (n % i == 0) {
                n /= i;
            }
        }
    }
    if (n > 1) {
        tmp = (tmp / n) * (n - 1);
    }
    return tmp;
}

ll euler2(ll n) {
    ll tmp = n;
    ll half = (ll)sqrt(n + 0.5);
    for (ll i = 2; i <= half; i++) {
        if (n % i == 0) {
            tmp -= tmp / i;
            while (n % i == 0) {
                n /= i;
            }
        }
    }
    if (n > 1) {
        tmp -= tmp / n;
    }
    return tmp;
}

int main() {
    bool flag{};
    for (ll i = 1; i < 1000000; i++) {
        if (euler(i) != euler2(i)) {
            flag = true;
        }
    }

    auto time1 = clock();
    for (ll i = 1; i < 1000000; i++) {
        ll tmp = euler(i);
    }
    auto time2 = clock();
    for (ll i = 1; i < 1000000; i++) {
        ll tmp = euler2(i);
    }
    auto time3 = clock();

    cout << flag << endl;
    cout << time2 - time1 << endl;
    cout << time3 - time2 << endl;

    return 0;
}
```

- 输出（运行三次）：
```
    0
    5202
    5194

    0
    5205
    5187

    0
    5201
    5188
```

- 结论：
`差异不大`
