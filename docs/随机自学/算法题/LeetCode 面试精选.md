# LeetCode 面试精选

## 面试 150 题

### 数组/字符串

#### [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

给你两个按 **非递减顺序** 排列的整数数组 `nums1` 和 `nums2`，另有两个整数 `m` 和 `n` ，分别表示 `nums1` 和 `nums2` 中的元素数目。

请你 **合并** `nums2` 到 `nums1` 中，使合并后的数组同样按 **非递减顺序** 排列。

**注意：** 最终，合并后数组不应由函数返回，而是存储在数组 `nums1` 中。为了应对这种情况，`nums1` 的初始长度为 `m + n`，其中前 `m` 个元素表示应合并的元素，后 `n` 个元素为 `0` ，应忽略。`nums2` 的长度为 `n` 。

题解——拆分为每个元素单独插入，维护一个已经合并完成的数组范围 `[start, end]`，然后遍历第二个数组将其插入第一个数组对应的范围内。

```cpp
class Solution {
public:
  void merge(vector<int> &nums1, int m, vector<int> &nums2, int n) {
    int start = 0, end = m;
    for (int i = 0; i < nums2.size(); i++) {
      merge(nums1, start, end, nums2[i]);
    }
  }

  void merge(vector<int> &nums1, int &start, int &end, int num) {
  	// 注意循环条件包括 start < end 限制
    while (start < end && nums1[start] < num) {
      start++;
    }

    for (int i = end; i > start; i--) {
      nums1[i] = nums1[i - 1];
    }

	// 插入新元素，并移动边界
    nums1[start] = num;
    end++;
  }
};
```



#### [27. 移除元素](https://leetcode.cn/problems/remove-element/)

