# 一、链表

## [141.环形链表(Easy)](https://leetcode-cn.com/problems/linked-list-cycle/)

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if(head==nullptr || head->next==nullptr) return false;
        ListNode* slow = head;
        ListNode* fast = head;
        while(fast!=nullptr&&fast->next!=nullptr)
        {
            slow=slow->next;
            fast=fast->next->next;
            if(slow==fast)  return true;
        }
        return false;
    }
};
```

## [142.环形链表II(Medium)](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(head==nullptr||head->next==nullptr) return nullptr;
        ListNode* slow=head;
        ListNode* fast=head;
        while(fast!=nullptr&&fast->next!=nullptr)
        {
            slow=slow->next;
            fast=fast->next->next;
            if(slow==fast)
            {
                fast=head;
                while(slow!=fast)
                {
                    slow=slow->next;
                    fast=fast->next;
                }
                return fast;
            }
        }
        return nullptr;
    }
};
```



## [206.反转链表(Easy)](https://leetcode-cn.com/problems/reverse-linked-list/)

​		[幻灯片演示](https://leetcode-cn.com/problems/reverse-linked-list/solution/dong-hua-yan-shi-206-fan-zhuan-lian-biao-by-user74/)

### （1）递归

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head==nullptr||head->next==nullptr)
            return head;
        //这里的node就是最后一个节点
        ListNode* node = reverseList(head->next);
        head->next->next=head;
        head->next=nullptr;
        return node;
    }
};
```

### （2）迭代

```java
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* cur=head;
        ListNode* pre=nullptr;
        while(cur!=nullptr)
        {
            ListNode* temp=cur->next;
            cur->next=pre;
            pre=cur;
            cur=temp;
        }
        return pre;
    }
};
```

## [92.反转链表II(Medium)](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

```C++
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        ListNode* dummy=new ListNode(0);
        dummy->next=head;n
        ListNode* pre=dummy;//p指针指向第一个要反转的节点的前面的节点
        ListNode* cur=dummy->next;//q指针指向第一个要反转的节点
        for(int i=0;i<left-1;i++)
        {
            pre=pre->next;
            cur=cur->next;
        }
        //头插法
        for(int i=0;i<right-left;i++)
        {
            ListNode* removed=cur->next;//将q后面的元素删除，然后添加到p的后面。也即头插法。
            cur->next=cur->next->next;

            removed->next=pre->next;
            pre->next=removed;
        }
        return dummy->next;
    }
};
```

## [剑指 Offer 06. 从尾到头打印链表(Easy)](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

### （1）递归

```c++
class Solution {
public:
    vector<int> res;
    vector<int> reversePrint(ListNode* head) {
        recur(head);
        return res;
    }
    void recur(ListNode* head){
        if(head==nullptr) return ;
        recur(head->next);
        res.push_back(head->val);
    }
};
```

### （2）辅助栈

```c++
class Solution {
public:

    vector<int> reversePrint(ListNode* head) {
        vector<int> res;
        stack<int> st;
        while(head!=nullptr)
        {
            st.push(head->val);
            head=head->next;
        }
        while(!st.empty())
        {
            res.push_back(st.top());
            st.pop();
        }
        return res;
    }
};
```

## [剑指 Offer 35. 复杂链表的复制(Medium)](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

```c++
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if(head==nullptr) return head;
        Node* cur=head;
        unordered_map <Node*,Node*> hash;
        while(cur)
        {
            hash[cur]=new Node(cur->val);
            cur=cur->next;
        }
        cur=head;
        while(cur)
        {
            hash[cur]->next=hash[cur->next];
            hash[cur]->random=hash[cur->random];
            cur=cur->next;
        }
        return hash[head];
    }
};
```

# 二、字符串

## [剑指 Offer 05. 替换空格(Easy)](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

```c++
class Solution {
public:
    string replaceSpace(string s) {
        int cnt=0;
        int size = s.size();
        for(int i=0;i<size;i++)
        {
            if(s[i]==' ') cnt++;
        }
        s.resize(size+2*cnt);
        for(int i=size-1,j=s.size()-1;i<j;i--,j--)
        {
            if(s[i]!=' ') s[j]=s[i];
            if(s[i]==' ')
            {
                s[j]='0';
                s[j-1]='2';
                s[j-2]='%';
                j-=2;
            }
        }
        return s;
    }
};
```

## [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

```c++
class Solution {
public:
    bool ScanInt(const string s,int& index)
    {
        if(s[index]=='+'||s[index]=='-')
        {
            index++;
        }
        return ScanUint(s,index);
    }
    bool ScanUint(const string s,int& index)
    {
        int begin=index;
        while(index!=s.size()&&s[index]>='0'&&s[index]<='9')
        {
            ++index;
        }
        return begin<index;
    }
    bool isNumber(string s) {
        if(s.size()==0) return false;
        int index=0;
        while(s[index]==' ') index++;
        bool numeric=ScanInt(s,index);//检查小数点前的有符号整数
        if(s[index]=='.')
        {
            ++index;
            numeric=ScanUint(s,index)||numeric;
        }
        if(s[index]=='e'||s[index]=='E')
        {
            ++index;
            numeric=ScanInt(s,index)&&numeric;
        }
        while(s[index]==' ')
            ++index;
        return numeric&&(index==s.size());
    }
};
```

## [剑指 Offer 58 - I. 翻转单词顺序(Easy)](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

```c++
class Solution {
public:
    string reverseWords(string s) {
        if(s.size()==0) return s;
        string res;
        int right=s.size()-1,left;
        while(right>=0)
        {
            while(right>=0&&s[right]==' ') right--;
            if(right < 0) break;
            left=right;
            while(left>=0&&s[left]!=' ') left--;
            res+=s.substr(left+1,right-left);
            res+=' ';
            right=left;
        }
        if(!res.empty())res.pop_back();
        return res;
    }
};
```

## [剑指 Offer 58 - II. 左旋转字符串(Easy)](https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof/)

```c++
class Solution {
public:
    string reverseLeftWords(string s, int n) {
        int size=s.size();
        s.resize(s.size()+n);
        for(int i=0;i<n;i++)
        {
            s[size+i]=s[i];
        }
        return s.substr(n,size);
    }
};
```

## [剑指 Offer 67. 把字符串转换成整数(Medium)](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

```c++
class Solution {
public:
    int strToInt(string str) {
        int res=0,bndry=INT_MAX/10;
        int i=0,sign=1,size=str.size();
        if(size==0) return 0;
        while(str[i]==' ')
            if(++i==size) return 0;
        if(str[i]=='-')sign=-1;
        if(str[i]=='-'||str[i]=='+') i++;
        for(int j=i;j<size;j++)
        {
            if(str[j]<'0'||str[j]>'9') break;
            if(res>bndry||(res==bndry&&str[j]>'7')) 
                return sign==1?INT_MAX:INT_MIN;
            res=res*10+(str[j]-'0');
        }
        return sign*res;
    }
};
```

# 三、栈和队列

## [剑指 Offer 09. 用两个栈实现队列(Easy)](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

```c++
class CQueue {
public:
    stack<int> A,B;
    CQueue() {

    }
    
