+++
date = '2025-01-04T09:52:08+08:00'
title = 'counting numbers — dynamic programming'
+++

## Problem
Count the number of integers between $a$ and $b$ where no two adjacent digits are the same. [Link to problem](https://cses.fi/problemset/task/2220)

**Input**

The only input line has two integers a and b.

**Output**

Print one integer: the answer to the problem.

**Constraints**

$0 \leq a \leq b \leq 10^{18}$

**Example**

Input: 
`123 321`
Output:
`171`

## Solution
If we can calculate the number of integers between $0$ and $n$ with no two adjacent digits for any nonnegative $n$, then the answer is just the count for $[0, b]$ and minus the count for $[0, a - 1]$.

First count the number of integers with $i$ digits zeros, with any leading digit (for example $1$), and with no adjacent digits without considering leading, call this $f_i$. Then we have $f_i = 9 f_{i-1}$ since the second digit cannot be the same as the first digit. Also $f_1 = 1$. We can preprocess this for $1 \leq i \leq 18$. 
```c++
const int N = 19;
long long l, r;
unsigned long long f[N];
int a[N];

int main() {
    cin >> l >> r;
    f[1] = 1LL;
    for (int i = 2; i < N; i++) {
        f[i] = 9LL * f[i - 1];
    }
    cout << solve(r) - solve(l - 1) << endl;
}
```
For our main function `solve`, we first need to convert the number into an array of digits. 
```c++
unsigned long long solve(long long n) {
    if (n < 0) return 0;
    if (n == 0) return 1;

    int len = 0;
    unsigned long long res = 0;
    while (n > 0) {
        len++;
        a[len] = n % 10;
        n /= 10;
    }
    // ......
}
```
To illustrate this with an example, consider `solve(2413)`. Then the `a` array and the variable `len` would be 
```
a = [0, 3, 1, 4, 2]
len = 4
```
Now we start the counting process, for a number with 4 digits and the leading digit being `0` and `1`, there is `f[4]` integers without two adjacent digits being the same. 
For leading digit `2` we have to consider the boundary so we handle that using next iteration. The next iteration is essentially counting the number of integers with 4 digits, starting with `2`, and no two adjacent digits are the same. For `0`, `1`, `2` and `3` on `a[3]` we can use our `f[3]` value directly and we handle the boundary `4` with next iteration. Therefore, our main loop is 
```c++
bool flag = false;
for (int i = len; i >= 1; i--) {
    res += (long long)a[i] * f[i];
    if (i < len && a[i] > a[i + 1]) res -= f[i];
    if (i < len && a[i] == a[i + 1]) {
        flag = true;
        break;
    }
}
if (!flag) res++;
```
Notice that when `a[i] > a[i + 1]`, we should not count the numbers with `a[i] = a[i + 1]` so we use `res -= f[i]` to handle that. In addition, when `a[i] = a[i + 1]` since we are essentially counting the numbers starting with `a[i+1 : len]` in later iterations and `a[i] = a[i + 1]` indicates the presence of two adjacent digits being the same, we break from the loop. Finally the `flag` and `res++` is used to handle the number itself. 

To deal with leading zeros, we remove the contribution from leading digits being zeros first by `res -= f[len]`, and then add the contribution from integers with `j` digits starting with `1` to `9` (ensuring no leading zeros) for `1 <= j <= len`. 
```c++
res -= f[len];
for (int j = 1; j < len; j++) {
    res += 9LL * f[j];
}
res++;
```
`res++` is used to handle the number `0`. This finishes this function and the complete solution code is as follows.
```c++
const int N = 19;
long long l, r;
unsigned long long f[N];
int a[N];

unsigned long long solve(long long n) {
    if (n < 0) return 0;
    if (n == 0) return 1;

    int len = 0;
    unsigned long long res = 0;
    while (n > 0) {
        len++;
        a[len] = n % 10;
        n /= 10;
    }
    bool flag = false;
    for (int i = len; i >= 1; i--) {
        res += (long long)a[i] * f[i];
        if (i < len && a[i] > a[i + 1]) res -= f[i];
        if (i < len && a[i] == a[i + 1]) {
            flag = true;
            break;
        }
    }
    if (!flag) res++;

    res -= f[len];
    for (int j = 1; j < len; j++) {
        res += 9LL * f[j];
    }
    res++;
    return res;
}

int main() {
    cin >> l >> r;
    f[1] = 1LL;
    for (int i = 2; i < N; i++) {
        f[i] = 9LL * f[i - 1];
    }
    cout << solve(r) - solve(l - 1) << endl;
}
```


