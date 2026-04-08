# LeetCode

## 题库

### 中等

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



#### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

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



#### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

给你一个字符串 `s`，找到 `s` 中最长的 回文 子串。

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



#### [7. 整数反转](https://leetcode.cn/problems/reverse-integer/)

给你一个 32 位的有符号整数 `x` ，返回将 `x` 中的数字部分反转后的结果。

如果反转后整数超过 32 位的有符号整数的范围 `[−231,  231 − 1]` ，就返回 0。

**假设环境不允许存储 64 位整数（有符号或无符号）。**

**示例 1：**

**输入：** x = 123
**输出：** 321

**示例 2：**

**输入：** x = -123
**输出：**-321

**示例 3：**

**输入：** x = 120
**输出：** 21

**示例 4：**

**输入：** x = 0
**输出：** 0

**提示：**

- `-231 <= x <= 231 - 1`

题解

```cpp
class Solution {
public:
  int reverse(int x) {
    if (x == INT_MAX || x + 1 == -INT_MAX)
      return 0;

    int sign = x > 0 ? 1 : -1;
    x *= sign;

    int y = 0;
    int top = INT_MAX;
    while (x > 0) {
      int r = x % 10;
      x = (x - r) / 10;
      if ((top - r) < y * 10)
        return 0;
      y = y * 10 + r;
    }

    return y * sign;
  }
};
```



#### [8. 字符串转换整数 (atoi)](https://leetcode.cn/problems/string-to-integer-atoi/)

请你来实现一个 `myAtoi(string s)` 函数，使其能将字符串转换成一个 32 位有符号整数。

函数 `myAtoi(string s)` 的算法如下：

1. **空格：** 读入字符串并丢弃无用的前导空格（`" "`）
2. **符号：** 检查下一个字符（假设还未到字符末尾）为 `'-'` 还是 `'+'`。如果两者都不存在，则假定结果为正。
3. **转换：** 通过跳过前置零来读取该整数，直到遇到非数字字符或到达字符串的结尾。如果没有读取数字，则结果为0。
4. **舍入：** 如果整数数超过 32 位有符号整数范围 `[−231,  231 − 1]` ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 `−231` 的整数应该被舍入为 `−231` ，大于 `231 − 1` 的整数应该被舍入为 `231 − 1` 。

返回整数作为最终结果。

题解