给你一个数组 `nums` 和一个值 `val`，你需要 **[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 移除所有数值等于 `val` 的元素。元素的顺序可能发生改变。然后返回 `nums` 中与 `val` 不同的元素的数量。

假设 `nums` 中不等于 `val` 的元素数量为 `k`，要通过此题，您需要执行以下操作：

- 更改 `nums` 数组，使 `nums` 的前 `k` 个元素包含不等于 `val` 的元素。`nums` 的其余元素和 `nums` 的大小并不重要。
- 返回 `k`。

题解——将符合要求的元素扔到数组尾部，因此需要维护一个尾部起点位置 `end`，循环范围不超过 `end`；需要注意交换元素后移动 `end`，此时当前元素可能依然符合要求，所以必须是循环；最后要确保 `end` 移动后不会反超过 `i`，导致破坏前面已完成的部分。

```cpp
class Solution {
public:
  int removeElement(vector<int> &nums, int val) {
    int end = nums.size();
    for (int i = 0; i < end; i++) {
      while (end > i && nums[i] == val) {
        swap(nums[i], nums[end - 1]);
        end--;
      }
    }
    return end;
  }
};
```



#### [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)

给你一个 **非严格递增排列** 的数组 `nums` ，请你 **[原地](http://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。元素的 **相对顺序** 应该保持 **一致** 。然后返回 `nums` 中唯一元素的个数。

考虑 `nums` 的唯一元素的数量为 `k` ，你需要做以下事情确保你的题解可以被通过：

- 更改数组 `nums` ，使 `nums` 的前 `k` 个元素包含唯一元素，并按照它们最初在 `nums` 中出现的顺序排列。`nums` 的其余元素与 `nums` 的大小不重要。
- 返回 `k` 。

题解——维护一个步长 `step`，每次将 `i` 元素向前移动 `step`，此时 `[0, i - step - 1]` 是已经完成的区域范围。如果检测到当前元素等于完成区域尾部元素，就增加 `step` 。

```cpp
class Solution {
public:
  int removeDuplicates(vector<int> &nums) {
    int step = 0;
    for (int i = 1; i < nums.size(); i++) {
      nums[i - step] = nums[i];
      if (nums[i] == nums[i - step - 1])
        step++;
    }
    return nums.size() - step;
  }
};
```



#### [80. 删除有序数组中的重复项 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/)

给你一个有序数组 `nums` ，请你 **[原地](http://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 删除重复出现的元素，使得出现次数超过两次的元素**只出现两次** ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 **[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95) 修改输入数组** 并在使用 O (1) 额外空间的条件下完成。

题解——需要检查出现超过两次的元素，所以检查 `nums[i],nums[i-2]` 是否相同。如果相同，就需要将后面的元素全部向前移动一位。但这样会比较浪费，因为后面如果还有重复，就可以直接向前移动两位甚至更多，减少操作次数。我们记录已经处理好的数组边界 `end=2`，到达 `i` 位置时，比较 `nums[i],nums[end-2]`，即完成部分的倒数第二个元素，确保该元素出现不多于 2 次。如果不同就增加 `end` 移动边界，否则固定边界。

>[!note]
>我们在代码中使用 `step` 记录当前位置和需要比较位置之间的步长，这与维护 `end` 是相同的。实际上 `i - 2 - step = end - 2` 含义相同。

```cpp
class Solution {
public:
  int removeDuplicates(vector<int> &nums) {
    if (nums.size() < 2)
      return nums.size();
    int step = 0;
    for (int i = 2; i < nums.size(); i++) {
      nums[i - step] = nums[i];
      if (nums[i] == nums[i - 2 - step]) {
        step++;
      }
    }
    return nums.size() - step;
  }
};
```



#### [169. 多数元素](https://leetcode.cn/problems/majority-element/)

给定一个大小为 `n` 的数组 `nums` ，返回其中的多数元素。多数元素是指在数组中出现次数 **大于** `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

题解——使用哈希表当然可以用 `O(n)` 空间和时间复杂度完成，但有更高效的解法。

>[!note]
>Boyer-Moore 投票算法

基本思路：多数元素超过一半，因此将所有不同元素对消以后，一定只剩下多数元素。所以我们可以维护一个当前元素 `current` 和该元素的个数 `count`，遇到相同元素则 `count++`，否则 `count--` 表示对消。如果 `count` 为零，则更新当前元素。最后剩下的 `current` 就是多数元素（题目保证存在多数元素）。

```cpp
class Solution {
public:
  int majorityElement(vector<int> &nums) {
    int current;
    int count = 0;
    for (int i = 0; i < nums.size(); i++) {
      // 如果当前元素没了，就更新元素
      if (count == 0)
        current = nums[i];

	  // 判断增加还是对消
      if (nums[i] == current)
        count++;
      else
        count--;
    }
    return current;
  }
};
```



#### [189. 轮转数组](https://leetcode.cn/problems/rotate-array/)

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

题解——递归交换。首先做 `k % n` 取余，然后执行旋转。从倒数第 `k + 1` 个元素开始，将元素与之后的第 `k` 个元素交换。这么做最终会剩下前面 `k` 个元素以某种顺序被轮换，例如

```
1 2 3 4 -> k = 3
4 2 3 1
```

此时前面 3 个元素还需要被轮换 `k - n % k` 次，这是因为每 `k` 个元素交换时得到的结果顺序不变，而每多一个元素交换，就需要额外轮换 `k - 1` 次来恢复顺序。于是得到

```cpp
class Solution {
public:
  void rotate(vector<int> &nums, int k) {
    if (nums.size() <= 1)
      return;
    rotate(nums, nums.size(), k % nums.size());
  }

  void rotate(vector<int> &nums, int n, int k) {
  	// 退出条件
    if (n == 0 || k == 0)
      return;

    for (int i = n - k - 1; i >= 0; i--) {
      swap(nums[i], nums[i + k]);
    }
    int m = n % k;
    if (m != 0)
      rotate(nums, k, k - m);
  }
};
```

注意当 `k` 整除 `n` 时，前 `k` 个元素不需要再轮换。



#### [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

题解——只需要维护一个历史最低位，计算在历史最低位买入，在当天卖出时的收益即可。

```cpp
class Solution {
public:
  int maxProfit(vector<int> &prices) {
    int minVal = prices[0];
    int maxVal = 0;
    for (int i = 1; i < prices.size(); i++) {
      minVal = min(minVal, prices[i]);
      int d = prices[i] - minVal;
      if (d > maxVal) {
        maxVal = d;
      }
    }
    return maxVal;
  }
};
```



#### [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

给你一个整数数组 `prices` ，其中 `prices[i]` 表示某支股票第 `i` 天的价格。

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。

返回 _你能获得的 **最大** 利润_ 。

题解——动态规划。令 $S(n)$ 表示前 $n$ 天能够获得的最大收益，则
$$
S(n+1) = max(S(n),S(n) + prices[n] - prices[n-1] )
$$
即增加一天的收益最多是新一天减去前一天的收益。

>[!note]
>注意到 $S(n)$ 有两种情况：在最后一天卖出以及在之前就卖出。对于前者，说明最后一天是一个高位，如果新增一天更高，则可以延后一天卖出，多收益差额，否则可以忽略新的一天；对于后者，说明最后一天相对之前是低位，如果新增一天更高，则在最后一天买入，新的一天卖出，多收益差额，否则可以忽略新的一天。

```cpp
class Solution {
public:
  int maxProfit(vector<int> &prices) {
    if (prices.size() < 2)
      return 0;
    int val = max(0, prices[1] - prices[0]);
    for (int i = 2; i < prices.size(); i++) {
      if (prices[i] > prices[i - 1])
        val += prices[i] - prices[i - 1];
    }
    return val;
  }
};
```



#### [55. 跳跃游戏](https://leetcode.cn/problems/jump-game/)

给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。

题解——遍历计算每个元素能够到达的最远位置。如果当前位置超出了最远位置，说明已经不可能到达当前位置，立即返回。

```cpp
class Solution {
public:
  bool canJump(vector<int> &nums) {
    int maxPos = 0;
    for (int i = 0; i < nums.size(); i++) {
      if (i > maxPos)
        return false;
      maxPos = max(maxPos, i + nums[i]);
    }
    return true;
  }
};
```



#### [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)

给定一个长度为 `n` 的 **0 索引**整数数组 `nums`。初始位置为 `nums[0]`。

每个元素 `nums[i]` 表示从索引 `i` 向后跳转的最大长度。换句话说，如果你在 `nums[i]` 处，你可以跳转到任意 `nums[i + j]` 处:

- `0 <= j <= nums[i]` 
- `i + j < n`

返回到达 `nums[n - 1]` 的最小跳跃次数。生成的测试用例可以到达 `nums[n - 1]`

题解——可以考虑将数组划分为多个区域，上一个区域存在某个位置可以跳转到下一个区域的边界。维护区域边界 `end` 和的当前区域可以跳转到的最远位置 `maxPos`，每当我们移动到当前区域的边界，就可以进入下一个区域。例如

```cpp
class Solution {
public:
  int jump(vector<int> &nums) {
    int step = 0, maxPos = 0, end = 0;
    for (int i = 0; i < nums.size() - 1; i++) {
      maxPos = max(maxPos, i + nums[i]);
      if (i == end) {
        end = maxPos;
        step++;
      }
    }
    return step;
  }
};
```



#### [274. H 指数](https://leetcode.cn/problems/h-index/)

给你一个整数数组 `citations` ，其中 `citations[i]` 表示研究者的第 `i` 篇论文被引用的次数。计算并返回该研究者的 **`h` 指数**。

根据维基百科上 [h 指数的定义](https://baike.baidu.com/item/h-index/3991452?fr=aladdin)：`h` 代表“高引用次数” ，一名科研人员的 `h` **指数** 是指他（她）至少发表了 `h` 篇论文，并且 **至少** 有 `h` 篇论文被引用次数大于等于 `h` 。如果 `h` 有多种可能的值，**`h` 指数** 是其中最大的那个。

题解——注意到 h 指数不会超过论文数量（数组长度），因此可以**用一个向量作为哈希表**。对于引用数超过 n 的论文全部计入 `count[n]`，其余按照引用数作为索引计入。最后，我们倒序遍历：索引 `i` 表示引用数，则 `count[i]` 就是对应引用数的论文数，用 `h` 累计不超过 ` i ` 个引用的所有论文的数量，当这些论文数超过引用数时，则当前记录的引用数 `i` 就是最大的 h 指数。

>[!note]
>注意到最大的 h 指数会达到一个平衡：引用数量从 `n` 开始减小，论文数量从 0 开始增加，直到它们相遇时恰好达到最大值。

```cpp
class Solution {
public:
  int hIndex(vector<int> &citations) {
    int n = citations.size();
    vector<int> count(n + 1, 0);	// 注意 count[n] 用来存超出 n 引用数的文章
    for (int i = 0; i < n; i++) {
      if (citations[i] >= n)
        count[n]++;
      else
        count[citations[i]]++;
    }

    int h = 0;
    for (int i = n; i >= 0; i--) {
      h += count[i];
      if (h >= i)
        return i;
    }
    return 0;
  }
};
```



#### [380. O(1) 时间插入、删除和获取随机元素](https://leetcode.cn/problems/insert-delete-getrandom-o1/)

实现`RandomizedSet` 类：

- `RandomizedSet()` 初始化 `RandomizedSet` 对象
- `bool insert(int val)` 当元素 `val` 不存在时，向集合中插入该项，并返回 `true` ；否则，返回 `false` 。
- `bool remove(int val)` 当元素 `val` 存在时，从集合中移除该项，并返回 `true` ；否则，返回 `false` 。
- `int getRandom()` 随机返回现有集合中的一项（测试用例保证调用此方法时集合中至少存在一个元素）。每个元素应该有 **相同的概率** 被返回。

你必须实现类的所有函数，并满足每个函数的 **平均** 时间复杂度为 `O(1)` 。

题解——使用 `vector` 实现均匀随机访问，使用哈希表实现随机插入删除。哈希表保存每个元素和它对应的位置，删除时获得元素位置，将其与最后一个元素交换，然后弹出，这样就能实现 `O(1)` 删除。

>[!note]
>注意删除元素时，交换元素同时也要更新哈希表中保存的元素位置。

```cpp
class RandomizedSet {
private:
  vector<int> arr;
  unordered_map<int, int> ind;

public:
  RandomizedSet() { srand(time(0)); }

  bool insert(int val) {
    if (ind.count(val) > 0)
      return false;
    ind[val] = arr.size();
    arr.emplace_back(val);
    return true;
  }

  bool remove(int val) {
    if (!ind.count(val))
      return false;

    int i = ind[val];
    swap(arr[i], arr.back());
    arr.pop_back();
    ind[arr[i]] = i;
    ind.erase(val);
    return true;
  }

  int getRandom() { return arr[rand() % arr.size()]; }
};
```



#### [238. 除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/)

给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积 。

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在  **32 位** 整数范围内。

请 **不要使用除法，** 且在 `O(n)` 时间复杂度内完成此题。

题解——双向遍历。遍历到 `i` 元素时，记录 `i` 之前的所有元素乘积，然后作为输出结果。然后反向遍历，记录 `i` 之后的所有元素乘积，以此与之前的输出结果相乘即可。

```cpp
class Solution {
public:
  vector<int> productExceptSelf(vector<int> &nums) {
    int p = 1;
    vector<int> ret(nums.size(), 1);
    for (int i = 0; i < nums.size(); i++) {
      ret[i] *= p;
      p *= nums[i];
    }
    p = 1;
    for (int i = nums.size() - 1; i >= 0; i--) {
      ret[i] *= p;
      p *= nums[i];
    }
    return ret;
  }
};
```

>[!note]
>注意到两次循环互不影响，因此可以合并为同一个循环。



#### [134. 加油站](https://leetcode.cn/problems/gas-station/)

在一条环路上有 `n` 个加油站，其中第 `i` 个加油站有汽油 `gas[i]` 升。

你有一辆油箱容量无限的的汽车，从第 `i` 个加油站开往第 `i+1` 个加油站需要消耗汽油 `cost[i]` 升。你从其中的一个加油站出发，开始时油箱为空。

给定两个整数数组 `gas` 和 `cost` ，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 `-1` 。如果存在解，则 **保证** 它是 **唯一** 的。

题解——循环遍历两次。用 `count` 记录从 0 开始出发到达后面站点时剩下的油量，为了考虑回到起点的情况，因此还需要多遍历 `n - 1` 个元素折返，例如

```
[2,3,4]		-- gas
[3,4,3] 	-- cost
[2,2,2,1,1] -- count
```

注意到当我们切换出发点为 `i` 时，相当于每个 `count` 元素都加上了 `gas[i] - count[i]`，例如以 1 为出发点，则每个 `count` 都加上 `3 - 2 = 1`；同时为了确保可以循环一圈，还要确保 `count` 增加以后要超过对应的 `cost` 。因此我们记录 `gas[i] - count[i], cost[i] - count[i]` 的最大值，要求前者大于后者时才能将 `i` 作为起点。

>[!note]
>注意 `gas[i] - count[i]` 只在 `i` 小于 `n` 时更新，因为起点不会出现在折返的位置，但是损失会出现在折返的时候。

```cpp
class Solution {
public:
  int canCompleteCircuit(vector<int> &gas, vector<int> &cost) {
    int n = gas.size();

    int index = 0;
    int count = gas[0];
    int maxDC = cost[0] - count;
    int maxDG = 0;
    int start = maxDG >= maxDC ? 0 : -1;
    while (++index < 2 * n - 1) {
      // 累计余量
      int i = index % n;
      int j = i == 0 ? n - 1 : i - 1;
      count += gas[i] - cost[j];

	  // 让路径上的余量都大于 0 时，需要增加 maxDC 的油
      maxDC = max(maxDC, cost[i] - count);

	  // 切换所有起点后，最多能够增加 maxDG 的油
      if (index < n)
        maxDG = max(maxDG, gas[i] - count);

	  // 如果切换起点不能增加足够的油，说明无法到达
      if (maxDG < maxDC)
        start = -1;
      if (start == -1 && maxDG >= maxDC)
        start = i;
    }
    return start;
  }
};
```



#### [13. 罗马数字转整数](https://leetcode.cn/problems/roman-to-integer/)

罗马数字包含以下七种字符: `I`， `V`， `X`， `L`，`C`，`D` 和 `M`。

|符号|值|
|---|---|
|I|1|
|V|5|
|X|10|
|L|50|
|C|100|
|D|500|
|M|1000|

例如， 罗马数字 `2` 写做 `II` ，即为两个并列的 1 。`12` 写做 `XII` ，即为 `X` + `II` 。 `27` 写做  `XXVII`, 即为 `XX` + `V` + `II` 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 `IIII`，而是 `IV`。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 `IX`。这个特殊的规则只适用于以下六种情况：

- `I` 可以放在 `V` (5) 和 `X` (10) 的左边，来表示 4 和 9。
- `X` 可以放在 `L` (50) 和 `C` (100) 的左边，来表示 40 和 90。 
- `C` 可以放在 `D` (500) 和 `M` (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。

题解——从后向前遍历，正常情况左边的数一定大于等于右边，此时加上当前数，如果小于就减去当前数。

```cpp
class Solution {
public:
  int romanToInt(string s) {
    auto ind = [](char c) {
      switch (c) {
      case 'I':
        return 1;
      case 'V':
        return 5;
      case 'X':
        return 10;
      case 'L':
        return 50;
      case 'C':
        return 100;
      case 'D':
        return 500;
      case 'M':
        return 1000;
      }
      return 0;
    };
    int n = 0;
    int prev = 0;
    for (int i = s.size() - 1; i >= 0; i--) {
      int current = ind(s[i]);
      if (prev <= current)
        n += current;
      else
        n -= current;
      prev = current;
    }
    return n;
  }
};
```



#### [12. 整数转罗马数字](https://leetcode.cn/problems/integer-to-roman/)

七个不同的符号代表罗马数字，其值如下：

|符号|值|
|---|---|
|I|1|
|V|5|
|X|10|
|L|50|
|C|100|
|D|500|
|M|1000|

罗马数字是通过添加从最高到最低的小数位值的转换而形成的。将小数位值转换为罗马数字有以下规则：

- 如果该值不是以 4 或 9 开头，请选择可以从输入中减去的最大值的符号，将该符号附加到结果，减去其值，然后将其余部分转换为罗马数字。
- 如果该值以 4 或 9 开头，使用 **减法形式**，表示从以下符号中减去一个符号，例如 4 是 5 (`V`) 减 1 (`I`): `IV` ，9 是 10 (`X`) 减 1 (`I`)：`IX`。仅使用以下减法形式：4 (`IV`)，9 (`IX`)，40 (`XL`)，90 (`XC`)，400 (`CD`) 和 900 (`CM`)。
- 只有 10 的次方（`I`, `X`, `C`, `M`）最多可以连续附加 3 次以代表 10 的倍数。你不能多次附加 5 (`V`)，50 (`L`) 或 500 (`D`)。如果需要将符号附加 4 次，请使用 **减法形式**。

给定一个整数，将其转换为罗马数字。

题解——最直接的方法就是条件分支减去编码对应的值

```cpp
class Solution {
public:
  string intToRoman(int num) {
    string s = "";
    while (num >= 1000) {
      s += 'M';
      num -= 1000;
    }

    if (num >= 900) {
      s += "CM";
      num -= 900;
    } else if (num >= 500) {
      s += 'D';
      num -= 500;
    } else if (num >= 400) {
      s += "CD";
      num -= 400;
    }

    while (num >= 100) {
      s += 'C';
      num -= 100;
    }

    if (num >= 90) {
      s += "XC";
      num -= 90;
    } else if (num >= 50) {
      s += 'L';
      num -= 50;
    } else if (num >= 40) {
      s += "XL";
      num -= 40;
    }

    while (num >= 10) {
      s += 'X';
      num -= 10;
    }

    if (num >= 9) {
      s += "IX";
      num -= 9;
    } else if (num >= 5) {
      s += 'V';
      num -= 5;
    } else if (num >= 4) {
      s += "IV";
      num -= 4;
    }

    while (num > 0) {
      s += 'I';
      num -= 1;
    }

    return s;
  }
};
```

更简单的方法是编码每一位的表示

```cpp
const string thousands[] = {"", "M", "MM", "MMM"};
const string hundreds[] = {"",  "C",  "CC",  "CCC",  "CD",
                           "D", "DC", "DCC", "DCCC", "CM"};
const string tens[] = {"",  "X",  "XX",  "XXX",  "XL",
                       "L", "LX", "LXX", "LXXX", "XC"};
const string ones[] = {"",  "I",  "II",  "III",  "IV",
                       "V", "VI", "VII", "VIII", "IX"};

class Solution {
public:
  string intToRoman(int num) {
    return thousands[num / 1000] + hundreds[num % 1000 / 100] +
           tens[num % 100 / 10] + ones[num % 10];
  }
};
```

计算每一位的值，然后将值直接替换为对应的编码。



#### [58. 最后一个单词的长度](https://leetcode.cn/problems/length-of-last-word/)

给你一个字符串 `s`，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中 **最后一个** 单词的长度。

**单词** 是指仅由字母组成、不包含任何空格字符的最大子字符串。

题解——从后向前遍历，找到第二个非连续空格时退出。

```cpp
class Solution {
public:
  int lengthOfLastWord(string s) {
    int count = 0;
    bool first = false;
    for (int i = s.size() - 1; i >= 0; i--) {
      if (s[i] == ' ') {
        if (first)
          break;
        continue;
      } else {
        first = true;
        count++;
      }
    }
    return count;
  }
};
```



#### [14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

题解——双重遍历，检查 `i` 号元素是否相同。

```cpp
class Solution {
public:
  string longestCommonPrefix(vector<string> &strs) {
    if (strs.empty())
      return "";
    if (strs.size() == 1)
      return strs[0];
    int i = 0;
    while (i < strs[0].size()) {
      char c = strs[0][i];
      for (int j = 1; j < strs.size(); j++) {
        if (strs[j][i] != c)
          return strs[0].substr(0, i);
      }
      i++;
    }
    return strs[0].substr(0, i);
  }
};
```



#### [151. 反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)

给你一个字符串 `s` ，请你反转字符串中 **单词** 的顺序。

**单词** 是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的 **单词** 分隔开。

返回 **单词** 顺序颠倒且 **单词** 之间用单个空格连接的结果字符串。

**注意：** 输入字符串 `s` 中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

题解——反向遍历，记录当前字符串的尾部位置，遇到空格时将当前位置到尾部范围的字符串添加到输出字符串中。

```cpp
class Solution {
public:
  string reverseWords(string s) {
    string ret = "";
    int end = -1;
    for (int i = s.size() - 1; i >= 0; i--) {
      if (s[i] == ' ') {
        // 处理非连续空格
        if (end != -1) {
          ret += s.substr(i + 1, end - i - 1) + ' ';
          end = -1;
        }
        continue;
      }
      // 当前没有字符串时才记录新的位置
      if (end == -1)
        end = i + 1;
    }
    // 处理头部没有空格的情况
    if (end != -1)
      ret += s.substr(0, end) + ' ';
    // 弹出尾部空格
    if (!ret.empty())
      ret.pop_back();
    return ret;
  }
};
```



#### [6. Z 字形变换](https://leetcode.cn/problems/zigzag-conversion/)

将一个给定字符串 `s` 根据给定的行数 `numRows` ，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `"PAYPALISHIRING"` 行数为 `3` 时，排列如下：

```
P   A   H   N
A P L S I I G
Y   I   R
```

之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`"PAHNAPLSIIGYIR"`。

请你实现这个将字符串进行指定行数变换的函数：

```cpp
string convert(string s, int numRows);
```

题解——直接计算索引：将每个 V 形看做一组计算索引。单独考虑第一行和最后一行；中间每次输出一对。

```cpp
class Solution {
public:
  string convert(string s, int numRows) {
    if (numRows == 1)
      return s;

	// 计算每个 V 形的字符数，不够就补全
    int m = numRows * 2 - 2;
    int cols = s.size() / m;
    if (s.size() % m != 0)
      cols++;

    string res = "";
    for (int i = 0; i < cols; i++) {
      int id = i * m;
      if (id < s.size())
        res += s[id];
    }

    for (int i = 1; i < numRows - 1; i++) {
      for (int j = 0; j < cols; j++) {
      	// 中间行的字符关于 V 形对称
        int id1 = j * m + i;
        int id2 = (j + 1) * m - i;
        if (id1 < s.size())
          res += s[id1];
        if (id2 < s.size())
          res += s[id2];
      }
    }

    for (int i = 0; i < cols; i++) {
      int id = i * m + numRows - 1;
      if (id < s.size())
        res += s[id];
    }

    return res;
  }
};
```



#### [28. 找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)

给你两个字符串 `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回  `-1` 。

题解——遍历匹配 `needle` 的第一个字符，找到后比较之后长度等于 `needle` 的 `haystack` 部分。

```cpp
class Solution {
public:
  int strStr(string haystack, string needle) {
    char c = needle[0];
    int m = haystack.size();
    int n = needle.size();
    if (m < n)
      return -1;
    for (int i = 0; i < m; i++) {
      if (c == haystack[i]) {
        if (m - i < n)
          return -1;
        if (haystack.substr(i, n) == needle)
          return i;
      }
    }
    return -1;
  }
};
```



### 双指针
#### [125. 验证回文串](https://leetcode.cn/problems/valid-palindrome/)

如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 **回文串** 。

字母和数字都属于字母数字字符。

给你一个字符串 `s`，如果它是 **回文串** ，返回 `true` ；否则，返回 `false` 。

题解——双向遍历，遇到不满足要求的字符就跳过。对于满足要求的字符，要考虑大小写转换。

```cpp
class Solution {
public:
  bool isPalindrome(string s) {
    auto check = [](char c) {
      return (c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') ||
             (c >= 'A' && c <= 'Z');
    };
    auto same = [](char c1, char c2) {
      if (c1 >= 'A' && c1 <= 'Z')
        c1 -= 'A' - 'a';
      if (c2 >= 'A' && c2 <= 'Z')
        c2 -= 'A' - 'a';
      return c1 == c2;
    };

    bool yes = true;
    int left = 0, right = s.size() - 1;
    while (left < right) {
      while (!check(s[left]) && left < right)
        left++;
      while (!check(s[right]) && left < right)
        right--;
      if (left >= right)
        break;
      if (!same(s[left], s[right])) {
        yes = false;
        break;
      }
      left++, right--;
    }
    return yes;
  }
};
```



#### [392. 判断子序列](https://leetcode.cn/problems/is-subsequence/)

给定字符串 **s** 和 **t** ，判断 **s** 是否为 **t** 的子序列。

字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，`"ace"`是`"abcde"`的一个子序列，而`"aec"`不是）。

**进阶：**

如果有大量输入的 S，称作 S1, S2, ... , Sk 其中 k >= 10亿，你需要依次检查它们是否为 T 的子序列。在这种情况下，你会怎样改变代码？

题解——匹配第一个字符，然后递归地匹配后面的字符。

>[!note]
>注意利用子序列的性质：如果 `s` 是 `t` 的子序列，那么匹配到第一个字符后，`s` 剩下的部分一定是 `t` 剩下部分的子序列，所以递归可以直接返回，不需要重新匹配第一个字符。

```cpp
class Solution {
public:
  bool isSubsequence(string s, string t) {
    if (s.size() > t.size())
      return false;
    return test(s, 0, t, 0);
  }

  bool test(const string &s, int s1, const string &t, int s2) {
    if (s1 == s.size())
      return true;
    for (int i = s2; i < t.size(); i++) {
      if (t[i] == s[s1]) {
        return test(s, s1 + 1, t, i + 1);
      }
    }
    return false;
  }
};
```


#### [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)

给你一个下标从 **1** 开始的整数数组 `numbers` ，该数组已按 **非递减顺序排列**  ，请你从数组中找出满足相加之和等于目标数 `target` 的两个数。如果设这两个数分别是 `numbers[index1]` 和 `numbers[index2]` ，则 `1 <= index1 < index2 <= numbers.length` 。

以长度为 2 的整数数组 `[index1, index2]` 的形式返回这两个整数的下标 `index1` 和 `index2`。

你可以假设每个输入 **只对应唯一的答案** ，而且你 **不可以** 重复使用相同的元素。

你所设计的解决方案必须只使用常量级的额外空间。

题解——双向遍历，如果和超过 `target` 则移动右指针；小于 `target` 则移动左指针。

```cpp
class Solution {
public:
  vector<int> twoSum(vector<int> &numbers, int target) {
    int left = 0, right = numbers.size() - 1;
    while (left < right) {
      int s = numbers[left] + numbers[right];
      if (s == target)
        return {left + 1, right + 1};
      else if (s > target)
        right--;
      else
        left++;
    }
    return {};
  }
};
```



### 滑动窗口
#### [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其总和大于等于 `target` 的长度最小的 **子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度。**如果不存在符合条件的子数组，返回 `0` 。

题解——滑动窗口。维护当前窗口区域，如果总和小于 `target` 就移动右指针（注意右指针边界），增加边界值；如果大于等于 `target` 就记录窗口长度并移动左指针。

```cpp
class Solution {
public:
  int minSubArrayLen(int target, vector<int> &nums) {
    int left = 0, right = 0;
    int sum = nums[0];
    int len = INT_MAX;
    while (right < nums.size()) {
      if (sum >= target) {
        len = min(len, right - left + 1);
        sum -= nums[left];
        left++;
      } else {
        if (right == nums.size() - 1)
          break;
        sum += nums[right + 1];
        right++;
      }
    }
    return len == INT_MAX ? 0 : len;
  }
};
```



### 矩阵

#### [289. 生命游戏](https://leetcode.cn/problems/game-of-life/)

根据 [百度百科](https://baike.baidu.com/item/%E7%94%9F%E5%91%BD%E6%B8%B8%E6%88%8F/2926434?fr=aladdin) ， **生命游戏** ，简称为 **生命** ，是英国数学家约翰·何顿·康威在 1970 年发明的细胞自动机。

给定一个包含 `m × n` 个格子的面板，每一个格子都可以看成是一个细胞。每个细胞都具有一个初始状态： `1` 即为 **活细胞** （live），或 `0` 即为 **死细胞** （dead）。每个细胞与其八个相邻位置（水平，垂直，对角线）的细胞都遵循以下四条生存定律：

1. 如果活细胞周围八个位置的活细胞数少于两个，则该位置活细胞死亡；
2. 如果活细胞周围八个位置有两个或三个活细胞，则该位置活细胞仍然存活；
3. 如果活细胞周围八个位置有超过三个活细胞，则该位置活细胞死亡；
4. 如果死细胞周围正好有三个活细胞，则该位置死细胞复活；

下一个状态是通过将上述规则同时应用于当前状态下的每个细胞所形成的，其中细胞的出生和死亡是 **同时** 发生的。给你 `m x n` 网格面板 `board` 的当前状态，返回下一个状态。

给定当前 `board` 的状态，**更新** `board` 到下一个状态。

**注意** 你不需要返回任何东西。

题解——用每一个元素编码状态，用高位存新的状态，低位存当前状态。

```cpp
class Solution {
private:
  int m, n;

public:
  void gameOfLife(vector<vector<int>> &board) {
    // 编码状态 00 01 10 11
    // 00 - 之前死，现在死
    // 10 - 之前死，现在活
    // 01 - 之前活，现在死
    // 11 - 之前活，现在活
    m = board.size();
    n = board.front().size();
    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        deal(board, i, j);
      }
    }

    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        board[i][j] = board[i][j] >> 1;
      }
    }
  }

  void deal(vector<vector<int>> &board, int i, int j) {
    int count = 0;
    for (int p = -1; p < 2; p++) {
      for (int q = -1; q < 2; q++) {
        if (p == 0 && q == 0)
          continue;
        if ((i + p) >= 0 && (i + p) < m && (j + q) >= 0 && (j + q) < n)
          count += board[i + p][j + q] & 1;
      }
    }

    if (board[i][j] & 1) {
      if (count == 2 || count == 3)
        board[i][j] |= 2;
      else
        board[i][j] &= 1;
    } else if (count == 3)
      board[i][j] |= 2;
  }
};
```



### 哈希表
#### [383. 赎金信](https://leetcode.cn/problems/ransom-note/)

给你两个字符串：`ransomNote` 和 `magazine` ，判断 `ransomNote` 能不能由 `magazine` 里面的字符构成。

如果可以，返回 `true` ；否则返回 `false` 。

`magazine` 中的每个字符只能在 `ransomNote` 中使用一次。

题解——注意到输入字符范围有限，因此可以用一个固定长度的数组保存每个字符出现的次数，然后在遍历过程中减少字符次数，如果无法减少就返回 `false` 。

```cpp
class Solution {
public:
  bool canConstruct(string ransomNote, string magazine) {
    int record[26];
    fill(record, record + 26, 0);
    for (int i = 0; i < magazine.size(); i++) {
      record[magazine[i] - 'a']++;
    }

    for (int i = 0; i < ransomNote.size(); i++) {
      int j = ransomNote[i] - 'a';
      if (record[j] == 0)
        return false;
      record[j]--;
    }
    return true;
  }
};
```



#### [205. 同构字符串](https://leetcode.cn/problems/isomorphic-strings/)

给定两个字符串 `s` 和 `t` ，判断它们是否是同构的。

如果 `s` 中的字符可以按某种映射关系替换得到 `t` ，那么这两个字符串是同构的。

每个出现的字符都应当映射到另一个字符，同时不改变字符的顺序。不同字符不能映射到同一个字符上，相同字符只能映射到同一个字符上，字符可以映射到自己本身。

题解——由于需要双射，因此记录两个哈希表。首先两个哈希表一定是同步的，因此在每次搜索时，必须同时找到或者同时找不到，否则不同构；然后检测是否相互映射到对方。

```cpp
class Solution {
public:
  bool isIsomorphic(string s, string t) {
    unordered_map<char, char> to;
    unordered_map<char, char> back;
    for (int i = 0; i < s.size(); i++) {
      auto toIt = to.find(s[i]);
      auto backIt = back.find(t[i]);
      if (toIt == to.end()) {
        if (backIt != back.end())
          return false;
        to[s[i]] = t[i];
        back[t[i]] = s[i];
      } else {
        if (backIt == back.end())
          return false;
        if (toIt->second != t[i] || backIt->second != s[i])
          return false;
      }
    }
    return true;
  }
};
```



#### [290. 单词规律](https://leetcode.cn/problems/word-pattern/)

给定一种规律 `pattern` 和一个字符串 `s` ，判断 `s` 是否遵循相同的规律。

这里的 **遵循** 指完全匹配，例如， `pattern` 里的每个字母和字符串 `s` 中的每个非空单词之间存在着双向连接的对应规律。

题解——维护两个哈希表。先考虑字符到字符串的映射，然后考虑反向映射判断是否是双射。注意边界情况：字符数量少于或多于字符串数量。

```cpp
class Solution {
public:
  bool wordPattern(string pattern, string s) {
    unordered_map<char, string> table;
    unordered_map<string, char> back;
    int i = 0, j = 0;
    while (j < pattern.size()) {
      char c = pattern[j];
      string str = "";
      while (i < s.size() && s[i] != ' ') {
        str += s[i];
        i++;
      }
      if (s[i] == ' ')
        i++;
      auto it = table.find(c);
      if (it == table.end()) {
        auto it = back.find(str);
        if (it != back.end())
          return false;
        back[str] = c;
        table[c] = str;
      } else if (it->second != str) {
        return false;
      }
      j++;
      if (i == s.size() && j < pattern.size())
        return false;
    }
    return i == s.size();
  }
};
```



#### [242. 有效的字母异位词](https://leetcode.cn/problems/valid-anagram/)

给定两个字符串 `s` 和 `t` ，编写一个函数来判断 `t` 是否是 `s` 的字母异位词。

题解——直接排序然后比较。或者记录每个字母出现的次数，然后对比是否相同。

```cpp
class Solution {
public:
  bool isAnagram(string s, string t) {
    if (s.size() != t.size())
      return false;

    sort(s.begin(), s.end());
    sort(t.begin(), t.end());
    return s == t;
  }
};
```



#### [202. 快乐数](https://leetcode.cn/problems/happy-number/)

编写一个算法来判断一个数 `n` 是不是快乐数。

**「快乐数」** 定义为：

- 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
- 然后重复这个过程直到这个数变为 1，也可能是 **无限循环** 但始终变不到 1。
- 如果这个过程 **结果为** 1，那么这个数就是快乐数。

如果 `n` 是 _快乐数_ 就返回 `true` ；不是，则返回 `false` 。

题解——用一个哈希表保存一路上的值，如果进入循环就判断是否是 1，不是说明非快乐数。

>[!note]
>注意到所有位上数字的平方和一定有限，当初始值足够大时，新的值一定更小。由于可能取到的值有限，因此最终必然进入循环。

```cpp
class Solution {
public:
  bool isHappy(int n) {
    unordered_set<int> path;

    while (true) {
      n = happy(n);
      auto it = path.find(n);
      if (it != path.end()) {
        if (n == 1)
          return true;
        else
          return false;
      }
      path.insert(n);
    }
    return true;
  }

  int happy(int n) {
    int h = 0;
    while (n > 0) {
      int a = n % 10;
      h += a * a;
      n /= 10;
    }
    return h;
  }
};
```

更好的方法是使用“快慢指针”检测周期：慢指针每次走一步，快指针每次走两步。如果它们得到的值相同，说明进入了循环（此时快指针恰好比慢指针多走几个周期）。

```cpp
class Solution {
public:
  bool isHappy(int n) {
    int slow = n, fast = n;
    do {
      slow = happy(slow);
      fast = happy(fast);
      fast = happy(fast);
    } while (slow != fast);
    return slow == 1;
  }

  int happy(int n) {
    int h = 0;
    while (n > 0) {
      int a = n % 10;
      h += a * a;
      n /= 10;
    }
    return h;
  }
};
```



#### [219. 存在重复元素 II](https://leetcode.cn/problems/contains-duplicate-ii/)

给你一个整数数组 `nums` 和一个整数 `k` ，判断数组中是否存在两个 **不同的索引** `i` 和 `j` ，满足 `nums[i] == nums[j]` 且 `abs(i - j) <= k` 。如果存在，返回 `true` ；否则，返回 `false` 。

题解——用哈希表记录前面 `k` 个元素，遍历数组检查当前元素是否在表中。

```cpp
class Solution {
public:
  bool containsNearbyDuplicate(vector<int> &nums, int k) {
    // 注意由于要求 i,j 不同，排除 k == 0
    if (k == 0)
      return false;
    unordered_set<int> record;
    k = min(k, (int)nums.size());
    for (int i = 0; i < nums.size(); i++) {
      auto it = record.find(nums[i]);
      if (it == record.end()) {
        // 在哈希表足够长之前不删除元素
        if (i >= k)
          record.erase(nums[i - k]);
        record.insert(nums[i]);
      } else
        return true;
    }
    return false;
  }
};
```



#### [128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

题解——我们要寻找连续序列开头的位置，因此遍历所有元素 `n`，检查 `n - 1` 是否存在，如果存在，说明它不是开头；找到开头后，递增当前元素并检查是否存在。

>[!note]
>注意遍历哈希表而非数组，从而跳过重复元素。

```cpp
class Solution {
public:
  int longestConsecutive(vector<int> &nums) {
    if (nums.empty())
      return 0;

    unordered_set<int> table;
    for (int i = 0; i < nums.size(); i++)
      table.insert(nums[i]);

    int maxLen = 1;
    for (auto &n : table) {
      if (table.count(n - 1))
        continue;

      int len = 1;
      while (table.count(n + len))
        len++;
      maxLen = max(maxLen, len);
    }
    return maxLen;
  }
};
```



### 区间

#### [228. 汇总区间](https://leetcode.cn/problems/summary-ranges/)

给定一个  **无重复元素** 的 **有序** 整数数组 `nums` 。

返回 _**恰好覆盖数组中所有数字** 的 **最小有序** 区间范围列表_ 。也就是说，`nums` 的每个元素都恰好被某个区间范围所覆盖，并且不存在属于某个范围但不属于 `nums` 的数字 `x` 。

列表中的每个区间范围 `[a,b]` 应该按如下格式输出：

- `"a->b"` ，如果 `a != b`
- `"a"` ，如果 `a == b`

题解——维护一个区间范围，当相邻数不连续时记录当前区间范围，然后刷新区间范围。

```cpp
class Solution {
public:
  vector<string> summaryRanges(vector<int> &nums) {
    vector<string> ret;
    int left = 0, right = 1;
    for (int i = 1; i < nums.size(); i++) {
      long d = (long)nums[i] - (long)nums[right - 1];
      if (d == 1)
        right++;
      else {
        string s = to_string(nums[left]);
        if (right - left > 1)
          s += "->" + to_string(nums[right - 1]);
        ret.push_back(s);
        left = i;
        right = i + 1;
      }
    }
    if (left < nums.size()) {
      string s = to_string(nums[left]);
      if (right - left > 1)
        s += "->" + to_string(nums[right - 1]);
      ret.push_back(s);
    }
    return ret;
  }
};
```



#### [56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 _一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间_ 。

题解——首先排序区间，然后维护一个区间范围。遍历所有区间，如果当前区间起点在所维护范围内，就将维护范围更新为最大范围；否则就记录维护范围，然后以当前区间为新的维护范围。

```cpp
Class Solution {
Public:
  vector<vector<int>> merge (vector<vector<int>> &intervals) {
    Sort (intervals.Begin (), intervals.End (),
         [](auto &a, auto &b) { return a[0] < b[0]; });

    Int left = intervals[0][0];
    Int right = intervals[0][1];
    vector<vector<int>> ret;
    For (int i = 1; i < intervals.Size (); i++) {
      If (intervals[i][0] <= right) {
        Right = max (right, intervals[i][1]);
        Continue;
      }
      Ret. Push_back ({left, right});
      Left = intervals[i][0];
      Right = intervals[i][1];
    }
    Ret. Push_back ({left, right});
    Return ret;
  }
};
```



#### [57. 插入区间](https://leetcode.cn/problems/insert-interval/)

给你一个 **无重叠的** ，按照区间起始端点排序的区间列表 `intervals`，其中 `intervals[i] = [starti, endi]` 表示第 `i` 个区间的开始和结束，并且 `intervals` 按照 `starti` 升序排列。同样给定一个区间 `newInterval = [start, end]` 表示另一个区间的开始和结束。

在 `intervals` 中插入区间 `newInterval`，使得 `intervals` 依然按照 `starti` 升序排列，且区间之间不重叠（如果有必要的话，可以合并区间）。

返回插入之后的 `intervals`。

**注意** 你不需要原地修改 `intervals`。你可以创建一个新数组然后返回它。

题解——首先找到插入位置，将新的区间插入，然后使用合并区间的代码。

```cpp
Class Solution {
Public:
  vector<vector<int>> insert (vector<vector<int>> &intervals,
                             vector<int> &newInterval) {
    Auto it = lower_bound (intervals.Begin (), intervals.End (), newInterval,
                          [](auto &a, auto &b) { return a[0] < b[0]; });
    Intervals.Insert (it, newInterval);
    Int left = intervals[0][0];
    Int right = intervals[0][1];
    vector<vector<int>> ret;
    For (int i = 1; i < intervals.Size (); i++) {
      If (intervals[i][0] <= right) {
        Right = max (right, intervals[i][1]);
        Continue;
      }
      Ret. Push_back ({left, right});
      Left = intervals[i][0];
      Right = intervals[i][1];
    }
    Ret. Push_back ({left, right});
    Return ret;
  }
};
```



#### [452. 用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)

有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组 `points` ，其中`points[i] = [xstart, xend]` 表示水平直径在 `xstart` 和 `xend`之间的气球。你不知道气球的确切 y 坐标。

一支弓箭可以沿着 x 轴从不同点 **完全垂直** 地射出。在坐标 `x` 处射出一支箭，若有一个气球的直径的开始和结束坐标为 `x_start`，`x_end`，且满足  `xstart ≤ x ≤ x``end`，则该气球会被 **引爆** 。可以射出的弓箭的数量 **没有限制** 。弓箭一旦被射出之后，可以无限地前进。

给你一个数组 `points` ，_返回引爆所有气球所必须射出的 **最小** 弓箭数_ 。

题解——假设最优解中有一支箭在 `x` 处射穿 `[x_start, x_end]` 气球，则它在 `x_end` 也一定射穿气球，这意味着我们总是可以把箭的位置右移到右端点。考虑**右端点位于最左侧**的气球，由于要射穿所有气球，则该右端点一定有一支箭，因此可以移除它射穿的所有气球，考虑剩下的气球。

```cpp
class Solution {
public:
  int findMinArrowShots(vector<vector<int>> &points) {
    // 按照右端点排序
    sort(
        points.begin(), points.end(),
        [](const vector<int> &a, const vector<int> &b) { return a[1] < b[1]; });
    int count = 1;
    int boundary = points[0][1];
    for (auto &v : points) {
      // 当某个气球左端点超出当前边界，说明需要射出第二支箭
      if (v[0] > boundary) {
        count++;
        boundary = v[1];
      }
    }
    return count;
  }
};
```



### 栈

#### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

题解——用一个栈保存括号，遇到开括号就推入，遇到闭括号则要与栈顶配对才弹出。如果当前为闭括号，而栈为空或者不配对，则无效。最后由于括号要完全匹配，因此栈为空才说明有效。

```cpp
class Solution {
public:
  bool isValid(string s) {
    map<char, char> close;
    close['('] = ')';
    close['['] = ']';
    close['{'] = '}';
    stack<char> table;
    for (auto &c : s) {
      if (close.count(c))
        table.push(c);
      else if (table.empty() || close[table.top()] != c)
        return false;
      else
        table.pop();
    }
    return table.size() == 0;
  }
};
```



#### [71. 简化路径](https://leetcode.cn/problems/simplify-path/)

给你一个字符串 `path` ，表示指向某一文件或目录的 Unix 风格 **绝对路径** （以 `'/'` 开头），请你将其转化为 **更加简洁的规范路径**。

在 Unix 风格的文件系统中规则如下：

- 一个点 `'.'` 表示当前目录本身。
- 此外，两个点 `'..'` 表示将目录切换到上一级（指向父目录）。
- 任意多个连续的斜杠（即，`'//'` 或 `'///'`）都被视为单个斜杠 `'/'`。
- 任何其他格式的点（例如，`'...'` 或 `'....'`）均被视为有效的文件/目录名称。

返回的 **简化路径** 必须遵循下述格式：

- 始终以斜杠 `'/'` 开头。
- 两个目录名之间必须只有一个斜杠 `'/'` 。
- 最后一个目录名（如果存在）**不能** 以 `'/'` 结尾。
- 此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 `'.'` 或 `'..'`）。

返回简化后得到的 **规范路径** 。

题解——先不考虑 `.` 的处理，只考虑 `/` 的个数，每次记录需要加入的新路径，遇到 `/` 就将其加入。然后考虑新路径由 `.` 组成的情况。

```cpp
Class Solution {
Public:
  String simplifyPath (string path) {
    // 补上最后一个 /，确保最后一个 cur 能够进入 newPath
    Path += '/';
    String cur = "";
    String newPath = "/";
    Int ndiv = 0;
    For (int i = 1; i < path.Size (); i++) {
      Char c = path[i];
      If (c == '/') {
        // 遇到第一个 / 才考虑记录 cur
        If (ndiv == 0) {
          // .. 返回上一层目录
          If (cur == "..") {
            If (newPath.Size () > 1) {
              NewPath. Pop_back ();
              While (newPath.Back () != '/')
                NewPath. Pop_back ();
            }
            // 如果 cur 非空且 cur 不是 . 才记录 cur
          } else if (cur != "" && cur != ".")
            NewPath += cur + '/';
          Cur = "";
        }
        Ndiv++;
        Continue;
      }
      Cur += c;
      Ndiv = 0;
    }
    // 最后一定会有一个 /，要删除
    If (newPath.Size () > 1)
      NewPath. Pop_back ();
    Return newPath;
  }
};
```



#### [155. 最小栈](https://leetcode.cn/problems/min-stack/)

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。

题解——用 `stack` 作为栈，同时维护一个 `map` 保存元素和其数量。由于 `map` 自动排序，因此第一个元素就是最小元素。

```cpp
class MinStack {
private:
  stack<int> st;
  map<int, int> vals;

public:
  MinStack() {}

  void push(int val) {
    st.push(val);
    auto it = vals.find(val);
    if (it == vals.end())
      vals.insert(make_pair(val, 1));
    else
      it->second++;
  }

  void pop() {
    int val = st.top();
    auto it = vals.find(val);
    it->second--;
    if (it->second == 0)
      vals.erase(it);
    st.pop();
  }

  int top() { return st.top(); }

  int getMin() { return vals.begin()->first; }
};
```



#### [150. 逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)

给你一个字符串数组 `tokens` ，表示一个根据 [逆波兰表示法](https://baike.baidu.com/item/%E9%80%86%E6%B3%A2%E5%85%B0%E5%BC%8F/128437) 表示的算术表达式。

请你计算该表达式。返回一个表示表达式值的整数。

**注意：**

- 有效的算符为 `'+'`、`'-'`、`'*'` 和 `'/'` 。
- 每个操作数（运算对象）都可以是一个整数或者另一个表达式。
- 两个整数之间的除法总是 **向零截断** 。
- 表达式中不含除零运算。
- 输入是一个根据逆波兰表示法表示的算术表达式。
- 答案及所有中间计算结果可以用 **32 位** 整数表示。

题解——利用逆波兰表示法，用栈保存左侧的数，当遇到符号时，弹出栈顶的两个元素计算，将结果再推入栈，以此类推，最终栈顶的元素就是计算结果。

```cpp
class Solution {
public:
  int evalRPN(vector<string> &tokens) {
    stack<int> st;
    for (auto &v : tokens) {
      if (v == "+" || v == "-" || v == "*" || v == "/") {
        int right = st.top();
        st.pop();
        int left = st.top();
        st.pop();

        if (v == "+")
          left += right;
        else if (v == "-")
          left -= right;
        else if (v == "*")
          left *= right;
        else
          left /= right;

        st.push(left);
      } else
        st.push(stoi(v));
    }
    return st.top();
  }
};
```



### 链表

>[!note]
>在链表处理中，常用的技巧是在头节点前面创建一个临时节点作为起点，避免头节点被删除造成的麻烦。



#### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

_如果链表中存在环_ ，则返回 `true` 。否则，返回 `false` 。

题解——使用哈希表保存所有指针，检查是否重复。

>[!note] 快慢指针
>要寻找循环时，快慢指针是常见的做法。维护两个指针，一个快指针，一个慢指针。快指针每次走两步，慢指针每次走一步，如果存在循环，那么它们一定会相遇。

```cpp
class Solution {
public:
  bool hasCycle(ListNode *head) {
    unordered_set<ListNode *> record;
    while (head != nullptr) {
      if (record.count(head))
        return true;
      record.insert(head);
      head = head->next;
    }
    return false;
  }
};
```



#### [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/)

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

题解——将两个链表和分解为一个数和一个链表的和

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
    ListNode *p = l1;
    ListNode *prev = nullptr;
    while (p != nullptr && l2 != nullptr) {
      // 如果存在进位，addOneNumber 会自动加长
      addOneNumber(p, l2->val);
      prev = p;
      p = p->next;
      l2 = l2->next;
    }

    // l2 更长，此时一定不会有进位
    if (p == nullptr)
      prev->next = l2;

    return l1;
  }

  // 考虑每次只加一个数
  ListNode *addOneNumber(ListNode *l, int val) {
    ListNode *p = l;
    ListNode *prev = nullptr;
    while (p != nullptr) {
      p->val += val;
      if (p->val >= 10) {
        p->val -= 10;
        val = 1;
      } else
        val = 0;

      prev = p;
      p = p->next;
    }

    if (val != 0) {
      prev->next = new ListNode;
      prev->next->val = val;
    }
    return l;
  }
};
```



#### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

题解——迭代法，将第一个链表的节点依次插入第二个链表，返回第二个链表的头部节点。为了避免头部被改写，交换头部以确保第一个链表插入在第二个链表头之后。

```cpp
class Solution {
public:
  ListNode *mergeTwoLists(ListNode *list1, ListNode *list2) {
    if (list1 == nullptr)
      return list2;
    if (list2 == nullptr)
      return list1;
    if (list2->val > list1->val)
      swap(list1, list2);

    ListNode *node = list1;
    ListNode *head = list2;
    ListNode *prev = nullptr;
    while (node != nullptr) {
      ListNode *next = node->next;
      prev = merge(prev, head, node);
      head = prev->next;
      node = next;
    }
    return list2;
  }

  ListNode *merge(ListNode *prev, ListNode *head, ListNode *node) {
    while (head != nullptr && head->val <= node->val) {
      prev = head;
      head = head->next;
    }

	// 此时插入在末尾，一定要注意断开 node 后面的节点，不然两个链表会连起来
    if (head == nullptr) {
      prev->next = node;
      node->next = nullptr;
    } else {
      // 此时插入在中间
      if (prev)
        prev->next = node;
      node->next = head;
    }
    return prev;
  }
};
```

更好的方案是使用递归，可以分为两种情况

- `l1->val > l2->val`，那么就将 `l2->next` 设为 `l1, l2->next` 合并后返回的节点
- `l1->val <= l2->val`，那么就将 `l1->next` 设为 `l1->next, l2` 合并后返回的节点

```cpp
class Solution {
public:
  ListNode *mergeTwoLists(ListNode *list1, ListNode *list2) {
    if (list1 == nullptr)
      return list2;
    if (list2 == nullptr)
      return list1;
    if (list1->val < list2->val) {
      list1->next = mergeTwoLists(list1->next, list2);
      return list1;
    } else {
      list2->next = mergeTwoLists(list1, list2->next);
      return list2;
    }
    return nullptr;
  }
};
```



#### [138. 随机链表的复制](https://leetcode.cn/problems/copy-list-with-random-pointer/)

给你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/%E6%B7%B1%E6%8B%B7%E8%B4%9D/22785317?fr=aladdin)**。深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

例如，如果原链表中有 `X` 和 `Y` 两个节点，其中 `X.random --> Y` 。那么在复制链表中对应的两个节点 `x` 和 `y` ，同样有 `x.random --> y` 。

返回复制链表的头节点。

用一个由 `n` 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 `[val, random_index]` 表示：

- `val`：一个表示 `Node.val` 的整数。
- `random_index`：随机指针指向的节点索引（范围从 `0` 到 `n-1`）；如果不指向任何节点，则为  `null` 。

你的代码 **只** 接受原链表的头节点 `head` 作为传入参数。

题解——保存节点到索引的映射，以及新节点的列表，然后利用映射关系构建连接

```cpp
class Solution {
public:
  Node *copyRandomList(Node *head) {
    if (head == nullptr)
      return nullptr;

	// 建立映射，包括空指针的映射
    unordered_map<Node *, int> record;
    record.insert(make_pair(nullptr, -1));
    int i = 0;
    Node *node = head;
    while (node != nullptr) {
      record.insert(make_pair(node, i++));
      node = node->next;
    }

	// 建立新节点，注意 record 里多一个 nullptr
    vector<Node *> nRecord(record.size() - 1);
    for (int i = 0; i < nRecord.size(); i++) {
      nRecord[i] = new Node(0);
    }

	// 建立连接关系
    for (int i = 0; i < nRecord.size(); i++) {
      nRecord[i]->val = head->val;
      if (i > 0)
        nRecord[i - 1]->next = nRecord[i];
      int j = record[head->random];
      if (j >= 0)
        nRecord[i]->random = nRecord[j];
      head = head->next;
    }

    return nRecord[0];
  }
};
```

注意到这种做法其实就是把旧节点与新节点通过索引联系起来，因此可以直接维护单一哈希表

```cpp

class Solution {
public:
  Node *copyRandomList(Node *head) {
    if (head == nullptr)
      return nullptr;

	// 记录新旧节点的映射
    unordered_map<Node *, Node *> record;
    Node *node = head;
    while (node != nullptr) {
      Node *n = new Node(node->val);
      record.insert(make_pair(node, n));
      node = node->next;
    }

	// 直接映射关系
    for (auto &&[n1, n2] : record) {
      if (n1->next)
        n2->next = record[n1->next];
      if (n1->random)
        n2->random = record[n1->random];
    }

    return record[head];
  }
};

```



#### [92. 反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii/)

给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置 `left` 到位置 `right` 的链表节点，返回 **反转后的链表** 。

题解——注意 `left` 从 1 开始。我们选择反转区间左右各一个节点，保存中间的节点，倒序反转。如果长度不超过 1，直接返回。如果 `left` 比较小，那么左节点应该是 `head` 的前一个，需要创建一个节点，注意由于节点数增加，所以 `left, right` 都要增加一次；另外这种情况带着 `head` 反转，因此还需要重新设置头节点。

```cpp
class Solution {
public:
  ListNode *reverseBetween(ListNode *head, int left, int right) {
    if (head == nullptr || head->next == nullptr || left == right)
      return head;

    ListNode *p = head;
    bool nHead = false;
    if (left < 2) {
      nHead = true;
      ListNode *n = new ListNode;
      n->next = head;
      p = n;
      left++;
      right++;
    }

    while (p != nullptr && left > 2) {
      p = p->next;
      left--;
      right--;
    }

    vector<ListNode *> nodes;
    ListNode *q = p->next;
    while (q != nullptr && right > 1) {
      nodes.push_back(q);
      q = q->next;
      right--;
    }

	// 反向连接
    for (int i = nodes.size() - 1; i > 0; i--)
      nodes[i]->next = nodes[i - 1];

    nodes[0]->next = q;
    p->next = nodes[nodes.size() - 1];

	// 设置头节点
    if (nHead)
      head = p->next;

    return head;
  }
```



#### [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

题解——记录每个节点

```cpp
/**
* Definition for singly-linked list.
* struct ListNode {
*     int val;
*     ListNode *next;
*     ListNode() : val(0), next(nullptr) {}
*     ListNode(int x) : val(x), next(nullptr) {}
*     ListNode(int x, ListNode *next) : val(x), next(next) {}
* };
*/
class Solution {
public:
  ListNode *removeNthFromEnd(ListNode *head, int n) {
    ListNode *p = head;
    vector<ListNode *> v;
    while (p != nullptr) {
      v.push_back(p);
      p = p->next;
    }

    if (n == v.size())
      return head->next;

    p = v[v.size() - n - 1];
    p->next = p->next->next;

    return head;
  }
};
```



#### [82. 删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/)

给定一个已排序的链表的头 `head` ， _删除原始链表中所有重复数字的节点，只留下不同的数字_ 。返回 _已排序的链表_ 。

题解——要删除重复两次或以上的节点（注意重复的节点本身也要删除），可以检查 `p,p->next` 是否相同，如果相同，就将 `p->prev` 记为初始位置，然后移动 `p` 直到 `p,p->next` 不同，此时将之前记录的位置连接过来。最后考虑边界情况

1. 头部开始重复：保存头部是否重复，最后移动头部节点
2. 尾部重复：由于需要检查到 `p,p->next` 不同才会裁剪，所以尾部重复不会被删去。最后检查初始位置是否为空，如果不为空说明需要删除后面的节点。

>[!note]
>在链表处理中，常用的技巧是在头节点前面创建一个临时节点作为起点，避免头节点被删除造成的麻烦。

```cpp
class Solution {
public:
  ListNode *deleteDuplicates(ListNode *head) {
    // 空指针
    if (head == nullptr)
      return nullptr;
      
    ListNode *p = head;
    ListNode *pp = nullptr;
    ListNode *left = nullptr;
    bool dhead = false;
    while (p->next != nullptr) {
      // 检查相邻
      if (p->val == p->next->val) {
        if (p == head)
          dhead = true;
        // 只记录一次最左边节点
        if (left == nullptr)
          left = pp;
        pp = p;
      } else if (left != nullptr) {
        pp = left;
        left->next = p->next;
        left = nullptr;
      } else {
        // 注意这一步不要忘记
        pp = p;
      }
      p = p->next;
    }

	// 删除尾部重复
    if (left != nullptr)
      left->next = nullptr;

	// 删除头部重复
    if (dhead) {
      int val = head->val;
      while (head != nullptr && head->val == val)
        head = head->next;
    }
    return head;
  }
};
```


#### [61. 旋转链表](https://leetcode.cn/problems/rotate-list/)

给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。

题解——记录链表节点，从倒数 `k` 位置切断链表重新连接。

>[!note]
>更省空间的做法是将链表连成环，利用取模运算的周期性找到断开位置。

```cpp
/**
* Definition for singly-linked list.
* struct ListNode {
*     int val;
*     ListNode *next;
*     ListNode () : val (0), next (nullptr) {}
*     ListNode (int x) : val (x), next (nullptr) {}
*     ListNode (int x, ListNode *next) : val (x), next (next) {}
* };
*/
Class Solution {
Public:
  ListNode *rotateRight (ListNode *head, int k) {
    If (k == 0 || head == nullptr)
      Return head;
    vector<ListNode *> list;
    ListNode *p = head;
    While (p != nullptr) {
      List. Push_back (p);
      P = p->next;
    }
    K = k % list.Size ();
    P = list[list.Size () - k - 1];
    List.Back ()->next = head;
    Head = p->next;
    P->next = nullptr;
    Return head;
  }
};
```



#### [146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)

请你设计并实现一个满足  [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

题解——使用一个哈希表保存 `[key, Node*]` 映射，使用双向链表保存时间顺序。新的键值对插入到头部，尾部是最少使用的键值对。

- `get` 获得对应 `key` 的值，同时要更新 `Node` 的位置到链表头部
- `put` 插入键值对：
	- 如果键在哈希表中，直接更新对应的值（这一步要在最前面）
	- 如果不在，才考虑总元素个数是否超过容量；超过则弹出尾部元素以及哈希表中的键值对；未超过则插入键值到链表头部以及哈希表中

```cpp
struct Node {
  Node *next = nullptr;
  Node *prev = nullptr;
  int key = 0;
  int val = 0;
};

class Link {
private:
  Node *head = nullptr;
  Node *tail = nullptr;

public:
  Link() {
    head = new Node;
    tail = new Node;
    head->next = tail;
    tail->prev = head;
  }

  Node *push(int key, int val) {
    Node *node = new Node;
    node->key = key;
    node->val = val;
    node->next = head->next;
    node->prev = head;
    head->next->prev = node;
    head->next = node;
    return node;
  }

  int pop() {
    Node *node = tail->prev;
    node->prev->next = tail;
    tail->prev = node->prev;
    int key = node->key;
    delete node;
    return key;
  }

  void update(Node *node) {
    node->prev->next = node->next;
    node->next->prev = node->prev;

    node->next = head->next;
    node->prev = head;
    head->next->prev = node;
    head->next = node;
  }

  ~Link() {
    while (head != nullptr) {
      Node *next = head->next;
      delete head;
      head = next;
    }
  }
};

class LRUCache {
private:
  int capacity;
  Link llist;
  unordered_map<int, Node *> indices;

public:
  LRUCache(int capacity) : capacity(capacity) {}

  int get(int key) {
    auto it = indices.find(key);
    if (it == indices.end())
      return -1;
    llist.update(it->second);
    return it->second->val;
  }

  void put(int key, int value) {
    auto it = indices.find(key);
    if (it == indices.end()) {
      if (indices.size() == capacity) {
        int k = llist.pop();
        indices.erase(k);
      }
      Node *node = llist.push(key, value);
      indices.insert(make_pair(key, node));
    } else {
      it->second->val = value;
      llist.update(it->second);
    }
  }
};
```



### 二叉树

#### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

给定一个二叉树 `root` ，返回其最大深度。

二叉树的 **最大深度** 是指从根节点到最远叶子节点的最长路径上的节点数。

题解——递归计算左右子树的最大深度。

```cpp
class Solution {
public:
  int maxDepth(TreeNode *root) {
    if (root == nullptr)
      return 0;
    return max(maxDepth(root->left), maxDepth(root->right)) + 1;
  }
};
```



#### [100. 相同的树](https://leetcode.cn/problems/same-tree/)

给你两棵二叉树的根节点 `p` 和 `q` ，编写一个函数来检验这两棵树是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

题解——左右子树递归比较即可。

```cpp
class Solution {
public:
  bool isSameTree(TreeNode *p, TreeNode *q) {
    if (p == nullptr && q == nullptr)
      return true;
    else if (p != nullptr && q != nullptr && p->val == q->val)
      return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
    else
      return false;
  }
};
```



#### [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。

题解——交换左右子树的根节点，然后递归翻转左右子树。

```cpp
class Solution {
public:
  TreeNode *invertTree(TreeNode *root) {
    if (root == nullptr)
      return nullptr;

    TreeNode *left = root->left;
    TreeNode *right = root->right;
    root->right = left;
    root->left = right;

    invertTree(left);
    invertTree(right);

    return root;
  }
};
```



#### [101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

给你一个二叉树的根节点 `root` ，检查它是否轴对称。

题解——注意到我们只需要判断左子树和右子树的翻转相同即可，因此进行对称比较。

```cpp
class Solution {
public:
  bool isSymmetric(TreeNode *root) {
    if (root == nullptr)
      return true;
    return isSame(root->left, root->right);
  }

  bool isSame(TreeNode *root1, TreeNode *root2) {
    if (root1 == nullptr && root2 == nullptr)
      return true;
    else if (root1 != nullptr && root2 != nullptr && root1->val == root2->val)
      // 比较左子树的左子树和右子树的右子树
      return isSame(root1->left, root2->right) &&
             isSame(root1->right, root2->left);
    else
      return false;
  }
};
```



#### [105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

题解——注意到先序遍历和中序遍历的数组可以分解为
$$
\begin{aligned}
S_{p}(root) &= root + S_{p}(root\to left) + S_{p}(root\to right) \\
S_{i}(root) &= S_{i}(root\to left) + root + S_{i}(root\to right)
\end{aligned}
$$
假设构造函数为 $F(S_{p},S_{i})$，可以得到
$$
F(S_{p}(root),S_{i}(root)) = 
\begin{cases}
root, \\
root\to left= F(S_{p}(root\to left),S_{i}(root\to left)),\\ 
root\to right= F(S_{p}(root\to right),S_{i}(root\to right))
\end{cases}
$$
这样就有了递归关系。由于先序遍历和中序遍历都一定得到相同长度的结果，因此只需要维护先序遍历起点、中序遍历起点和数组长度。由于起点总是在数组内部，因此总是有效，只需要判断数组长度是否为零。

```cpp
class Solution {
public:
  TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder) {
    return buildTree(preorder, inorder, 0, 0, inorder.size());
  }

  TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder, int pi,
                      int ii, int size) {
    if (size == 0)
      return nullptr;
    TreeNode *root = new TreeNode(preorder[pi]);
    auto it = find(inorder.begin() + ii, inorder.begin() + ii + size,
                          preorder[pi]);
    int leftSize = it - inorder.begin() - ii;
    int rightSize = size - leftSize - 1;
    root->left = buildTree(preorder, inorder, pi + 1, ii, leftSize);	// 左子树，只有先序遍历需要跳过 root
    root->right = buildTree(preorder, inorder, pi + 1 + leftSize,		// 右子树，跳过左子树和 root，跳过长度相同
                            ii + leftSize + 1, rightSize);
    return root;
  }
};
```



#### [106. 从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

给定两个整数数组 `inorder` 和 `postorder` ，其中 `inorder` 是二叉树的中序遍历， `postorder` 是同一棵树的后序遍历，请你构造并返回这颗 _二叉树_ 。

题解——注意到中序遍历和后序遍历的数组可以分解为
$$
\begin{aligned}
S_{i}(root) &= S_{i}(root\to left) + root + S_{i}(root\to right)\\
S_{p}(root) &= S_{i}(root\to left) + S_{i}(root\to right) + root 
\end{aligned}
$$
递归关系与之前相同。

```cpp
class Solution {
Public:
  TreeNode *buildTree (vector<int> &inorder, vector<int> &postorder) {
    Return buildTree (inorder, postorder, 0, 0, inorder.Size ());
  }

  TreeNode *buildTree (vector<int> &inorder, vector<int> &postorder, int ii,
                      Int pi, int size) {
    If (size == 0)
      Return nullptr;

    TreeNode *root = new TreeNode (postorder[pi + size - 1]);
    Auto it =
        Find (inorder.Begin () + ii, inorder.Begin () + ii + size, root->val);
    Int leftSize = it - inorder.Begin () - ii;
    Int rightSize = size - leftSize - 1;
    Root->left = buildTree (inorder, postorder, ii, pi, leftSize);		// 左子树，都不需要移动索引
    Root->right = buildTree (inorder, postorder, ii + leftSize + 1,		// 右子树，中序遍历跳过左子树和 root，后序遍历跳过左子树
                            Pi + leftSize, rightSize);
    Return root;
  }
};
```



#### [117. 填充每个节点的下一个右侧节点指针 II](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node-ii/)

给定一个二叉树：

```cpp
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```

填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 `NULL` 。

初始状态下，所有 next 指针都被设置为 `NULL` 。

题解——基本结构仿照之前。首先是向下深入的过程要依次考虑 `lefr,right,next`；然后是考虑每层的首节点，如果它是叶节点，就以 `next` 为首节点；最后是连接循环，`p` 要连接左，如果左为空则连接右；连接后 `p` 向右移动，如果左为空则移动到右，如果右为空则不动。

```cpp
class Solution {
public:
  Node *connect(Node *root) {
    Node *p = root;
    while (p != nullptr) {
      link(p);
      if (p->left)
        p = p->left;
      else if (p->right)
        p = p->right;
      else
        p = p->next;
    }
    return root;
  }

  void link(Node *first) {
    if (first->left == nullptr && first->right == nullptr) {
      if (first->next)
        link(first->next);
      return;
    }

    Node *p = nullptr;
    while (first != nullptr) {
      if (p)
        p->next = first->left ? first->left : first->right;
      if (first->left)
        first->left->next = first->right;
      if (first->right)
        p = first->right;
      else if (first->left)
        p = first->left;
      first = first->next;
    }
  }
};
```



#### [114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)

给你二叉树的根结点 `root` ，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode` ，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null` 。
- 展开后的单链表应该与二叉树 [**先序遍历**](https://baike.baidu.com/item/%E5%85%88%E5%BA%8F%E9%81%8D%E5%8E%86/6442839?fr=aladdin) 顺序相同。

题解——递归获得左右子树展开的结果（首尾节点）。注意要解决边界问题

- 如果恰好是叶节点，则首尾节点设为叶节点
- 如果左子树展开为空，则 `root` 要接右子树起点
- 如果右子树展开为空，则返回的尾节点应该是左子树的尾节点

```cpp
class Solution {
public:
  void flatten(TreeNode *root) {
    TreeNode *first, *last;
    flatten(root, first, last);
  }

  void flatten(TreeNode *root, TreeNode *&first, TreeNode *&last) {
    if (root == nullptr) {
      first = last = nullptr;
      return;
    }
    if (root->left == nullptr && root->right == nullptr) {
      first = last = root;
      return;
    }

    TreeNode *first1 = nullptr, *last1 = nullptr;
    TreeNode *first2 = nullptr, *last2 = nullptr;

    flatten(root->left, first1, last1);
    flatten(root->right, first2, last2);

    root->right = first1;
    root->left = nullptr;

    if (last1) {
      last1->right = first2;
      last1->left = nullptr;
    } else {
      root->right = first2;
    }

    first = root;
    // 注意修改尾部节点
    last = last2 ? last2 : last1;
  }
};
```

更简单的方案，是获得前驱节点。考虑当前节点的左右子树，右子树的前一个节点应该是左子树最右下方的节点，我们找到这个节点（蓝色），将其连接到右子树，最后将左子树设为当前节点的右子树。现在当前节点的位置已经正确，现在切换到右子节点重复操作。

![[image-20250417134558568.png|650]]

>[!note]
>注意如果左子树为空，则不应该改变右子树。

```cpp
class Solution {
private:
  void dfs(TreeNode *root) {
    if (root == nullptr)
      return;

    TreeNode *left = root->left;
    TreeNode *right = root->right;
    // 修改左子树为右子树
    root->left = nullptr;
    if (left)
      root->right = left;

	// 找到前驱节点
    while (left && left->right) {
      left = left->right;
    }
    if (left)
      left->right = right;

    dfs(root->right);
  }

public:
  void flatten(TreeNode *root) { dfs(root); }
};
```

>[!note]
>尾递归可以优化为循环。



#### [112. 路径总和](https://leetcode.cn/problems/path-sum/)

给你二叉树的根节点 `root` 和一个表示目标和的整数 `targetSum` 。判断该树中是否存在 **根节点到叶子节点** 的路径，这条路径上所有节点值相加等于目标和 `targetSum` 。如果存在，返回 `true` ；否则，返回 `false` 。

**叶子节点** 是指没有子节点的节点。

题解——递归计算左右子树。注意路径必须抵达叶节点，因此不能用当前节点为空作为达成条件，反而应该失败。要求左右节点为空并且目标值恰好为当前节点值才成立。

```cpp
class Solution {
public:
  bool hasPathSum(TreeNode *root, int targetSum) {
    if (root == nullptr)
      return false;
    else if (root->left == nullptr && root->right == nullptr &&
             targetSum == root->val)
      return true;
    return hasPathSum(root->left, targetSum - root->val) ||
           hasPathSum(root->right, targetSum - root->val);
  }
};
```



#### [129. 求根节点到叶节点数字之和](https://leetcode.cn/problems/sum-root-to-leaf-numbers/)

给你一个二叉树的根节点 `root` ，树中每个节点都存放有一个 `0` 到 `9` 之间的数字。

每条从根节点到叶节点的路径都代表一个数字：

- 例如，从根节点到叶节点的路径 `1 -> 2 -> 3` 表示数字 `123` 。

计算从根节点到叶节点生成的 **所有数字之和** 。

**叶节点** 是指没有子节点的节点。

题解——深度优先搜索遍历所有路径。仍然要注意必须到达叶节点，因此当前节点为空时什么也不做，否则记录当前字符。如果是叶节点则将路径转换为数值，否则对左右子树递归。

```cpp
class Solution {
private:
  int ret = 0;
  string path = "";

  void dfs(TreeNode *root) {
    if (root == nullptr)
      return;

    path += '0' + root->val;
    if (root->right == nullptr && root->left == nullptr)
      ret += stoi(path);

    dfs(root->left);
    dfs(root->right);
    path.pop_back();
  }

public:
  int sumNumbers(TreeNode *root) {
    dfs(root);
    return ret;
  }
};
```



#### [173. 二叉搜索树迭代器](https://leetcode.cn/problems/binary-search-tree-iterator/)

实现一个二叉搜索树迭代器类`BSTIterator` ，表示一个按中序遍历二叉搜索树（BST）的迭代器：

- `BSTIterator(TreeNode root)` 初始化 `BSTIterator` 类的一个对象。BST 的根节点 `root` 会作为构造函数的一部分给出。指针应初始化为一个不存在于 BST 中的数字，且该数字小于 BST 中的任何元素。
- `boolean hasNext()` 如果向指针右侧遍历存在数字，则返回 `true` ；否则返回 `false` 。
- `int next()`将指针向右移动，然后返回指针处的数字。

注意，指针初始化为一个不存在于 BST 中的数字，所以对 `next()` 的首次调用将返回 BST 中的最小元素。

你可以假设 `next()` 调用总是有效的，也就是说，当调用 `next()` 时，BST 的中序遍历中至少存在一个下一个数字。

题解——深度优先搜索，利用栈保存路径。调用 `next` 时，一直向左子树移动，得到以 `node` 为根节点的子树中的最小元素返回。然后将 `node` 更新为最小元素的左子节点，等下次调用时初始化；如果左子节点为空，则获得栈顶元素作为新的根节点。

```cpp
class BSTIterator {
private:
  TreeNode *node;
  stack<TreeNode *> path;

public:
  BSTIterator(TreeNode *root) : node(root) {}

  int next() {
    while (node != nullptr) {
      path.push(node);
      node = node->left;
    }
    node = path.top();
    path.pop();
    int ret = node->val;
    node = node->right;
    return ret;
  }

  bool hasNext() { return !path.empty() || node; }
};
```



#### [222. 完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/)

给你一棵 **完全二叉树** 的根节点 `root` ，求出该树的节点个数。

[完全二叉树](https://baike.baidu.com/item/%E5%AE%8C%E5%85%A8%E4%BA%8C%E5%8F%89%E6%A0%91/7773232?fr=aladdin) 的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 `h` 层（从第 0 层开始），则该层包含 `1~ 2h` 个节点。

题解——最直接的方案就是递归遍历所有节点

```cpp
class Solution {
public:
  int countNodes(TreeNode *root) {
    if (root == nullptr)
      return 0;
    return countNodes(root->left) + countNodes(root->right) + 1;
  }
};
```



#### [236. 二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

题解——首先遍历整棵树，存放每个节点对应的父节点。然后从 `p` 向上追溯，记录路径上的节点，最后从 `q` 向上追溯，直到第一次遇到记录的节点就返回。

```cpp
class Solution {
private:
  unordered_map<TreeNode *, TreeNode *> parents;
  unordered_set<TreeNode *> path;

public:
  TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *p, TreeNode *q) {
    parents[root] = nullptr;
    inorder(root);
    while (p != nullptr) {
      path.insert(p);
      p = parents[p];
    }

    while (q != nullptr) {
      if (path.count(q))
        return q;
      q = parents[q];
    }
    return root;
  }

  void inorder(TreeNode *root) {
    if (root) {
      if (root->left)
        parents[root->left] = root;
      if (root->right)
        parents[root->right] = root;
      inorder(root->left);
      inorder(root->right);
    }
  }
};
```



### 二叉树层序遍历

#### [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

题解——问题的关键在于，沿着右子树移动后，可能无法到达树的最深层，此时还需要考虑左子树中超过右子树深度的部分。所以我们递归调用，获得当前节点和深度，只有深度超过已经记录的元素数时才记录当前节点。并且我们优先对右子树递归，确保右侧元素总是最先被输出。

```cpp
class Solution {
private:
  vector<int> ret;

  void rec(TreeNode *root, int depth) {
    if (root == nullptr)
      return;
    if (depth >= ret.size())
      ret.push_back(root->val);
    rec(root->right, depth + 1);
    rec(root->left, depth + 1);
  }

public:
  vector<int> rightSideView(TreeNode *root) {
    rec(root, 0);
    return ret;
  }
};
```



#### [637. 二叉树的层平均值](https://leetcode.cn/problems/average-of-levels-in-binary-tree/)

给定一个非空二叉树的根节点 `root` , 以数组的形式返回每一层节点的平均值。与实际答案相差 `10^-5` 以内的答案可以被接受。

题解——用一个数组保存每层的总和，一个数组保存每层节点数。递归调用，当深度超过数组长度就记录新的层，最后计算平均值。

```cpp
class Solution {
private:
  vector<double> ret;
  vector<int> count;

  void dfs(TreeNode *root, int depth) {
    if (root == nullptr)
      return;

    if (depth >= ret.size()) {
      ret.push_back(root->val);
      count.push_back(1);
    } else {
      ret[depth] += root->val;
      count[depth]++;
    }

    dfs(root->left, depth + 1);
    dfs(root->right, depth + 1);
  }

public:
  vector<double> averageOfLevels(TreeNode *root) {
    dfs(root, 0);
    for (int i = 0; i < ret.size(); i++) {
      ret[i] /= count[i];
    }
    return ret;
  }
};
```



#### [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

题解——广度优先搜索。用队列记录边界元素，从边界中取出元素，将其子节点加入到队列中。注意到需要区分层级，因此每次记录该层的元素数后，再取出队列中该层的所有元素并添加子节点。

```cpp
class Solution {
public:
  vector<vector<int>> levelOrder(TreeNode *root) {
    if (root == nullptr)
      return {};
    vector<vector<int>> ret;
    queue<TreeNode *> boundary;
    boundary.push(root);
    while (!boundary.empty()) {
      ret.emplace_back(vector<int>{});
      int size = boundary.size();
      for (int i = 0; i < size; i++) {
        TreeNode *front = boundary.front();
        boundary.pop();
        if (front != nullptr) {
          TreeNode *left = front->left;
          TreeNode *right = front->right;
          ret.back().push_back(front->val);
          boundary.push(left);
          boundary.push(right);
        }
      }
    }
    ret.pop_back();
    return ret;
  }
};
```



#### [103. 二叉树的锯齿形层序遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/)

给你二叉树的根节点 `root` ，返回其节点值的 **锯齿形层序遍历** 。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

题解——用两个栈保存当前层和下一层的节点，用栈是因为入栈顺序和出栈顺序相反。记录下一层节点后就交换两个栈。注意推入左右节点的顺序也需要在每层循环后反转。

```cpp
class Solution {
public:
  vector<vector<int>> zigzagLevelOrder(TreeNode *root) {
    if (root == nullptr)
      return {};
    vector<vector<int>> ret;
    stack<TreeNode *> s1, s2;
    auto *bd1 = &s1;
    auto *bd2 = &s2;
    bool zig = false;

    bd1->push(root);
    while (!bd1->empty()) {
      zig = !zig;
      swap(bd1, bd2);
      ret.emplace_back(vector<int>{});
      while (!bd2->empty()) {
        TreeNode *front = bd2->top();
        bd2->pop();
        if (front != nullptr) {
          TreeNode *left = front->left;
          TreeNode *right = front->right;
          ret.back().push_back(front->val);
          if (zig) {
            bd1->push(left);
            bd1->push(right);
          } else {
            bd1->push(right);
            bd1->push(left);
          }
        }
      }
    }
    ret.pop_back();
    return ret;
  }
};
```



### 二叉搜索树

#### [530. 二叉搜索树的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/)

给你一个二叉搜索树的根节点 `root` ，返回 **树中任意两不同节点值之间的最小差值** 。

差值是一个正数，其数值等于两值之差的绝对值。

题解——注意到中序遍历可以得到递增的数组，只需要比较相邻两个元素差，获得最小值即可。因此使用中序遍历，将比较操作放在中间，记录前一个元素的值。

```cpp
class Solution {
private:
  int diff = INT_MAX;
  int prev = INT_MAX;

  void inorder(TreeNode *root) {
    if (root == nullptr)
      return;

    inorder(root->right);
    diff = min(diff, abs(prev - root->val));
    prev = root->val;
    inorder(root->left);
  }

public:
  int getMinimumDifference(TreeNode *root) {
    inorder(root);
    return diff;
  }
};
```



#### [230. 二叉搜索树中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)

给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 小的元素（从 1 开始计数）。

题解——同样利用中序遍历，记录中序元素的个数，第 $k$ 个中序元素就是第 $k$ 小的元素，找到后可以直接返回，避免重复。

```cpp
class Solution {
private:
  int count = 0;
  int kth = 0;

  void inorder(TreeNode *root, int k) {
    if (root == nullptr)
      return;

    inorder(root->left, k);
    count++;
    if (count == k) {
      kth = root->val;
      return;
    }
    inorder(root->right, k);
  }

public:
  int kthSmallest(TreeNode *root, int k) {
    inorder(root, k);
    return kth;
  }
};
```



#### [98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)

给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

**有效** 二叉搜索树定义如下：

- 节点的左子树只包含 **小于** 当前节点的数。
- 节点的右子树只包含 **大于** 当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。

题解——递归检测，需要获得左右子树的范围，然后比较

```cpp
class Solution {
public:
  bool isValidBST(TreeNode *root) {
    auto [valid, _, _] = isValid(root);
    return valid;
  }

  tuple<bool, long, long> isValid(TreeNode *root) {
    if (root == nullptr) {
      return {true, LONG_MAX, -LONG_MAX};
    }
    auto [valid1, min1, max1] = isValid(root->left);
    auto [valid2, min2, max2] = isValid(root->right);
    return {valid1 && valid2 && (max1 < root->val && root->val < min2),
            min(min1, (long)root->val), max(max2, (long)root->val)};
  }
};
```

注意到可以将检测倒过来：一开始设置区间 `[MIN, MAX]`，从根节点开始，向左或向右

- 向左 `[MIN, root->val]`
- 向右 `[root->val, MAX]`

要求子树在此区间范围内。

```cpp
class Solution {
public:
  bool helper(TreeNode *root, long long lower, long long upper) {
    if (root == nullptr) {
      return true;
    }
    if (root->val <= lower || root->val >= upper) {
      return false;
    }
    return helper(root->left, lower, root->val) &&
           helper(root->right, root->val, upper);
  }
  bool isValidBST(TreeNode *root) { return helper(root, LONG_MIN, LONG_MAX); }
};
```



### 图

#### [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

题解——递归搜索即可。遍历每个元素，向周围延伸，直到遇到边界或者已经标记过的元素/水。

```cpp
class Solution {
private:
  bool rec(vector<vector<char>> &grid, int i, int j) {
    if (i < 0 || i >= grid.size() || j < 0 || j >= grid[0].size() ||
        grid[i][j] != '1')
      return false;

	// 注意标记防止重复访问
    grid[i][j] = '2';

    rec(grid, i + 1, j);
    rec(grid, i - 1, j);
    rec(grid, i, j + 1);
    rec(grid, i, j - 1);

    return true;
  }

public:
  int numIslands(vector<vector<char>> &grid) {
    int m = grid.size();
    int n = grid[0].size();
    int count = 0;
    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        if (grid[i][j] == '1')
          count += rec(grid, i, j);
      }
    }
    return count;
  }
};
```



#### [130. 被围绕的区域](https://leetcode.cn/problems/surrounded-regions/)

给你一个 `m x n` 的矩阵 `board` ，由若干字符 `'X'` 和 `'O'` 组成，**捕获** 所有 **被围绕的区域**：

- **连接：** 一个单元格与水平或垂直方向上相邻的单元格连接。
- **区域：连接所有** `'O'` 的单元格来形成一个区域。
- **围绕：** 如果您可以用 `'X'` 单元格 **连接这个区域**，并且区域中没有任何单元格位于 `board` 边缘，则该区域被 `'X'` 单元格围绕。

通过 **原地** 将输入矩阵中的所有 `'O'` 替换为 `'X'` 来 **捕获被围绕的区域**。你不需要返回任何值。

题解——直接的想法是遍历所有元素，递归检测周围元素，扩展直到碰到边界。但是这种做法很难判断是否被包围，因为无法确定深度优先搜索路径的方向，即使路径没有碰到边界，也有可能其实在另一个方向上能够到达边界。所以我们可以反过来，从边界出发填充所有连通的 `O`，最后对这些标记赋值。

```cpp
class Solution {
private:
  void rec(vector<vector<char>> &board, int i, int j) {
    if (i < 0 || i >= board.size() || j < 0 || j >= board[0].size() ||
        board[i][j] != 'O')
      return;

    board[i][j] = 'Y';
    rec(board, i + 1, j);
    rec(board, i - 1, j);
    rec(board, i, j + 1);
    rec(board, i, j - 1);
  }

public:
  void solve(vector<vector<char>> &board) {
    int m = board.size();
    int n = board[0].size();
    // 边界出发遍历
    for (int i = 0; i < m; i++) {
      rec(board, i, 0);
      rec(board, i, n - 1);
    }
    for (int j = 1; j < n - 1; j++) {
      rec(board, 0, j);
      rec(board, m - 1, j);
    }
    // 标记赋值
    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        if (board[i][j] == 'O')
          board[i][j] = 'X';
        else if (board[i][j] == 'Y')
          board[i][j] = 'O';
      }
    }
  }
};
```



#### [133. 克隆图](https://leetcode.cn/problems/clone-graph/)

给你无向 **[连通](https://baike.baidu.com/item/%E8%BF%9E%E9%80%9A%E5%9B%BE/6460995?fr=aladdin)** 图中一个节点的引用，请你返回该图的 [**深拷贝**](https://baike.baidu.com/item/%E6%B7%B1%E6%8B%B7%E8%B4%9D/22785317?fr=aladdin)（克隆）。

图中的每个节点都包含它的值 `val`（`int`） 和其邻居的列表（`list[Node]`）。

```coo
class Node {
    public int val;
    public List<Node> neighbors;
}
```

**测试用例格式：**

简单起见，每个节点的值都和它的索引相同。例如，第一个节点值为 1（`val = 1`），第二个节点值为 2（`val = 2`），以此类推。该图在测试用例中使用邻接列表表示。

**邻接列表** 是用于表示有限图的无序列表的集合。每个列表都描述了图中节点的邻居集。

给定节点将始终是图中的第一个节点（值为 1）。你必须将 **给定节点的拷贝** 作为对克隆图的引用返回。

题解——广度优先搜索。我们建立新节点与旧节点的映射关系，维护优先队列，当旧节点无法映射到新节点时，就创建新节点并将旧节点添加到队列中。

>[!note]
>注意这题不能修改旧节点的值，所以通过改动旧节点值来判断是否重复添加节点并不可行。

```cpp
class Solution {
private:
  unordered_map<Node *, Node *> nmap;

public:
  Node *cloneGraph(Node *node) {
    if (node == nullptr)
      return nullptr;

    nmap[node] = new Node(node->val);
    queue<Node *> boundary;
    boundary.push(node);
    while (!boundary.empty()) {
      int size = boundary.size();
      for (int i = 0; i < size; i++) {
        Node *front = boundary.front();
        boundary.pop();

        auto nfront = nmap[front];
        for (auto &n : front->neighbors) {
          if (nmap.find(n) == nmap.end()) {
            nmap[n] = new Node(n->val);
            boundary.push(n);
          }
          nfront->neighbors.push_back(nmap[n]);
        }
      }
    }
    return nmap[node];
  }
};
```



#### [399. 除法求值](https://leetcode.cn/problems/evaluate-division/)

给你一个变量对数组 `equations` 和一个实数值数组 `values` 作为已知条件，其中 `equations[i] = [Ai, Bi]` 和 `values[i]` 共同表示等式 `Ai / Bi = values[i]` 。每个 `Ai` 或 `Bi` 是一个表示单个变量的字符串。

另有一些以数组 `queries` 表示的问题，其中 `queries[j] = [Cj, Dj]` 表示第 `j` 个问题，请你根据已知条件找出 `Cj / Dj = ?` 的结果作为答案。

返回 **所有问题的答案** 。如果存在某个无法确定的答案，则用 `-1.0` 替代这个答案。如果问题中出现了给定的已知条件中没有出现的字符串，也需要用 `-1.0` 替代这个答案。

**注意：** 输入总是有效的。你可以假设除法运算中不会出现除数为 0 的情况，且不存在任何矛盾的结果。

**注意：** 未在等式列表中出现的变量是未定义的，因此无法确定它们的答案。

题解——只需要建立图之后，找到连接起点和终点的路径。已知 `a / b` 时自然得到 `b / a`，因此图一定是双向的。首先检查起点和终点是否在图中；然后开始深度优先搜索，当起点终点重合就记录值。用一个哈希表保存已访问的元素，避免重复访问。为了剪枝，当已经计算得到值后就不再递归。如果最后没有计算出值，则推入 `-1` 表示无法确定。

```cpp
class Solution {
private:
  unordered_map<string, vector<tuple<string, double>>> emap;
  unordered_set<string> visited;
  vector<double> ret;

