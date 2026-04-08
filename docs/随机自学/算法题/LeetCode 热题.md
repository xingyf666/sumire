# LeetCode 热题

## 热题 100

### 哈希

#### [1. 两数之和](https://leetcode.cn/problems/two-sum/)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** _`target`_  的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

题解——注意返回的是索引，不能直接排序然后双指针。所以构建一个数组同时保存索引和值，然后排序。

```cpp
class Solution {
public:
  vector<int> twoSum(vector<int> &nums, int target) {
    vector<tuple<int, int>> table;
    for (int i = 0; i < nums.size(); i++)
      table.push_back(make_tuple(i, nums[i]));
    sort(table.begin(), table.end(),
         [](auto &a, auto &b) { return get<1>(a) < get<1>(b); });
    int left = 0, right = table.size() - 1;
    while (left < right) {
      int sum = get<1>(table[left]) + get<1>(table[right]);
      if (sum == target)
        return {get<0>(table[left]), get<0>(table[right])};
      else if (sum < target)
        left++;
      else
        right--;
    }
    return {};
  }
};
```



#### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

题解——对每个单词**排序**，作为键保存即可。

>[!note]
>还可以记录每个字母出现的次数。

```cpp
class Solution {
public:
  vector<vector<string>> groupAnagrams(vector<string> &strs) {
    unordered_map<string, int> groups;
    vector<vector<string>> ret;
    for (const string &s : strs) {
      string key = s;
      sort(key.begin(), key.end());
      if (groups.find(key) == groups.end()) {
        groups[key] = ret.size();
        ret.push_back({s});
      } else {
        ret[groups[key]].push_back(s);
      }
    }
    return ret;
  }
};
```



### 双指针

#### [283. 移动零](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

题解——维护一个已经处理完成的数组的边界 `end`，遇到非零元素就将元素交换到边界，同时移动边界。

```cpp
class Solution {
public:
  void moveZeroes(vector<int> &nums) {
    int end = 0;
    for (int i = 0; i < nums.size(); i++) {
      if (nums[i] == 0)
        continue;
      swap(nums[i], nums[end++]);
    }
  }
};
```



#### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：** 你不能倾斜容器。

题解——贪心剪枝：最直接的方法是计算任意两条线之间的盛水量，但这样显然复杂度过高。考虑进行剪枝：首先考虑最左和最右两条线，然后开始移动左线或右线。遍历方法需要固定左线，右线向左移动，到达最左边后再移动左线，将右线放回最右边。那么什么情况可以完全不考虑呢？

- 如果初始左线低于右线，很显然无论怎么移动右线，容量都不会增加，因为宽度不会更大，而高度也不会高于左线，因此此时遍历右线就完全没有必要；
- 反之，如果初始右线低于左线，那么左线也没必要遍历；

因此遍历过程中只需要移动较低的线，这意味着遍历一遍就可以得到最大容量。

```cpp
class Solution {
public:
  int maxArea(vector<int> &height) {
    int left = 0, right = height.size() - 1;
    int area = 0;
    while (left < right) {
      int w = right - left;
      int h = min(height[left], height[right]);
      area = max(area, w * h);
      if (height[left] < height[right])
        left++;
      else
        right--;
    }
    return area;
  }
};
```



#### [15. 三数之和](https://leetcode.cn/problems/3sum/)

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：** 答案中不可以包含重复的三元组。

题解——先排序，然后固定一个索引，其余两个索引从两头向中间遍历，同时跳过重复元素避免三元组重复。

```cpp
class Solution {
public:
  vector<vector<int>> threeSum(vector<int> &nums) {
    vector<vector<int>> ret;
    sort(nums.begin(), nums.end());

    int i = 0;
    while (i < nums.size()) {
      int ri = nums[i];
      int j = i + 1, k = nums.size() - 1;
      while (j < k) {
        int rj = nums[j];
        int rk = nums[k];
        int sum = ri + rj + rk;
        if (sum == 0) {
          ret.push_back({ri, rj, rk});
        }
        if (sum <= 0) {
          while (j++ < k && rj == nums[j])
            ;
        }
        if (sum >= 0) {
          while (j < --k && rk == nums[k])
            ;
        }
      }
      while (++i < nums.size() && ri == nums[i])
        ;
    }

    return ret;
  }
};
```



### 滑动窗口

#### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。

**示例 1:**

**输入:** s = "abcabcbb"
**输出:** 3 
**解释:** 因为无重复字符的最长子串是 `"abc"`，所以其长度为 3。

**示例 2:**

**输入:** s = "bbbbb"
**输出:** 1
**解释:** 因为无重复字符的最长子串是 `"b"`，所以其长度为 1。

**示例 3:**

**输入:** s = "pwwkew"
**输出:** 3
**解释:** 因为无重复字符的最长子串是 `"wke"`，所以其长度为 3。
     请注意，你的答案必须是 **子串** 的长度，`"pwke"` 是一个_子序列，_不是子串。

**提示：**

- `0 <= s.length <= 5 * 104`
- `s` 由英文字母、数字、符号和空格组成

题解——维护滑动窗口

```cpp
class Solution {
public:
  int lengthOfLongestSubstring(string s) {
    if (s.size() == 0)
        return 0;

    int maxLength = 1;
    int left = 0, right = 1;
    for (int i = 1; i < s.size(); i++) {
      for (int j = right - 1; j >= left; j--) {
        if (s[j] == s[i])
        {
          left = j + 1;
          break;
        }
      }
      right++;
      maxLength = max(maxLength, right - left);
    }
    return maxLength;
  }
};
```



#### [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

题解——最直接的方法，是维护长度为 `p.size()` 的滑动窗口，然后排序窗口中的字符进行比较。但是这样复杂度很高，并且没有利用到之前的比较结果。我们可以改为维护一个 `count` 数组，记录窗口中不同字母的个数，如果这些字符个数与 `p` 中的字符个数相同，就说明构成异位词。还可以进一步简化

- 用 `count` 记录窗口与 `p` 中字符个数的差
- 用 `diff` 记录不同字符的个数

当 `diff` 归零，说明构成异位词。移动窗口，左端字符被弹出，右端加入新字符。检查弹出和加入过程中对应 `count` 值的变化，如果变化之前是零，则 `diff++`；如果变化之后是零，则 `diff--` 。注意跳出循环之后还要判断一次 `diff` 值。

```cpp
class Solution {
public:
  vector<int> findAnagrams(string s, string p) {
    if (s.size() < p.size())
      return {};
    vector<int> ret;
    vector<int> count(26, 0);
    for (int i = 0; i < p.size(); i++) {
      count[p[i] - 'a']++;
      count[s[i] - 'a']--;
    }
    int diff = 0;
    for (auto &n : count)
      diff += n == 0 ? 0 : 1;

    for (int i = p.size(); i < s.size(); i++) {
      if (diff == 0)
        ret.push_back(i - p.size());

      if (count[s[i - p.size()] - 'a'] == 0)
        diff++;
      count[s[i - p.size()] - 'a']++;
      if (count[s[i - p.size()] - 'a'] == 0)
        diff--;

      if (count[s[i] - 'a'] == 0)
        diff++;
      count[s[i] - 'a']--;
      if (count[s[i] - 'a'] == 0)
        diff--;
    }
    if (diff == 0)
      ret.push_back(s.size() - p.size());

    return ret;
  }
};
```



### 子串

#### [560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 _该数组中和为 `k` 的子数组的个数_ 。

子数组是数组中元素的连续非空序列。

题解——前缀和问题。考虑以 `i` 位置结尾的子数组，注意到
$$
S(0,i) = S(0,j-1)+S(j,i) \implies S(j,i) = S(0,i)-S(0,j-1)
$$
这意味着 `j,i` 范围和可以由 `i,j-1` 两个前缀和得到。因此
$$
S(j,i) == k \implies S(0,i) - k= S(0,j-1)
$$
所以我们保存所有的 $S(0,j),j<i$，然后检查 $S(0,i)-k$ 是否已经保存。如果是，说明存在 $S(0,j-1)$ 满足上式。考虑到可能有多个 $S(0,j-1)$ 满足要求，因此记录每个和的个数。注意要保存 $S(0,0)$ 作为起点。

```cpp
class Solution {
public:
  int subarraySum(vector<int> &nums, int k) {
    int count = 0;
    int sum = 0;
    unordered_map<int, int> record;
    record[sum] = 1;
    for (auto &n : nums) {
      sum += n;
      if (record.find(sum - k) != record.end())
        count += record[sum - k];
      record[sum]++;
    }
    return count;
  }
};
```



