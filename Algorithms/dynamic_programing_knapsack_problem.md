# 動態規劃 - 背包問題

有一個容量為 capacity 的背包，有 n 件物品，第 i 件物品重 weight[i]，第 i 件物品價值 value[i]

01 背包 - 每個物品只有一個，求能放入背包的最大價值  
完全背包 - 每個物品無限多個，求能放入背包的最大價值  
多重背包 - 每個物品的數目有限，求能放入背包的最大價值  

01 背包、完全背包、多重背包在二維陣列都可以從小到大走迴圈，但是在一維陣列  
01 背包的內層迴圈必須從大走到小  
完全背包的內層迴圈必須從小走到大  
多重背包的內層迴圈必須從大走到小  

## 01 背包 - 0/1 Knapsack Problem

背包重量為 4  
有四個物品，分別為青銅、白銀、黃金、鑽石  
重量分別為 1、2、3、4  
價值分別為 100、130、160、240

令 dp[i][j] 從 i 個物品選擇放入容量 j 的背包的最大價值

dp[i][j] 的來源可從以下選出最大:
- 不選物品<sub>i</sub> 或者是塞不下，從 i-1 個物品選擇放入容量 j 的背包的最大價值，也就是 dp[i-1][j]
- 可選物品<sub>i</sub>，從 i-1 個物品選擇放入容量 ( j - 物品重量<sub>i</sub> ) 的背包的最大價值 加上 物品價值<sub>i</sub>，也就是 dp[i-1][j - weight[i]] + value[i] -> <b>注意這裡是各個背包解法不一樣的地方</b>

考慮到 j 可能放不下物品<sub>i</sub>，dp[i][j] 的堆導改寫如下
- 不選物品<sub>i</sub> 或者是塞不下，只有一種選擇，也就是 dp[i-1][j]
- 可選物品<sub>i</sub>，從兩者中選最大，也就是 Max( dp[i-1][j], dp[i-1][j - weight[i]] + value[i] )

| 縱座標為物品<br/>橫座標為背包 | 0 | 1   | 2   | 3   | 4   |
|-------------------------------|---|-----|-----|-----|-----|
| <b>0</b>                      | 0 | 0   | 0   | 0   | 0   |
| <b>青銅</b>                   | 0 | 100 | 100 | 100 | 100 |
| <b>白銀</b>                   | 0 | 100 | 130 | 230 | 230 |
| <b>黃金</b>                   | 0 | 100 | 130 | 230 | 260 |
| <b>鑽石</b>                   | 0 | 100 | 130 | 230 | 260 |

以二維陣列來說，可以寫成下列程式碼
```csharp
for (var i = 1; i <= n; i++)
{
    for (var j = 1; j <= capacity; j++)
    {
        if (j < weight[i])
        {
            dp[i][j] = dp[i - 1][j];
        }
        else
        {
            dp[i][j] = Math.Max(dp[i - 1][j], dp[i - 1][j - value[i]] + value[i]);
        }
    }
}
```

如果想改寫用一維陣列，要注意用來計算的值不能先被蓋掉，那麼就要注意 j - value[i]，這個值算出來肯定比 j 小，如果我們仍維持從小到大走迴圈，會導致被拿來計算值已先被蓋掉。  
所以我們必須把內層迴圈反過來，改從大走到小，才可以避免要被拿來計算的值先被蓋掉
```csharp
for (var i = 1; i <= n; i++)
{
    for (var j = capacity; j >= 1; j--)
    {
        if (j <= weight[i])
        {
            dp[j] = Math.Max(dp[j], dp[j - value[i]] + value[i]);
        }
    }
}
```

---

## 完全背包 - Unbounded Knapsack Problem

背包重量為 4  
有四個物品，分別為青銅、白銀、黃金、鑽石  
重量分別為 1、2、3、4  
價值分別為 100、130、160、240

令 dp[i][j] 從 i 個物品選擇放入容量 j 的背包的最大價值

dp[i][j] 的來源可從以下選出最大:
- 不選物品<sub>i</sub> 或者是塞不下，從 i-1 個物品選擇放入容量 j 的背包的最大價值，也就是 dp[i-1][j]
- 可選物品<sub>i</sub>，從 i 個物品選擇放入容量 ( j - 物品重量<sub>i</sub> ) 的背包的最大價值 加上 物品價值<sub>i</sub>，也就是 dp[i][j - weight[i]] + value[i] -> <b>注意這裡是各個背包解法不一樣的地方</b>

考慮到 j 可能放不下物品<sub>i</sub>，dp[i][j] 的堆導改寫如下
- 不選物品<sub>i</sub> 或者是塞不下，只有一種選擇，也就是 dp[i-1][j]
- 可選物品<sub>i</sub>，從兩者中選最大，也就是 Max( dp[i-1][j], dp[i][j - weight[i]] + value[i] )

