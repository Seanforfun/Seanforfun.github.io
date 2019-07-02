---
layout: post
title:  "Algorithm | Stock Buy and Sell Questions"
date:   2019-07-02 16:19
author: Botao Xiao
categories: Algorithm
comment: true
description: Leetcode stock buy and sell questions.
---

### Methodology
1. Always use dp.
2. Correct initialization.
3. Figure out the states and their relationship.(Transaction function)

### Question 1:
* [121. Best Time to Buy and Sell Stock](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/121.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock.md)
    1. O(n) time complexity
    ```Java
    class Solution {
        public int maxProfit(int[] prices) {
            int profit = 0, min = Integer.MAX_VALUE;
            for(int i = 0; i < prices.length; i++){
                min = Math.min(min, prices[i]);
                profit = Math.max(profit, prices[i] - min);
            }
            return profit;
        }
    }
    ```

### Question 2:
* [122. Best Time to Buy and Sell Stock II](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/122.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock%20II.md)
    * State transfer function:
        1. Holding a stock, decide to sell or keep it. buys[i] = max(buys[i - 1], sells[i - 1] - prices[i - 1])
        2. Not holding any stock, decide to buy or wait for a day. sells[i] = max(sells[i - 1], buys[i - 1] + prices[i - 1])
    ```Java
    class Solution {
        public int maxProfit(int[] prices) {
            int len = prices.length;
            int[] buys = new int[len + 1];
            int[] sells = new int[len + 1];
            buys[1] = -prices[0];
            for(int i = 2; i <= len; i++){
                buys[i] = Math.max(buys[i - 1], sells[i - 1] - prices[i - 1]);
                sells[i] = Math.max(sells[i - 1], buys[i - 1] + prices[i - 1]);
            }
            return sells[len];
        }
    }
    ```

### Question 3: With at most 2 transactions
* [123. Best Time to Buy and Sell Stock III](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/123.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock%20III.md)
    1. Initialization: 
        * buys[1][1] = -profit[0], buys[1][2] = Integer.MIN_VALUE.
    2. Transaction function:
        * buys[i][j] = max(buys[i - 1][j], sells[i - 1][j - 1] - profit[i - 1]), decide to unhold or buy at current day.
        * sells[i][j] = max(sells[i - 1][j], buys[i - 1][j] + profit[i - 1]), decide to hold or sell.
    ```Java
    class Solution {
        public int maxProfit(int[] prices) {
            if(prices == null || prices.length <= 1) return 0;
            int len = prices.length;
            int[][] buys = new int[len + 1][3];
            int[][] sells = new int[len + 1][3];
            buys[1][1] = -prices[0];
            buys[1][2] = Integer.MIN_VALUE;
            for(int i = 2; i <= len; i++){
                for(int j = 1; j <= 2; j++){
                    buys[i][j] = Math.max(buys[i - 1][j], sells[i - 1][j - 1] - prices[i - 1]);
                    sells[i][j] = Math.max(sells[i - 1][j], buys[i - 1][j] + prices[i - 1]);
                }
            }       
            return Math.max(sells[len][0], Math.max(sells[len][1], sells[len][2]));
        }
    }
    ```

### Question 4: K transactions
* [124. Best Time to Buy and Sell Stock IV](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/123.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock%20IV.md)
    1. We need to pay attention to the initialization.
	2. Transaction function is same as [123. Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/description/).
	3. If k is larger than prices length, we treat it as question [122. Best Time to Buy and Sell Stock II](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/122.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock%20II.md).
    ```Java
    class Solution {
        public int maxProfit(int k, int[] prices) {
            if(k == 0 || prices == null || prices.length <= 1) return 0;
            int len = prices.length;
            if(k > len) return maxProfit(prices);
            int[][] buys = new int[len + 1][k + 1];
            int[][] sells = new int[len + 1][k + 1];
            for(int i = 2; i <= k; i++){
                for(int j = 0; j < i; j++){
                    buys[j][i] = Integer.MIN_VALUE;
                }
            }
            int sum = 0;
            for(int i = 1; i <= k; i++){
                sum -= prices[i - 1];
                buys[i][i] = sum;
            }
            for(int i = 2; i <= len; i++){
                for(int j = 1; j <= k; j++){
                    buys[i][j] = Math.max(buys[i - 1][j], sells[i - 1][j - 1] - prices[i - 1]);
                    sells[i][j] = Math.max(sells[i - 1][j], buys[i - 1][j] + prices[i - 1]);
                }
            }        
            int profit = 0;
            for(int i = 0; i <= k; i++){
                profit = Math.max(profit, sells[len][i]);
            }
            return profit;
        }
        
        private int maxProfit(int[] prices) {
                int len = prices.length;
                int[] buys = new int[len + 1];
                int[] sells = new int[len + 1];
                buys[1] = -prices[0];
                for(int i = 2; i <= len; i++){
                    buys[i] = Math.max(buys[i - 1], sells[i - 1] - prices[i - 1]);
                    sells[i] = Math.max(sells[i - 1], buys[i - 1] + prices[i - 1]);
                }
                return sells[len];
            }
    }
    ```

### Question 5: With cooldown
* [309. Best Time to Buy and Sell Stock with Cooldown](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/309.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock%20with%20Cooldown.md)
    1. We just need to write correct transaction function
    ```Java
    class Solution {
        public int maxProfit(int[] prices) {
            if(prices == null || prices.length <= 1) return 0;
            int len = prices.length;
            int[] buys = new int[len + 1];
            int[] sells = new int[len + 1];
            int[] cools = new int[len + 1];
            buys[1] = -prices[0];
            for(int i = 2; i <= len; i++){
                buys[i] = Math.max(buys[i - 1], cools[i - 1] - prices[i - 1]);
                cools[i] = Math.max(cools[i - 1], sells[i - 1]);
                sells[i] = Math.max(buys[i - 1] + prices[i - 1], sells[i - 1]);
            }
            return sells[len];
        }
    }
    ```

### Question 6: with fee
* [714. Best Time to Buy and Sell Stock with Transaction Fee](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/714.%20Best%20Time%20to%20Buy%20and%20Sell%20Stock%20with%20Transaction%20Fee.md)
    ```Java
    class Solution {
        public int maxProfit(int[] prices, int fee) {
            if(prices == null || prices.length <= 1) return 0;
            int len = prices.length;
            int[] buys = new int[len + 1];
            int[] sells = new int[len + 1];
            buys[1] = -prices[0];
            for(int i = 2; i <= len; i++){
                buys[i] = Math.max(buys[i - 1], sells[i - 1] - prices[i - 1]);
                sells[i] = Math.max(sells[i - 1], buys[i - 1] + prices[i - 1] - fee);
            }
            return sells[len];
        }
    }
    ```