    void appendTail(int value) {
        A.push(value);
    }
    
    int deleteHead() {
        if(B.empty())
        {
            while(!A.empty())
            {
                B.push(A.top());
                A.pop();
            }
        }
        if(B.empty()) return -1;
        int res=B.top();
        B.pop();
        return res;
    }
};
```

## [剑指 Offer 30. 包含min函数的栈(Easy)](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

```C++
class MinStack {
public:
    /** initialize your data structure here. */
    stack<int>A,B;
    MinStack() {

    }
    
    void push(int x) {
        A.push(x);
        if(B.empty())B.push(x);
        else if(!B.empty()&&B.top()>=x) B.push(x);
    }
    
    void pop() {
        if(B.top()==A.top()) B.pop();
        A.pop();

    }
    
    int top() {
        return A.top();
    }
    
    int min() {
        return B.top();
    }
};
```

## [剑指 Offer 59 - II. 队列的最大值(Medium)](https://leetcode-cn.com/problems/dui-lie-de-zui-da-zhi-lcof/)

```c++
class MaxQueue {
public:
    queue<int> A;
    deque<int> B;//递减
    MaxQueue() {
    
    }
    
    int max_value() {
        return B.empty()?-1:B.front();
    }
    
    void push_back(int value) {
        A.push(value);
        while(!B.empty()&&B.back()<=value) B.pop_back();//保证递减
        B.push_back(value);
    }
    
    int pop_front() {
        if(A.empty())return -1;
        int tmp=A.front();
        A.pop();
        if(tmp==B.front())B.pop_front();
        return tmp;
    }
};
```



# 四、查找算法

## [剑指 Offer 04. 二维数组中的查找(Medium)](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

```c++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        if(matrix.empty()) return false;
        int a=0;
        int b=matrix[0].size()-1;
        int i=matrix.size();
        while(a<i&&b>=0)
        {
            if(matrix[a][b]<target) a++;
            else if(matrix[a][b]>target) b--;
            else return true;
        }
        return false;
    }
};
```

# 五、滑动窗口

## [剑指 Offer 59 - I. 滑动窗口的最大值(Hard)](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n=nums.size();
        deque<int> dq;//单调递减
        vector<int> ret;
        for(int i=0;i<n;i++)
        {
            if(i>=k&&nums[i-k]==dq.front())
                dq.pop_front();
            while(!dq.empty()&&dq.back()<nums[i])
                dq.pop_back();//保持dq递减
            dq.push_back(nums[i]);
            
            if(i>=k-1)
                ret.push_back(dq.front());
            
        }
        return ret;
    }
};
```

# 六、动态规划

## [剑指 Offer 10- I. 斐波那契数列(Easy)](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

```c++
class Solution {
public:
    int fib(int n) {
        int a=0,b=1,sum;
        for(int i=0;i<n;i++)
        {
            sum=(a+b)%1000000007;
            a=b;
            b=sum;
        }
        return a;
    }
};
```

## [剑指 Offer 10- II. 青蛙跳台阶问题(Easy)](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