  bool dfs(string s, string e, double val) {
    if (s == e) {
      ret.push_back(val);
      return true;
    }

    bool finish = false;
    for (auto &sv : emap[s]) {
      auto [ss, v] = sv;
      if (visited.find(ss) != visited.end())
        continue;
      visited.insert(ss);
      finish = dfs(ss, e, val * v);
      visited.erase(ss);

      if (finish)
        return true;
    }
    return false;
  }

public:
  vector<double> calcEquation(vector<vector<string>> &equations,
                              vector<double> &values,
                              vector<vector<string>> &queries) {
    for (int i = 0; i < values.size(); i++) {
      auto &ss = equations[i];
      auto v = values[i];
      emap[ss[0]].push_back(make_tuple(ss[1], v));
      emap[ss[1]].push_back(make_tuple(ss[0], 1 / v));
    }

    for (int i = 0; i < queries.size(); i++) {
      if (emap.find(queries[i][0]) == emap.end() ||
          emap.find(queries[i][1]) == emap.end()) {
        ret.push_back(-1);
        continue;
      }
      if (!dfs(queries[i][0], queries[i][1], 1))
        ret.push_back(-1);
    }
    return ret;
  }
};
```



#### [207. 课程表](https://leetcode.cn/problems/course-schedule/)

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程  `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

