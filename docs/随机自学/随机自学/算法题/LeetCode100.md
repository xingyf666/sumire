# LeetCode

## 题库

### 中等

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
    Return true;
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



#### [59. 螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)

给你一个正整数 `n` ，生成一个包含 `1` 到 `n 2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

题解——类似于螺旋矩阵，将记录顺序改为设置值。

```cpp
Class Solution {
Private:
  vector<vector<int>> matrix;

Public:
  vector<vector<int>> generateMatrix (int n) {
    matrix.Assign (n, vector<int>(n));
    GenerateMatrix (0, n, 1);
    Return matrix;
  }

  Void generateMatrix (int i, int n, int val) {
    If (n == 0)
      Return;
    If (n == 1) {
      Matrix[i][i] = val;
      Return;
    }
    For (int j = 0; j < n - 1; j++)
      Matrix[i][i + j] = val++;
    For (int j = 0; j < n - 1; j++)
      Matrix[i + j][i + n - 1] = val++;
    For (int j = 0; j < n - 1; j++)
      Matrix[i + n - 1][i + n - 1 - j] = val++;
    For (int j = 0; j < n - 1; j++)
      Matrix[i + n - 1 - j][i] = val++;
    GenerateMatrix (i + 1, n - 2, val);
  }
};
```



#### [61. 旋转链表](https://leetcode.cn/problems/rotate-list/)

给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。

题解——记录链表节点，从倒数 `k` 位置切断链表重新连接。

> >更省空间的做法是将链表连成环，利用取模运算的周期性找到断开位置。

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



#### [62. 不同路径](https://leetcode.cn/problems/unique-paths/)

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

题解——动态规划，用 $S (i, j)$ 表示到达 `(i, j)` 位置的路径数，它等于到达左边或上边的路径数之和。状态方程
$$
S (i, j) = S (i-1, j) + S (i, j-1)
$$
由于每次更新一行都只与上一行有关，因此只需要一行数组记录，最左边始终为 1 。

```cpp
class Solution {
public:
  int uniquePaths(int m, int n) {
    vector<int> table(n, 1);
    for (int i = 1; i < m; i++) {
      for (int j = 1; j < n; j++) {
        table[j] = table[j] + table[j - 1];
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
min(S(i-1,j),S(i,j-1),S(i-1,j-1))+1, & word 1[i]\neq word 2[j]\\
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



#### [75. 颜色分类](https://leetcode.cn/problems/sort-colors/)

给定一个包含红色、白色和蓝色、共 `n` 个元素的数组 `nums` ，**[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

我们使用整数 `0`、 `1` 和 `2` 分别表示红色、白色和蓝色。

必须在不使用库内置的 sort 函数的情况下解决这个问题。

题解——遍历数组，维护一个标记 `0,1,2` 最大有序范围的数组。如果遇到 `0`，就将其与 `0` 的最大范围处的元素交换，然后扩大最大范围。注意有可能出现

```
0 1 2 0
// 最大有序范围
0 : 0 - 1
1 : 1 - 2
2 : 2 - 3
// 交换 nums[3] = 0 与 nums[1] (0 的最大有序范围)
```

这里将最后一个 `0` 与 `0` 的最大范围处的 `1` 交换

```
0 0 2 1
// 最大有序范围
0 : 0 - 2
1 : 2 - 2
2 : 2 - 3
// 交换 nums[3] = 1 与 nums[2] (1 的最大有序范围)
```

但是此时前面有一个 `2`，因此还要再将 `1` 与 `1` 的最大范围处的 `2` 交换

```
0 0 1 2
// 最大有序范围
0 : 0 - 2
1 : 2 - 3
2 : 3 - 3
```

最终代码为

```cpp
class Solution {
public:
  void sortColors(vector<int> &nums) {
    int ends[] = {0, 0, 0};
    ends[nums[0]] = 1;
    for (int i = 1; i < nums.size();) {
      // 1 的最大有序范围不小于 0 的最大有序范围
      ends[1] = max(ends[0], ends[1]);
      ends[2] = max(ends[1], ends[2]);

      int a = nums[i];
      swap(nums[ends[nums[i]]], nums[i]);
      ends[a]++;

      // 只有前面元素更小时才增加索引
      if (nums[i] >= nums[i - 1])
        i++;
    }
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
class Solution {
private:
  vector<vector<int>> ret;
  vector<int> path;

  void dfs(int left, int right, int k) {
    if (k == 0) {
      ret.push_back(path);
      return;
    }

    for (int i = left; i < right; i++) {
      path.push_back(i + 1);
      dfs(i + 1, right, k - 1);
      path.pop_back();
    }
  }

public:
  vector<vector<int>> combine(int n, int k) {
    dfs(0, n, k);
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
class Solution {
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



#### [80. 删除有序数组中的重复项 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/)

给你一个有序数组 `nums` ，请你 **[原地](http://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 删除重复出现的元素，使得出现次数超过两次的元素**只出现两次** ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 **[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95) 修改输入数组** 并在使用 O(1) 额外空间的条件下完成。

题解——需要检查出现超过两次的元素，所以检查 `nums[i],nums[i-2]` 是否相同。如果相同，就需要将后面的元素全部向前移动一位。但这样会比较浪费，因为后面如果还有重复，就可以直接向前移动两位甚至更多，减少操作次数。我们记录已经处理好的数组边界 `end=2`，到达 `i` 位置时，比较 `nums[i],nums[end-2]`，即完成部分的倒数第二个元素，确保该元素出现不多于 2 次。如果不同就增加 `end` 移动边界，否则固定边界。

> >我们在代码中使用 `step` 记录当前位置和需要比较位置之间的步长，这与维护 `end` 是相同的。实际上 `i - 2 - step = end - 2` 含义相同。

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



#### [81. 搜索旋转排序数组 II](https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/)

已知存在一个按非降序排列的整数数组 `nums` ，数组中的值不必互不相同。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转** ，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,4,4,5,6,6,7]` 在下标 `5` 处经旋转后可能变为 `[4,5,6,6,7,0,1,2,4,4]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，请你编写一个函数来判断给定的目标值是否存在于数组中。如果 `nums` 中存在这个目标值 `target` ，则返回 `true` ，否则返回 `false` 。

你必须尽可能减少整个操作步骤。

题解——参考“搜索旋转排序数组 I”，区别在于现在检测点为 `l,mid,r`，如果这三个检测点处相同，就要同时移动 `l,r` 缩减边界范围。

```cpp
class Solution {
public:
  bool search(vector<int> &nums, int target) {
    int n = nums.size();
    if (n == 0) {
      return false;
    }
    if (n == 1) {
      return nums[0] == target;
    }
    int l = 0, r = n - 1;
    while (l <= r) {
      int mid = (l + r) / 2;
      if (nums[mid] == target) {
        return true;
      }
      if (nums[l] == nums[mid] && nums[mid] == nums[r]) {
        ++l;
        --r;
      } else if (nums[l] <= nums[mid]) {
        if (nums[l] <= target && target < nums[mid]) {
          r = mid - 1;
        } else {
          l = mid + 1;
        }
      } else {
        if (nums[mid] < target && target <= nums[r]) {
          l = mid + 1;
        } else {
          r = mid - 1;
        }
      }
    }
    return false;
  }
};
```



#### [82. 删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/)

给定一个已排序的链表的头 `head` ， _删除原始链表中所有重复数字的节点，只留下不同的数字_ 。返回 _已排序的链表_ 。

题解——要删除重复两次或以上的节点（注意重复的节点本身也要删除），可以检查 `p,p->next` 是否相同，如果相同，就将 `p->prev` 记为初始位置，然后移动 `p` 直到 `p,p->next` 不同，此时将之前记录的位置连接过来。最后考虑边界情况

1. 头部开始重复：保存头部是否重复，最后移动头部节点
2. 尾部重复：由于需要检查到 `p,p->next` 不同才会裁剪，所以尾部重复不会被删去。最后检查初始位置是否为空，如果不为空说明需要删除后面的节点。

> >在链表处理中，常用的技巧是在头节点前面创建一个临时节点作为起点，避免头节点被删除造成的麻烦。

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



#### [86. 分隔链表](https://leetcode.cn/problems/partition-list/)

给你一个链表的头节点 `head` 和一个特定值 `x` ，请你对链表进行分隔，使得所有 **小于** `x` 的节点都出现在 **大于或等于** `x` 的节点之前。

你应当 **保留** 两个分区中每个节点的初始相对位置。

题解——首先找到第一个大于等于 `x` 的节点，以这个节点的前一个节点作为边界。然后遍历后面的节点，将每个小于 `x` 节点插入到边界位置，同时移动边界。

```cpp
class Solution {
private:
  ListNode *h;

public:
  ListNode *partition(ListNode *head, int x) {
    if (head == nullptr)
      return nullptr;
    h = head;

	// 找到第一个大于等于 x 的节点
    ListNode *first = nullptr;
    ListNode *p = head;
    while (p != nullptr && p->val < x) {
      first = p;
      p = p->next;
    }
    // 此时 first 是前一个节点
    
    ListNode *prev = first;
    while (p != nullptr) {
      ListNode *q = p->next;
      if (p->val < x) {
        remove(prev, p);
        insert(first, p);
        first = p;
      } else {
        prev = p;
      }
      p = q;
    }
    return h;
  }

  // 辅助删除 prev 后的节点
  void remove(ListNode *prev, ListNode *node) {
    if (prev == nullptr)
      return;
    prev->next = node->next;
  }

  // 在 pos 后面插入节点。如果 pos 为空，就将 node 作为头节点
  void insert(ListNode *pos, ListNode *node) {
    if (pos == nullptr) {
      node->next = h;
      h = node;
      return;
    }
    node->next = pos->next;
    pos->next = node;
  }
};
```



#### [89. 格雷编码](https://leetcode.cn/problems/gray-code/)

**n 位格雷码序列** 是一个由 `2n` 个整数组成的序列，其中：

- 每个整数都在范围 `[0, 2n - 1]` 内（含 `0` 和 `2n - 1`）
- 第一个整数是 `0`
- 一个整数在序列中出现 **不超过一次**
- 每对 **相邻** 整数的二进制表示 **恰好一位不同** ，且
- **第一个** 和 **最后一个** 整数的二进制表示 **恰好一位不同**

给你一个整数 `n` ，返回任一有效的 **n 位格雷码序列** 。

题解——观察不同长度的序列

```
  00   01   11   10
 000  001  011  010  110  111  101
0000 0001 0011 0010 0110 0111 0101
```

注意到 3 长度序列是先将 2 长度序列添加 0 开头，然后将开头改为 1，将 2 长度序列倒过来。因此可以得到状态方程
$$
S(n) = S(0;n-1)\cup S^{-1}(1;n-1)
$$
注意到右式中有 $S^{-1}$，这意味着 $S(0;n-1)$ 递归完成后，$S^{-1}$ 应该倒过来执行。然而由于由于异或运算的性质，同一位异或两次就会恢复，因此 $S^{-1}=S$，只不过每一位恢复的顺序相反。

```cpp
class Solution {
private:
  vector<int> ret;
  int path;

  void dfs(int i, int n) {
    if (i == n) {
      ret.push_back(path);
      return;
    }

    dfs(i + 1, n);
    path ^= 1 << i;	// 处理当前位
    dfs(i + 1, n);
  }

public:
  vector<int> grayCode(int n) {
    path = 0;
    dfs(0, n);
    return ret;
  }
};
```



#### [90. 子集 II](https://leetcode.cn/problems/subsets-ii/)

给你一个整数数组 `nums` ，其中可能包含重复元素，请你返回该数组所有可能的 子集（幂集）。

解集 **不能** 包含重复的子集。返回的解集中，子集可以按 **任意顺序** 排列。

题解——考虑状态方程
$$
S(left,n) = S(arr[left];left+1,n-1)\cup S(left+1,n)
$$
右式第二部分可以展开为循环。由于可能出现重复，例如

```
[2 2 2]
// 重复子集
[2 2]
[2   2]
[  2 2]
```

要避免重复，可以强制要求只能选择

```
[2]
[2 2]
[2 2 2]
```

我们首先排序数组，然后可以记录每个元素是否被取用。如果有相邻的相同元素，只有前一个元素被取用的情况下才选择当前元素。

```cpp
class Solution {
private:
  vector<vector<int>> ret;
  vector<int> path;
  vector<int> records;

  void dfs(vector<int> &nums, int left) {
    for (int i = left; i < nums.size(); i++) {
      if (i > 0 && nums[i] == nums[i - 1] && !records[i - 1])
        continue;
      records[i] = 1;
      path.push_back(nums[i]);
      dfs(nums, i + 1);
      records[i] = 0;
      path.pop_back();
    }
    ret.push_back(path);
  }

public:
  vector<vector<int>> subsetsWithDup(vector<int> &nums) {
    sort(nums.begin(), nums.end());
    records.resize(nums.size(), 0);
    dfs(nums, 0);
    return ret;
  }
};
```



#### [91. 解码方法](https://leetcode.cn/problems/decode-ways/)

一条包含字母 `A-Z` 的消息通过以下映射进行了 **编码** ：

`"1" -> 'A'   "2" -> 'B'   ...   "25" -> 'Y'   "26" -> 'Z'`

然而，在 **解码** 已编码的消息时，你意识到有许多不同的方式来解码，因为有些编码被包含在其它编码当中（`"2"` 和 `"5"` 与 `"25"`）。

例如，`"11106"` 可以映射为：

- `"AAJF"` ，将消息分组为 `(1, 1, 10, 6)`
- `"KJF"` ，将消息分组为 `(11, 10, 6)`
- 消息不能分组为  `(1, 11, 06)` ，因为 `"06"` 不是一个合法编码（只有 "6" 是合法的）。

注意，可能存在无法解码的字符串。

给你一个只含数字的 **非空** 字符串 `s` ，请计算并返回 **解码** 方法的 **总数** 。如果没有合法的方式解码整个字符串，返回 `0`。

题目数据保证答案肯定是一个 **32 位** 的整数。

题解——直观的想法是使用回溯法递归，考虑状态方程
$$
S(i) = S(arr[i];i+1) + S(arr[i], arr[i+1];i+2)
$$
因此可以双递归。但是注意到我们只需要计算解码总数，而不需要保存路径。因此对于相同的 `i`，其递归后的结果应该也相同，即
$$
S(i) = S(i+1) + S(i+2)
$$
与选择的元素无关，所以可以将结果保存起来。我们从后向前遍历，将从 `i` 开始后面所有的解码数保存起来，根据上式计算当前位置的解码数。

```cpp
class Solution {
public:
  int numDecodings(string s) {
    vector<int> table(s.size() + 1, 0);
    // 注意多一个元素作为初始值
    table[s.size()] = 1;
    int count = 0;
    for (int i = s.size() - 1; i >= 0; i--) {
      // 只选一个数的情况下，排除 0
      if (s[i] != '0')
        table[i] += table[i + 1];

      if (i < s.size() - 1) {
        if (s[i] == '1' || (s[i] == '2' && s[i + 1] <= '6')) {
          table[i] += table[i + 2];
        } else if (s[i + 1] == '0') {
          // 选两个数时，当后一个为 0 说明无效
          return 0;
        }
      }
    }
    return table[0];
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



#### [93. 复原 IP 地址](https://leetcode.cn/problems/restore-ip-addresses/)

**有效 IP 地址** 正好由四个整数（每个整数位于 `0` 到 `255` 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

- 例如：`"0.1.2.201"` 和 `"192.168.1.1"` 是 **有效** IP 地址，但是 `"0.011.255.245"`、`"192.168.1.312"` 和 `"192.168@1.1"` 是 **无效** IP 地址。

给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能的**有效 IP 地址**，这些地址可以通过在 `s` 中插入 `'.'` 来形成。你 **不能** 重新排序或删除 `s` 中的任何数字。你可以按 **任何** 顺序返回答案。

题解——回溯搜索，考虑状态方程
$$
S(i) = S(arr[i,i];i+1) \cup S(arr[i,i+1];i+2) \cup S(arr[i,i+2];i+3)
$$
对于 3 位数，限制其不超过 255；注意到不能有前导零，因此要求 2 位数不小于 10，3 位数不小于 100 。

```cpp
class Solution {
private:
  vector<string> ret;
  vector<int> path;

  void dfs(const string &s, int left, int count) {
    if (count == 0) {
      if (left < s.size())
        return;

      string s = "";
      for (auto &i : path)
        s += to_string(i) + ".";
      ret.push_back(s.substr(0, s.size() - 1));
      return;
    }

    if (left < s.size()) {
      auto num = stoi(s.substr(left, 1));
      path.push_back(num);
      dfs(s, left + 1, count - 1);
      path.pop_back();
    }

    if (left < s.size() - 1) {
      auto num = stoi(s.substr(left, 2));
      if (num >= 10) {
        path.push_back(num);
        dfs(s, left + 2, count - 1);
        path.pop_back();
      }
    }

    if (left < s.size() - 2) {
      auto num = stoi(s.substr(left, 3));
      if (num < 256 && num >= 100) {
        path.push_back(num);
        dfs(s, left + 3, count - 1);
        path.pop_back();
      }
    }
  }

public:
  vector<string> restoreIpAddresses(string s) {
    if (s.size() < 4)
      return ret;
    dfs(s, 0, 4);
    return ret;
  }
};
```



#### [95. 不同的二叉搜索树 II](https://leetcode.cn/problems/unique-binary-search-trees-ii/)

给你一个整数 `n` ，请你生成并返回所有由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的不同 **二叉搜索树** 。可以按 **任意顺序** 返回答案。

题解——考虑后面的状态方程，计算出 $S(1,i-1),S(i+1,n)$ 的所有可能的子树，然后双循环拼接到当前节点左右

```cpp
class Solution {
public:
  vector<TreeNode *> generateTrees(int start, int end) {
    if (start > end)
      return {nullptr};

    vector<TreeNode *> allTrees;
    for (int i = start; i <= end; i++) {
      // 获得所有可能的左右子树
      vector<TreeNode *> leftTrees = generateTrees(start, i - 1);
      vector<TreeNode *> rightTrees = generateTrees(i + 1, end);

      // 从左子树集合中选出一棵左子树，从右子树集合中选出一棵右子树，拼接到根节点上
      for (auto &left : leftTrees) {
        for (auto &right : rightTrees) {
          TreeNode *currTree = new TreeNode(i);
          currTree->left = left;
          currTree->right = right;
          allTrees.emplace_back(currTree);
        }
      }
    }
    return allTrees;
  }

  vector<TreeNode *> generateTrees(int n) {
    if (n == 0)
      return {};
    return generateTrees(1, n);
  }
};
```



#### [96. 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/)

给你一个整数 `n` ，求恰由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的 **二叉搜索树** 有多少种？返回满足题意的二叉搜索树的种数。

题解——考虑状态方程
$$
\begin{aligned}
S(1,n) &=\sum_{i=1}^{n} S(1,i-1)\times S(i+1,n)\\
&= \sum_{i=1}^{n} S(1,i-1)\times S(1,n - i) 
\end{aligned}
$$
我们只需要依次计算并保存 `S(1,j)` 即可，并令 $S(i,j)=1,j\leq i$ 表示空的情况

```cpp
class Solution {
public:
  int numTrees(int n) {
    vector<int> table(n + 2, 1);

	// S(1, 1) -> S(1, n)
    for (int i = 1; i < n + 1; i++) {
      int c = 0;
      // S(1, i) = sum S(1, j - 1) x S(1, i - j)
      for (int j = 1; j < i + 1; j++) {
        c += table[j - 1] * table[i - j];
      }
      table[i] = c;
    }
    return table[n];
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



#### [99. 恢复二叉搜索树](https://leetcode.cn/problems/recover-binary-search-tree/)

给你二叉搜索树的根节点 `root` ，该树中的 **恰好** 两个节点的值被错误地交换。_请在不改变其结构的情况下，恢复这棵树_ 。

题解——中序遍历会得到升序的数列，可以记录并检查相邻的元素，如果当前元素小于前一个元素，说明前一个元素的位置不对，标记为第一个元素。如果第一个元素已经标记，此时当前元素应该被标记为第二个元素。注意到两种情况

- 交换节点不相邻，此时按照前面的方式标记
- 交换节点相邻，此时应该同时标记前一个元素和当前元素

因此我们可以每次都将当前元素标记为第二个元素。

```cpp
class Solution {
private:
  vector<TreeNode *> record;

public:
  void recoverTree(TreeNode *root) {
    inorder(root);
    int first = -1, second = -1;
    for (int i = 1; i < record.size(); i++) {
      if (record[i - 1]->val > record[i]->val) {
        if (first == -1)
          first = i - 1;
        second = i;
      }
    }
    swap(record[first]->val, record[second]->val);
  }

  void inorder(TreeNode *node) {
    if (node == nullptr)
      return;
    inorder(node->left);
    record.push_back(node);
    inorder(node->right);
  }
};
```

注意到我们只需要比较相邻两个元素，因此不需要总是存放所有节点。可以用一个栈保存当前遍历路径上的节点，这样遍历时，栈顶元素就是前一个节点。