```c++
class Solution {
public:
    int numWays(int n) {
        vector<int> dp;
        dp.push_back(1);
        dp.push_back(1);
        dp.push_back(2);
        for(int i=3;i<=n;i++)
        {
            dp.push_back((dp[i-1]+dp[i-2])%1000000007);
        }
        return dp[n];
    }
};
```

## [剑指 Offer 14- I. 剪绳子(Medimun)](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

```c++
class Solution {
public:
    int cuttingRope(int n) {
        vector<int> dp(n+1,0);
        if(n<=3) return n-1;
        dp[0]=1;
        dp[1]=1;
        dp[2]=1;
        dp[3]=2;
        for(int i=4;i<=n;i++)
        {
            for(int j=2;j<=3;j++)
            {
                //dp[i]：存储最大值
                //j*(i-j)：将绳子切成j和(i-j)两份，之后不再切分
                //j*dp[i-j]：将绳子切成j和(i-j)两份后，(i-j)这份继续切分
                dp[i]=max(dp[i],max(j*(i-j),j*dp[i-j]));
            }
        }
        return dp[n];
    }
};
```

## [剑指 Offer 42. 连续子数组的最大和(Easy)](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int res=nums[0];
        for(int i=1;i<nums.size();i++)
        {
            nums[i]+=max(nums[i-1],0);
            res=max(nums[i],res);
        }    
        return res;
    }
};
```

## [剑指 Offer 46. 把数字翻译成字符串(Medium)](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)	

```c++
class Solution {
public:
    int translateNum(int num) {
        string s=to_string(num);
        int a=1,b=1,len=s.size();// “无数字” 和 “第 1 位数字” 的翻译方法数量均为 1
        for(int i=2;i<=len;i++)
        {
            string tmp=s.substr(i-2,2);
            int c=tmp>="10"&&tmp<="25"?a+b:b;
            a=b;
            b=c;
        }
        return b;
    }
};

```

## [剑指 Offer 47. 礼物的最大价值(Medium)](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

```c++
class Solution {
public:
    int maxValue(vector<vector<int>>& grid) {
        int m=grid.size();
        int n=grid[0].size();
        for(int i=1;i<n;i++)
        {
            grid[0][i]+=grid[0][i-1];
        }
        for(int i=1;i<m;i++)
        {
            grid[i][0]+=grid[i-1][0];
        }
        for(int i=1;i<m;i++)
        {
            for(int j=1;j<n;j++)
            {
                grid[i][j]+=max(grid[i-1][j],grid[i][j-1]);
            }
        }
        return grid[m-1][n-1];
    }
};
```

## [剑指 Offer 48. 最长不含重复字符的子字符串(Medium)](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

- 状态定义： 设动态规划列表 dp ，dp[j] 代表以字符 s[j] 为结尾的 “最长不重复子字符串” 的长度。
- 转移方程： 固定右边界 j ，设字符 s[j]左边距离最近的相同字符为 s[i] ，即 s[i]=s[j] 。

1. 当 i < 0，即 s[j] 左边无相同字符，则 dp[j]=dp[j−1]+1 ；
2. 当 dp[j - 1] < j - i，说明字符 s[i] 在子字符串 dp[j−1] 区间之外 ，则 dp[j] = dp[j - 1] + 1；
3. 当 dp[j − 1] ≥ j − i ，说明字符 s[i] 在子字符串 dp[j−1] 区间之中 ，则 dp[j] 的左边界由s[i] 决定，即 dp[j] = j - i；

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<char,int> hash;
        int res=0,temp=0,size=s.size(),i;
        for(int j=0;j<size;j++)
        {
            if(hash.find(s[j])==hash.end()) i=-1;
            else i=hash.find(s[j])->second;
            hash[s[j]]=j;
            temp=temp<j-i?temp+1:j-i;
            res=max(res,temp);
        }
        return res;
    }
};
```

## [剑指 Offer 49. 丑数(Medium)](https://leetcode-cn.com/problems/chou-shu-lcof/)

```c++
class Solution {
public:
    int nthUglyNumber(int n) {
        int dp[2000];
        int a=0,b=0,c=0;
        dp[0]=1;
        for(int i=1;i<n;i++)
        {
            int n2=dp[a]*2,n3=dp[b]*3,n5=dp[c]*5;
            dp[i]=min(min(n2,n3),n5);
            if(dp[i]==n2) a++;
            if(dp[i]==n3) b++;
            if(dp[i]==n5) c++;
        }
        return dp[n-1];
    }
};
```

## [剑指 Offer 60. n个骰子的点数(Medium)](https://leetcode-cn.com/problems/nge-tou-zi-de-dian-shu-lcof/)

[图解](https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/ozsdss/)

```c++
class Solution {
public:
    vector<double> dicesProbability(int n) {
        vector<double> dp(6,1.0/6);
        for(int i=2;i<=n;i++)
        {
            vector<double> temp(5*i+1,0);
            for(int j=0;j<dp.size();j++)
            {
                for(int k=0;k<6;k++)
                {
                    temp[j+k]+=dp[j]/6.0;
                }
            }
            dp=temp;
        }
        return dp;
    }
};
```

