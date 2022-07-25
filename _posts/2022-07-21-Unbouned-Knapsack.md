---
title: "Unbounded Knapsack"
categories: [Concepts, Dynamic Programming]
tags: [knapsack]     # TAG names should always be lowercase
author: Ayush Koul
layout: post
toc: true
math: true
---

### Problem Statement:
We are given an array of `n` items, each with a `weight` ($\boldsymbol{W_i}$) and a `profit` ($\boldsymbol{P_i}$),and a bag (aka knapsack) which has a `limited capacity` of $\boldsymbol{C}$ (where $\boldsymbol{C}$ is the maximum amount of weight that the bag can hold). The goal is to find a subset of the items such that their `total weight is <= C` and the sum of their profits is `maximum`. 

Each item has an infinite quantity and can be used any amount of times.

Given:

- $C$
- $W : [W_0, W_1, ... W_{n-1}] \ (Size = n)$
- $P : [P_0, P_1, ... P_{n-1}] \ (Size = n)$

> Note: Each item can be used **any number of times** (unlike 0/1 Knapsack)
{: .prompt-warning }

### Brute-Force/Recursive Approach
Unlike [0/1 Knapsack]({% post_url 2022-06-29-01-Knapsack %}), we can use an item any number of times: $0, 1, ... \infty$. Therefore, here the approach is slightly different. Here, the main idea still remains the same: for each item, we have 2 choices: For each item, we can make `2 choices`: to `include` it or to `exclude` it. However, the only difference is, if we `include` an item, we don't move on. We have to also consider the possibility that it can be included again (Therefore, when we include an item, we dont decrement `i`):

#### Code

```cpp
#include <bits/stdc++.h>
using namespace std;
/**
 * @param W array of item weights
 * @param P array of item profits
 * @param Space current capacity of the bag (before choosing item i)
 * @param i current item index (start from n-1)
*/
int knapsack(const vector<int> &W, const vector<int> &P, int Space, int i)
{
    if (i == -1 || Space == 0)
        return 0;
    if (W[i] > Space) 
        return knapsack(W, P, Space, i - 1); //Exclude the item
    else
        return max(knapsack(W, P, Space, i - 1),            //Exclude the item
                   P[i] + knapsack(W, P, Space - W[i], i)); //Include the item
}

int main()
{
    vector<int> W = {7, 2, 4}, P = {10, 5, 6};
    int C = 7;
    cout << knapsack(W, P, C, W.size() - 1);

    return 0;
}
```
#### Complexity Analysis

- **Time complexity: $\boldsymbol{O(2^n)}$** - Size of recursion tree will be $\boldsymbol{2^n}$
- **Space complexity: $\boldsymbol{O(n)}$** - The depth of the recursion tree can go up to $\boldsymbol{n}$.






### Recursive (Top-down) + Memoization
Here, we will just cache each result so it is not re-computed

#### Code
```cpp
#include <bits/stdc++.h>
using namespace std;

// A hash function used to hash a pair of any kind
struct hash_pair {
    template <class T1, class T2>
    size_t operator()(const pair<T1, T2>& p) const
    {
        auto hash1 = hash<T1>{}(p.first);
        auto hash2 = hash<T2>{}(p.second);
        return hash1 ^ hash2;
    }
};
unordered_map<pair<int, int>, int, hash_pair> dp;

/**
 * @param W array of item weights
 * @param P array of item profits
 * @param Space current capacity of the bag (before choosing item i)
 * @param i current item index (start from n-1)
*/
int knapsack(const vector<int> &W, const vector<int> &P, int Space, int i)
{
    if(dp.find({i, Space}) != dp.end())
        return dp[{i, Space}];
    if (i == -1 || Space == 0)
        return dp[{i, Space}] = 0;
    if (W[i] > Space)
        return dp[{i, Space}] = knapsack(W, P, Space, i - 1); //Exclude the item
    else
        return dp[{i, Space}] = max(knapsack(W, P, Space, i - 1),            //Exclude the item
                                    P[i] + knapsack(W, P, Space - W[i], i)); //Include the item
}

int main()
{
    vector<int> W = {7, 2, 4}, P = {10, 5, 6};
    int C = 7;
    cout << knapsack(W, P, C, W.size() - 1);

    return 0;
}
```

#### Complexity Analysis

- **Time complexity: $\boldsymbol{O(n\*C)}$** - This is because the recursive function will only branch or call 
other recursive functions if it hasn't cached the query yet. The number of possible queries is $\boldsymbol{n\*C}$
(For each item, we have to explore for all `capacities <= C`)
- **Space complexity: $\boldsymbol{O(n + \text{size of recursive call stack})}$** - We are caching $\boldsymbol
{n\*C}$ results in our Hash Map.

### Bottom-Up (Tabular)
In order to understand this approach, we need to refer back to the recurrence relation (how and when we call the
recursive calls) we used in the Top-down method.
```
//Base case:
dp(i, 0) = 0
dp(-1, Space) = 0

//Recurrence Relation
if (W[i] > Space)
    dp(i, Space) = dp(i-1, Space)
else
    dp(i, Space) = max(dp(i-1, Space), P[i] + dp(i, Space - W[i]))
```
We know that `i` will be in the range of `[0, n-1]` and Space will be in the range of `[1, C]`. We can use this to 
change dp into a 2D matrix/array for caching the results. In addition, we will be generating the answer from the 
bottom up (i.e. calculating and storing the results of smaller sub problems then using those to calculate and store 
the larger ones). In top down, we start from the top (the answer we need, and recursively call the small sub 
problems), but here, we start from the bottom.

One more point to note is that, our base case checks to see if `i` is -1 which we can't use if we have an array 
(since they only go to index 0). Therefore, we will shift the values of `i` by +1 (make it 1 indexed) and reserve 0 
for the -1 case (basically, in the code, `dp[0][Space] = 0` since 0 works like -1)

#### Code
```cpp
#include <bits/stdc++.h>
using namespace std;
/**
 * @param W array of item weights
 * @param P array of item profits
 * @param C capacity of the bag
*/
int knapsack(const vector<int> &W, const vector<int> &P, int C)
{
    int n = W.size();

    vector<vector<int>> dp(n + 1, vector<int>(C + 1));
    //dp[0][Space] = 0 (by default in vector)
    //dp[i][0] = 0 (by default in vector)
    for (int i = 1; i <= n; ++i)
    {
        for (int Space = 1; Space <= C; ++Space)
        {
            if (W[i - 1] > Space)
                dp[i][Space] = dp[i - 1][Space];
            else
                dp[i][Space] = max(dp[i - 1][Space], P[i - 1] + dp[i][Space - W[i - 1]]);
        }
    }

    return dp[n][C];
}
int main()
{
    vector<int> W = {7, 2, 4}, P = {10, 5, 6};
    int C = 7;
    cout << knapsack(W, P, C, W.size() - 1);

    return 0;
}
```
#### Complexity Analysis

- **Time complexity: $\boldsymbol{O(n\*C)}$** - We have 2 nested for loops that run in total of $\boldsymbol{n\*C}$ 
times.
- **Space complexity: $\boldsymbol{O(n\*C)}$** - We are caching $\boldsymbol{n\*C}$ results in our 2D matrix/array.