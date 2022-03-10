---
title: 第45届ICPC亚洲区域赛济南站题解
date: 2021-10-7
top: true
tags: [ICPC,C++]
categories: 题解
mathjax: true
---
# A.Matrix Equation
* **题意**
给定$n\times n$的$01$矩阵$A,B$，求满足$A\times C=B\cdot C$的矩阵个数。
* **解题思路**
由于是矩阵乘法，所以对于$C$的每一列都可以单独考虑，即可以设置C的每一列一开始都有$n$个未知数，那么自然可以得到$n$条异或方程组，然后利用高斯消元求解出自由变元个数$x$，这些是可以自由取值的，所以矩阵个数则为$2^{x}$。

* **AC代码**

```cpp
#include<bits/stdc++.h>

using namespace std;
using ll = long long;

const int N = 2e2 + 10, P = 998244353;
int n, A[N][N], B[N][N];
int a[N][N], free_num;
int equ, var;
int pow2[N];
void cal(int n){
    pow2[0] = 1;
    for(int i = 1; i <= n; ++ i){
        pow2[i] = 1LL * pow2[i - 1] * 2 % P;
    }
}
void init(){
    for(int i = 0; i < n; ++ i){
        for(int j = 0; j < n; ++ j){
            a[i][j] = A[i][j];
        }
    }
}
int Gauss(){
    int max_r, col, k;
    for(k = 0, col = 0; k < equ && col < var; ++ k, ++ col){
        max_r = k;
        for(int i = k + 1; i < equ; ++ i){
            if(abs(a[i][col]) > abs(a[max_r][col])){
                max_r = i;
            }
        }
        if(!a[max_r][col]){
            -- k;
            continue;
        }
        if(max_r != k){
            for(int j = k; j < var + 1; ++ j){
                swap(a[k][j], a[max_r][j]);
            }
        }
        for(int i = k + 1; i < equ; ++ i){
            if(a[i][col]){
                for(int j = col; j < var + 1; ++ j){
                    a[i][j] ^= a[k][j];
                }
            }
        }
    }
    for(int i = k; i < equ; ++ i){
        if(a[i][col]){
            return - 1;
        }
    }
    if(k < var)return var - k;//自由变元个数。
    return 0;
}
int main(){
    scanf("%d", &n);
    cal(n);
    for(int i = 0; i < n; ++ i){
        for(int j = 0; j < n; ++ j){
            scanf("%d", &A[i][j]);
        }
    }
    for(int i = 0; i < n; ++ i){
        for(int j = 0; j < n; ++ j){
            scanf("%d", &B[i][j]);
        }
    }
    equ = var = n;
    ll res = 1;
    for(int j = 0; j < n; ++ j){
        //枚举第一行乘c的解。
        init();
        for(int i = 0; i < n; ++ i){
            a[i][i] = (A[i][i] - B[i][j] + 2) % 2;
        }
        free_num = Gauss();
        if(free_num != -1){
            res = res * pow2[free_num] % P;
        }
    }
    printf("%lld\n", res);
    return 0;
}
```

# C.	Stone Game
* **题意**
有$a_1$个一颗石头的堆，$a_2$个两颗石头的堆，有$a_3$个三颗石头的堆。你可以将两个堆进行合并，合并后石头数量为两堆石头数量之和$(x+y)$，代价为$(x\mod 3)(y\mod 3)$。你需要将它们合并最后变成一个堆，问最小代价总和。

* **解题思路**
我们注意到，用$x\mod 3=0$的堆去合并，是不需要任何代价的，所以实际上我们可以不用考虑$x\mod 3=0$的堆。那么考虑其他两个，我们发现，用余数为$1$和余数为$2$的进行合并是最优的，因为代价少，且可以合成一个不用考虑的堆。
所以我们的策略就是先抵消掉$min(a_1,a_2)$，然后对剩下那个不为$0$的堆进行消耗，也就是只剩下余数为$1$或者余数为$2$的堆，这个时候我们发现$3$个$3$个一起会最优，因为可以凑出一个$3$，且代价最小。故据此处理题解。

* **AC代码**

```cpp
#include<bits/stdc++.h>

using namespace std;
using ll = long long;
int a[3];
int main(){
    for(int i = 0; i < 3; ++ i)cin >> a[i];
    ll ans = 0;
    int x = min(a[0], a[1]);
    a[0] -= x, a[1] -= x;
    a[2] += x;
    ans += 2 * x;
    x = a[0] / 3;
    a[0] %= 3;
    ans += 3 * x;
    x = a[1] / 3;
    a[1] %= 3;
    ans += 6 * x;
    if(a[0] == 2)ans += 1;
    if(a[1] == 2)ans += 4;
    cout << ans << endl;
    return 0;
}
```
# D.	Fight against involution
* **题意**
有$n$个学生，每个学生论文字数为$w_i\in [L_i,R_i]$，若有$k_i$个学生$j\in [1,n]$满足$w_j>w_i$，那么学生$i$的分数为$n-k_i$。最开始所有学生写了$R_i$个字，现在你可以在每个学生分数不减小的情况下调整每个学生的论文字数，问最小的字数和是多少。
* **解题思路**
由题意得，上限$R$实际上就决定了每个学生的分数，所以我们只要控制原先大于他的$w_i$不变，即可。所以我们可以将$R$升序排列，那么同一个$R$的肯定是取最大的$L$（因为得分一样，取最大的$L$是为了让他们都能取到相同的值），但这个值同样需要大于等于前面$R$选定的值。
据此，我们实际上就可以用$last$变量来记录上一轮选定的字数，然后双指针滑过去取每一段$R$相同的区间即可。