## [剑指 Offer 63. 股票的最大利润(Medium)](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int cost=INT_MAX,profit=0;
        for(int i=0;i<prices.size();i++)
        {
            cost=min(cost,prices[i]);
            profit=max(profit,prices[i]-cost);
        }
        return profit;
    }
};
```

# 七、搜索与回溯算法

## [剑指 Offer 12. 矩阵中的路径(Medium)](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

```c++
class Solution {
public:
    int m,n;

    bool exist(vector<vector<char>>& board, string word) {
        m=board.size();
        n=board[0].size();
        for(int i=0;i<m;i++)
        {
            for(int j=0;j<n;j++)
            {
                if (dfs(board,word,i,j,0)) return true;
            }
        }
        return false;
    }
    bool dfs(vector<vector<char>>& board,string word,int i,int j,int k)
    {
        if(i>=m||i<0||j>=n||j<0||board[i][j]!=word[k]) return false;
        if(k==word.size()-1) return true;
        board[i][j]='\0';//访问过
        bool res = dfs(board,word,i+1,j,k+1)||dfs(board,word,i-1,j,k+1)||dfs(board,word,i,j+1,k+1)||dfs(board,word,i,j-1,k+1);
        board[i][j]=word[k];
        return res;

    }
};
```

## [剑指 Offer 13. 机器人的运动范围(Medium)](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

```c++
class Solution {
public:
    int movingCount(int m, int n, int k) {
        vector<vector<bool>> visited(m,vector<bool>(n,false));
        return dfs(0,0,m,n,k,visited);
    }
    int dfs(int i,int j,int m,int n,int k,vector<vector<bool>>& visited)
    {
        if(i>=m||j>=n||i<0||j<0||visited[i][j]||cal(i)+cal(j)>k) return 0;
        visited[i][j]=true;
        return dfs(i+1,j,m,n,k,visited)+dfs(i,j+1,m,n,k,visited)+1;
    }
    int cal(int x)
    {
        return x/10+x%10;
    }
};
```

## [剑指 Offer 26. 树的子结构(Medium)](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

```c++
class Solution {
public:
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        //只有A,B均不为空才能有左右子树
        return (A!=nullptr&&B!=nullptr)&&(dfs(A,B)||isSubStructure(A->left,B)||isSubStructure(A->right,B));
    }
    bool dfs(TreeNode* A,TreeNode* B)
    {
        if(B==nullptr) return true;
        if(A==nullptr||A->val!=B->val) return false;
        return dfs(A->left,B->left)&&dfs(A->right,B->right); 
    }
};
```

## [剑指 Offer 27. 二叉树的镜像(Easy)](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

```c++
class Solution {
public:
    TreeNode* mirrorTree(TreeNode* root) {
        swap(root);
        return root;
    }
    void swap(TreeNode* root)
    {
        if(root==nullptr) return ;
        TreeNode* temp=root->left;
        root->left=root->right;
        root->right=temp;
        swap(root->left);
        swap(root->right);
    }
};
```

## [剑指 Offer 28. 对称的二叉树(Easy)](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

```c++
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        return root==nullptr||judge(root->left,root->right);
    }
    bool judge(TreeNode* left,TreeNode* right)
    {
        if(left==nullptr&&right==nullptr) return true;
        if(left==nullptr||right==nullptr||left->val!=right->val) return false;
        return judge(left->left,right->right)&&judge(left->right,right->left);
    }
};
```

## [剑指 Offer 32 - I. 从上到下打印二叉树(Medium)](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

```c++
class Solution {
public:
    vector<int> levelOrder(TreeNode* root) {
        vector<int> res;
        queue<TreeNode*> q;
        if(root) q.push(root);
        while(!q.empty())
        {
            TreeNode* tmp=q.front();
            q.pop();
            if(tmp->left) q.push(tmp->left);
            if(tmp->right) q.push(tmp->right);
            res.push_back(tmp->val);
        }
        return res;
    }
};
```

## [剑指 Offer 32 - II. 从上到下打印二叉树 II(Easy)](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> q;
        vector<vector<int>> res;
        if(root) q.push(root);
        while(!q.empty())
        {
            vector<int> temp;
            int size=q.size();
            for(int i=0;i<size;i++)
            {
                TreeNode* tmp=q.front();
                q.pop();
                temp.push_back(tmp->val);
                if(tmp->left) q.push(tmp->left);
                if(tmp->right) q.push(tmp->right);
                
            }
            res.push_back(temp);
        }
        return res;
    }
};
```

## [剑指 Offer 32 - III. 从上到下打印二叉树 III(Medium)](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> q;
        vector<vector<int>> res;
        if(root) q.push(root);
        int cnt=1;
        while(!q.empty())
        {
            vector<int> temp;
            int size=q.size();
            for(int i=0;i<size;i++)
            {
                TreeNode* tmp=q.front();
                q.pop();
                temp.push_back(tmp->val);
                if(tmp->left) q.push(tmp->left);
                if(tmp->right) q.push(tmp->right);
                
            }
            if(cnt%2==0) reverse(temp.begin(),temp.end());
            res.push_back(temp);
            cnt++;
        }
        return res;
    }
};
```

## [剑指 Offer 34. 二叉树中和为某一值的路径(Medium)](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> path;
    vector<vector<int>> pathSum(TreeNode* root, int target) {
        recur(root,target);
        return res;
    }
    void recur(TreeNode* root,int target)
    {
        if(root==nullptr) return ;
        path.push_back(root->val);
        target-=root->val;
        if(target==0&&root->left==nullptr&&root->right==nullptr) 
            res.push_back(path);
        recur(root->left,target);
        recur(root->right,target);
        path.pop_back();
    }
};
```