题解——根据课程之间的依赖关系，建立有向图。注意到输入有课程总数，因此不必借助邻接表，直接**用固定长度的数组，索引对应课程，其元素为邻接元素的数组**。我们要检测图中的环，需要沿着一条路径移动直到碰到路径上的节点。同样可以**用定长数组保存节点是否被访问**

- 用 `0` 表示未访问
- 用 `1` 表示在路径中

由于这是有向图，不能保证从一个节点出发到达任意节点，需要遍历每个节点递归，这样开销就很大。注意到以某个节点为起点，如果深度优先搜索后找不到环，那么它一定不是环的一部分，所以不用再访问。所以

- 用 `2` 表示已完成

对应的节点不再递归，能够确保每个节点只访问一次。

```cpp
class Solution {
private:
  vector<vector<int>> nmap;
  vector<int> visited;
  bool finish = true;

  void dfs(int val) {
    if (!finish)
      return;

    visited[val] = 1;
    for (auto &n : nmap[val]) {
      if (visited[n] == 0) {
        dfs(n);
      } else if (visited[n] == 1) {
        finish = false;
        return;
      }
    }
    visited[val] = 2;
  }

public:
  bool canFinish(int numCourses, vector<vector<int>> &prerequisites) {
    if (prerequisites.empty())
      return true;

    visited.resize(numCourses, 0);
    nmap.resize(numCourses);
    for (auto &p : prerequisites) {
      auto i = p[0];
      auto j = p[1];
      nmap[i].push_back(j);
    }

    for (int i = 0; i < numCourses; i++) {
      if (finish)
        dfs(i);
      else
        return false;
    }
    return finish;
  }
};
```