### 矩阵
#### [73. 矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/)

给定一个 `_m_ x _n_` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。请使用 **[原地](http://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 算法。

题解——直接记录需要置零的行和列。

```cpp
class Solution {
public:
  void setZeroes(vector<vector<int>> &matrix) {
    int m = matrix.size();
    int n = matrix[0].size();
    vector<int> rows;
    vector<int> cols;
    for (int i = 0; i < m; i++) {
      for (int j = 0; j < n; j++) {
        if (matrix[i][j] == 0) {
          rows.push_back(i);
          cols.push_back(j);
        }
      }
    }
    for (int i = 0; i < rows.size(); i++) {
      for (int j = 0; j < n; j++) {
        matrix[rows[i]][j] = 0;
      }
    }
    for (int i = 0; i < cols.size(); i++) {
      for (int j = 0; j < m; j++) {
        matrix[j][cols[i]] = 0;
      }
    }
  }
};
```



#### [54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

题解——每次输出最外层，然后递归调用（注意矩阵不是方阵）。

```cpp
class Solution {
private:
  vector<int> ret;

public:
  vector<int> spiralOrder(vector<vector<int>> &matrix) {
    int m = matrix.size();
    int n = matrix[0].size();
    spiralOrder(matrix, 0, m, n);
    return ret;
  }

  void spiralOrder(vector<vector<int>> &matrix, int i, int m, int n) {
    if (m * n == 0)
      return;
    if (m == 1) {
      for (int j = 0; j < n; i++)
        ret.push_back(matrix[i][i + j]);
      return;
    }
    if (n == 1) {
      for (int j = 0; j < m; i++)
        ret.push_back(matrix[i + j][i]);
      return;
    }
    for (int j = 0; j < n - 1; j++)
      ret.push_back(matrix[i][i + j]);
    for (int j = 0; j < m - 1; j++)
      ret.push_back(matrix[i + j][i + n - 1]);
    for (int j = 0; j < n - 1; j++)
      ret.push_back(matrix[i + m - 1][i + n - 1 - j]);
    for (int j = 0; j < m - 1; j++)
      ret.push_back(matrix[i + m - 1 - j][i]);
    spiralOrder(matrix, i + 1, m - 2, n - 2);
  }
};
```



#### [48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

给定一个 _n_ × _n_ 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在 **[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。

题解——每次旋转最外层，然后递归即可。

```cpp
class Solution {
public:
  void rotate(vector<vector<int>> &matrix) {
    int n = matrix.size();
    for (int i = 0; i < n / 2; i++) {
      rotate(matrix, i, n - i * 2);
    }
  }

  void rotate(vector<vector<int>> &matrix, int i, int n) {
    for (int j = 0; j < n - 1; j++) {
      swap(matrix[i + j][i], matrix[i + n - 1][i + j]);
      swap(matrix[i + n - 1][i + j], matrix[i + n - 1 - j][i + n - 1]);
      swap(matrix[i + n - 1 - j][i + n - 1], matrix[i][i + n - 1 - j]);
    }
  }
};
```



#### [240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

编写一个高效的算法来搜索 `_m_ x _n_` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

题解——显然可以遍历查找、按行二分查找。注意到左上角元素总是最小元素，右下角元素总是最大元素。因此

- 我们二分查找第一行，找到的位置如果不是 `target`，则一定大于 `target`，此时以它为左上角的子矩阵中的元素一定大于 `target`，所以应该搜索左半矩阵；
- 二分查找第一列，找到的位置如果不是 `target`，则一定大于 `target`，此时以它为左上角的子矩阵中的元素一定大于 `target`，所以应该搜索上半矩阵。

但是注意到这种递归方式可能陷入死循环，因为搜索到的位置可能越过边界，导致矩阵规模不变。因此更简单直接的方法，是从右上角开始，如果当前位置大于 `target`，则向左移动（搜索左半）；如果小于 `target`，则向下移动（搜索下半）。

```cpp
class Solution {
public:
  bool searchMatrix(vector<vector<int>> &matrix, int target) {
    int n = matrix.size(), m = matrix[0].size();
    int x = 0, y = m - 1;
    while (x < n && y >= 0) {
      if (matrix[x][y] == target)
        return true;
      if (matrix[x][y] < target)
        x++;
      else
        y--;
    }
    return false;
  }
};
```



### 链表

#### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)

给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null` 。

图示两个链表在节点 `c1` 开始相交：

[![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)

题目数据 **保证** 整个链式结构中不存在环。

**注意**，函数返回结果后，链表必须 **保持其原始结构** 。

**自定义评测：**

**评测系统** 的输入如下（你设计的程序 **不适用** 此输入）：

- `intersectVal` - 相交的起始节点的值。如果不存在相交节点，这一值为 `0`
- `listA` - 第一个链表
- `listB` - 第二个链表
- `skipA` - 在 `listA` 中（从头节点开始）跳到交叉节点的节点数
- `skipB` - 在 `listB` 中（从头节点开始）跳到交叉节点的节点数

评测系统将根据这些输入创建链式数据结构，并将两个头节点 `headA` 和 `headB` 传递给你的程序。如果程序能够正确返回相交节点，那么你的解决方案将被 **视作正确答案** 。

题解——使用哈希表显然可以解决；更省空间的方法是先遍历一遍，然后对齐链表长度，再进行遍历，找到相同的两个头节点。

```cpp
class Solution {
public:
  ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
    int countA = 0;
    ListNode *nodeA = headA;
    while (nodeA) {
      countA++;
      nodeA = nodeA->next;
    }

    int countB = 0;
    ListNode *nodeB = headB;
    while (nodeB) {
      countB++;
      nodeB = nodeB->next;
    }

    if (countA < countB) {
      swap(headA, headB);
      swap(countA, countB);
    }

    while (countA > countB) {
      headA = headA->next;
      countA--;
    }

    while (headA) {
      if (headA == headB)
        return headA;
      headA = headA->next;
      headB = headB->next;
    }

    return nullptr;
  }
};
```



#### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/)

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

题解——遍历同时反指。注意头节点要指向空指针。

```cpp
class Solution {
public:
  ListNode *reverseList(ListNode *head) {
    if (head == nullptr || head->next == nullptr)
      return head;
    ListNode *prev = head;
    head = head->next;
    prev->next = nullptr;	// 头节点指向空
    while (head) {
      ListNode *next = head->next;
      head->next = prev;
      prev = head;
      head = next;
    }
    return prev;
  }
};
```



#### [234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。

题解——使用列表保存节点可以很容易验证。如果要在 `O(1)` 空间复杂度下完成，可以利用快慢指针：慢指针移动到中间，然后将之后的节点全部反转，这样就可以对称比较。

```cpp
class Solution {
public:
  bool isPalindrome(ListNode *head) {
    vector<ListNode *> record;
    while (head) {
      record.push_back(head);
      head = head->next;
    }

    for (int i = 0; i < record.size() / 2; i++) {
      if (record[i]->val != record[record.size() - i - 1]->val)
        return false;
    }
    return true;
  }
};
```



#### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/)

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

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



#### [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 _如果链表无环，则返回 `null`。_

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

题解——使用快慢指针，注意到它们相遇时，快指针移动距离为 $a+n(b+c)$，其中 $n$ 表示在环中转的圈数；慢指针移动距离为 $a+b$ 。于是
$$
a+n(b+c)=2(a+b)\implies a= (n-1)(b+c)+c
$$
即入环位置等于转 $n-1$ 圈以后再转 $c$ 。由于慢指针到入环位置距离为 $c$，因此在快慢指针相遇后，再创建一个头节点，与慢指针一起移动，它们最终会在入环位置相遇。

![[image-20250417091200831.png|500]]

```cpp
class Solution {
public:
  ListNode *detectCycle(ListNode *head) {
    if (head == nullptr)
      return nullptr;
    ListNode *fast = head;
    ListNode *slow = head;
    do {
      fast = fast->next;
      if (fast == nullptr)
        break;
      fast = fast->next;
      slow = slow->next;
    } while (fast && fast != slow);

    if (fast != slow)
      return nullptr;

    ListNode *node = head;
    while (node != slow) {
      node = node->next;
      slow = slow->next;
    }
    return node;
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



#### [24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

题解——递归移动后面所有的节点

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
  ListNode *swapPairs(ListNode *head) {
    if (head == nullptr || head->next == nullptr)
      return head;

    ListNode *p = head->next;
    swapPairs(nullptr, head);
    return p;
  }

  void swapPairs(ListNode *prev, ListNode *cur) {
    if (cur == nullptr || cur->next == nullptr)
      return;

    ListNode *p = cur->next;
    cur->next = p->next;
    p->next = cur;
    if (prev != nullptr)
      prev->next = p;
    swapPairs(cur, cur->next);
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



#### [94. 二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

给定一个二叉树的根节点 `root` ，返回 _它的 **中序** 遍历_ 。

题解——递归中序遍历。

```cpp
class Solution {
private:
  vector<int> ret;

  void inorder(TreeNode *root) {
    if (root == nullptr)
      return;

    inorder(root->left);
    ret.push_back(root->val);
    inorder(root->right);
  }

public:
  vector<int> inorderTraversal(TreeNode *root) {
    inorder(root);
    return ret;
  }
};
```



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



#### [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

给你一棵二叉树的根节点，返回该树的 **直径** 。

二叉树的 **直径** 是指树中任意两个节点之间最长路径的 **长度** 。这条路径可能经过也可能不经过根节点 `root` 。

两节点之间路径的 **长度** 由它们之间边数表示。

题解——递归计算左右子树的深度，同时记录左右子树的深度和，作为经过当前节点的最长路径长度。

```cpp

class Solution {
private:
  int maxLen= 0;

  int dfs(TreeNode *root) {
    if (root == nullptr)
      return 0;

    int left = dfs(root->left);
    int right = dfs(root->right);
    maxLen = max(maxLen, left + right + 1);
    return max(left, right) + 1;
  }

public:
  int diameterOfBinaryTree(TreeNode *root) {
    dfs(root);
    return maxLen - 1;
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



#### [108. 将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵平衡二叉搜索树。

题解——先序遍历创建节点，节点值为数组中最中间的值。

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



#### [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

题解——问题的关键在于，沿着右子树移动后，可能无法到达树的最深层，此时还需要考虑左子树中超过右子树深度的部分。所以我们递归调用，获得当前节点和深度，只有深度超过已经记录的元素数时才记录当前节点。并且我们**优先对右子树递归**，确保右侧元素总是最先被输出。

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



#### [437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/)

给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目。

**路径** 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

题解——前缀和。注意到

```
sum(a->b->c->d->e) = sum(a->b->c) + sum(d->e)
sum(d->e) = sum(a->b->c->d->e) - sum(a->b->c)
```

因此从 `a` 开始向下时，我们记录前缀和 `sum(a->...)`，只要在哈希表中查找

```
sum(a->b->c->d->e) - target
```

如果存在，就说明有 `sum(a->b->c)` 满足上面等式，则 `sum(d->e)` 等于目标值。相同的值可以累计，此时有多个不同的前缀和满足等式，因此累加。

>[!note]
>注意 `0` 是一个天然的前缀和，必须在最开始添加。

我们递归返回满足要求的路径数，注意由于子树递归以后要向上返回，因此最后要减少前缀和计数。

```cpp
class Solution {
private:
  unordered_map<long long, int> prefix;

  int dfs(TreeNode *root, long long cur, int targetSum) {
    if (root == nullptr)
      return 0;

    int ret = 0;
    cur += root->val;
    if (prefix.count(cur - targetSum))
      ret = prefix[cur - targetSum];

    prefix[cur]++;
    ret += dfs(root->left, cur, targetSum);
    ret += dfs(root->right, cur, targetSum);
    prefix[cur]--;

    return ret;
  }

public:
  int pathSum(TreeNode *root, int targetSum) {
    prefix[0]++;
    return dfs(root, 0, targetSum);
  }
};
```



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



#### [994. 腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)

在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。

返回 _直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1` _ 。

题解——广度优先搜索，用队列记录所有腐烂橘子的位置，记录新鲜橘子的个数。

```cpp
class Solution {
public:
  int orangesRotting(vector<vector<int>> &grid) {
    int n = grid.size(), m = grid[0].size();
    int fresh = 0;
    queue<tuple<int, int>> old;
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < m; j++)
        if (grid[i][j] == 2)
          old.push(make_tuple(i, j));
        else if (grid[i][j] == 1)
          fresh++;
    }

    int times = 0;
    while (!old.empty()) {
      if (fresh == 0)
        return times;
      times++;

      int size = old.size();
      for (int k = 0; k < size; k++) {
        auto [i, j] = old.front();
        old.pop();

        if (i > 0 && grid[i - 1][j] == 1) {
          old.push(make_tuple(i - 1, j));
          grid[i - 1][j] = 2;
          fresh--;
        }
        if (i + 1 < n && grid[i + 1][j] == 1) {
          old.push(make_tuple(i + 1, j));
          grid[i + 1][j] = 2;
          fresh--;
        }
        if (j > 0 && grid[i][j - 1] == 1) {
          old.push(make_tuple(i, j - 1));
          grid[i][j - 1] = 2;
          fresh--;
        }
        if (j + 1 < m && grid[i][j + 1] == 1) {
          old.push(make_tuple(i, j + 1));
          grid[i][j + 1] = 2;
          fresh--;
        }
      }
    }
    return fresh == 0 ? times : -1;
  }
};
```



#### [207. 课程表](https://leetcode.cn/problems/course-schedule/)

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程  `bi` 。

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



### 回溯

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



#### [78. 子集](https://leetcode.cn/problems/subsets/)

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

题解——考虑状态方程
$$
S (left, right)=S (left+1, right)\cup S (arr[left]; left+1, right)
$$
其中右边两种状态分别对应是否选择 `arr[left]` 元素，并且第一种状态可以展开为循环。

```cpp
class Solution {
private:
  vector<vector<int>> ret;
  vector<int> path;

  void dfs(vector<int> &nums, int left, int right) {
    if (left == right) {
      ret.push_back(path);
      return;
    }

    path.push_back(nums[left]);
    dfs(nums, left + 1, right);
    path.pop_back();

    // 尾递归优化，等价于循环
    dfs(nums, left + 1, right);
  }

public:
  vector<vector<int>> subsets(vector<int> &nums) {
    dfs(nums, 0, nums.size());
    return ret;
  }
};
```



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



#### [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**示例 1：**

**输入：** s = "aab"
**输出：** `[["a","a","b"],["aa","b"]]`

**示例 2：**

**输入：** s = "a"
**输出：** `[["a"]] `

**提示：**

- `1 <= s.length <= 16`
- `s` 仅由小写英文字母组成

题解——首先考虑深度优先搜索（回溯），例如

```
aabb
a -> a -> b -> b (搜索完成，回到上一节点)
       -> bb
```

每次需要判断子串是否是回文串，但这样会造成重复检测，例如

```
aabab
a (o) -> a (o) -> b (o) -> a (o) -> b (o)
                        -> ab (x)
               -> ba (x)
               -> bab (o)
               -> ab (x)
      -> aba (o) -> b (o) 2
      -> abab (x)
aa (o) -> b (o) -> a (o) -> b(o)
                -> ab (x) 2
       -> ba (x) 2
       -> bab (o)
       ...
```

注意到搜索过程中 `b,ab,ba` 都被检测了两次，因此可以预先判断回文串。注意到状态转移方程
$$
f(i,j) = 
\begin{cases}
1, & i=j \\
f(i+1,j-1) \land (s[i]=s[j])
\end{cases}
$$
通过二维表格预存判断结果。

```cpp
class Solution {
private:
  vector<vector<int>> f;
  vector<vector<string>> ret;
  vector<string> path; // 记录深度搜索路径
  int n;

public:
  void dfs(string &s, int left) {
    // 深度达到最大，记录路径
    if (left == n)
      ret.push_back(path);

    for (int j = left; j < n; j++) {
      if (f[left][j]) {
        // 推入当前子串，然后以此字串为基点进行深搜
        path.push_back(s.substr(left, j - left + 1));
        dfs(s, j + 1);
        path.pop_back();
        // 弹出最后一个节点，回到上一个节点开始深搜
      }
    }
  }

  vector<vector<string>> partition(string s) {
    n = s.size();
    f.resize(n, vector<int>(n, true));

    // 状态转移 f(i,j) = f(i+1,j-1) and (s[i] == s[j])
    // 标记 i,j 范围是否是回文，从左下到右上
    for (int i = n - 1; i >= 0; i--) {
      for (int j = i + 1; j < n; j++) {
        f[i][j] = f[i + 1][j - 1] && (s[i] == s[j]);
      }
    }

    dfs(s, 0);
    return ret;
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