## [剑指 Offer 36. 二叉搜索树与双向链表(Medium)](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

```c++
class Solution {
public:
    Node* pre;
    Node* head;
    Node* treeToDoublyList(Node* root) {
        if(!root) return nullptr;
        dfs(root);
        head->left=pre;
        pre->right=head;
        return head;
    }
    void dfs(Node* cur)
    {
        if(cur==nullptr) return ;
        dfs(cur->left);
        if(pre==nullptr)head=cur;
        else pre->right=cur;
        cur->left=pre;
        pre=cur;
        dfs(cur->right);
    }
};
```

## [46. 全排列(Medium)](https://leetcode-cn.com/problems/permutations/)

```c++
class Solution {
public:

    void backtrack(vector<vector<int>>& res,vector<int>& output,int first,int len)
    {
        if(first==len)
        {
            res.push_back(output);
            return ;
        }
        for(int i=first;i<len;i++)
        {
            swap(output[i],output[first]);
            backtrack(res,output,first+1,len);
            swap(output[i],output[first]);
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> res;
        backtrack(res,nums,0,nums.size());
        return res;
    }
};
```

## [47. 全排列 II(Medium)](https://leetcode-cn.com/problems/permutations-ii/)

```c++
class Solution {
public:
    
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<vector<int>> res;
        dfs(res,nums,0,nums.size());
        return res;
    }
    void dfs(vector<vector<int>>& res,vector<int>& nums,int first,int len)
    {
        if(first==len-1)
        {
            res.push_back(nums);
            return ;
        }
        set<int> st;//当字符串存在重复字符时，排列方案中也存在重复的排列方案。为排除重复方案，需在固定某位字符时，保证 “每种字符只在此位固定一次” ，即遇到重复字符时不交换，直接跳过。
        for(int i=first;i<len;i++)
        {
            if(st.find(nums[i])!=st.end()) 
                continue;
            st.insert(nums[i]);
            swap(nums[i],nums[first]);
            dfs(res,nums,first+1,len);
            swap(nums[i],nums[first]);
        }

    }
};
```

## [剑指 Offer 38. 字符串的排列(Medium)](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

```c++
class Solution {
public:
    vector<string> permutation(string s) {
        vector<string> res;
        dfs(res,s,0);
        return res;
    }
    void dfs(vector<string>& res,string s,int x)
    {
        if(s.size()-1==x)
        {
            res.push_back(s);
            return ;
        }
        set<char> st;
        for(int i=x;i<s.size();i++)
        {
            if(st.find(s[i])!=st.end()) continue;
            st.insert(s[i]);
            swap(s[i], s[x]);                       // 交换，将 s[i] 固定在第 x 位
            dfs(res,s, x + 1);                          // 开启固定第 x + 1 位字符
            swap(s[i], s[x]);
        }
    }
};
```

## [剑指 Offer 54. 二叉搜索树的第k大节点(Easy)](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

```c++
class Solution {
public:
    int res,k;
    int kthLargest(TreeNode* root, int k) {
        this->k = k;
        dfs(root);
        return res;
    }
    void dfs(TreeNode* root)
    {
        if(root==nullptr) return ;
        dfs(root->right);
        k--;
        if(k==0) res = root->val;
        dfs(root->left);
    }
};
```

## [剑指 Offer 55 - I. 二叉树的深度(Easy)](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(root==nullptr) return 0;
        return max(maxDepth(root->left),maxDepth(root->right))+1;
    }
};
```

## [剑指 Offer 55 - II. 平衡二叉树(Easy)](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

```c++
class Solution {
public:
    bool isBalanced(TreeNode* root) {
        return recur(root)!=-1;
    }
    int recur(TreeNode* root)
    {
        if(root==nullptr) return 0;
        int left=recur(root->left);
        if(left==-1) return -1;
        int right=recur(root->right);
        if(right==-1) return -1;
        return abs(left-right)<=1?max(left,right)+1:-1;
    }
};
```

## [剑指 Offer 64. 求1+2+…+n(Medium)](https://leetcode-cn.com/problems/qiu-12n-lcof/)

```c++
class Solution {
public:
    int sumNums(int n) {
        if(n==1) return 1;
        return n+sumNums(n-1);
    }
};
```

## [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先(Easy)](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

### （1）迭代

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(p->val>q->val) swap(p,q);
        while(root!=nullptr)
        {
            if(root->val<p->val) root=root->right;
            else if(root->val>q->val) root=root->left;
            else break;
        }
        return root;
    }
};
```