#### [210. 课程表 II](https://leetcode.cn/problems/course-schedule-ii/)

现在你总共有 `numCourses` 门课需要选，记为 `0` 到 `numCourses - 1`。给你一个数组 `prerequisites` ，其中 `prerequisites[i] = [ai, bi]` ，表示在选修课程 `ai` 前 **必须** 先选修 `bi` 。

- 例如，想要学习课程 `0` ，你需要先完成课程 `1` ，我们用一个匹配来表示：`[0,1]` 。

返回你为了学完所有课程所安排的学习顺序。可能会有多个正确的顺序，你只要返回 **任意一种** 就可以了。如果不可能完成所有课程，返回 **一个空数组** 。

题解——与前面类似，但不同的是路径记录。考虑到依赖关系，要修读一门，就以它为起点，深度优先搜索到达叶节点，即没有依赖关系的节点，将该节点添加到路径中。根据递归思想，对所有相邻节点递归后，就可以修读当前课程，所以在递归末尾记录课程，并标记为“已修读”。

- 用 `0` 表示未访问
- 用 `1` 表示在路径中
- 用 `2` 表示已修读

另外还需要注意图中的环，如果递归过程中遇到路径上的节点，表示出现环，不能全部修读。考虑到图不一定连通，需要遍历所有节点。

```cpp
class Solution {
private:
  vector<vector<int>> nmap;
  vector<int> visited;
  vector<int> path;
  bool finish = true;

  void dfs(int val) {
    if (!finish || visited[val] == 2)
      return;

    visited[val] = 1;
    for (auto &n : nmap[val]) {
      if (visited[n] == 1) {
        finish = false;
        return;
      }
      dfs(n);
    }
    path.push_back(val);
    visited[val] = 2;
  }

public:
  vector<int> findOrder(int numCourses, vector<vector<int>> &prerequisites) {
    visited.resize(numCourses, 0);
    nmap.resize(numCourses);
    for (auto &p : prerequisites) {
      auto i = p[0];
      auto j = p[1];
      nmap[i].push_back(j);
    }

    for (int i = 0; i < numCourses; i++) {
      dfs(i);
    }
    if (finish)
      return path;
    else
      return {};
  }
};
```



### 图的广度优先搜索

#### [909. 蛇梯棋](https://leetcode.cn/problems/snakes-and-ladders/)

