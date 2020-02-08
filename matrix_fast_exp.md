# 快速幂与矩阵快速幂

### 快速幂
---

#### 朴素

在[有关用幂运算设置正负的思考](https://github.com/grislux55/C_Thinking/blob/master/pow.md)中我们可以显而易见地发现，幂运算其实是一个效率很低的运算。

并且在常见的比赛中通常需要对一个较大的数求一个很大的幂然后取余。

我们可以想到的第一个朴素的想法就是暴力运算，时间复杂度`O(N)`。

```
int pow_mod(int n, int level, int mod) {
    int tmp = n;
    for (int i = 1; i < level; i++) {
        tmp = (tmp * n) % mod;
    }
    return tmp;
}
```

>
    注意：模运算并不干涉乘法运算。

然而人家是不会让你这么简单就过了的。

我们来考虑一下数的二进制表示法。

#### 快速幂

我们以求`2 ^ 14`为例，`14`在二进制表示下为：`1110`。
```
14 = 0 * 2 ^ 0 + 1 * 2 ^ 1 + 1 * 2 ^ 2 + 1 * 2 ^ 3
==> 14 = 2 ^ 1 + 2 ^ 2 + 2 ^ 3
```
所以 `2 ^ 14 = 2 ^ (2 ^ 1) * 2 ^ (2 ^ 2) * 2 ^ (2 ^ 3)`

所以我们需要计算如下的数的积：
```
2 ^ (2 ^ 1)
2 ^ (2 ^ 2)
2 ^ (2 ^ 3)
```

进一步，我们可以引入一个中间变量`tmp`，以及一个答案变量`ans`：
```
ans = 1;
tmp = 2;
tmp *= tmp; // 2 ^ (2 ^ 1)
ans *= tmp;
tmp *= tmp; // 2 ^ (2 ^ 2)
ans *= tmp;
tmp *= tmp; // 2 ^ (2 ^ 3)
ans *= tmp
```

由于次方数的每一位上的二进制位为1时才修改答案，我们可以用这样的判别式：`level & 1`。

每次循环时要将已经使用的位数丢弃,使用：`level >>= 1`。

可以抽象出通用代码：
```
while (level) {
    if (level & 1) {
        ans *= tmp;
    }
    tmp *= tmp;
    level >>= 1;
}
```

加入取余部分：
```
while (level) {
    if (level & 1) {
        ans = (ans * tmp) % mod;
    }
    tmp *= tmp;
    level >>= 1;
}
```

这样就得到了通用的一个快速幂函数：
```
int fast_pow_mod(int n, int level, int mod) {
    int tmp = n, ans = 1;
    while (level) {
        if (level & 1) {
            ans = (ans * tmp) % mod;
        }
        tmp *= tmp;
        level >>= 1;
    }
    return tmp;
}
```

时间复杂度：`O(log2(N))`

### 矩阵快速幂

快速幂似乎已经完结了，但是转念一想，矩阵也是支持乘法的，那么我们似乎可以将其推广到求一个矩阵的n次方上去。

定义如下一个矩阵：
```
struct Matrix {
    vector<vector<int>> mat;
    int rows, cols;

    Matrix(vector<vector<int>> values)
        : mat(values), rows(values.size()), cols(values[0].size()) {}

    static Matrix identity_matrix(int n) {
        vector<vector<int>> values(n, vector<int>(n, 0));
        for (int i = 0; i < n; i++) {
            values[i][i] = 1;
        }
        return values;
    }

    Matrix operator*(const Matrix &other) const {
        int n = rows, m = other.cols;
        vector<vector<int>> result(rows, vector<int>(m, 0));
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                for (int k = 0; k < cols; k++) {
                    result[i][j] =
                        result[i][j] + mat[i][k] * 1ll * other.mat[k][j];
                }
            }
        }
        return Matrix(result);
    }

    Matrix multi_mod(const Matrix &other, int mod) const {
        int n = rows, m = other.cols;
        vector<vector<int>> result(rows, vector<int>(m, 0));
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                for (int k = 0; k < cols; k++) {
                    result[i][j] =
                        (result[i][j] + mat[i][k] * 1ll * other.mat[k][j]) %
                        mod;
                }
            }
        }
        return Matrix(result);
    }
};
```

哦呼！太长不看！总之就是实现了乘法和求模乘法的矩阵类型，支持使用`vector<vector<int>>`进行初始化，内置n和m表示行列长度，还有一个静态方法生成单位矩阵。

既然已经实现了矩阵类型了，那么怎么进行快速乘法呢？

答案很简单，直接将其当作整数用就好了，不过开始相乘的时候需要的基底变量是一个单位矩阵：
```
Matrix fast_exp(Matrix square, int power) {
    Matrix result = Matrix::identity_matrix(square.rows);

    while (power) {
        if (power & 1) {
            result = result * square;
        }
        square = square * square;
        power >>= 1;
    }

    return result;
}
```

加入取余：
```
Matrix fast_exp_mod(Matrix square, int power, int mod) {
    Matrix result = Matrix::identity_matrix(square.rows);

    while (power) {
        if (power & 1) {
            result = result.multi_mod(square, mod);
        }
        square = square.multi_mod(square, mod);
        power >>= 1;
    }

    return result;
}
```

### 矩阵快速幂加速递推公式

oof，好像单独写个矩阵快速幂没啥用的感觉，那我们来加速一下递推公式。

>
    题目简述：
    给定A，B，定义一个递推公式：
    F(N)=A * F(N-1) + B * F(N-2)，F(2)=F(1)=1
    计算F(N)结果对1E7取余。

>
    输入：若干组A，B，N

>
    输出：F(N) % 1E7

首先我们来写一下矩阵：由于右侧给出了两个函数用法：`F(N-1)`和`F(N-2)`

所以矩阵如下：
```
|  F(N)  |
| F(N-1) |
```

设一个矩阵为base使其满足：
```
|  F(N)  |              | F(N-1) |
|        |  =  base  *  |        |
| F(N-1) |              | F(N-2) |
```

由于两侧均为两行一列的矩阵，所以base的大小为两行两列。

先看第二行，很明显`F(N-1)=1*F(N-1)+0*F(n-2)`，所以第二行为`1 0`。

再来看第一行`F(N)=A*F(N-1)+B*F(N-2)`，所以第一行为`A B`。

整个矩阵：
```
| A B |
| 1 0 |
```

这个式子就可以展开为：
```
|  F(N)  |                      | F(2) |
|        |  =  base ^ (N-2)  *  |      |
| F(N-1) |                      | F(1) |
```

代码写作：
```
int a, b, n;
while (cin >> a >> b >> n) {
    if (n <= 2) {
        cout << 1 << '\n';
    } else {
        auto initial = Matrix({{1}, {1}});
        auto m = Matrix({{a, b}, {1, 0}});
        auto exp = fast_exp_mod(m, n - 2, 1E7);
        cout << exp.multi_mod(initial, 1E7).mat[0][0] << '\n';
    }
}
```
