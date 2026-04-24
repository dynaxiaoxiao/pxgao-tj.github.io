---
title: "力扣算法题小结"
date: 2026-04-24
#image: "cover.jpg"   # 封面图（可选）
math: true            # 如果有公式，记得开启
hidden: true
categories:
    - "数据结构算法"      # 这里填你想划分的板块名
tags:
    - "数据结构算法"
---

```cpp
#C++中定义长度不确定的二维数组
 vector<vector<int>> map(m, vector<int>(n, 0));
#重新赋值为全0
map.assign(m, vector<int>(n, 0));

#字符串转数值
int i = std::stoi("42");          
long l = std::stol("9999999999");
float f = std::stof("3.14");
double d = std::stod("2.71828");
#数值转字符串
std::string s1 = std::to_string(42);      
std::string s2 = std::to_string(3.14159); 
```

```cpp
DFS算法模板
class Solution {
public:
    int m,n;
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
        m=heights.size();
        n=heights[0].size();
        vector<vector<int>> map(m,vector<int>(n,0));
        vector<vector<int>> map2(m,vector<int>(n,0));
        vector<vector<int>> ans;
        for(int i=0;i<m;i++){
            dfs(map,heights,i,0,0);
        }
        for(int j=0;j<n;j++){
            dfs(map,heights,0,j,0);
        }
        for(int i=0;i<m;i++){
            dfs(map2,heights,i,n-1,0);
        }
        for(int j=0;j<n;j++){
            dfs(map2,heights,m-1,j,0);
        }
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(map[i][j]+map2[i][j]==2){
                    ans.push_back({i,j});
                }
            }
        }
        return ans;
    }
    void dfs(vector<vector<int>>& map,vector<vector<int>>& heights,int i,int j,int pre){
        if(i<0||i>=m||j<0||j>=n||map[i][j]==1)return;//越界
        int num =heights[i][j];
        if(pre<=num){
            map[i][j]=1;//可以
            dfs(map,heights,i+1,j,num);
            dfs(map,heights,i-1,j,num);
            dfs(map,heights,i,j+1,num);
            dfs(map,heights,i,j-1,num);
        }
        return;
    }
};


BFS算法模板
class Solution {
    static constexpr int DIRS[4][2] = {{0, -1}, {0, 1}, {-1, 0}, {1, 0}}; // 左右上下

public:
    int nearestExit(vector<vector<char>>& maze, vector<int>& entrance) {
        int m = maze.size(), n = maze[0].size();
        vector vis(m, vector<int8_t>(n));

        int sx = entrance[0], sy = entrance[1]; // 起点
        vis[sx][sy] = true; // 访问标记
        vector<pair<int, int>> q = {{sx, sy}};
        for (int ans = 1; !q.empty(); ans++) {
            auto tmp = q;
            q.clear();
            for (auto& [i, j] : tmp) {
                // 注意起点不算终点，不能在这里判断 p 是不是终点
                for (auto [dx, dy] : DIRS) { // 访问相邻的格子
                    int x = i + dx, y = j + dy;
                    if (0 <= x && x < m && 0 <= y && y < n && maze[x][y] == '.' && !vis[x][y]) { // 之前没有访问过
                        if (x == 0 || y == 0 || x == m - 1 || y == n - 1) { // 到达终点
                            return ans;
                        }
                        vis[x][y] = true; // 访问标记
                        q.emplace_back(x, y);
                    }
                }
            }
        }
        return -1; // 无法到达终点
    }
};









0-1 BFS算法模板
class Solution {
    static constexpr int DIRS[4][2]={{1,0},{-1,0},{0,-1},{0,1}};
public:
    int m,n;
    bool findSafeWalk(vector<vector<int>>& grid, int health) {
        m=grid.size();
        n=grid[0].size();
        vector<vector<int>> dis(m,vector<int>(n,INT_MAX));
        deque<pair<int,int>> qu;
        qu.emplace_front(0,0);
        dis[0][0]=grid[0][0];
        while(!qu.empty()){//队列非空，开始bfs
            auto tmp=qu.front();
            qu.pop_front();
            int x=tmp.first,y=tmp.second;//队首的横纵坐标
            for(auto dir:DIRS){
                int x1=x+dir[0],y1=y+dir[1];
                if(x1>=0&&x1<m&&y1>=0&&y1<n){
                    if(dis[x][y]+grid[x1][y1]<dis[x1][y1]){
                        dis[x1][y1] = dis[x][y]+grid[x1][y1];
                        grid[x1][y1] == 0? qu.emplace_front(x1,y1):qu.emplace_back(x1,y1);
                    }
                }
            }
        }
        return dis[m-1][n-1] < health;
    }
};



前缀和+哈希表
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int ans = 0;
        int n = nums.size();
        vector<int> s(n+1);
        for(int i = 0;i<n;i++){
            s[i+1]=s[i]+nums[i];
        }
        unordered_map<int,int> cnt;
        for(int sj:s){
            ans+= cnt.contains(sj-k)?cnt[sj-k]:0;
            cnt[sj]++;
        }
        return ans;
    }
};

路径总和（前缀和+哈希表一般情况）
class Solution {
public:
    int target;
    int ans;
    int pathSum(TreeNode* root, int targetSum) {
        unordered_map<long long,int> cnt={{0,1}};
        ans=0;
        target = targetSum;
        dfs(root,0,cnt);
        return ans;
    }
    void dfs(TreeNode* root, long long s, unordered_map<long long,int>& cnt){
        if(root == nullptr){
            return;
        }
        s+=root->val;
        ans+=cnt.contains(s-target)?cnt[s-target]:0;
        cnt[s]++;
        dfs(root->left,s,cnt);
        dfs(root->right,s,cnt);
        cnt[s]--;
        return;
    }
};

电话号码（回溯）
class Solution {
public:
    static constexpr string map[10]={"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    vector<string> ans;
    int n;
    vector<string> letterCombinations(string digits) {
        n = digits.size();
        string str = "";
        dfs(0,str,digits);
        return ans;
    }
    void dfs(int i,string str,string digits){
        if(i==n){
            ans.push_back(str);
            return;
        }
        char ch=digits[i];
        int num = ch - '0';
        for(char c : map[num]){
            string str1=str;
            str1 += c;
            dfs(i+1,str1,digits);
        }
        return;
    }
};
```