### （2）递归

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root->val < p->val && root->val < q->val)
            return lowestCommonAncestor(root->right, p, q);
        if(root->val > p->val && root->val > q->val)
            return lowestCommonAncestor(root->left, p, q);
        return root;
    }
};
```

## [剑指 Offer 68 - II. 二叉树的最近公共祖先(Easy)](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root==nullptr||root==p||root==q) return root;
        TreeNode* left=lowestCommonAncestor(root->left,p,q);
        TreeNode* right=lowestCommonAncestor(root->right,p,q);
        if(left==nullptr&&right==nullptr) return nullptr;
        if(left==nullptr) return right;
        if(right==nullptr) return left;
        return root;//left!=nullptr&&right!=nullptrm
    }
};
```

# 八、分治算法

## [剑指 Offer 07. 重建二叉树(Medium)](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

```c++
class Solution {
public:
    vector<int> preorder;
    unordered_map<int,int> hash;
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        this->preorder=preorder;
        for(int i=0;i<inorder.size();i++)
        {
            hash[inorder[i]]=i;
        }
        return recur(0,0,inorder.size()-1);
    }
    //递推参数： 根节点在前序遍历的索引 root 、子树在中序遍历的左边界 left 、子树在中序遍历的右边界 right ；
    TreeNode* recur(int root,int left,int right)
    {
        if(left>right) return nullptr;
        TreeNode* node=new TreeNode(preorder[root]);
        int i=hash[preorder[root]];//查找根节点在中序遍历 inorder 中的索引 i
        node->left=recur(root+1,left,i-1);
        node->right=recur(root+i-left+1,i+1,right);//根节点索引 + 左子树长度 + 1
        return node;
    }
};
```

## [剑指 Offer 16. 数值的整数次方(Medium)](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

```c++
class Solution {
public:
    double myPow(double x, int n) {
        long long N=n;
        if(N==0) return 1;
        if(N==1) return x;
        if(N<0)
        {
            x=1/x;
            N=-N;
        }
        double res=myPow(x,N/2);
        return N%2?res*res*x:res*res;
    }
};
```

## [剑指 Offer 17. 打印从1到最大的n位数(Easy)](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

```c++
class Solution {
public:
    vector<int> res;
    vector<int> printNumbers(int n) {
        string path(n,'0');
        dfs(n,0,path);
        return res;
    }
    void dfs(int n,int index,string path)
    {
        if(n==index)
        {
            int val=stoi(path);
            if(val!=0) res.push_back(val);
            return ;
        }
        for(int i=0;i<10;i++)
        {
            path[index]=i+'0';
            dfs(n,index+1,path);
        }
    }
};
```

## [剑指 Offer 33. 二叉搜索树的后序遍历序列(Medium)](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

### （1）分治

```c++
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        return recur(postorder,0,postorder.size()-1);
    }
    bool recur(vector<int>& postorder,int i,int j)
    {
        if(i>=j) return true;
        int p=i;
        while(postorder[p]<postorder[j]) p++;
        int m=p;
        while(postorder[p]>postorder[j]) p++;//左子树区间 [i,m−1] 、右子树区间 [m, j-1] 、根节点索引j
        return p==j&&recur(postorder,i,m-1)&&recur(postorder,m,j-1);
    }
};
```

### （2）辅助单调栈

[图解](https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/5vwbf6/)

```c++
class Solution {
public:
    bool verifyPostorder(vector<int>& postorder) {
        stack<int> st;
        int root=INT_MAX;
        int size=postorder.size();
        for(int i=size-1;i>=0;i--)
        {
            if(postorder[i]>root) return false;
            while(!st.empty()&&st.top()>postorder[i])
            {
                root=st.top();
                st.pop();
            }
            st.push(postorder[i]);
        }
        return true;
    }
};
```

## [剑指 Offer 51. 数组中的逆序对(Hard)](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

```c++
class Solution {
    
    int reversePairs(vector<int>& nums) {
        vector<int> tmp(nums.size());
        return mergeSort(0, nums.size() - 1, nums, tmp);
    }
    int mergeSort(int l, int r, vector<int>& nums, vector<int>& tmp) {
        // 终止条件
        if (l >= r) return 0;
        // 递归划分
        int m = (l + r) / 2;
        int res = mergeSort(l, m, nums, tmp) + mergeSort(m + 1, r, nums, tmp);
        // 合并阶段
        int i = l, j = m + 1;
        for (int k = l; k <= r; k++)
            tmp[k] = nums[k];
        for (int k = l; k <= r; k++) {
            if (i == m + 1)
                nums[k] = tmp[j++];
            else if (j == r + 1 || tmp[i] <= tmp[j])
                nums[k] = tmp[i++];
            else {
                nums[k] = tmp[j++];
                res += m - i + 1; // 统计逆序对
            }
        }
        return res;
    }
};
```

# 九、排序算法

## [剑指 Offer 40. 最小的k个数(Easy)](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

### （1）快速排序

