# LeetCode

## 题解

### 中等

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
public:
  TreeNode *buildTree(vector<int> &inorder, vector<int> &postorder) {
    return buildTree(inorder, postorder, 0, 0, inorder.Size ());
  }

  TreeNode *buildTree(vector<int> &inorder, vector<int> &postorder, int ii,
                      int pi, int size) {
    if (size == 0)
      Return nullptr;

    treeNode *root = new TreeNode (postorder[pi + size - 1]);
    auto it =
        find(inorder.begin() + ii, inorder.begin() + ii + size, root->val);
    int leftSize = it - inorder.begin() - ii;
    int rightSize = size - leftSize - 1;
    root->left = buildTree(inorder, postorder, ii, pi, leftSize);		// 左子树，都不需要移动索引
    root->right = buildTree(inorder, postorder, ii + leftSize + 1,		// 右子树，中序遍历跳过左子树和 root，后序遍历跳过左子树
                            pi + leftSize, rightSize);
    return root;
  }
};
```



#### [107. 二叉树的层序遍历 II](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)

给你二叉树的根节点 `root` ，返回其节点值 **自底向上的层序遍历** 。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

题解——直接使用层序遍历的代码，最后反转数组。

```cpp
class Solution {
public:
  vector<vector<int>> levelOrderBottom(TreeNode *root) {
    if (root == nullptr)
      return {};
    vector<vector<int>> ret;
    queue<TreeNode *> boundary;
    boundary.push(root);
    while (!Boundary.Empty()) {
      int size = boundary.size();
      ret.push_back(vector<int>{});
      for (int i = 0; i < size; i++) {
        TreeNode *p = boundary.front();
        boundary.pop();
        if (p != nullptr) {
          boundary.push(p->left);
          boundary.push(p->right);
          ret.back().push_back(p->val);
        }
      }
    }
    ret.pop_back();
    reverse(ret.begin(), ret.end());
    return ret;
  }
};
```



#### [109. 有序链表转换二叉搜索树](https://leetcode.cn/problems/convert-sorted-list-to-binary-search-tree/)

给定一个单链表的头节点  `head` ，其中的元素 **按升序排序** ，将其转换为平衡二叉搜索树。



#### [113. 路径总和 II](https://leetcode.cn/problems/path-sum-ii/)

给你二叉树的根节点 `root` 和一个整数目标和 `targetSum` ，找出所有 **从根节点到叶子节点** 路径总和等于给定目标和的路径。

**叶子节点** 是指没有子节点的节点。

题解——深度优先搜索。注意叶节点会判断左右空节点，因此不能以 `target == 0` 作为返回条件，否则会重复推入两次。

```cpp
class Solution {
private:
  vector<vector<int>> ret;
  vector<int> path;