* **AC代码**

```cpp
#include<bits/stdc++.h>
#define L second 
#define R first

using namespace std;
using pii = pair<int, int>;
using ll = long long;

const int N = 1e5 + 10;

int n;
pii a[N];
int main(){
    scanf("%d", &n);
    for(int i = 1; i <= n; ++ i){
        scanf("%d%d", &a[i].L, &a[i].R);
    }
    sort(a + 1, a + 1 + n);
    ll res = 0;
    int last = 0;
    for(int l = 1, r; l <= n; l = r){
        r = l;
        while(r <= n && a[r].R == a[l].R)++ r;
        //贪心取最左端。
        int tmp = max(last, a[r - 1].L);
        res += 1LL * (r - l) * tmp;
        last = tmp;
    }
    printf("%lld\n", res);
    return 0;
}
```

# G.Xor Transformation
* **题意**
给定$X,Y$，其中$Y<X$。现在你可以进行不超过$5$次的操作，每次操作可以选取值$A,A\in [0,X)$，然后使$X=X\ xor A$。问你是否可以将$X$变成$Y$。

* **解题思路**
如果我们直接让$X \ xor\ A=Y$，那么$A=X\ xor\ Y$，那么如果$A\in [0,X)$，那么直接一步操作即可。否则，说明$X\ xor\ Y>X$，而我们知道异或两次相同的值，那么值是不变的，所以我们可以让$X=X  \ xor  \ Y$，那么再异或一次$X$即可。

* **AC代码**

```cpp
#include<bits/stdc++.h>

using namespace std;
using ll = long long;
ll x, y;
int main(){
    cin >> x >> y;
    if((x ^ y) > x){
        puts("2");
        printf("%lld %lld", y, x);   
    }
    else{
        puts("1");
        printf("%lld\n", x ^ y);
    }
    return 0;
}
```
# J.Tree Constructer
* **题意**
有一颗$n$个结点的数，需要你构造结点的权值$a_i$，使得边$(x,y)$存在当且仅当$a_x| a_y=2^{60}-1$。

* **解题思路**
我们可以先利用染色法构造二分图，将其分成左部和右部，为了让这两部分内的顶点互不影响，所以我们可以让左部顶点空出第一位，右部顶点空出第二位，这样可以保证左部顶点与左部顶点、右部顶点与右部顶点没有边。然后对于左部顶点与右部顶点的关系，针对每个左部顶点，我们可以拿出一位，然后让右部顶点中与该点没有边的点该位都为$0$，这样能够保证题目要求。由于我们是要在左部顶点上拿位，所以我们要使得左部顶点数量尽可能少，所以如果左部比右部多，交换一下即可。具体看$AC$代码。

* **AC代码**

```cpp
#include<bits/stdc++.h>

using namespace std;
using ll = long long;

int n;
bool g[110][110];
vector<int> p[2];//二分图。
int color[110];//color[u]表示u所染的颜色
void dfs(int u, int fu, bool op){
    p[op].push_back(u);
    color[u] = op;
    for(int v = 1; v <= n; ++ v){
        if(v == fu)continue;
        if(g[u][v])dfs(v, u, op ^ 1);
    }
}
void solve(){
    dfs(1, -1, 0);
    //因为我们是通过p[0]来构造p[1]。
    if(p[0].size() > p[1].size())swap(p[0], p[1]);
    vector<ll> res(n + 1, (1LL << 60) - 1);
    for(auto u : p[0])res[u] ^= 1;
    for(auto v : p[1])res[v] ^= 2;
    ll x = 4;
    for(auto u : p[0]){
        res[u] ^= x;
        for(auto v : p[1]){
            //将没有联系的且处于不同部的都清除这个点。
            if(!g[u][v] && color[u] != color[v])res[v] ^= x;
        }
        x *= 2;
    }
    for(int i = 1; i <= n; ++ i){
        printf("%lld ", res[i]);
    }
    puts("");
}
int main(){
    scanf("%d", &n);
    for(int i = 1, u, v; i < n; ++ i){
        scanf("%d%d", &u, &v);
        g[u][v] = g[v][u] = true;
    }
    solve();
    return 0;
}
```
# L.Bit Sequence
* **题意**
定义$f(x)$为二进制下1的数量，现在给定长度为m的01序列a，你需要找出有多少$x(x\in [0, L])$满足对于所有$a_i$，都满足$f(x + i) \mod 2 = a_i$ 。

