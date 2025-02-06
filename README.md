# codepratice-day15
代码随想录第15天

补
二叉树的递归和迭代遍历，搜索树，验证树。

[最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/description/)
给定一个不重复的整数数组 nums 。 最大二叉树 可以用下面的算法从 nums 递归地构建:

1.创建一个根节点，其值为 nums 中的最大值。
2.递归地在最大值 左边 的 子数组前缀上 构建左子树。
3.递归地在最大值 右边 的 子数组后缀上 构建右子树。
返回 nums 构建的 最大二叉树 。

构造树一般采用前序遍历，因为先构造中间节点，然后递归构造左子树和右子树。
1.确定递归函数的参数和返回值。
参数传入的是存放元素的数组，返回该数组构造的二叉树的头节点，返回类型是指向节点的指针。

2.确定终止条件
题目说了输入数组大小大于等于1.

递归遍历的时候，如果传入的数组大小为1，说明遍历到了叶子节点。
那么应该定义一个新节点，把这个数组的数值赋给新的节点，然后返回这个节点。
```CPP
TreeNode* node=new TreeNode(0);
if(nums.size()==1)
{
    node->val=nums[0];
    return node;
}
```

3.确定单层递归的逻辑
这里有三步
(1).先找到数组中最大的值和对应的下标，最大的值构造根节点，下标用来下一步分割数组。
(2).最大值所在的下标左区间构造左子树。
(3).最大值所在的下标右区间构造右子树。

整体代码（先只学这种好理解的代码）：
```CPP
class Solution {
public:
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        TreeNode* node = new TreeNode(0);
        if (nums.size() == 1) {
            node->val = nums[0];
            return node;
        }
        // 找到数组中最大的值和对应的下标
        int maxValue = 0;
        int maxValueIndex = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] > maxValue) {
                maxValue = nums[i];
                maxValueIndex = i;
            }
        }
        node->val = maxValue;
        // 最大值所在的下标左区间 构造左子树
        if (maxValueIndex > 0) {
            vector<int> newVec(nums.begin(), nums.begin() + maxValueIndex);
            node->left = constructMaximumBinaryTree(newVec);
        }
        // 最大值所在的下标右区间 构造右子树
        if (maxValueIndex < (nums.size() - 1)) {
            vector<int> newVec(nums.begin() + maxValueIndex + 1, nums.end());
            node->right = constructMaximumBinaryTree(newVec);
        }
        return node;
    }
};
```

优化代码，不用定义新的数组，而是通过下标索引直接在原数组上操作。
简介的原因是允许空节点进入递归，所以不用在递归的时候加判断节点是否为空。
```CPP
class Solution {
private:
    // 在左闭右开区间[left, right)，构造二叉树
    TreeNode* traversal(vector<int>& nums, int left, int right) {
        if (left >= right) return nullptr;

        // 分割点下标：maxValueIndex
        int maxValueIndex = left;
        for (int i = left + 1; i < right; ++i) {
            if (nums[i] > nums[maxValueIndex]) maxValueIndex = i;
        }

        TreeNode* root = new TreeNode(nums[maxValueIndex]);

        // 左闭右开：[left, maxValueIndex)
        root->left = traversal(nums, left, maxValueIndex);

        // 左闭右开：[maxValueIndex + 1, right)
        root->right = traversal(nums, maxValueIndex + 1, right);

        return root;
    }
public:
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        return traversal(nums, 0, nums.size());
    }
};
```
什么时候递归函数前面加if，什么时候不加if：
一般情况来说：如果让空节点（空指针）进入递归，就不加if，如果不让空节点进入递归，就加if限制一下， 终止条件也会相应的调整。

[合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/)
给你两棵二叉树： root1 和 root2 。
想象一下，当你将其中一棵覆盖到另一棵之上时，两棵树上的一些节点将会重叠（而另一些不会）。你需要将这两棵树合并成一棵新二叉树。合并的规则是：如果两个节点重叠，那么将这两个节点的值相加作为合并后节点的新值；否则，不为 null 的节点将直接作为新二叉树的节点。
返回合并后的二叉树。

如何同时遍历两个二叉树。 传入两个树的节点，同时操作即可。

其实想一想，那种遍历方式都可以。

递归3部曲
1.确定递归函数的参数和返回值
参数两个二叉树的根节点，返回值就是合并之后二叉树的节点。