给你一个大小为 `n x n` 的整数矩阵 `board` ，方格按从 `1` 到 `n2` 编号，编号遵循 [转行交替方式](https://baike.baidu.com/item/%E7%89%9B%E8%80%95%E5%BC%8F%E8%BD%AC%E8%A1%8C%E4%B9%A6%E5%86%99%E6%B3%95/17195786) ，**从左下角开始** （即，从 `board[n - 1][0]` 开始）的每一行改变方向。

你一开始位于棋盘上的方格  `1`。每一回合，玩家需要从当前方格 `curr` 开始出发，按下述要求前进：

- 选定目标方格 `next` ，目标方格的编号在范围 `[curr + 1, min(curr + 6, n2)]` 。
    - 该选择模拟了掷 **六面体骰子** 的情景，无论棋盘大小如何，玩家最多只能有 6 个目的地。
- 传送玩家：如果目标方格 `next` 处存在蛇或梯子，那么玩家会传送到蛇或梯子的目的地。否则，玩家传送到目标方格 `next` 。 
- 当玩家到达编号 `n2` 的方格时，游戏结束。

如果 `board[r][c] != -1` ，位于 `r` 行 `c` 列的棋盘格中可能存在 “蛇” 或 “梯子”。那个蛇或梯子的目的地将会是 `board[r][c]`。编号为 `1` 和 `n2` 的方格不是任何蛇或梯子的起点。

注意，玩家在每次掷骰的前进过程中最多只能爬过蛇或梯子一次：就算目的地是另一条蛇或梯子的起点，玩家也 **不能** 继续移动。

- 举个例子，假设棋盘是 `[[-1,4],[-1,3]]` ，第一次移动，玩家的目标方格是 `2` 。那么这个玩家将会顺着梯子到达方格 `3` ，但 **不能** 顺着方格 `3` 上的梯子前往方格 `4` 。（简单来说，类似飞行棋，玩家掷出骰子点数后移动对应格数，遇到单向的路径（即梯子或蛇）可以直接跳到路径的终点，但如果多个路径首尾相连，也不能连续跳多个路径）

返回达到编号为 `n2` 的方格所需的最少掷骰次数，如果不可能，则返回 `-1`。

题解——广度优先搜索。题目要求从左下角出发，走 Z 形到达右上角。每次可以走 `1~6` 格，如果遇到的格子里的值不为 `-1`，就要跳转到值对应的编号位置，所以需要提供一个编号到位置的转换函数。

>[!note]
>注意边界问题：如果编号超过范围，返回位置就是右上角。

然后维护队列，利用贪心思路，如果后面 6 格没有跳转，就只移动到最远的格子；如果存在跳转，就移动到不跳转的最远格子。记录新格子时，更新到达的最远位置。

```cpp
class Solution {
public:
  int snakesAndLadders(vector<vector<int>> &board) {
    int n = board.size();
    auto f = [n](int x) -> tuple<int, int> {
      int r = x / n;
      int c = x % n;
      if (r % 2 == 1) {
        c = n - c - 1;
      }
      r = min(r, n - 1);
      return make_tuple(n - 1 - r, c);
    };

    int step = 0;
    int maxPos = 1;
    queue<int> boundary;
    boundary.push(1);
    while (maxPos < n * n) {
      if (step > n * n / 6)
        return -1;

      int size = boundary.size();
      for (int i = 0; i < size; i++) {
        int front = boundary.front();
        boundary.pop();

        bool hasMax = false;
        for (int j = 6; j >= 1; j--) {
          auto [ti, tj] = f(front + j - 1);
          if (board[ti][tj] != -1)
            boundary.push(board[ti][tj]);
          else if (!hasMax) {
            hasMax = true;
            boundary.push(front + j);
          }
          maxPos = max(maxPos, boundary.back());
        }
      }
      step++;
    }
    return step;
  }
};
```



#### [433. 最小基因变化](https://leetcode.cn/problems/minimum-genetic-mutation/)

基因序列可以表示为一条由 8 个字符组成的字符串，其中每个字符都是 `'A'`、`'C'`、`'G'` 和 `'T'` 之一。

假设我们需要调查从基因序列 `start` 变为 `end` 所发生的基因变化。一次基因变化就意味着这个基因序列中的一个字符发生了变化。

- 例如，`"AACCGGTT" --> "AACCGGTA"` 就是一次基因变化。

另有一个基因库 `bank` 记录了所有有效的基因变化，只有基因库中的基因才是有效的基因序列。（变化后的基因必须位于基因库 `bank` 中）

给你两个基因序列 `start` 和 `end` ，以及一个基因库 `bank` ，请你找出并返回能够使 `start` 变化为 `end` 所需的最少变化次数。如果无法完成此基因变化，返回 `-1` 。

注意：起始基因序列 `start` 默认是有效的，但是它并不一定会出现在基因库中。

题解——首先建立邻接表。为了方便检索，将 `bank` 中的基因和 `startGene` 存在哈希表中，同时确认 `endGene` 在该表中。然后遍历库中的基因，依次修改一个字符，如果修改后的基因在库中，就记录连接关系。最后进行广度优先搜索，用一个哈希表保存已经访问过的基因，避免重复访问。 `

```cpp
class Solution {
public:
  int minMutation(string startGene, string endGene, vector<string> &bank) {
    unordered_map<string, vector<string>> graph;
    unordered_set<string> bankSet;
    unordered_set<string> visited;

    bankSet.insert(startGene);
    for (auto &s : bank)
      bankSet.insert(s);
    if (bankSet.find(endGene) == bankSet.end())
      return -1;

    char C[] = {'A', 'C', 'G', 'T'};
    for (auto &s : bankSet) {
      for (int i = 0; i < 8; i++) {
        string str = s;
        for (int j = 0; j < 4; j++) {
          str[i] = C[j];
          if (bankSet.find(str) != bankSet.end())
            graph[s].push_back(str);
        }
      }
    }

    int step = 0;
    queue<string> boundary;
    boundary.push(startGene);
    visited.insert(startGene);
    while (!boundary.empty()) {
      int size = boundary.size();
      for (int i = 0; i < size; i++) {
        string str = boundary.front();
        boundary.pop();
        
        if (str == endGene)
          return step;

        for (auto &s : graph[str]) {
          if (visited.find(s) == visited.end()) {
            boundary.push(s);
            visited.insert(s);
          }
        }
      }
      step++;
    }
    return -1;
  }
};
```



#### [208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)

**[Trie](https://baike.baidu.com/item/%E5%AD%97%E5%85%B8%E6%A0%91/9825209?fr=aladdin)**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补全和拼写检查。

请你实现 Trie 类：

- `Trie()` 初始化前缀树对象。
- `void insert(String word)` 向前缀树中插入字符串 `word` 。
- `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false` 。
- `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix` ，返回 `true` ；否则，返回 `false` 。

题解——前缀树利用节点路径保存字符串，每个节点有 26 个子节点，并且保存一个布尔标记。当节点标记为叶节点，表示到这个节点为止是一个字符串。例如 `apple, app` 插入树中，则 `e, p` 节点被标记为叶节点。默认情况下，新建的节点不是叶节点，插入完成后将最后一个节点标记为叶节点。

```cpp
struct Node {
  bool leaf;
  Node *children[26];

  Node() : leaf(false) {
    for (int i = 0; i < 26; i++)
      children[i] = nullptr;
  }
};

class Trie {
private:
  Node *root;

public:
  Trie() { root = new Node; }

  void insert(string word) {
    Node *node = root;
    for (auto &c : word) {
      int i = c - 'a';
      if (node->children[i] == nullptr)
        node->children[i] = new Node;
      node = node->children[i];
    }
    node->leaf = true;
  }

  bool search(string word) {
    Node *node = root;
    for (auto &c : word) {
      int i = c - 'a';
      if (node->children[i] == nullptr)
        return false;
      node = node->children[i];
    }
    return node->leaf;
  }

  bool startsWith(string prefix) {
    Node *node = root;
    for (auto &c : prefix) {
      int i = c - 'a';
      if (node->children[i] == nullptr)
        return false;
      node = node->children[i];
    }
    return true;
  }
};
```

注意到这里 `search, startsWith` 只有最后一行不一样。



#### [211. 添加与搜索单词 - 数据结构设计](https://leetcode.cn/problems/design-add-and-search-words-data-structure/)

请你设计一个数据结构，支持 添加新单词 和 查找字符串是否与任何先前添加的字符串匹配 。

实现词典类 `WordDictionary` ：

- `WordDictionary()` 初始化词典对象
- `void addWord(word)` 将 `word` 添加到数据结构中，之后可以对它进行匹配
- `bool search(word)` 如果数据结构中存在字符串与 `word` 匹配，则返回 `true` ；否则，返回  `false` 。`word` 中可能包含一些 `'.'` ，每个 `.` 都可以表示任何一个字母。

题解——使用前缀树结构保存词。插入过程与前面一致；由于 `.` 的存在，我们利用递归查找。遍历每个字符，如果不是 `.`，就按照之前的查找方法；如果是 `.`，就遍历所有子节点递归搜索，没找到就直接返回 `false` 。

```cpp
struct Node {
  bool leaf;
  Node *children[26];

  Node() : leaf(false) {
    for (int i = 0; i < 26; i++)
      children[i] = nullptr;
  }
};

class WordDictionary {
private:
  Node *root;

  bool search(Node *node, string word) {
    for (int k = 0; k < word.size(); k++) {
      char c = word[k];
      if (c == '.') {
        for (int i = 0; i < 26; i++) {
          if (node->children[i] &&
              search(node->children[i], word.substr(k + 1)))
            return true;
        }
        return false;
      } else {
        int i = c - 'a';
        if (node->children[i] == nullptr)
          return false;
        node = node->children[i];
      }
    }
    return node->leaf;
  }

public:
  WordDictionary() { root = new Node; }

  void addWord(string word) {
    Node *node = root;
    for (auto &c : word) {
      int i = c - 'a';
      if (node->children[i] == nullptr)
        node->children[i] = new Node;
      node = node->children[i];
    }
    node->leaf = true;
  }

  bool search(string word) { return search(root, word); }
};
```



### 回溯

#### [17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2021/11/09/200px-telephone-keypad2svg.png)

题解——建表映射，每当添加 `m` 种新字符时，在原来的 ` n ` 个字符串后依次附加后 `m-1` 个字符后推入容器，第一个字符直接在原始字符串上修改。

```cpp
const string table[] = {"abc", "def",  "ghi", "jkl",
                        "mno", "pqrs", "tuv", "wxyz"};
                        
class Solution {
public:
  vector<string> letterCombinations(string digits) {
    if (digits == "")
      return {};

    vector<string> ret = {""};
    for (int i = 0; i < digits.size(); i++) {
      const string &str = table[digits[i] - '2'];
      int len = ret.size();
      for (int k = 0; k < len; k++) {
        for (int j = 1; j < str.size(); j++) {
          ret.push_back(ret[k] + str[j]);
        }
        ret[k] += str[0];
      }
    }

    return ret;
  }
};
```



#### [77. 组合](https://leetcode.cn/problems/combinations/)

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

你可以按 **任何顺序** 返回答案。

题解——回溯递归：考虑状态方程
$$
S (left, right, k)=S (left+1, right, k)\cup S (arr[left]; left+1, right, k-1)
$$
其中右边两种状态分别对应是否选择 `arr[left]` 元素，并且第一种状态可以展开为循环。

```cpp
Class Solution {
Private:
  vector<vector<int>> ret;
  vector<int> path;

  Void dfs (int left, int right, int k) {
    If (k == 0) {
      Ret. Push_back (path);
      Return;
    }

    For (int i = left; i < right; i++) {
      Path. Push_back (i + 1);
      Dfs (i + 1, right, k - 1);
      Path. Pop_back ();
    }
  }

Public:
  vector<vector<int>> combine (int n, int k) {
    Dfs (0, n, k);
    Return ret;
  }
};
```



#### [46. 全排列](https://leetcode.cn/problems/permutations/)

给定一个不含重复数字的数组 `nums` ，返回其 _所有可能的全排列_ 。你可以 **按任意顺序** 返回答案。

题解——深度优先搜索。考虑状态方程
$$
S(i) = \bigcup_{k=i,\cdots n}S(arr[k];i+1)
$$
其中 $S(i)$ 表示从 `i` 位置开始后面所有的全排列，$S(arr[k];i+1)$ 表示选择 `arr[k]` 后从 `i + 1` 位置开始后面所有的全排列。注意到后面元素的顺序不影响全排列结果，因此可以将 `arr[i], arr[k]` 交换来实现选取。

```cpp
class Solution {
private:
  vector<vector<int>> ret;

  void dfs(vector<int> &nums, int start) {
    if (start == nums.size()) {
      ret.emplace_back(nums);
      return;
    }

    for (int i = start; i < nums.size(); i++) {
      swap(nums[start], nums[i]);
      dfs(nums, start + 1);
      swap(nums[start], nums[i]);
    }
  }

public:
  vector<vector<int>> permute(vector<int> &nums) {
    dfs(nums, 0);
    return ret;
  }
};
```



#### [39. 组合总和](https://leetcode.cn/problems/combination-sum/)

给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。

`candidates` 中的 **同一个** 数字可以 **无限制重复被选取** 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

题解——回溯深度优先搜索，每个值搜索次数未知，需要额外一条回溯路径。组合方程满足
$$
S(arr,tar,idx) = S(arr, tar, idx+1)\cup (S(arr,tar-arr[idx],idx)+arr[idx])
$$
对应不选择当前值和选择当前值的回溯路径，后者 `emplace_back` 元素。

```cpp
class Solution {
public:
  void dfs(vector<int> &candidates, int target, vector<vector<int>> &ans,
           vector<int> &combine, int idx) {
    if (idx == candidates.size())
      return;
    if (target == 0) {
      ans.emplace_back(combine);
      return;
    }

    // 沿着路径深入
    dfs(candidates, target, ans, combine, idx + 1);
    // 到达终点，开始记录当前数
    if (target - candidates[idx] >= 0) {
      combine.emplace_back(candidates[idx]);
      // 推入当前节点，以当前节点为基准继续计算
      dfs(candidates, target - candidates[idx], ans, combine, idx);
      combine.pop_back();
    }
  }

  vector<vector<int>> combinationSum(vector<int> &candidates, int target) {
    vector<vector<int>> ans;
    vector<int> combine;
    dfs(candidates, target, ans, combine, 0);
    return ans;
  }
};
```

>[!note]
>这里第一条路径可以展开为循环。



#### [22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

题解——回溯法。用 $S(open,close)$ 表示有 `open,close` 个开/闭括号时的状态，考虑状态方程
$$
S(open,close)=S(open+1,close)\cup S(open,close+1)
$$
然后排除掉无效的组合：

- 只有 `open` 不超过 `n` 个时才增加开括号；
- 只有 `close` 不超过 `open` 时才增加闭括号

```cpp
class Solution {
private:
  vector<string> ret;
  string path;

  void dfs(int open, int close, int n) {
    if (path.size() == 2 * n) {
      ret.push_back(path);
      return;
    }

    if (open < n) {
      path += '(';
      dfs(open + 1, close, n);
      path.pop_back();
    }
    if (close < open) {
      path += ')';
      dfs(open, close + 1, n);
      path.pop_back();
    }
  }

public:
  vector<string> generateParenthesis(int n) {
    dfs(0, 0, n);
    return ret;
  }
};
```



#### [79. 单词搜索](https://leetcode.cn/problems/word-search/)

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

题解——从每个单元格出发，开始深度优先搜索。状态方程
$$
\begin{aligned}
S (i, j, w) &= S (i+1,j,w (1:))\lor S (i-1,j,w (1:))\\
&\lor S (i, j+1,w (1:))\lor S (i, j-1,w (1:))
\end{aligned}
$$
在递归过程中要进行边界判断，并且为了避免重复，使用临时数组 `tmp` 保存每次增加递归深度时所在的字符，使用 `swap` 替换掉用过的字符来避免重复。

```cpp

class solution {
private:
  vector<char> tmp;

public:
  bool exist(vector<vector<char>> &board, string word) {
    tmp.resize(word.size(), '0');
    int m = board.size();
    int n = board[0].size();
    // 考虑 1 x 1 表格
    if (m == 1 && n == 1 && word.size() == 1 && board[0][0] == word[0])
      return true;
    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        if (dfs(board, i, j, 0, word))
          return true;
      }
    }
    return false;
  }

  bool dfs(vector<vector<char>> &board, int i, int j, int left,
           const string &word) {
    if (left >= word.size())
      return true;
    if (board[i][j] != word[left])
      return false;

    int m = board.size();
    int n = board[0].size();
    bool r = false;
    // 使用 swap 记录并替换使用过的元素
    swap(tmp[left], board[i][j]);
    if (!r && i + 1 < m)
      r = r || dfs(board, i + 1, j, left + 1, word);
    if (!r && i - 1 >= 0)
      r = r || dfs(board, i - 1, j, left + 1, word);
    if (!r && j + 1 < n)
      r = r || dfs(board, i, j + 1, left + 1, word);
    if (!r && j - 1 >= 0)
      r = r || dfs(board, i, j - 1, left + 1, word);
    swap(tmp[left], board[i][j]);
    // 递归完成后需要恢复状态
    return r;
  }
};
```



### 分治

#### [108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵平衡二叉搜索树。

题解——划分为左右两部分，递归转换即可。

```cpp
class Solution {
public:
  TreeNode *sortedArrayToBST(vector<int> &nums) {
    return sortedArrayToBST(nums.begin(), nums.end());
  }

  TreeNode *sortedArrayToBST(vector<int>::iterator beg,
                             vector<int>::iterator end) {
    if (beg == end)
      return nullptr;

    int d = (end - beg) / 2;
    auto node = new TreeNode(*(beg + d));
    node->left = sortedArrayToBST(beg, beg + d);
    node->right = sortedArrayToBST(beg + d + 1, end);
    return node;
  }
};
```



#### [148. 排序链表](https://leetcode.cn/problems/sort-list/)

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。

题解——直接思路是使用快速排序或者归并排序。首先给出快速排序：添加一个空的头节点便于处理边界；两个指针从头部出发，`left->next` 寻找第一个大于头节点的位置，`right->next` 寻找第一个小于头节点的位置。注意 `right` 只需要初始化一次，之后直接从当前位置出发。

```cpp
class Solution {
private:
  void sort(ListNode *head, ListNode *end) {
    if (head->next == end)
      return;

    ListNode *left = head;
    ListNode *right = end;
    while (true) {
      while (left->next != end && left->next->val <= head->next->val) {
        left = left->next;
      }
      if (left->next == end)
        break;
      if (right == end)
        right = left->next;
      while (right->next != end && right->next->val >= head->next->val) {
        right = right->next;
      }
      if (right->next == end)
        break;
      swap(left->next->val, right->next->val);
    }
    swap(head->next->val, left->val);

    sort(head, left);
    sort(left, end);
  }

public:
  ListNode *sortList(ListNode *head) {
    if (head == nullptr)
      return head;

    srand(time(0));
    ListNode *node = new ListNode;
    node->next = head;
    sort(node, nullptr);

    return head;
  }
};
```

然而快速排序在处理顺序和逆序两种测试样例时速度过慢，因此可以改为归并排序。这里运用技巧

>[!note]
>快慢指针：快指针每次走两步，慢指针每次走一步。当快指针到达末尾时，慢指针位于中间。

找到中间位置后，将链表断开，然后递归排序，最后归并链表。

```cpp
class Solution {
private:
  ListNode *sort(ListNode *head) {
    if (head == nullptr || head->next == nullptr)
      return head;

    ListNode *fast = head;
    ListNode *slow = head;
    while (fast != nullptr && fast->next != nullptr) {
      fast = fast->next->next;
      if (fast == nullptr)
        break;
      slow = slow->next;
    }
    ListNode *mid = slow->next;
    slow->next = nullptr;
    return merge(sort(head), sort(mid));
  }

