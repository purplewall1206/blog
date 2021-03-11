---
title: dynamicprogramming.md
date: 2020-11-15 18:36:17
tags: [leetcode,algorithm]
categories: 
- leetcode
---

# 动态规划 Dynamic Programming

动态规划(dynamic programming)是运筹学的一个分支，是求解决策过程(decision process)最优化的数学方法。20世纪50年代初美国数学家R.E.Bellman等人在研究多阶段决策过程(multistep decision process)的优化问题时，提出了著名的最优化原理(principle of optimality)，把多阶段过程转化为一系列单阶段问题，利用各阶段之间的关系，逐个求解，创立了解决这类过程优化问题的新方法——动态规划。1957年出版了他的名著《Dynamic Programming》，这是该领域的第一本著作。

## tricky
一般不要求记录解空间具体内容的，但是看起来好像需要从所有解中提取结果的都可以试试看动态规划，设计不好dp不妨试试看从0到i的物品的xxx，dp有可能是1维2维甚至3维。

如果要求解空间具体内容的上backtrack也不错，而且dp的题也可以用backtrack或者dfs碰碰运气。

<!-- more -->
## 硬币找零问题 coin change
[leetcode-322](https://leetcode-cn.com/problems/coin-change/)

>You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.  
>> Example 1:    
>> Input: coins = [1, 2, 5], amount = 11    
>> Output: 3       
>> Explanation: 11 = 5 + 5 + 1      


解析：每种coins的数量是不限的，因此假设dp[i]是从0到金额i，所需要的最小硬币的数量
初始化假设dp值均为 amount+1
`dp[i] = min(dp[i-coin[x]]+1, dp[i])`
循环遍历每个coin的值，获得dp最后的值，即相应的结果

```
int coinChange(vector<int>& coins, int amount) {
        // dp[i] = min(dp[i-coink]+1, dp[i])
        if (amount == 0)
            return 0;
        vector<int> dp(amount+1, amount+1);
        // 此处初始化为amount+1，便于之后取最小值
        dp[0] = 0;
        for (int i = 1;i <= amount;i++) {
            for (int j = 0;j < coins.size();j++) {
                if (i >= coins[j]) {
                    // cout << i << "  " << coins[j] <<"  " << dp[i] << "  ";
                    dp[i] = min(dp[i], dp[i-coins[j]]+1);
                    // cout << dp[i] << endl;
                }
            }
        }
        return (dp[amount] == amount+1) ? -1 : dp[amount];
    }
```


## 01背包问题 

01背包是解决什么问题？

答：当书包容量大小固定，面对1堆重量固定而且带有价格的物品。在不超出包容量前提下，选择那些物品使包里物品总价值最大。

其中代表重量的数组 w[0...i....n], 代表价值的数组 v[0...i...n]。

**一般可以通过画表格确定解空间（还没研究明白）**

动态规划的思路是设计 dp[N+1][S+1] 其中N代表物品数量，S代表背包总重量，即在N件物品，重量为S的情况下，物品的价值多大
当w[i-1] < j(当前剩余重量)时间
分别代表装入或不装入
`dp[i][j] = min(dp[i-1][j], dp[i-1][j-w[i-1]]+v[i-1])`


### 例题 target sum
[leetcode-494](https://leetcode-cn.com/problems/target-sum/)

> You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.  
> Find out how many ways to assign symbols to make sum of integers equal to target S.  
>> Example 1:  
>> Input: nums is [1, 1, 1, 1, 1], S is 3.   
>> Output: 5  
>> Explanation:   
>>> -1+1+1+1+1 = 3  
>>> +1-1+1+1+1 = 3  
>>> +1+1-1+1+1 = 3  
>>> +1+1+1-1+1 = 3  
>>> +1+1+1+1-1 = 3  
>>> There are 5 ways to assign symbols to make the sum of nums be target 3.  

根据题意可以假设集合nums有两个真子集A,B，其中集合A表示所有为正数的集合，集合B表示所有未负数的集合，目标值未target，那么有
```
sum(A) + sum(B) = sum(nums)
sum(A) - sum(B) = target
2 * sum(A) = sum(nums) + target
```
由上述递推公式将该问题转化为01背包问题，题解如下

```
int findTargetSumWays(vector<int>& nums, int S) {
        int n = nums.size();
        long sum = 0;
        for (int i : nums) sum += i;
        if ((S + sum) % 2 == 1 || S > sum) {
            return 0;
        } 
        S = (sum + S) / 2;
        // 传统dp
        // 对于dp(i,j)就表示可选物品为i到n且背包容量为j(总重量)时背包中所放物品的最大价值
        vector<vector<int>> dp(n+1, vector<int>(S+1, 0));
        // 0个数和为0的个数为1
        dp[0][0] = 1;
        for (int i = 1;i <=n;i++ ) {
            for (int j = 0;j < S+1;j++) {
                if(j-nums[i-1] < 0)//背包容量不足，不能放入第i个物品
                {
                    dp[i][j] = dp[i-1][j];//其实就是表格的左边界
                }
                else
                {
                    dp[i][j] = dp[i-1][j-nums[i-1]] + dp[i-1][j];//装入第i个物品或者不装入
                }
            }
        }
        return dp[n][S];      
    }
```

根据上述代码我们发现 `dp[i][j] = dp[i-1][...]` 的关系，即i至于i-1有关系，因此我们可以减少一维向量，用一维向量记录上一轮的运算结果。

```
int findTargetSumWays(vector<int>& nums, int S) {
        int n = nums.size();
        long sum = 0;
        for (int i : nums) sum += i;
        if ((S + sum) % 2 == 1 || S > sum) {
            return 0;
        } 
        S = (sum + S) / 2;
        // dp优化
        vector<int> dp(S+1, 0);
        dp[0] = 1;
        for (int i = 0;i < n;i++) {
            for (int j = S;j >= nums[i];j--) {
                dp[j] += dp[j-nums[i]];
            }
        } 
        return dp[S];
        
    }
```