| 縱座標為物品<br/>橫座標為背包 | 0 | 1   | 2   | 3   | 4   |
|-------------------------------|---|-----|-----|-----|-----|
| <b>0</b>                      | 0 | 0   | 0   | 0   | 0   |
| <b>青銅</b>                   | 0 | 100 | 200 | 300 | 400 |
| <b>白銀</b>                   | 0 | 100 | 200 | 300 | 400 |
| <b>黃金</b>                   | 0 | 100 | 200 | 300 | 400 |
| <b>鑽石</b>                   | 0 | 100 | 200 | 300 | 400 |

以二維陣列來說，可以寫成下列程式碼
```csharp
for (var i = 1; i <= n; i++)
{
    for (var j = 1; j <= capacity; j++)
    {
        if (j < weight[i])
        {
            dp[i][j] = dp[i - 1][j];
        }
        else
        {
            dp[i][j] = Math.Max(dp[i - 1][j], dp[i][j - value[i]] + value[i]);
        }
    }
}
```

如果想改寫用一維陣列，要注意用來計算的值不能先被蓋掉，那麼就要注意 j - value[i]，這個值算出來肯定比 j 小，我們必須從小到大走迴圈，才能保證被拿來計算值是正確的。  
```csharp
for (var i = 1; i <= n; i++)
{
    for (var j = 1; j <= capacity; j++)
    {
        if (j <= weight[i])
        {
            dp[j] = Math.Max(dp[j], dp[j - value[i]] + value[i]);
        }
    }
}
```

---

## 多重背包 - bounded Knapsack Problem

背包重量為 4  
有四個物品，分別為青銅、白銀、黃金、鑽石  
重量分別為 1、2、3、4  
數量分別為 3、3、3、3  
價值分別為 100、130、160、240

令 dp[i][j] 從 i 個物品選擇放入容量 j 的背包的最大價值

dp[i][j] 的來源可從以下選出最大:
- 不選物品<sub>i</sub> 或者是塞不下，從 i-1 個物品選擇放入容量 j 的背包的最大價值，也就是 dp[i-1][j]
- 可選物品<sub>i</sub>，從 i-1 個物品選擇放入容量 ( j - k * 物品重量<sub>i</sub> ) 的背包的最大價值 加上 k * 物品價值<sub>i</sub>，0 <= k <= 物品數量限制<sub>i</sub>，也就是 dp[i-1][j - k * weight[i]] + k * value[i]，0 <= k <= 數量[i] -> <b>注意這裡是各個背包解法不一樣的地方</b>

考慮到 j 可能放不下物品<sub>i</sub>，dp[i][j] 的堆導改寫如下
- 不選物品<sub>i</sub> 或者是塞不下，只有一種選擇，也就是 dp[i-1][j]
- 可選物品<sub>i</sub>，從以下選最大，也就是 Max( dp[i-1][j], dp[i-1][j - k * weight[i]] + k * value[i] )，0 <= k <= 數量[i]

| 縱座標為物品<br/>橫座標為背包 | 0 | 1   | 2   | 3   | 4   |
|-------------------------------|---|-----|-----|-----|-----|
| <b>0</b>                      | 0 | 0   | 0   | 0   | 0   |
| <b>青銅</b>                   | 0 | 100 | 200 | 300 | 300 |
| <b>白銀</b>                   | 0 | 100 | 200 | 300 | 330 |
| <b>黃金</b>                   | 0 | 100 | 200 | 300 | 330 |
| <b>鑽石</b>                   | 0 | 100 | 200 | 300 | 330 |

以二維陣列來說，可以寫成下列程式碼
```csharp
for (var i = 1; i <= n; i++)
{
    for (var j = 1; j <= capacity; j++)
    {
        for(var k = 0; k <= quantity[i-1] && k * weight[i-1] <= j; k++)
        {
            dp[i][j] = Math.Max(dp[i][j], dp[i-1][j - k * weight[i-1]] + k * value[i-1])
        }
    }
}
```

如果想改寫用一維陣列，要注意用來計算的值不能先被蓋掉，那麼就要注意 j - k * weight[i-1]，這個值算出來肯定比 j 小，如果我們仍維持從小到大走迴圈，會導致被拿來計算值已先被蓋掉。  
所以我們必須把內層迴圈反過來，改從大走到小，才可以避免要被拿來計算的值先被蓋掉
```csharp
for (var i = 1; i <= n; i++)
{
    for (var j = capacity; j >= 1; j--)
    {
        for(var k = 1; k <= quantity[i-1] && k * weight[i-1] <= j; k++)
        {
            dp[i][j] = Math.Max(dp[i][j], dp[i-1][j - k * weight[i-1]] + k * value[i-1])
        }
    }
}
```