  ListNode *merge(ListNode *head1, ListNode *head2) {
    if (head1 == nullptr)
      return head2;
    if (head2 == nullptr)
      return head1;

    if (head1->val < head2->val) {
      head1->next = merge(head1->next, head2);
      return head1;
    } else {
      head2->next = merge(head1, head2->next);
      return head2;
    }
  }

public:
  ListNode *sortList(ListNode *head) {
    if (head == nullptr)
      return head;

    return sort(head);
  }
};
```

如果考虑到归并和排序过程中递归的开销，还可以进行简化。首先归并可以从递归改为循环；其次，排序过程可以改成自底向上——先划分为长度为 1 的子链表，然后两两合并为长度为 2 的子链表，以此类推来代替递归。



#### [427. 建立四叉树](https://leetcode.cn/problems/construct-quad-tree/)

给你一个 `n * n` 矩阵 `grid` ，矩阵由若干 `0` 和 `1` 组成。请你用四叉树表示该矩阵 `grid` 。

你需要返回能表示矩阵 `grid` 的 四叉树 的根结点。

四叉树数据结构中，每个内部节点只有四个子节点。此外，每个节点都有两个属性：

- `val`：储存叶子结点所代表的区域的值。1 对应 **True**，0 对应 **False**。注意，当 `isLeaf` 为 **False** 时，你可以把 **True** 或者 **False** 赋值给节点，两种值都会被判题机制 **接受** 。
- `isLeaf`: 当这个节点是一个叶子结点时为 **True**，如果它有 4 个子节点则为 **False** 。

```cpp
class Node {
    public boolean val;
    public boolean isLeaf;
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;
}
```

我们可以按以下步骤为二维区域构建四叉树：

1. 如果当前网格的值相同（即，全为 `0` 或者全为 `1`），将 `isLeaf` 设为 True ，将 `val` 设为网格相应的值，并将四个子节点都设为 Null 然后停止。
2. 如果当前网格的值不同，将 `isLeaf` 设为 False， 将 `val` 设为任意值，然后如下图所示，将当前网格划分为四个子网格。
3. 使用适当的子网格递归每个子节点。

题解——一种解法是在 4 分递归时，在递归到单个单元格时构建节点，然后在上层获得子节点，判断子节点的值是否相同，如果相同就合并节点，不同就设置节点，称为**自底向上**递归。但是这种方法会导致合并节点，原先申请的内存需要释放，所以开销更大。我们选择**自顶向下**递归，先检查当前范围网格是否相同，如果不同则进行递归。

```cpp
class Solution {
private:
  Node *build(vector<vector<int>> &grid, int si, int sj, int ei, int ej) {
    int mi = (si + ei) / 2;
    int mj = (sj + ej) / 2;
    for (int i = si; i < ei; i++) {
      for (int j = sj; j < ej; j++) {
        if (grid[i][j] != grid[si][sj]) {
          return new Node(0, false, build(grid, si, sj, mi, mj),
                          build(grid, si, mj, mi, ej),
                          build(grid, mi, sj, ei, mj),
                          build(grid, mi, mj, ei, ej));
        }
      }
    }
    return new Node(grid[si][sj], true);
  }

public:
  Node *construct(vector<vector<int>> &grid) {
    return build(grid, 0, 0, grid.size(), grid.size());
  }
};
```



### Kadane 算法

#### [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。

题解——动态规划，考虑以 `i` 元素结尾的子数组的最大和。于是有状态方程
$$
S(i) = max(arr(i), arr(i)+S(i-1))
$$
记录最大的 $S(i)$ 即可。

```cpp
class Solution {
public:
  int maxSubArray(vector<int> &nums) {
    int m = nums[0];
    for (int i = 1; i < nums.size(); i++) {
      nums[i] = max(nums[i], nums[i] + nums[i - 1]);
      m = max(m, nums[i]);
    }
    return m;
  }
};
```



#### [918. 环形子数组的最大和](https://leetcode.cn/problems/maximum-sum-circular-subarray/)

给定一个长度为 `n` 的**环形整数数组** `nums` ，返回 _`nums` 的非空 **子数组** 的最大可能和_ 。

**环形数组** 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。

**子数组** 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。

题解——直观的方法是使用滑动窗口，构造一个 `2 * n - 1` 长度的数组，然后维护一个数组范围，要求范围内元素和递增或者尽可能不减小。这需要 $O(n)$ 空间复杂度。注意到最大和有两种

- 位于数组内部范围
- 跨过数组边界范围

第一种情况可以通过最大子数组和算法得到；第二种情况的数组和应该是所有元素和减去最小子数组和。

![[image-20250412155110695.png|475]]

于是我们维护最大和和最小和，记录前一次的最大值和最小值，使用动态规划更新，同时记录所有元素和。最终的结果应该是 `maxVal, sum - minVal` 中的最大值。另外，注意到有可能 `minVal` 恰好取得所有元素，那么剩余数组为空，这个值没有意义，因此要返回 `maxVal` 。

```cpp
class Solution {
public:
  int maxSubarraySumCircular(vector<int> &nums) {
    int maxVal = nums[0];
    int minVal = nums[0];
    int prevMax = nums[0];
    int prevMin = nums[0];
    int sum = nums[0];
    for (int i = 1; i < nums.size(); i++) {
      prevMax = max(prevMax + nums[i], nums[i]);
      prevMin = min(prevMin + nums[i], nums[i]);
      maxVal = max(maxVal, prevMax);
      minVal = min(minVal, prevMin);
      sum += nums[i];
    }
    if (maxVal > 0)
      return max(maxVal, sum - minVal);
    else
      return maxVal;
  }
};
```



### 二分查找

#### [35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

题解——标准二分查找，比想象中麻烦。首先是退出条件，如果这样写

```cpp
class Solution {
public:
  int searchInsert(vector<int> &nums, int target) {
    int left = 0, right = nums.size(), mid = 0;
    while (left < right) {
      mid = (left + right) / 2;
      if (nums[mid] == target)
        return mid;
      else if (nums[mid] < target)
        left = mid;
      else
        right = mid;
    }
    return mid;
  }
};
```

看上去是对的，但是如果 `left == right - 1`，则 `mid == left`，此时循环无法退出，所以似乎可以改成

```cpp
class Solution {
public:
  int searchInsert(vector<int> &nums, int target) {
    int left = 0, right = nums.size(), mid = 0;
    while (left + 1 < right) {
      mid = (left + right) / 2;
      if (nums[mid] == target)
        return mid;
      else if (nums[mid] < target)
        left = mid;
      else
        right = mid;
    }
    return mid;
  }
};
```

这样能够退出，但会引入新的问题

- 由于返回 `mid`，循环的退出条件与 `mid` 无关，因此可能出现 `mid == left, mid == right` 两种情况，这会导致麻烦的边界问题。

首先解决循环问题：只需要让 `left = mid + 1`，这样即使 `left, right` 相邻，也能够确保退出；其次是返回位置，因为右边界是开的，所以直接返回 `left` 。

```cpp
class Solution {
public:
  int searchInsert(vector<int> &nums, int target) {
    int left = 0, right = nums.size(), mid = 0;
    while (left < right) {
      mid = (left + right) / 2;
      if (nums[mid] == target)
        return mid;
      else if (nums[mid] < target)
        left = mid + 1;
      else
        right = mid;
    }
    return left;
  }
};
```

我们验证 `left` 就是正确的位置：

- 如果 `nums[mid] == target`，则返回 `mid`
- 如果 `nums[mid] < target`，目标总是插入在右侧，那么 `left = mid + 1` 满足要求
- 如果 `nums[mid] > target`，目标要插入在右侧，但是 `right` 是开的，因此设为 `mid` 满足要求

并且显然不可能出现 `left > right`，所以返回位置是唯一的。



#### [74. 搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按非严格递增顺序排列。
- 每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

题解——先对第一列二分查找，再对找到的行二分查找。注意考虑查找边界情况。

```cpp
class Solution {
public:
  bool searchMatrix(vector<vector<int>> &matrix, int target) {
    vector<int> tar{target};
    auto rit = lower_bound(matrix.begin(), matrix.end(), tar,
                           [](auto &a, auto &b) { return a[0] < b[0]; });
    if (rit != matrix.end() && rit->front() == target)
      return true;
    if (rit != matrix.begin())
      rit--;
    auto cit = lower_bound(rit->begin(), rit->end(), target);
    return cit != rit->end() && *cit == target;
  }
};
```



#### [162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/)

峰值元素是指其值严格大于左右相邻值的元素。

给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 **任何一个峰值** 所在位置即可。

你可以假设 `nums[-1] = nums[n] = -∞` 。

你必须实现时间复杂度为 `O(log n)` 的算法来解决此问题。

题解——注意到左右两端都一定是下坡，因此从任何位置出发，只要朝着上坡方向就一定能够到达一个峰值。由于只要找到一个峰值，因此可以二分查找：从中点出发，只要不是峰值，就可以找到上坡的方向。注意在左右边界一定是下坡。

```cpp
class Solution {
public:
  int findPeakElement(vector<int> &nums) {
    int left = 0, right = nums.size(), mid = 0;
    while (left < right) {
      mid = (left + right) / 2;
      bool tl = mid == 0 ? true : nums[mid] > nums[mid - 1];
      bool tr = mid == nums.size() - 1 ? true : nums[mid] > nums[mid + 1];
      if (tl && tr)
        return mid;
      else if (tl)
        left = mid + 1;
      else
        right = mid;
    }
    return left;
  }
};
```

>[!note]
>一个关键条件是测试样例会保证相邻元素不同，这样上下坡总是存在。



#### [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)

整数数组 `nums` 按升序排列，数组中的值 **互不相同** 。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,5,6,7]` 在下标 `3` 处经旋转后可能变为 `[4,5,6,7,0,1,2]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，如果 `nums` 中存在这个目标值 `target` ，则返回它的下标，否则返回 `-1` 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

题解——使用 3 个检测点 `0,mid,n-1`，分为 5 种情况

- `nums[mid]==target`
- `nums[0]<=nums[mid]` 此时 `mid` 在左边有序部分，因此 `[0,mid-1]` 有序
	- `nums[0]<=target && target<nums[mid]` => `target` 在 `[0,mid-1]`
	- `target>nums[mid]` 否则在 `[mid+1,n-1]`
- `nums[0]>nums[mid]` 此时 `mid` 在右边有序部分，因此 `[mid+1,n-1]` 有序
	- `nums[mid]<target && target<=nums[n-1]` => `target` 在 `[mid+1,n-1]
	- `target<nums[mid]` 否则在 `[0,mid-1]`

注意与 `0,n-1` 边界的比较都要有等于。

```cpp
class Solution {
public:
  int search(vector<int> &nums, int target) {
    int n = (int)nums.size();
    if (!n) {
      return -1;
    }
    if (n == 1) {
      return nums[0] == target ? 0 : -1;
    }
    int l = 0, r = n - 1;
    while (l <= r) {
      int mid = (l + r) / 2;
      if (nums[mid] == target)
        return mid;
      if (nums[0] <= nums[mid]) {
        if (nums[0] <= target && target < nums[mid]) {
          r = mid - 1;
        } else {
          l = mid + 1;
        }
      } else {
        if (nums[mid] < target && target <= nums[n - 1]) {
          l = mid + 1;
        } else {
          r = mid - 1;
        }
      }
    }
    return -1;
  }
};
```



#### [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

题解——使用两个二分查找。

```cpp
class Solution {
public:
  vector<int> searchRange(vector<int> &nums, int target) {
    vector<int> range{-1, -1};
    if (nums.empty())
      return range;

    int left = 0, right = nums.size();
    while (left + 1 < right) {
      int mid = (left + right) / 2;
      if (nums[mid] > target)
        right = mid;
      else
        left = mid;
    }

    if (nums[left] != target)
      return range;
    range[1] = left;
    if (nums[0] == target) {
      range[0] = 0;
      return range;
    }

    left = 0;
    while (left + 1 < right) {
      int mid = (left + right) / 2;
      if (nums[mid] >= target)
        right = mid;
      else
        left = mid;
    }
    range[0] = min(int(nums.size()) - 1, right);
    return range;
  }
};
```



#### [153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)

已知一个长度为 `n` 的数组，预先按照升序排列，经由 `1` 到 `n` 次 **旋转** 后，得到输入数组。例如，原数组 `nums = [0,1,2,4,5,6,7]` 在变化后可能得到：

- 若旋转 `4` 次，则可以得到 `[4,5,6,7,0,1,2]`
- 若旋转 `7` 次，则可以得到 `[0,1,2,4,5,6,7]`

注意，数组 `[a[0], a[1], a[2], ..., a[n-1]]` **旋转一次** 的结果为数组 `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]` 。

给你一个元素值 **互不相同** 的数组 `nums` ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 **最小元素** 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

题解——旋转后的数组必然在中间有跳跃，最大值和最小值相邻。我们检查中间位置

- `nums[0] <= nums[mid]` 说明 `mid` 在左侧上升范围，跳跃位置在右侧
- 否则说明跳跃位置在左侧

由于 `left` 总是在 `mid` 基础上向右，因此当 `mid` 位于最大值时，`left` 就位于最小值；当 `mid` 位于最小值时，`right` 就位于最小值。最后返回位置就是跳跃位置。但是需要注意，如果完全升序，那么返回位置是最大值位置，因此需要与 `nums[0]` 比较。

```cpp
class Solution {
public:
  int findMin(vector<int> &nums) {
    int left = 0, right = nums.size() - 1, mid = 0;
    while (left < right) {
      mid = (left + right) / 2;
      if (nums[0] <= nums[mid])
        left = mid + 1;
      else
        right = mid;
    }
    return min(nums[0], nums[left]);
  }
};
```



### 堆

#### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `k` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

题解——利用快速排序的划分过程查找。这里需要强调快排的写法：边界问题非常麻烦，所以必须记住正确的流程。

1. 首先，为了防止边界的复杂性，我们让左右指针都位于有效位置，它们重合时得到结果。
2. 然后考虑如何移动：直观上，左指针向右移动到第一个大于 `choice` 的位置，右指针向左移动到第一个小于 `choice` 的位置，然后交换。
3. 光这样并不够，因为会出现 `[0,1],[1,0]` 两种情况，前者会导致右指针向左越过边界，后者会导致左指针向右越过边界，因此还要判断左右位置是否顺序。并且退出循环后，左右指针的位置都很难把握。
4. 因此更好的方案时左指针向右移动到**第一个大于等于** `choice` 的位置，右指针同理。这样右指针不会越过边界，但是左指针还存在问题。并且假设交换的两个元素值相同，第二次 `while` 循环就无法移动指针。
5. 可以使用 `do while` 解决第二个问题，将左右指针先移出边界，抵消 `do while` 的第一次移动。
6. 注意到第一次循环时，左指针移动一次就会停止，因为它遇到 `choice`；第一次交换后，`choice` 被换到后面，这样左指针遇到后面的 `choice` 就会停止。于是我们顺便就解决了左指针越过边界的问题。

通过上述分析，我们已经得到了有效的、能解决越界问题的方案。并且对于数组中元素都相同的情况，由于两个 `do while` 都至少会移动一次指针，因此循环一定会终止。现在考虑退出循环后的位置：选择左指针还是右指针？有两种情况

- 左右指针重合
- 退出的前一次循环，左右指针相邻，则退出后右指针在左指针的左侧

前一种情况选哪个都可以。后一种情况，右指针应该位于 `<= choice` 的位置，左指针位于 `>= choice` 的位置。右指针一定可以划分到左边，左指针一定可以划分到右边，因此下面两种判断似乎等价

```cpp
if (k <= j)
  return findKth(nums, left, j, k);
else
  return findKth(nums, j + 1, right, k);

if (k < i)
  return findKth(nums, left, i - 1, k);
else
  return findKth(nums, i, right, k);
```

但左右指针的关键区别在于：左指针不会直接移动到最右边，但是右指针可能直接移动到最左边！这导致一个关键结果

>[!note]
>循环结束时，左指针可能没动，但是右指针一定移动了。由于左指针要划到右边，如果选择右边递归，就会导致数组长度不变。

例如

```
1 2 3 4, k = 0
```

左指针由于开始移动到 `choice`，所以会停下；但是紧接着右指针会直接移动到左指针位置。如果使用左指针判断，就会进入

```cpp
findKth(nums, i, right, k);
```

然而左指针 `i` 没动，所以接下来进入的数组长度没变，导致无限递归。而右指针总是会移动：考虑

```
1 2 3 1
```

此时第一轮循环，左右指针都没动；第二轮循环，右指针至少会动一下。所以应该**选择右指针**递归。

```cpp
class Solution {
private:
  int findKth(vector<int> &nums, int left, int right, int k) {
    if (left == right)
      return nums[k];

    int choice = nums[left];
    int i = left - 1, j = right + 1;
    while (i < j) {
      do {
        i++;
      } while (nums[i] < choice);
      do {
        j--;
      } while (nums[j] > choice);
      if (i < j)
        swap(nums[i], nums[j]);
    }
    if (k <= j)
      return findKth(nums, left, j, k);
    else
      return findKth(nums, j + 1, right, k);
  }

public:
  int findKthLargest(vector<int> &nums, int k) {
    return findKth(nums, 0, nums.size() - 1, nums.size() - k);
  }
};
```



#### [373. 查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/)

给定两个以 **非递减顺序排列** 的整数数组 `nums1` 和 `nums2` , 以及一个整数 `k` 。

定义一对值 `(u,v)`，其中第一个元素来自 `nums1`，第二个元素来自 `nums2` 。

请找到和最小的 `k` 个数对 `(u1,v1)`,  `(u2,v2)`  ...  `(uk,vk)` 。

题解——假设 $(i,j)$ 是第 $k$ 小的数对的索引，那么第 $k+1$ 小的数对只能在 $(i+1,j),(i,j+1)$ 之中。所以我们维护一个最小堆，保存数对的索引，索引映射到数对和作为排序函数。如果从 $(0,0)$ 开始，那么后面索引 $(i+1,j),(i,j+1)$ 可能出现重复，例如

```
(0,0) -> (1,0), (0,1)
	  -> (2,0), (1,1), (1,1), (0,1)
```

所以我们可以预先推入 $(0,0),(1,0),\cdots,(k-1,0)$，然后只添加 $(i,j+1)$ 作为新索引，避免重复。容器直接使用优先队列，其内部实现就是堆。

```cpp
class Solution {
public:
  vector<vector<int>> kSmallestPairs(vector<int> &nums1, vector<int> &nums2,
                                     int k) {
    auto compare = [&nums1, &nums2](auto &a, auto &b) {
      return nums1[get<0>(a)] + nums2[get<1>(a)] >
             nums1[get<0>(b)] + nums2[get<1>(b)];
    };
    vector<vector<int>> pairs;
    priority_queue<tuple<int, int>, vector<tuple<int, int>>, decltype(compare)>
        q(compare);
    for (int i = 0; i < k && i < nums1.size(); i++)
      q.emplace(i, 0);
    while (k-- > 0) {
      auto [x, y] = q.top();
      q.pop();
      pairs.push_back(vector<int>{nums1[x], nums2[y]});
      if (y + 1 < nums2.size())
        q.emplace(x, y + 1);
    }
    return pairs;
  }
};
```



### 位运算

#### [67. 二进制求和](https://leetcode.cn/problems/add-binary/)

给你两个二进制字符串 `a` 和 `b` ，以二进制字符串的形式返回它们的和。

题解——获得最长长度，然后倒过来逐字符相加，弹出末尾多余的前导 0，最后反转字符串。

```cpp
class Solution {
public:
  string addBinary(string a, string b) {
    string ret = "";
    int r = 0;
    int n = max(a.size(), b.size());
    for (int i = 0; i < n + 1; i++) {
      if (i < a.size())
        r += (a[a.size() - 1 - i] - '0');
      if (i < b.size())
        r += b[b.size() - 1 - i] - '0';
      ret += '0' + r % 2;
      r = r / 2;
    }
    while (ret.size() > 1 && ret.back() == '0')
      ret.pop_back();
    reverse(ret.begin(), ret.end());
    return ret;
  }
};
```



#### [190. 颠倒二进制位](https://leetcode.cn/problems/reverse-bits/)

颠倒给定的 32 位无符号整数的二进制位。

**提示：**