```cpp
class Solution {
public:
  int myAtoi(string s) {
    int i = 0;
    while (s[i] == ' ') {
      i++;
      if (i >= s.size())
        return 0;
    }

    int sign = s[i] == '-' ? -1 : 1;
    if (s[i] == '+' || s[i] == '-')
      i++;

    int y = 0;
    int top = INT_MAX;
    while (i < s.size()) {
      if (s[i] < '0' || s[i] > '9')
        break;

      if (s[i] == '0' && y == 0) {
        i++;
        continue;
      }

      int r = s[i] - '0';
      if (sign == 1 && (top - r) / 10 < y)
        return INT_MAX;
      if (sign == -1) {
        // 针对 -INT_MAX - 1 的情况
        if ((-INT_MAX + 7) / 10 >= -y && r >= 8)
          return -INT_MAX - 1;
        // 一般情况
        if ((-INT_MAX - 1 + r) / 10 > -y)
          return -INT_MAX - 1;
      }

      y = y * 10 + r;
      i++;
    }

    return y * sign;
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
- 只有 10 的次方（`I`, `X`, `C`, `M`）最多可以连续附加 3 次以代表 10 的倍数。你不能多次附加 5 (`V`)，50 (`L`) 或 500 (`D`)。如果需要将符号附加4次，请使用 **减法形式**。

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



#### [16. 最接近的三数之和](https://leetcode.cn/problems/3sum-closest/) 

给你一个长度为 `n` 的整数数组 `nums` 和 一个目标值 `target`。请你从 `nums` 中选出三个整数，使它们的和与 `target` 最接近。

返回这三个数的和。

假定每组输入只存在恰好一个解。

题解——与三数之和类似

```cpp
class Solution {
public:
  int threeSumClosest(vector<int> &nums, int target) {
    sort(nums.begin(), nums.end());

    auto fs = [&nums](int i, int j, int k) {
      return nums[i] + nums[j] + nums[k];
    };
    auto fr = [&nums, target](int i, int j, int k) {
      return abs(nums[i] + nums[j] + nums[k] - target);
    };

    int i = 0, j = 1, k = 2;
    int minr = fr(i, j, k);
    int nearest = fs(i, j, k);
    while (i < nums.size()) {
      j = i + 1, k = nums.size() - 1;
      while (j < k) {
        int r = fr(i, j, k);
        int s = fs(i, j, k);
        if (r < minr) {
          nearest = s;
          minr = r;
        }

        if (s == target)
          return target;
        else if (s < target)
          j++;
        else
          k--;
      }
      i++;
    }

    return nearest;
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



#### [18. 四数之和](https://leetcode.cn/problems/4sum/)

给你一个由 `n` 个整数组成的数组 `nums` ，和一个目标值 `target` 。请你找出并返回满足下述全部条件且**不重复**的四元组 `[nums[a], nums[b], nums[c], nums[d]]` （若两个四元组元素一一对应，则认为两个四元组重复）：

- `0 <= a, b, c, d < n`
- `a`、`b`、`c` 和 `d` **互不相同**
- `nums[a] + nums[b] + nums[c] + nums[d] == target`

你可以按 **任意顺序** 返回答案。

题解——类似于三数之和，可以优化剪枝（注意考虑整型溢出）

```cpp
class Solution {
public:
  vector<vector<int>> fourSum(vector<int> &nums, int target) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> ret;

    int i = 0;
    while (i + 3 < nums.size()) {
      int ri = nums[i];
      int j = i + 1;
      while (j + 2 < nums.size()) {
        int rj = nums[j];
        int k = j + 1, l = nums.size() - 1;
        while (k < l) {
          int rk = nums[k];
          int rl = nums[l];
          long sum = long(ri) + rj + rk + rl;
          if (sum == target) {
            ret.push_back({ri, rj, rk, rl});
          }
          if (sum >= target) {
            while (k < --l && rl == nums[l])
              ;
          }
          if (sum <= target) {
            while (++k < l && rk == nums[k])
              ;
          }
        }
        while (++j < nums.size() && rj == nums[j])
          ;
      }
      while (++i < nums.size() && ri == nums[i])
        ;
    }

    return ret;
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



#### [29. 两数相除](https://leetcode.cn/problems/divide-two-integers/)

给你两个整数，被除数 `dividend` 和除数 `divisor`。将两数相除，要求 **不使用** 乘法、除法和取余运算。

整数除法应该向零截断，也就是截去（`truncate`）其小数部分。例如，`8.345` 将被截断为 `8` ，`-2.7335` 将被截断至 `-2` 。

返回被除数 `dividend` 除以除数 `divisor` 得到的 **商** 。

**注意：** 假设我们的环境只能存储 **32 位** 有符号整数，其数值范围是 `[−231,  231 − 1]` 。本题中，如果商 **严格大于** `231 − 1` ，则返回 `231 − 1` ；如果商 **严格小于** `-231` ，则返回 `-231` 。

题解

```cpp
class Solution {
public:
  int divide(int dividend, int divisor) {
    if (dividend == -INT_MAX - 1 && divisor == 1)
      return -INT_MAX - 1;
    if (dividend <= -INT_MAX && divisor == -1)
      return INT_MAX;

    // 转为负数除法，不容易溢出
    if (dividend > 0)
      return -divide(-dividend, divisor);
    if (divisor > 0)
      return -divide(dividend, -divisor);

    int q = 0;
    while (dividend < 0) {
      dividend -= divisor;
      q++;
    }

	// 这一步单独放出来，防止多减一个 divisor 导致溢出
    if (dividend == 0)
      return q;
    else
      return q - 1;
  }
};
```



#### [31. 下一个排列](https://leetcode.cn/problems/next-permutation/)

整数数组的一个 **排列**  就是将其所有成员以序列或线性顺序排列。

- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。

整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。

给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。

必须 **[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)** 修改，只允许使用额外常数空间。

题解——从后向前检查，如果升序，则不存在下一个排列；直到遇见第一个降序元素，将其与后面第一个大于该元素的值交换，然后排序。

```cpp
class Solution {
public:
  void nextPermutation(vector<int> &nums) {
    bool up = true;
    int first = nums.size() - 1;
    for (int i = first - 1; i >= 0 && up; i--) {
      up = nums[i] >= nums[i + 1];
      first--;
    }

    for (int i = nums.size() - 1; i > first; i--) {
      if (nums[i] > nums[first]) {
        swap(nums[i], nums[first]);
        break;
      }
    }

    sort(nums.begin() + first + 1 - up, nums.end());
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



#### [36. 有效的数独](https://leetcode.cn/problems/valid-sudoku/)

请你判断一个 `9 x 9` 的数独是否有效。只需要 **根据以下规则** ，验证已经填入的数字是否有效即可。

1. 数字 `1-9` 在每一行只能出现一次。
2. 数字 `1-9` 在每一列只能出现一次。
3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次。（请参考示例图）

**注意：**

- 一个有效的数独（部分已被填充）不一定是可解的。
- 只需要根据以上规则，验证已经填入的数字是否有效即可。
- 空白格用 `'.'` 表示。

题解——开二维数组计数。

```cpp
class Solution {
public:
  bool isValidSudoku(vector<vector<char>> &board) {
    int rows[9][9]{};
    int cols[9][9]{};
    int square[9][9]{};

    for (int i = 0; i < 9; i++) {
      for (int j = 0; j < 9; j++) {
        int c = board[i][j];
        if (c == '.')
          continue;
        int a = c - '1';
        int inds = (i / 3) * 3 + (j / 3);
        if (rows[i][a] == 1 || cols[j][a] == 1 || square[inds][a] == 1)
          return false;
        square[inds][a]++;
        rows[i][a]++;
        cols[j][a]++;
      }
    }
    return true;
  }
};
```



#### [38. 外观数列](https://leetcode.cn/problems/count-and-say/)

「外观数列」是一个数位字符串序列，由递归公式定义：

- `countAndSay(1) = "1"`
- `countAndSay(n)` 是 `countAndSay(n-1)` 的行程长度编码。

[行程长度编码](https://baike.baidu.com/item/%E8%A1%8C%E7%A8%8B%E9%95%BF%E5%BA%A6%E7%BC%96%E7%A0%81/2931940)（RLE）是一种字符串压缩方法，其工作原理是通过将连续相同字符（重复两次或更多次）替换为字符重复次数（运行长度）和字符的串联。例如，要压缩字符串 `"3322251"` ，我们将 `"33"` 用 `"23"` 替换，将 `"222"` 用 `"32"` 替换，将 `"5"` 用 `"15"` 替换并将 `"1"` 用 `"11"` 替换。因此压缩后字符串变为 `"23321511"`。

给定一个整数 `n` ，返回 **外观数列** 的第 `n` 个元素。

题解——直接递归计算。

```cpp
class Solution {
public:
  string countAndSay(int n) {
    if (n == 1)
      return "1";
    return encode(countAndSay(n - 1));
  }

  string encode(const string &s) {
    char c = s[0];
    string ret = "";
    int count = 1;
    for (int i = 1; i < s.size(); i++) {
      if (s[i] == c)
        count++;
      else {
        ret += '0' + count;
        ret += c;
        c = s[i];
        count = 1;
      }
    }
    ret += '0' + count;
    ret += c;
    return ret;
  }
};
```



#### [39. 组合总和](https://leetcode.cn/problems/combination-sum/)

给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的 所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。

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

> >这里第一条路径可以展开为循环。



#### [40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)

给定一个候选人编号的集合 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的每个数字在每个组合中只能使用 **一次** 。

**注意：** 解集不能包含重复的组合。

题解——回溯深度优先搜索，每个值只搜索一次，因此只需要一个递归。组合方程
$$
S(arr,tar,idx) = S(arr, tar, idx+1)\cup (S(arr,tar-arr[idx],idx+1)+arr[idx])
$$
将右边第一项展开可以得到循环。

```cpp
class Solution {
public:
  vector<vector<int>> res;
  vector<int> temp;

  void backtrack(vector<int> &candidates, int target, int index) {
    if (target == 0) {
      res.push_back(temp);
      return;
    }
    for (int i = index; i < candidates.size() && target - candidates[i] >= 0;
         i++) {
      if (i > index && candidates[i] == candidates[i - 1])
        continue;
      temp.push_back(candidates[i]);
      backtrack(candidates, target - candidates[i], i + 1);
      temp.pop_back();
    }
  }

  vector<vector<int>> combinationSum2(vector<int> &candidates, int target) {
    sort(candidates.begin(), candidates.end());
    backtrack(candidates, target, 0);
    return res;
  }
};
```



#### [43. 字符串相乘](https://leetcode.cn/problems/multiply-strings/)

给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。

**注意：** 不能使用任何内置的 BigInteger 库或直接将输入转换为整数。

题解——使用数组模拟竖式。

```cpp
class Solution {
public:
  string multiply(string num1, string num2) {
    if (num1 == "0" || num2 == "0")
      return "0";

    int m1 = num1.size(), m2 = num2.size();
    vector<int> ans(m1 + m2, 0);
    for (int i = m1 - 1; i >= 0; i--) {
      int c1 = num1[i] - '0';
      for (int j = m2 - 1; j >= 0; j--) {
        int c2 = num2[j] - '0';
        int r = c1 * c2;
        int idx = ans.size() - 2 - i - j;
        ans[idx] += r % 10;
        ans[idx + 1] += r / 10;
      }
    }

    for (int i = 0; i < ans.size() - 1; i++) {
      int r = ans[i];
      ans[i] %= 10;
      ans[i + 1] += (r - ans[i]) / 10;
    }

    string ret = "";
    for (int i = ans.size() - 1; i >= 0; i--) {
      if (ans[i] == 0 && ret == "")
        continue;
      ret += '0' + ans[i];
    }
    return ret;
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



#### [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/)

给定一个可包含重复数字的序列 `nums` ，_**按任意顺序**_ 返回所有不重复的全排列。

题解——首先排序，这样能够确保相同元素是相邻的。用一个数组记录每个元素是否已经被选取，反复遍历整个数组，这样能够得到所有排列。然后考虑排除重复情况

```cpp
i > 0 && nums[i] == nums[i - 1] && vis[i - 1]
```

我们要求当元素重复时，只能选取第一个元素，因此要检查前一个元素是否被选取。

```cpp
class Solution {
  vector<int> vis;

public:
  void backtrack(vector<int> &nums, vector<vector<int>> &ans, int idx,
                 vector<int> &perm) {
    if (idx == nums.size()) {
      ans.emplace_back(perm);
      return;
    }
    for (int i = 0; i < nums.size(); ++i) {
      if (vis[i] || (i > 0 && nums[i] == nums[i - 1] && vis[i - 1])) {
        continue;
      }
      perm.emplace_back(nums[i]);
      vis[i] = 1;
      backtrack(nums, ans, idx + 1, perm);
      vis[i] = 0;
      perm.pop_back();
    }
  }

  vector<vector<int>> permuteUnique(vector<int> &nums) {
    vector<vector<int>> ans;
    vector<int> perm;
    vis.resize(nums.size());
    sort(nums.begin(), nums.end());
    backtrack(nums, ans, 0, perm);
    return ans;
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



#### [49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

题解——对每个单词排序作为键保存即可

> >还可以记录每个字母出现的次数。

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