2.确定终止条件
如果root1是空，合并就是root2,反之一样。

3.确定单层递归的逻辑
重复利用树root1，t1就是合并之后树的根节点（就是修改了原来树的结构）
单层递归中，把两棵树的元素加到一起
```CPP
//合并两棵树的元素
root1->val+=root2->val;
```
接下来，root1的左子树是，合并 root1左子树 和 root2左子树 之后的左子树
```CPP
root1->left=mergeTrees(root1->left,root2->left);
root1->right=mergeTrees(root1->left,root2->right);
return root1;
```

完整代码：
```CPP
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) return t2; // 如果t1为空，合并之后就应该是t2
        if (t2 == NULL) return t1; // 如果t2为空，合并之后就应该是t1
        // 修改了t1的数值和结构
        t1->val += t2->val;                             // 中
        t1->left = mergeTrees(t1->left, t2->left);      // 左
        t1->right = mergeTrees(t1->right, t2->right);   // 右
        return t1;
    }
};
```

中序遍历和后序遍历：
```CPP
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) return t2; // 如果t1为空，合并之后就应该是t2
        if (t2 == NULL) return t1; // 如果t2为空，合并之后就应该是t1
        // 修改了t1的数值和结构
        t1->left = mergeTrees(t1->left, t2->left);      // 左
        t1->val += t2->val;                             // 中
        t1->right = mergeTrees(t1->right, t2->right);   // 右
        return t1;
    }
};
```
```CPP
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) return t2; // 如果t1为空，合并之后就应该是t2
        if (t2 == NULL) return t1; // 如果t2为空，合并之后就应该是t1
        // 修改了t1的数值和结构
        t1->left = mergeTrees(t1->left, t2->left);      // 左
        t1->right = mergeTrees(t1->right, t2->right);   // 右
        t1->val += t2->val;                             // 中
        return t1;
    }
};
```

也可以不修改树的结果
```CPP
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) return t2;
        if (t2 == NULL) return t1;
        // 重新定义新的节点，不修改原有两个树的结构
        TreeNode* root = new TreeNode(0);
        root->val = t1->val + t2->val;
        root->left = mergeTrees(t1->left, t2->left);
        root->right = mergeTrees(t1->right, t2->right);
        return root;
    }
};
```