* **解题思路**
由于$m$特别小，其只会影响后面的七位，而对从第八位开始的位也只有进位时才会影响，所以我们可以暴力处理后七位的情况，然后我们可以用$cnt$来记录从第八位开始的连续$1$的个数，同时，用$sum$表示从$x$第八位开始的数位之和，用$f(i,j)$后面的七位数和$i$进位相加的值二进制下$1$的个数。所以，如果进位，则$f(x+i)=sum-cnt+f(i+j)$（因为进位的$1$被抵消了，我们这里多算了cnt）；如果不进位的话，则$f(x+i)=sum+f(i+j)$。针对$f(i+j)$，由于其不会超过$256$，所以我们可以预处理出$256$一下的函数值。然后，同时由于是要对$2$取余，所以我们实际上只在乎其奇偶性，同时由于加减法我们可以用异或运算实现，所以我们可以统一按异或处理。
经过以上分析，且此题就是数位$dp$的风格，所以我们可以从高位到低位依次考虑，用$dp[pos][sum][cnt][limit]$来表示当前处理位为$pos$且数位和奇偶性为$sum$，连续$1$个数奇偶性为$cnt$，$limit$表示该位是否限定了上界的方案数。
故我们需要开$dp[65][2][2][2]$，由于状态数经过奇偶性处理后实际上非常少了，为了降低时间复杂度，减少不必要的搜索，我们可以使用记忆化搜索。
具体见$AC$代码。

* **AC代码**

```cpp
#include<bits/stdc++.h>

using namespace std;
using ll = long long;

int t, m, a[110], digit[65];
int f[300];//提前计算好256前的数位和奇偶性。
ll L, dp[65][2][2][2];
void init(){
    for(int i = 1; i <= 256; ++ i){
        f[i] = (f[i >> 1] + (i & 1)) % 2;
    }
}
ll cal(bool sum, bool cnt, bool limit){
    int up = limit ? L % 128 : 127;
    ll ans = 0;
    for(int i = 0; i <= up; ++ i){
        bool flag = true;
        for(int j = 0; j < m && flag; ++ j){
            if(i + j < 128){
                //说明没有进位。
                flag = (f[i + j] ^ sum) == a[j];
            }
            else{
                //说明存在进位，此时需要将多余的cnt给删掉。
                flag = (f[i + j] ^ cnt ^ sum) == a[j];
            }
        }
        ans += flag;
    }
    return ans;
}
ll dfs(int pos, bool sum, bool cnt, bool limit){
    //pos为当前正在处理的为，sum为当前数位和奇偶性，cnt为从第八位开始连续1的个数的奇偶性，limit为是否限定上界。
    ll &x = dp[pos][sum][cnt][limit];
    //cout << pos << " " << sum << " " << cnt << " " << limit << endl;
    if(~x)return x;
    if(pos <= 6){
        return x = cal(sum, cnt, limit);
    }
    int up = limit ? digit[pos] : 1;
    ll ans = 0;
    for(int i = 0; i <= up; ++ i){
        int one = cnt;
        if(i)one ^= 1;
        else one = 0;
        ans += dfs(pos - 1, sum ^ i, one, limit && i == up);
    }
    return x = ans;
}
void solve(){
    int len = 0;
    memset(dp, -1, sizeof(dp));
    for(int i = 63; i >= 0; -- i){
        digit[i] = (L >> i) & 1;
        if(digit[i]){
            len = max(len, i);
        }
    }
    printf("%lld\n", dfs(len, 0, 0, 1));
}
int main(){
    scanf("%d", &t);
    init();
    while(t -- ){
        scanf("%d%lld", &m, &L);
        for(int i = 0; i < m; ++ i){
            scanf("%d", &a[i]);
        }
        solve();
    }
    return 0;
}
```

# M.Cook Pancakes!
* **题意**
有$n$个煎饼，$k$个锅，每个煎饼需要煎两面才可以熟，煎一面需要$1s$，问将煎饼全煎熟最小需要多少分钟。

* **解题思路**
经典小学数学问题，即是相当于有$2n$个数：$1...n,-1....-n$。每次可以删掉$k$个数，其中$x,-x$不能一起删，问最少几次删完，按照上述，顺序删过去即可，答案则是$\lceil\frac{2n}{k}\rceil$。注意$k>n$两次就可以删除。

* **AC代码**

```cpp
#include<bits/stdc++.h>

using namespace std;

int n, k;
int main(){
    cin >> n >> k;
    if(k > n){
        cout << 2 << endl; 
    }
    else{
        cout << (2 * n + k - 1) / k << endl;
    }
    return 0;
}
```
