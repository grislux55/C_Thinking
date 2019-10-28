### 经过仔细测算，C语言里面并不适合使用pow函数来计算正负（即 `pow(-1,n)`）
以下为测试用例，这个用例使用精心设计的算法，保证了性能最大化：
```
#include <stdio.h>
#include <time.h>

int main() {
    long x;
    double sum = 0.0;
    scanf("%ld", &x);
    long before = clock();
    for (; x >= 1; x--) {
        sum = 1.0 / (2 * x - 1) - sum;
    }
    long after = clock();
    printf("program runs: %ld ms\n", after - before);
    printf("result is: %lf", sum);
    return 0;
}
```

所测得的结果为：
```
9000000
program runs: 39 ms
result is: 0.785398

90000000
program runs: 519 ms
result is: 0.785398
```

以下为使用pow函数计算：
```
#include <stdio.h>
#include <time.h>
#include <math.h>

int main() {
    long x;
    double sum = 0.0;
    scanf("%ld", &x);
    long before = clock();
    for (; x >= 1; x--) {
        sum += pow(-1,x+1)/(2*x-1);
    }
    long after = clock();
    printf("program runs: %ld ms\n", after - before);
    printf("result is: %lf", sum);
    return 0;
}
```

所测得的结果为：
```
9000000
program runs: 1097 ms
result is: 0.785398

90000000
program runs: 10892 ms
result is: 0.785398
```

哈哈哈哈！
### pow函数的算法提供了一个慢20多倍的算法！