迭代法
同时处理两颗二叉树，在迭代法中，[对称二叉树](https://leetcode.cn/problems/symmetric-tree/description/)这道题也有,
求二叉树对称的时候就是把两个树的节点同时加入队列进行比较。
```CPP
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) return t2;
        if (t2 == NULL) return t1;
        queue<TreeNode*> que;
        que.push(t1);
        que.push(t2);
        while(!que.empty()) {
            TreeNode* node1 = que.front(); que.pop();
            TreeNode* node2 = que.front(); que.pop();
            // 此时两个节点一定不为空，val相加
            node1->val += node2->val;

            // 如果两棵树左节点都不为空，加入队列
            if (node1->left != NULL && node2->left != NULL) {
                que.push(node1->left);
                que.push(node2->left);
            }
            // 如果两棵树右节点都不为空，加入队列
            if (node1->right != NULL && node2->right != NULL) {
                que.push(node1->right);
                que.push(node2->right);
            }

            // 当t1的左节点 为空 t2左节点不为空，就赋值过去
            if (node1->left == NULL && node2->left != NULL) {
                node1->left = node2->left;
            }
            // 当t1的右节点 为空 t2右节点不为空，就赋值过去
            if (node1->right == NULL && node2->right != NULL) {
                node1->right = node2->right;
            }
        }
        return t1;
    }
};
```

[二叉搜索树中的搜索](https://leetcode.cn/problems/search-in-a-binary-search-tree/)
给定二叉搜索树（BST）的根节点 root 和一个整数值 val。

你需要在 BST 中找到节点值等于 val 的节点。 返回以该节点为根的子树。 如果节点不存在，则返回 null 

二叉搜索树是一个有序树：

1.若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2.若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
3.它的左、右子树也分别为二叉搜索树

递归法
1.确定递归函数的参数和返回值
参数传入的就是根节点和要搜索的数值，返回以这个搜索数值所在的节点。

2.确定终止条件
如果root为空，或者找到这个数值，就返回root节点

3.确定单层递归的逻辑
二叉搜索树的节点是有序的，所以可以有方向的去搜索

root->val > val 搜索左子树， root->val < val 搜索右子树，如果都没有搜索到，返回nullptr

整体代码
```CPP
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if (root == NULL || root->val == val) return root;
        TreeNode* result = NULL;
        if (root->val > val) result = searchBST(root->left, val);
        if (root->val < val) result = searchBST(root->right, val);
        return result;
    }
};
```
或
```CPP
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if (root == NULL || root->val == val) return root;
        if (root->val > val) return searchBST(root->left, val);
        if (root->val < val) return searchBST(root->right, val);
        return NULL;
    }
};
```

迭代法
对于二叉搜索树，不需要回溯的过程，因为节点的有序性就帮我们确定了搜索的方向。 利用二叉搜索树有序性
```CPP
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        while (root != NULL) {
            if (root->val > val) root = root->left;
            else if (root->val < val) root = root->right;
            else return root;
        }
        return NULL;
    }
};
```

[验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)
给你一个二叉树的根节点 root ，判断其是否是一个有效的二叉搜索树。

有效 二叉搜索树定义如下：
节点的左子树只包含 小于 当前节点的数。
节点的右子树只包含 大于 当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。


中序遍历下，输出的二叉搜索树节点的数值是有序序列
这样，验证二叉搜索树，转化为判断一个序列是不是递增的 问题了。

可以递归中序遍历将二叉搜索树转化成一个数组
```CPP
class Solution {
private:
    vector<int> vec;
    void traversal(TreeNode* root) {
        if (root == NULL) return;
        traversal(root->left);
        vec.push_back(root->val); // 将二叉搜索树转换为有序数组
        traversal(root->right);
    }
public:
    bool isValidBST(TreeNode* root) {
        vec.clear(); // 不加这句在leetcode上也可以过，但最好加上
        traversal(root);
        for (int i = 1; i < vec.size(); i++) {
            // 注意要小于等于，搜索树里不能有相同元素
            if (vec[i] <= vec[i - 1]) return false;
        }
        return true;
    }
};
```

这道题的问题是
不能单纯的比较左节点小于中间节点，右节点大于中间节点就完事了。
我们要比较的是 左子树所有节点小于中间节点，右子树所有节点大于中间节点。

样例中最小节点 可能是int的最小值，如果这样使用最小的int来比较也是不行的。

递归三部曲
1.确定递归函数，返回值以及参数
定义一个long long的全局遍历，比较遍历的节点是否有序。
只有寻找某一条边（或者一个节点）的时候，递归函数会右bool类型的返回值。

2.确定终止条件
二叉搜索树可以为空

3.确定单层递归的逻辑
中序遍历，一直更新maxVal，一旦发现maxVal >= root->val，就返回false，注意元素相同时候也要返回false。

```CPP
class Solution {
public:
    long long maxVal = LONG_MIN; // 因为后台测试数据中有int最小值
    bool isValidBST(TreeNode* root) {
        if (root == NULL) return true;

        bool left = isValidBST(root->left);
        // 中序遍历，验证遍历的元素是不是从小到大
        if (maxVal < root->val) maxVal = root->val;
        else return false;
        bool right = isValidBST(root->right);

        return left && right;
    }
};
```
如果节点有比这个更小的值，那么不用初始化最小值，而是取最左边节点的数值比较的方式。
```CPP
class Solution {
public:
    TreeNode* pre = NULL; // 用来记录前一个节点
    bool isValidBST(TreeNode* root) {
        if (root == NULL) return true;
        bool left = isValidBST(root->left);

        if (pre != NULL && pre->val >= root->val) return false;
        pre = root; // 记录前一个节点

        bool right = isValidBST(root->right);
        return left && right;
    }
};
```

迭代法
```CPP
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        stack<TreeNode*> st;
        TreeNode* cur = root;
        TreeNode* pre = NULL; // 记录前一个节点
        while (cur != NULL || !st.empty()) {
            if (cur != NULL) {
                st.push(cur);
                cur = cur->left;                // 左
            } else {
                cur = st.top();                 // 中
                st.pop();
                if (pre != NULL && cur->val <= pre->val)
                return false;
                pre = cur; //保存前一个访问的结点

                cur = cur->right;               // 右
            }
        }
        return true;
    }
};
```