[快排图解](https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/ohwddh/)

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
        if(k>=arr.size()) return arr;
        return quickSort(0,arr.size()-1,k,arr);
    }
    vector<int> quickSort(int l,int r,int k,vector<int>& arr)
    {
        int i=l,j=r;
        //若均采用i<=j,则容易产生越界，例如[3,2,1]
        while(i<j)
        {
            while(i<j&&arr[j]>=arr[l]) j--;
            while(i<j&&arr[i]<=arr[l]) i++;
            swap(arr[i],arr[j]);
        }
        swap(arr[i],arr[l]);
        if(i>k) return quickSort(l,i-1,k,arr);
        if(i<k) return quickSort(i+1,r,k,arr);
        vector<int> res;
        for(int i=0;i<k;i++)
        {
            res.push_back(arr[i]);
        }
        return res;
    }
};
```

### （2）堆排序（优先队列）

```c++
class Solution {
public:
    vector<int> getLeastNumbers(vector<int>& arr, int k) {
            if(k>=arr.size()) return arr;
        vector<int> res;
        priority_queue<int> q;
        if(arr.empty()||k==0) return res;
        for(int i=0;i<k;i++)
        {
            q.push(arr[i]);
        }
        for(int i=k;i<arr.size();i++)
        {
            if(arr[i]<q.top())
            {
                q.pop();
                q.push(arr[i]);
            }
        }
        while(q.size())
        {
            res.push_back(q.top());
            q.pop();
        }
        return res; 
    }
};
```

## [剑指 Offer 41. 数据流中的中位数(Hard)](https://leetcode-cn.com/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/)

```c++
class MedianFinder {
public:
    /** initialize your data structure here. */
    priority_queue<int,vector<int>,greater<int>> A;//小顶堆，存储较大的一部分
    priority_queue<int,vector<int>,less<int>> B;//大顶堆，存储较小的一部分
    MedianFinder() {

    }
    
    void addNum(int num) {
        if(A.size()!=B.size())
        {
            A.push(num);
            B.push(A.top());
            A.pop();
        }
        else{
            B.push(num);
            A.push(B.top());
            B.pop();
        }
    }
    
    double findMedian() {
        return A.size()==B.size()?(A.top()+B.top())/2.0:A.top();
    }
};
```

## [剑指 Offer 45. 把数组排成最小的数(Medium)](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

```c++
class Solution {
public:
    string minNumber(vector<int>& nums) {
        vector<string> strs;
        for(int i=0;i<nums.size();i++)
        {
            strs.push_back(to_string(nums[i]));
        }
        quickSort(strs,0,strs.size()-1);
        string res;
        for(string s:strs)
        {
            res.append(s);
        }
        return res;
    }
    void quickSort(vector<string>& strs,int l,int r)
    {
        if(l>=r) return ;
        int i=l,j=r;
        //若拼接字符串 x + y > y + x  ，则 x “大于” y；
        //若 x + y < y + x ，则 x “小于” y ；
        //x “小于” y 代表：排序完成后，数组中 x 应在 y 左边；“大于” 则反之。
        while(i<j)
        {
            while(i<j && strs[j]+strs[l]>=strs[l]+strs[j]) j--;
            while(i<j && strs[i]+strs[l]<=strs[l]+strs[i]) i++;
            swap(strs[i],strs[j]);
        }
        swap(strs[i],strs[l]);
        quickSort(strs,l,i-1);
        quickSort(strs,i+1,r);
    }
};
```

## [剑指 Offer 61. 扑克牌中的顺子(Easy)](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

```c++
class Solution {
public:
    bool isStraight(vector<int>& nums) {
        int joker=0;
        sort(nums.begin(),nums.end());
        for(int i=0;i<4;i++)
        {
            if(nums[i]==0) joker++;
            else if(nums[i]==nums[i+1]) return false;
        }
        return nums[4]-nums[joker]<5;
    }
};
```

# 十、查找算法

## [剑指 Offer 03. 数组中重复的数字(Easy)](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

### （1）哈希表

```c++
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        unordered_map<int,bool> hash;
        for(int num:nums)
        {
            if(hash[num]) return num;
            else hash[num]=true;
        }
        return -1;
    }
};
```

### （2）原地交换

```c++
class Solution {
public:
    int findRepeatNumber(vector<int>& nums) {
        int n=nums.size();
        for(int i=0;i<n;i++)
        {
            if(nums[i]==i) continue;
            if(nums[nums[i]]==nums[i]) return nums[i];
            swap(nums[i],nums[nums[i]]);
        }
        return -1;
    }
};
```

## [剑指 Offer 04. 二维数组中的查找(Medium)](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

```c++
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) {
        if(matrix.empty()) return false;
        int n=matrix.size();
        int m=matrix[0].size();
        int i=0,j=m-1;
        while(i<n && j>=0)
        {
            if(matrix[i][j]>target) j--;
            else if(matrix[i][j]<target) i++;
            else return true;
        }
        return false;
    }
};
```

## [剑指 Offer 50. 第一个只出现一次的字符(Easy)](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

### （1）哈希表

```c++
class Solution {
public:
    char firstUniqChar(string s) {
        unordered_map<char,bool> hash;
        for(char c:s)
        {
            hash[c] = hash.find(c)==hash.end();
        }
        for(char c:s)
        {
            if(hash[c]) return c;
        }
        return ' ';
    }
};
```

### （2）有序哈希表

```c++
class Solution {
public:
    char firstUniqChar(string s) {
        unordered_map<char,bool> hash;
        vector<char> key;
        for(char c:s)
        {
            if(hash.find(c)==hash.end()) key.push_back(c);
            hash[c] = hash.find(c)==hash.end();
        }
        for(char c:key)
        {
            if(hash[c]) return c;
        }
        return ' ';
    }
};
```

# 十一、双指针

# 十二、位运算

## [剑指 Offer 15. 二进制中1的个数(Easy)](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

- (n−1) 解析： 二进制数字 n 最右边的 1 变成 0 ，此 1 右边的 0 都变成 1 。
- n \& (n - 1) 解析： 二进制数字 n 最右边的 1 变成 0 ，其余不变。

![image-20210721141517838](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210721141517838.png)

```c++
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int res=0;
        while(n!=0)
        {
            res++;
            n=n&n-1;
        }
        return res;
    }
};
```

## [剑指 Offer 56 - I. 数组中数字出现的次数(Medium)](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

[解析](https://leetcode-cn.com/leetbook/read/illustration-of-algorithm/euj1a1/)

```c++
class Solution {
public:
    vector<int> singleNumbers(vector<int>& nums) {
        int x = 0, y = 0, n = 0, m = 1;
        for(int num : nums)         // 1. 遍历异或
            n ^= num;
        while((n & m) == 0)         // 2. 循环左移，计算 m，获取整数 x⊕y 首位 1
            					    //异或结果为1说明x和y在这一位上不同，那用只有这一位为1的数字m去分别异或x和y
            						//得到的结果一定不同，也就把x和y分到了不同的子数组
            m <<= 1;
        for(int num : nums) {       // 3. 拆分 nums 为两个子数组，分别遍历两个子数组执行异或
            if(num & m) x ^= num;   // 4. 当 num & m != 0
            else y ^= num;          // 4. 当 num & m == 0
        }
        return vector<int> {x, y};  // 5. 返回出现一次的数字
    }
};
```

## [剑指 Offer 56 - II. 数组中数字出现的次数 II(Medium)](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int count[32]={0};
        for(int num:nums)
        {
            for(int i=0;i<32;i++)
            {
                count[i]+=num&1;// 更新第 i 位 1 的个数之和
                num>>=1;// 第 i 位 --> 第 i+1 位
            }
        }
        int res=0,m=3;
        for(int i=31;i>=0;i--)
        {
            res<<=1;
            res|=count[i]%m;// 恢复第 i 位
        }
        return res;
    }
};
```