- 请注意，在某些语言（如 Java）中，没有无符号整数类型。在这种情况下，输入和输出都将被指定为有符号整数类型，并且不应影响您的实现，因为无论整数是有符号的还是无符号的，其内部的二进制表示形式都是相同的。
- 在 Java 中，编译器使用[二进制补码](https://baike.baidu.com/item/%E4%BA%8C%E8%BF%9B%E5%88%B6%E8%A1%A5%E7%A0%81/5295284)记法来表示有符号整数。因此，在 **示例 2** 中，输入表示有符号整数 `-3`，输出表示有符号整数 `-1073741825`。

题解——类似于十进制位反转，但由于位数限制，只需要固定 32 次循环

```cpp
class Solution {
public:
  uint32_t reverseBits(uint32_t n) {
    uint32_t ret = 0;
    for (int i = 0; i < 32; i++) {
      ret = ret << 1;
      ret += n % 2;
      n = n >> 1;
    }
    return ret;
  }
};
```



#### [191. 位1的个数](https://leetcode.cn/problems/number-of-1-bits/)

给定一个正整数 `n`，编写一个函数，获取一个正整数的二进制形式并返回其二进制表达式中 设置位 的个数（也被称为[汉明重量](https://baike.baidu.com/item/%E6%B1%89%E6%98%8E%E9%87%8D%E9%87%8F)）。

题解——直接循环记录每一位中的 1 即可。

```cpp
class Solution {
public:
  int hammingWeight(int n) {
    int count = 0;
    while (n > 0) {
      count += n % 2;
      n = n >> 1;
    }
    return count;
  }
};
```



#### [136. 只出现一次的数字](https://leetcode.cn/problems/single-number/)

给你一个 **非空** 整数数组 `nums` ，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

你必须设计并实现线性时间复杂度的算法来解决此问题，且该算法只使用常量额外空间。

题解——利用异或运算，将数组中所有元素进行异或，则所有重复两次的元素会消去，只剩下唯一的元素。

```cpp
class Solution {
public:
  int singleNumber(vector<int> &nums) {
    int n = nums[0];
    for (int i = 1; i < nums.size(); i++)
      n = n ^ nums[i];
    return n;
  }
};
```



#### [137. 只出现一次的数字 II](https://leetcode.cn/problems/single-number-ii/)

给你一个整数数组 `nums` ，除某个元素仅出现 **一次** 外，其余每个元素都恰出现 **三次。** 请你找出并返回那个只出现了一次的元素。

你必须设计并实现线性时间复杂度的算法且使用常数级空间来解决此问题。

题解——我们需要消去一个位上出现 3 次的元素。注意到可以直接对整个位上累计的值取余得到只出现一次的元素在该位上的值，因此遍历每一位，将所有元素在该位上的值累计即可。

```cpp
class Solution {
public:
  int singleNumber(vector<int> &nums) {
    int ret = 0;
    for (int i = 0; i < 32; i++) {
      int count = 0;
      for (auto &n : nums) {
        count += (n >> i) & 1;
      }
      ret += (1 << i) * (count % 3);
    }
    return ret;
  }
};
```



#### [201. 数字范围按位与](https://leetcode.cn/problems/bitwise-and-of-numbers-range/)

给你两个整数 `left` 和 `right` ，表示区间 `[left, right]` ，返回此区间内所有数字 **按位与** 的结果（包含 `left` 、`right` 端点）。

题解——通过观察可以得到：如果两个数位数不同，则结果一定为零，例如

```
1010 0110 -> 0000
```

如果位数相同，则从不同的位开始归零

```
1101 1110 -> 1100
```

我们可以总结为：从高位开始遍历，连续相同的位保留，如果出现不同的位，则之后的位都为零。

```cpp
class Solution {
public:
  int rangeBitwiseAnd(int left, int right) {
    // 输入都为正 <= 2^31 - 1，因此只需要 2^30 开始
    int r = 1 << 30;
    int ret = 0;
    for (int i = 0; i < 31; i++) {
      int lr = left & r;
      int rr = right & r;
      // 出现不同的位，后面都为零
      if (lr != rr)
        break;
      ret += lr;
      r = r >> 1;
    }
    return ret;
  }
};
```



### 数学

#### [9. 回文数](https://leetcode.cn/problems/palindrome-number/)

给你一个整数 `x` ，如果 `x` 是一个回文整数，返回 `true` ；否则，返回 `false` 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

- 例如，`121` 是回文，而 `123` 不是。

题解——通过转换为字符串可以容易判断，但是有很大优化空间。首先负数以及个位为 0 的非 0 数不是回文；其次，对于回文，当反转到中间时

- 如果有偶数位，则 `x, reverted` 应该相同
- 如果有奇数位，则通过除以 10 去掉最中间的位

循环退出条件就是 `x > reverted`，因为到这里就没必要再转换了。

```cpp
class Solution {
public:
  bool isPalindrome(int x) {
    if (x < 0 || (x % 10 == 0 && x != 0)) {
      return false;
    }

    int revertedNumber = 0;
    while (x > reverted) {
      reverted = reverted * 10 + x % 10;
      x /= 10;
    }
    return x == reverted || x == reverted / 10;
  }
};
```



#### [66. 加一](https://leetcode.cn/problems/plus-one/)

给定一个由 **整数** 组成的 **非空** 数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储**单个**数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

题解——从最后一位开始向前递增，并且如果没有进位就可以直接返回。

```cpp
class Solution {
public:
  vector<int> plusOne(vector<int> &digits) {
    for (int i = digits.size() - 1; i >= 0; i--) {
      digits[i]++;
      if (digits[i] < 10)
        return digits;
      digits[i] = 0;
    }
    digits.insert(digits.begin(), 1);
    return digits;
  }
};
```



#### [172. 阶乘后的零](https://leetcode.cn/problems/factorial-trailing-zeroes/)

给定一个整数 `n` ，返回 `n!` 结果中尾随零的数量。

提示 `n! = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1`

题解——只有 `2 * 5` 可以构成尾随零，并且 2 数量远远多于 5，所以只需要考虑 5 的数量。并且 25 会提供两个 5，因此只需要检查所有 5 的幂的数量。

```cpp
class Solution {
public:
  int trailingZeroes(int n) {
    int count = 0;
    while (n > 0)
      count += (n /= 5);
    return count;
  }
};
```



#### [69. x 的平方根](https://leetcode.cn/problems/sqrtx/)

给你一个非负整数 `x` ，计算并返回 `x` 的 **算术平方根** 。

由于返回类型是整数，结果只保留 **整数部分** ，小数部分将被 **舍去 。**

**注意：** 不允许使用任何内置指数函数和算符，例如 `pow(x, 0.5)` 或者 `x ** 0.5` 。

题解——二分查找即可，注意边界条件。为了确保循环退出，`left, right` 都要在 `mid` 基础上移动。为了防止溢出，需要使用长整型。

```cpp
class Solution {
public:
  int mySqrt(int x) {
    int left = 0, right = x, mid = 0;
    while (left <= right) {
      mid = (left + right) / 2;
      long d = (long)mid * mid;
      if (d == x)
        return mid;
      else if (d < x)
        left = mid + 1;
      else
        right = mid - 1;
    }
    return left - 1;
  }
};
```



#### [50. Pow(x, n)](https://leetcode.cn/problems/powx-n/)

实现 [pow(_x_, _n_)](https://www.cplusplus.com/reference/valarray/pow/) ，即计算 `x` 的整数 `n` 次幂函数（即，`xn` ）。

题解——将负数幂转换为正数，注意考虑 $|x|=1$ 以及 `n` 位于负数边界的情况。之后对于奇数幂转换为偶数幂递归，偶数幂通过自乘使指数减半。

```cpp
class Solution {
public:
  double myPow(double x, int n) {
    if (abs(x) == 1) {
      if (n % 2 == 0)
        return 1;
      else
        return x;
    }
    if (n == -INT_MAX - 1)
      return 0;
    if (n < 0) {
      x = 1 / x;
      n = -n;
    }

    double res = 1.0;
    while (n > 0) {
      if (n % 2 == 1)
        return x * myPow(x, n - 1);
      res *= x;
      x *= x;
      n /= 2;
    }
    return res;
  }
};
```



### 一维动态规划

#### [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

题解——动态规划，保存到达前 1 个台阶和前 2 个台阶的方法数，依次累计。

```cpp
class Solution {
public:
  int climbStairs(int n) {
    int n0 = 0, n1 = 1, n2;
    for (int i = 0; i < n; i++) {
      n2 = n0 + n1;
      n0 = n1;
      n1 = n2;
    }
    return n2;
  }
};
```



#### [198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

题解——状态转移方程
$$
S(i) = \max(S(i-1),S(i-2)+arr[i])
$$
其中 $S(i)$ 表示 $i$ 范围内能偷到的最大金额。

```cpp
class Solution {
public:
  int rob(vector<int> &nums) {
    if (nums.size() == 1)
      return nums[0];
    nums[1] = max(nums[0], nums[1]);
    for (int i = 2; i < nums.size(); i++)
      nums[i] = max(nums[i - 2] + nums[i], nums[i - 1]);
    return nums[nums.size() - 1];
  }
};
```



#### [139. 单词拆分](https://leetcode.cn/problems/word-break/)

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：** 不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

题解——用一个数组 `dp` 保存字符串的划分情况。状态方程
$$
S(i) = \lor_{j}(S(j)\land (s[j,i-1]\in Dict))
$$
每个元素 $S(i)$ 表示它是否可以是一个字符串的边界，因此遍历每个位置，然后从头开始检查 `dp[j]`，即先找到一个边界，然后检查 `[j, i-1]` 范围的字符串是否在字典中，如果在其中，就说明 `dp[i]` 是可以通过拼接得到的边界，此时可以直接跳出循环。最后只要 `dp[s.size()]` 是边界，就说明 `s` 可以拼接得到。

```cpp
class Solution {
public:
  bool wordBreak(string s, vector<string> &wordDict) {
    unordered_set<string> words;
    for (auto &s : wordDict)
      words.insert(s);

    vector<int> dp(s.size() + 1, 0);
    dp[0] = 1;
    for (int i = 1; i < s.size() + 1; i++) {
      for (int j = 0; j < i; j++) {
        if (dp[j] && words.find(s.substr(j, i - j)) != words.end()) {
          dp[i] = 1;
          break;
        }
      }
    }
    return dp[s.size()];
  }
};
```



#### [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。

题解——假设最后一枚硬币为 `coins[j]`，那么只需要计算出组成 `amount - coins[j]` 需要的最少硬币数然后加一即可。因此注意到状态方程
$$
S(i) = \min_{j=0,\cdots,n-1}S(i-coins[j])+1
$$
我们用 `count` 数组保存组成总金额 `i` 所需要的最少硬币数，显然不会超过 `amount + 1` 枚，用来作为边界判断，如果最终 `count[amount]` 没变就说明无法组成。

```cpp
class Solution {
public:
  int coinChange(vector<int> &coins, int amount) {
    vector<int> count(amount + 1, amount + 1);
    count[0] = 0;
    for (int i = 1; i < amount + 1; i++) {
      // 遍历每个硬币面额检测
      for (int j = 0; j < coins.size(); j++) {
        // 在 count[i - coins[j]] 的基础上 +1
        if (i >= coins[j])
          count[i] = min(count[i], count[i - coins[j]] + 1);
      }
    }
    return count[amount] == amount + 1 ? -1 : count[amount];
  }
};
```



#### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

题解——注意到状态方程
$$
S(i) = \max(S(i),\max_{j=0,\cdots,i-1}S(j)+1)
$$
只有在 $arr[i]>arr[j]$ 时更新状态，因此后面的元素不一定比前面保存的结果大，需要维护一个最大长度返回。

```cpp
class Solution {
public:
  int lengthOfLIS(vector<int> &nums) {
    int maxLen = 1;
    vector<int> count(nums.size(), 1);
    for (int i = 1; i < nums.size(); i++) {
      for (int j = 0; j < i; j++) {
        if (nums[i] > nums[j]) {
          count[i] = max(count[i], count[j] + 1);
          maxLen = max(maxLen, count[i]);
        }
      }
    }
    return maxLen;
  }
};
```

另一种更好的方法是贪心法：我们希望长度为 $i$ 的子串尽可能以最小的元素结尾，这样后续的子串才可能更长。使用 $d[i]$ 表示这个最小的元素，则 $d[i]$ 显然应当单增。于

1. 初始化 $d[1]=arr[0],n=1$，然后遍历数组 $i=1,2,\cdots$
2. 对于 $i$ 元素
	1. 如果 $arr[i]>d[n]$，则令 $d[n+1]=arr[i],n=n+1$ 
	2. 否则二分搜索 $d[1],\cdots,d[n]$，找到 $arr[i]$ 的插入位置 $j$，令其替换 $d[j]$ 

这个过程保持 $d$ 的单调性，并且 $i\geq n$，因此每次更新确实可以替换 $d$ 中的元素。



### 多维动态规划

#### [120. 三角形最小路径和](https://leetcode.cn/problems/triangle/)

给定一个三角形 `triangle` ，找出自顶向下的最小路径和。

每一步只能移动到下一行中相邻的结点上。**相邻的结点** 在这里指的是 **下标** 与 **上一层结点下标** 相同或者等于 **上一层结点下标 + 1** 的两个结点。也就是说，如果正位于当前行的下标 `i` ，那么下一步可以移动到下一行的下标 `i` 或 `i + 1` 。

题解——原地动态规划，注意左右边界直接从上一层边界计算。

```cpp
class Solution {
public:
  int minimumTotal(vector<vector<int>> &triangle) {
    for (int i = 1; i < triangle.size(); i++) {
      triangle[i][0] += triangle[i - 1][0];
      for (int j = 1; j < triangle[i].size() - 1; j++) {
        triangle[i][j] += min(triangle[i - 1][j - 1], triangle[i - 1][j]);
      }
      triangle[i].back() += triangle[i - 1].back();
    }
    int minVal = INT_MAX;
    for (int j = 0; j < triangle.back().size(); j++)
      minVal = min(minVal, triangle.back()[j]);
    return minVal;
  }
};
```



#### [64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/)

给定一个包含非负整数的 `_m_ x _n_` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

**说明：** 每次只能向下或者向右移动一步。

题解——与不同路径类似，只需要记录到达 `(i, j)` 位置的最小和。状态方程
$$
S (i, j) = grid[i][j]+min (S (i-1, j), S (i, j-1))
$$
注意第一行需要累计第一行元素，第一列需要累计第一列元素。

```cpp
class Solution {
public:
  int minPathsum(vector<vector<int>> &grid) {
    int m = grid.size();
    int n = grid[0].size();
    vector<int> table(n);
    table[0] = grid[0][0];
    for (int i = 1; i < n; i++)
      table[i] = table[i - 1] + grid[0][i];
    for (int i = 1; i < m; i++) {
      table[0] += grid[i][0];
      for (int j = 1; j < n; j++) {
        table[j] = grid[i][j] + min(table[j - 1], table[j]);
      }
    }
    return table[n - 1];
  }
};
```



#### [63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

给定一个 `m x n` 的整数数组 `grid`。一个机器人初始位于 **左上角**（即 `grid[0][0]`）。机器人尝试移动到 **右下角**（即 `grid[m - 1][n - 1]`）。机器人每次只能向下或者向右移动一步。

网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。机器人的移动路径中不能包含 **任何** 有障碍物的方格。

返回机器人能够到达右下角的不同路径数量。

测试用例保证答案小于等于 `2 * 109`。

题解——与不同路径相同，区别在于首先检测第一行，如果遇到障碍物，之后的列全部设为 0；然后对于剩下的行，检测第一列，如果遇到障碍物，此位置永久设为 0；对于之后的列，如果有障碍物就设为 0，否则设为左边和上边路径数之和。

```cpp
class Solution {
public:
  int uniquePathsWithObstacles(vector<vector<int>> &obstacleGrid) {
    if (obstacleGrid[0][0] == 1)
      return 0;
    int m = obstacleGrid.size();
    int n = obstacleGrid[0].size();
    vector<int> table(n, 1);
    for (int i = 1; i < n; i++) {
      if (obstacleGrid[0][i] == 1)
        table[i] = 0;
      else
        table[i] = table[i - 1];
    }

    for (int i = 1; i < m; i++) {
      if (obstacleGrid[i][0] == 1)
        table[0] = 0;
      for (int j = 1; j < n; j++) {
        if (obstacleGrid[i][j] == 1)
          table[j] = 0;
        else
          table[j] = table[j] + table[j - 1];
      }
    }
    return table[n - 1];
  }
};
```



#### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

**示例 1：**

**输入：** s = "babad"
**输出：**"bab"
**解释：**"aba" 同样是符合题意的答案。

**示例 2：**

**输入：** s = "cbbd"
**输出：**"bb"

**提示：**

- `1 <= s.length <= 1000`
- `s` 仅由数字和英文字母组成

题解——分别以每个字符为中心向两边扩展，还要考虑 `aa` 这种以缝隙为中心的情况

```cpp
class Solution {
public:
  string longestPalindrome(string s) {
    int longestLeft = 0, longestRight = 0;
    for (int i = 0; i < s.size(); i++) {
      // 以 i 为中心和以 i,i+1 为中心
      int left = i, right = i;
      while (left >= 0 && right < s.size() && s[left] == s[right]) {
        left--;
        right++;
      }

      if (right - left - 2 > longestRight - longestLeft) {
        longestLeft = left + 1;
        longestRight = right - 1;
      }

      left = i, right = i + 1;
      while (left >= 0 && right < s.size() && s[left] == s[right]) {
        left--;
        right++;
      }

      if (right - left - 2 > longestRight - longestLeft) {
        longestLeft = left + 1;
        longestRight = right - 1;
      }
    }

    string res = "";
    for (int i = longestLeft; i <= longestRight; i++)
      res += s[i];
    return res;
  }
};
```



#### [97. 交错字符串](https://leetcode.cn/problems/interleaving-string/)

给定三个字符串 `s1`、`s2`、`s3`，请你帮忙验证 `s3` 是否是由 `s1` 和 `s2` **交错** 组成的。

两个字符串 `s` 和 `t` **交错** 的定义与过程如下，其中每个字符串都会被分割成若干 **非空** 子字符串：

- `s = s1 + s2 + ... + sn`
- `t = t1 + t2 + ... + tm`
- `|n - m| <= 1`
- **交错** 是 `s1 + t1 + s2 + t2 + s3 + t3 + ...` 或者 `t1 + s1 + t2 + s2 + t3 + s3 + ...`

**注意：**`a + b` 意味着字符串 `a` 和 `b` 连接。

题解——动态规划，令 $S(i,j)$ 表示 `s1` 的前 `i` 个元素与 `s2` 的前 `j` 个元素可以组成 `s3` 的前 `i+j` 个元素。于是有
$$
\begin{aligned}
S(i,j) = &(S(i-1,j)\land (s_{1}[i-1]==s_{3}[i+j-1])) \lor\\
&(S(i,j-1)\land (s_{2}[j-1]==s_{3}[i+j-1]))
\end{aligned}
$$
注意当 `s1,s2` 长度和不为 `s3` 长度时，显然不可能。

```cpp
class Solution {
public:
  bool isInterleave(string s1, string s2, string s3) {
    if (s1.size() + s2.size() != s3.size()) {
      return false;
    }

    vector<vector<bool>> table(s1.size() + 1,
                               vector<bool>(s2.size() + 1, false));
    // 前 0 个元素当然可以交错组成前 0 个字符
    table[0][0] = true;

    // 计算 s1 前 i 个字符是否可以组成 s3 的前 i 个字符
    for (int i = 1; i < s1.size() + 1; i++) {
      if (s1[i - 1] == s3[i - 1]) {
        table[i][0] = table[i - 1][0];
      }
    }

    // 计算 s2 前 j 个字符是否可以组成 s3 的前 j 个字符
    for (int j = 1; j < s2.size() + 1; j++) {
      if (s2[j - 1] == s3[j - 1]) {
        table[0][j] = table[0][j - 1];
      }
    }

    for (int i = 1; i < s1.size() + 1; i++) {
      for (int j = 1; j < s2.size() + 1; j++) {
        bool test1 = false;
        if (s1[i - 1] == s3[i + j - 1]) {
          test1 = table[i - 1][j];
        }
        bool test2 = false;
        if (s2[j - 1] == s3[i + j - 1]) {
          test2 = table[i][j - 1];
        }
        table[i][j] = test1 || test2;
      }
    }

    return table[s1.size()][s2.size()];
  }
};
```



#### [72. 编辑距离](https://leetcode.cn/problems/edit-distance/)

给你两个单词 `word 1` 和 `word 2`， _请返回将 `word 1` 转换成 `word 2` 所使用的最少操作数_  。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

题解——动态规划。由于对每个单词的操作顺序不影响变换后的结果，因此可以只考虑对最后一个字符的操作。用 $S(i,j)$ 表示将 `word1[0,i]` 转换成 `word2[0,j]` 所需的最少操作数。状态方程
$$
S(i,j) =
\begin{cases}
\min(S(i-1,j),S(i,j-1),S(i-1,j-1))+1, & word 1[i]\neq word 2[j]\\
S(i-1,j-1), & word 1[i]=word 2[j]
\end{cases}
$$
如果 `i,j` 字符相同，就直接等于 $S(i-1,j-1)$；如果不同，有三种情况

1. 先将 `word1[0,i-1]` 变成 `word2[0,j]`，然后删掉 `word1` 最后一个字符
2. 先将 `word1[0,i]` 变成 `word2[0,j-1]`，然后添加 `word2` 最后一个字符
3. 先将 `word[0,i-1]` 变成 `word2[0,j-1]`，然后将 `word1[i]` 替换为 `word2[j]`

还要考虑字符串为空的情况，因此实际代码中用 $S(i,j)$ 表示 `word1[0,i+1]` 转换为 `word2[0,j+1]` 的最少操作数，这样可以预先填充 $S(0,j),S(i,0)$ 元素。

```cpp
class Solution {
public:
  int minDistance(string word1, string word2) {
    int m = word1.size();
    int n = word2.size();
    vector<vector<int>> table(m + 1, vector<int>(n + 1, 0));
    for (int i = 0; i < m + 1; i++)
      table[i][0] = i;
    for (int j = 0; j < n + 1; j++)
      table[0][j] = j;
    for (int i = 1; i < m + 1; i++) {
      for (int j = 1; j < n + 1; j++) {
        if (word1[i - 1] == word2[j - 1])
          table[i][j] = table[i - 1][j - 1];
        else
          table[i][j] =
              min(table[i - 1][j], min(table[i - 1][j - 1], table[i][j - 1])) +
              1;
      }
    }
    return table[m][n];
  }
};
```



#### [221. 最大正方形](https://leetcode.cn/problems/maximal-square/)

在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。

题解——注意到状态方程
$$
S(i,j) = 
\begin{cases}
\min(S(i-1,j-1),S(i-1,j),S(i,j-1))+1, & arr[i][j]=1 \\
0, & arr[i][j]==0
\end{cases}
$$
即以当前位置为右下角的最大正方形以左上 3 个格子为右下角的最大正方形中最小的那个为基准。

```cpp
class Solution {
public:
  int maximalSquare(vector<vector<char>> &matrix) {
    int maxSize = 0;
    
    auto f = [](char c) { return c - '0'; };
    vector<vector<int>> table(matrix.size(), vector<int>(matrix[0].size(), 0));
    for (int i = 0; i < matrix[0].size(); i++) {
      table[0][i] = f(matrix[0][i]);
      maxSize = max(maxSize, table[0][i]);
    }
    
    for (int i = 1; i < matrix.size(); i++) {
      table[i][0] = f(matrix[i][0]);
      maxSize = max(maxSize, table[i][0]);
    }
    
    for (int i = 1; i < matrix.size(); i++) {
      for (int j = 1; j < matrix[i].size(); j++) {
        table[i][j] = min(table[i][j], min(table[i][j - 1], table[i - 1][j]));
        if (f(matrix[i][j]) == 0)
          table[i][j] = 0;
        else
          table[i][j] =
              min(table[i - 1][j - 1], min(table[i][j - 1], table[i - 1][j])) +
              1;
        maxSize = max(maxSize, table[i][j]);
      }
    }
    return maxSize * maxSize;
  }
};
```