  void dfs(TreeNode *node, int target) {
    if (node == nullptr)
      return;
    if (target == node->val && node->left == nullptr &&
        node->right == nullptr) {
      path.push_back(node->val);
      ret.push_back(path);
      path.pop_back();
      return;
    }

    path.push_back(node->val);
    dfs (node->left, target - node->val);
    path.pop_back();

    path.push_back (node->val);
    dfs(node->right, target - node->val);
    path.pop_back ();
  }

public:
  vector<vector<int>> pathSum(TreeNode *root, int targetSum) {
    dfs(root, targetSum);
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



#### [116. 填充每个节点的下一个右侧节点指针](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)

给定一个 **完美二叉树** ，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：

```cpp
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```

填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 `NULL`。

初始状态下，所有 next 指针都被设置为 `NULL`。

题解——以已经连接好的一层最左侧的节点开始，连接子节点。由于是完全二叉树，因此只需要向左子节点迭代即可。

```cpp
class Solution {
public:
  Node *connect(Node *root) {
    Node *p = root;
    while (p != nullptr) {
      link(p);
      p = p->left;
    }
    return root;
  }

  void link(Node *first) {
    if (first->left == nullptr)
      return;

    Node *p = nullptr;
    while (first != nullptr) {
      if (p)
        p->next = first->left;
      first->left->next = first->right;
      p = first->right;
      first = first->next;
    }
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



#### [131. 分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**示例 1：**

**输入：** s = "aab"
**输出：** `[["a","a","b"],["aa","b"]]`

**示例 2：**

**输入：** s = "a"
**输出：** `["a"]("a") `

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



#### [1328. 破坏回文串](https://leetcode.cn/problems/break-a-palindrome/)

给你一个由小写英文字母组成的回文字符串 `palindrome` ，请你将其中 **一个** 字符用任意小写英文字母替换，使得结果字符串的 **字典序最小** ，且 **不是** 回文串。

请你返回结果字符串。如果无法做到，则返回一个 **空串** 。

如果两个字符串长度相同，那么字符串 `a` 字典序比字符串 `b` 小可以这样定义：在 `a` 和 `b` 出现不同的第一个位置上，字符串 `a` 中的字符严格小于 `b` 中的对应字符。例如，`"abcc”` 字典序比 `"abcd"` 小，因为不同的第一个位置是在第四个字符，显然 `'c'` 比 `'d'` 小。

题解——首先不考虑回文，只需要找到第一个不是 `a` 的元素替换为 `a`；如果全是 `a`，那么将最后一个元素加一；然后考虑回文：输入单独 `a` 的情况以及替换的恰好是中间的字符时需要排除。

```cpp
class Solution {
public:
  string breakPalindrome(string palindrome) {
    int n = palindrome.size();
    bool mid = n % 2;
    for (int i = 0; i < palindrome.size(); i++) {
      int cur = palindrome[i] - 'a';
      if (cur > 0 && (!mid || i != n / 2)) {
        palindrome[i] = 'a';
        return palindrome;
      }
    }
    if (palindrome.size() > 1)
      palindrome[palindrome.size() - 1]++;
    else
      palindrome = "";
    return palindrome;
  }
};
```

> >注意到输入是回文，因此只需要考虑前半部分，这样就不需要判断是否恰好是中间。



#### [2588. 统计美丽子数组数目](https://leetcode.cn/problems/count-the-number-of-beautiful-subarrays/)

给你一个下标从 **0** 开始的整数数组 `nums` 。每次操作中，你可以：

- 选择两个满足 `0 <= i, j < nums.length` 的不同下标 `i` 和 `j` 。
- 选择一个非负整数 `k` ，满足 `nums[i]` 和 `nums[j]` 在二进制下的第 `k` 位（下标编号从 **0** 开始）是 `1` 。
- 将 `nums[i]` 和 `nums[j]` 都减去 `2k` 。

如果一个子数组内执行上述操作若干次后，该子数组可以变成一个全为 `0` 的数组，那么我们称它是一个 **美丽** 的子数组。

请你返回数组 `nums` 中 **美丽子数组** 的数目。

子数组是一个数组中一段连续 **非空** 的元素序列。

题解——前缀和问题。容易看出只有当一个子数组中所有元素的异或为零时，这个子数组是美丽子数组。因此我们考虑 `i,j` 范围数组的异或
$$
\begin{aligned}
S(i,j) &= \bigoplus_{k=i\cdots j}arr[k]\\
S(i,j) &= S(i,k) \oplus S(k+1,j)
\end{aligned}
$$
因此 $S(k+1,j)$ 是美丽子数组当且仅当 $S(i,j)=S(i,k)$ 成立。取 `i=0`，我们只需要遍历整个数组，记录所有的 $S(0,k)$，后面出现重复的 $S(0,j)$，说明 $[k+1,j]$ 是美丽子数组，即有
$$
S(0,k) = S(0,j),\quad k=0,\cdots j -1\implies S(k+1,j)
$$
假设访问到 `j` 时出现多个重复值，例如 $S(0,k_{1})=S(0,k_{2})=S(0,j)$，说明
$$
[k_{1}+1,j],[k_{2}+1,j]
$$
都是美丽子数组，因此计数需要加上之前重复的次数。如果 $[0,j]$ 是第一个美丽子数组，那么需要之前重复次数为 1 才行。

```cpp
class Solution {
public:
  long long beautifulSubarrays(vector<int> &nums) {
    int xors = 0;
    long long count = 0;
    unordered_map<int, int> table;
    // 恰好是美丽子数组的情况，默认重复次数为 1
    table[0] = 1;
    for (auto &a : nums) {
      xors = xors ^ a;
      count += table[xors];		// 加上之前的重复次数
      table[xors]++;			// 重复次数增加
    }
    return count;
  }
};
```



#### [2597. 美丽子集的数目](https://leetcode.cn/problems/the-number-of-beautiful-subsets/)

给你一个由正整数组成的数组 `nums` 和一个 **正** 整数 `k` 。

如果 `nums` 的子集中，任意两个整数的绝对差均不等于 `k` ，则认为该子数组是一个 **美丽** 子集。

返回数组 `nums` 中 **非空** 且 **美丽** 的子集数目。

`nums` 的子集定义为：可以经由 `nums` 删除某些元素（也可能不删除）得到的一个数组。只有在删除元素时选择的索引不同的情况下，两个子集才会被视作是不同的子集。

题解——首先通过回溯枚举所有子集，考虑状态方程
$$
S(left) = S(left+1)\cup S(arr[left];left+1)
$$
其中只有当 `arr[left]` 与路径上所有数的差不等于 `k` 时才做第二步递归。使用一个二维数组记录两两数之间的差是否满足要求，路径只记录数的索引，便于查询数组中的记录。

```cpp
class Solution {
private:
  int count;
  vector<int> path;
  vector<vector<int>> D;

  void dfs(vector<int> &nums, int left, int k) {
    if (left == nums.size()) {
      count++;
      return;
    }

    dfs(nums, left + 1, k);
    for (int i = 0; i < path.size(); i++) {
      int j = path[i];
      if (D[j][left] == -1) {
        D[j][left] = abs(nums[j] - nums[left]) != k ? 1 : 0;
        D[left][j] = D[j][left];
      }
      if (D[j][left] == 0)
        return;
    }
    path.push_back(left);
    dfs(nums, left + 1, k);
    path.pop_back();
  }

public:
  int beautifulSubsets(vector<int> &nums, int k) {
    count = 0;
    D.assign(nums.size(), vector<int>(nums.size(), -1));
    dfs(nums, 0, k);
    return count - 1;
  }
};
```