## [剑指 Offer 65. 不用加减乘除做加法(Easy)](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

![image-20210721150149734](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210721150149734.png)

​		循环求 n 和 c，直至进位 c = 0 ；此时 s = n ，返回 n即可。

```c++
class Solution {
public:
    int add(int a, int b) {
        while(b!=0)
        {
            int c=(unsigned int)(a&b)<<1;//进位
            a^=b;//非进位
            b=c;
        }
        return a;
    }
};
```

# 十三、数学

## [剑指 Offer 14- II. 剪绳子 II(Medium)](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

```c++
class Solution {
public:
    int cuttingRope(int n) {
        if(n<=3) return n-1;
        long res=1;
        while(n>4)
        {
            res=res*3%1000000007;
            n-=3;
        }
        return res*n%1000000007;
    }
};
```

## [剑指 Offer 39. 数组中出现次数超过一半的数字(Easy)](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

```c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int res=0,votes=0;
        for(int num:nums)
        {
            if(votes==0) res=num;
            votes+=num==res?1:-1;
        }
        return res;
    }
};
```

## [剑指 Offer 43. 1～n 整数中 1 出现的次数(Hard)](https://leetcode-cn.com/problems/1nzheng-shu-zhong-1chu-xian-de-ci-shu-lcof/)

```c++
class Solution {
public:
    int countDigitOne(int n) {
        long digit=1;
        int high=n/10,cur=n%10,low=0,res=0;
        while(high!=0||cur!=0)
        {
            if(cur==0) res=res+high*digit;
            else if(cur==1) res=res+high*digit+low+1;
            else res=res+(high+1)*digit;
            low+=cur*digit;
            cur=high%10;
            high/=10;
            digit*=10;
        }
        return res;
    }
};
```

# 十四、模拟

## [剑指 Offer 29. 顺时针打印矩阵(Easy)](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```c++
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix)
    {
        if (matrix.empty()) return {};
        int l = 0, r = matrix[0].size() - 1, t = 0, b = matrix.size() - 1;
        vector<int> res;
        while(true)
        {
            for (int i = l; i <= r; i++) res.push_back(matrix[t][i]); // left to right
            if (++t > b) break;
            for (int i = t; i <= b; i++) res.push_back(matrix[i][r]); // top to bottom
            if (l > --r) break;
            for (int i = r; i >= l; i--) res.push_back(matrix[b][i]); // right to left
            if (t > --b) break;
            for (int i = b; i >= t; i--) res.push_back(matrix[i][l]); // bottom to top
            if (++l > r) break;
        }
        return res;
    }
};
```

## [剑指 Offer 31. 栈的压入、弹出序列(Medium)](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

```c++
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        stack<int> stk;
        int i = 0;
        for(int num : pushed) {
            stk.push(num); // num 入栈
            while(!stk.empty() && stk.top() == popped[i]) { // 循环判断与出栈
                stk.pop();
                i++;
            }
        }
        return stk.empty();
    }
};
```