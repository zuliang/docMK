# power algorithm
## 分治处理
x和n是整数，以下算法复杂度为O(lgn),将阶乘一分为二递归，需要注意：
1. n的奇偶情况
2. n为负数这个坑
```python
def pow(x, n):
    if n == 0:
        return 1
    if n < 0:
        return 1.0 / pow(x, -n)
    if n % 2:
        return x * pow(x, n-1)
    else:
        return pow(x*x, n/2)


def pow_nonrevcursive(x, n):
    if n < 0:
        x = 1.0 / x
        n = -n
    powe = 1
    while n:
        if n & 1:
            powe *= x
        x *= x
        n >>= 1
    return powe
```