```cpp
递归搜索 + 保存计算结果 =记忆化搜索


Java这样似乎可以存储一个二元组
Queue<int[]> queue = new LinkedList<>();
        for(int i = 0; i < m; i++){
          for(int j = 0; j < n; j++){
            if(grid[i][j] == 2){ //烂的入队列
                queue.add(new int[]{i,j});
            }else if(grid[i][j] == 1){
                count++;
            }
          }
        }




二分查找常用写法


private int lowerBound(int[] nums, int target) {
        int left = 0, right = nums.length - 1; // 闭区间 [left, right]
        while (left <= right) { // 区间不为空
            // 循环不变量：
            // nums[left-1] < target
            // nums[right+1] >= target
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid + 1; // 范围缩小到 [mid+1, right]
            } else {
                right = mid - 1; // 范围缩小到 [left, mid-1]
            }
        }
        return left;
    }

    // 左闭右开区间写法
    private int lowerBound2(int[] nums, int target) {
        int left = 0, right = nums.length; // 左闭右开区间 [left, right)
        while (left < right) { // 区间不为空
            // 循环不变量：
            // nums[left-1] < target
            // nums[right] >= target
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid + 1; // 范围缩小到 [mid+1, right)
            } else {
                right = mid; // 范围缩小到 [left, mid)
            }
        }
        return left; // 返回 left 还是 right 都行，因为循环结束后 left == right
    }

    // 开区间写法
    private int lowerBound3(int[] nums, int target) {
        int left = -1, right = nums.length; // 开区间 (left, right)
        while (left + 1 < right) { // 区间不为空
            // 循环不变量：
            // nums[left] < target
            // nums[right] >= target
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid; // 范围缩小到 (mid, right)
            } else {
                right = mid; // 范围缩小到 (left, mid)
            }
        }
        return right;
    }








合理设计的哈希键能够有效地整合原始信息，找出对于解题有用的结果信息
常见哈希键设计：

在字符串和数组当中，当每个元素的顺序不重要时，可以使用排序后的字符串或数组作为键
如果只关心每个值的偏移量，例如第一个值的偏移量，则可以使用偏移量作为键
在树中，我们通常会用子树的序列化表述作为键




关于HashMap的一些运用
public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap();
        List<List<String>> ans;

        for(String str : strs){
            char[] arr = str.toCharArray();
            Arrays.sort(arr);
            String str0 = new String(arr);
            List<String> list = map.getOrDefault(str0, new ArrayList());
            list.add(str);
            map.put(str0, list);
        }
        ans = new ArrayList(map.values());
        return ans;
    }



最小路径问题可以考虑用二维数组dp



单调栈

单调栈是一种特殊的栈结构，它维护了栈内的元素单调性，即元素要么单调递增，要么单调递减。单调栈常用于解决与数组元素的顺序相关的问题，比如找到下一个或上一个更大的元素、计算最大矩形面积等。单调栈的核心思想是利用栈的后进先出（LIFO）特性，以及栈内元素的单调性，来高效地处理问题。



增强for循环似乎比for(i) 循环快/
(第一段代码使用的是增强型for循环（for-each loop），它直接遍历HashSet中的元素。这种循环方式简洁且通常性能较好，因为它直接利用了HashSet的快速查找特性。

第二段代码使用的是传统的for循环（for loop），并且以数组的索引来遍历HashSet。这种方式在性能上通常不如增强型for循环，因为它需要额外的步骤来从数组中获取元素，并且数组的长度可能比HashSet中的元素个数要多，因为HashSet会自动去除重复元素。)




前缀和
构建前缀和数组，以快速计算区间和；
注意在计算区间和的时候，下标有偏移。


单调队列套路
入（元素进入队尾，同时维护队列单调性）
出（元素离开队首）
记录/维护答案（根据队首）



最长递增子序列
class Solution {
    private int[] nums;
    private int[] memo;

    public int lengthOfLIS(int[] nums) {
        this.nums = nums;
        int n = nums.length;
        memo = new int[n]; // 本题可以初始化成 0，表示没有计算过
        int ans = 0;
        for (int i = 0; i < n; i++) {
            ans = Math.max(ans, dfs(i));
        }
        return ans;
    }

    private int dfs(int i) {
        if (memo[i] > 0) { // 之前计算过
            return memo[i];
        }
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                memo[i] = Math.max(memo[i], dfs(j));
            }
        }
        return ++memo[i]; // 加一提到循环外面
    }
}




以归并排序的方法，将两个链表合并排序

class Solution {
    public ListNode sortList(ListNode head) {
        // 如果链表为空或者只有一个节点，无需排序
        if (head == null || head.next == null) {
            return head;
        }
        // 找到中间节点，并断开 head2 与其前一个节点的连接
        // 比如 head=[4,2,1,3]，那么 middleNode 调用结束后 head=[4,2] head2=[1,3]
        ListNode head2 = middleNode(head);
        // 分治
        head = sortList(head);
        head2 = sortList(head2);
        // 合并
        return mergeTwoLists(head, head2);
    }

    // 876. 链表的中间结点（快慢指针）
    private ListNode middleNode(ListNode head) {
        ListNode pre = head;
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            pre = slow; // 记录 slow 的前一个节点
            slow = slow.next;
            fast = fast.next.next;
        }
        pre.next = null; // 断开 slow 的前一个节点和 slow 的连接
        return slow;
    }

    // 21. 合并两个有序链表（双指针）
    private ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(); // 用哨兵节点简化代码逻辑
        ListNode cur = dummy; // cur 指向新链表的末尾
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                cur.next = list1; // 把 list1 加到新链表中
                list1 = list1.next;
            } else { // 注：相等的情况加哪个节点都是可以的
                cur.next = list2; // 把 list2 加到新链表中
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 != null ? list1 : list2; // 拼接剩余链表
        return dummy.next;
    }
}




最大乘积子数组的解决

class Solution {
    public int maxProduct(int[] nums) {
        int[] fMax = new int[20001];
        int[] fMin = new int[20001];
        fMax[0] = fMin[0] = nums[0];
        for(int i = 1; i < nums.length; i++){
            fMax[i] = Math.max(fMax[i - 1]*nums[i], Math.max(fMin[i - 1]*nums[i], nums[i]));
            fMin[i] = Math.min(fMax[i - 1]*nums[i], Math.min(fMin[i - 1]*nums[i], nums[i]));
        }
        int ans = -11;
        for(int i = 0; i < nums.length; i++){
            ans = Math.max(ans, fMax[i]);
        }
        return ans;
    }
}
















线段树
线段树是一种二叉树，也就是对于一个线段，我们会用一个二叉树来表示。
 

void bui(int id,int l,int r)//创建线段树,id表示存储下标,区间[L,r]
{
  if(l == r)//左端点等于右端点，即为叶子节点(区间长度为1)，直接赋值即可
  {
    tr[id] = a[l];
    return ;
  }
// 否则将当前区间中间拆开成两个区间
  int mid = (l + r) / 2;//mid则为中间点，左儿子的结点区间为[l,mid],右儿子的结点区间为[mid + 1,r]
  bui(id * 2,l,mid); //递归构造左儿子结点
  bui(id * 2 + 1,mid + 1,r); //递归构造右儿子结点
// 左右两个区间计算完成以后
// 合并到当前区间
  tr[id] = max(tr[id * 2],tr[id * 2 + 1]);//更新父节点
}
```