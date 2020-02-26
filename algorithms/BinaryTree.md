# 1. 二叉树（Binary Tree）

<!-- TOC -->

- [1. 二叉树（Binary Tree）](#1-二叉树binary-tree)
    - [1.1. 二叉树的定义](#11-二叉树的定义)
    - [1.2. 二叉树的性质](#12-二叉树的性质)
    - [1.3. 二叉树的存储结构](#13-二叉树的存储结构)
    - [1.4. 二叉树的遍历操作](#14-二叉树的遍历操作)
        - [1.4.1. 前序遍历](#141-前序遍历)
        - [1.4.2. 中序遍历](#142-中序遍历)
        - [1.4.3. 后序遍历](#143-后序遍历)
        - [1.4.4. 层序遍历](#144-层序遍历)
    - [1.5. 构建二叉树](#15-构建二叉树)
        - [1.5.1. 通过 前序+中序 构建二叉树](#151-通过-前序中序-构建二叉树)
        - [1.5.2. 通过 后序+中序 构建二叉树](#152-通过-后序中序-构建二叉树)

<!-- /TOC -->

## 1.1. 二叉树的定义

**二叉树：** 是n(>=0)个结点的有限集合，该集合或为空集（空二叉树），或者由一个根结点和两棵互不相交的、分别称为左子树（left subtree）和右子树（right subtree）的二叉树构成  
**特点：**

- 每个结点的度不大于2（每个结点最多有两个子树）
- 二叉树是有序的结构，即使树中的某个结点只有一个子树，也要区分它是左子树还是右子树。所以，树和二叉树是两种结构  

**二叉树的五种基本形态：**

- 空二叉树
- 只有一个根结点
- 根结点只有左子树
- 根结点只有右子树
- 根结点既有左子树又有右子树
  
**斜树：**
>所有结点只有左子树称为左斜树（left oblique tree）；所有结点只有右子树称为右斜树（right oblique tree）。  
>**特点：** 每一层只有一个结点，所以结点的个数与深度相同

**满二叉树（full binary tree）：**
>二叉树中，所有的结点都有左子树和右子树，且所有的叶子在同一层上  
>**特点：** 叶子全在最下一层；没有度为1的结点。

**完全二叉树（complete binary tree）：**
>对一棵n个结点的二叉树按层序编号，如果编号为i(1~n)的结点与同样深度的满二叉树中编号为i的结点在二叉树中的位置完全相同，则称为完全二叉树（就是从一个满二叉树拿走最后几个叶子）  
>**特点：**
>
>- 叶子结点全在最下两层，且最下层的叶子结点都集中在二叉树的左侧；
>- 若有度为1的结点，则只可能有一个，且该结点只有左孩子。

## 1.2. 二叉树的性质

- 二叉树的第k层最多有$2^{k-1}$个结点
- 一课深度为k的二叉树，最多有$2^k-1$个结点（等比数列求和），最少有k个结点
- 叶子结点个数为$n_0$,度为2的结点个数为$n_2$，则$n_0 = n_2 +1$; (所有结点的进分支=出分支：$n-1 = n_1 + 2n_2$)

**完全二叉树：**

>- 深度：$logn+1$(向下取整)
>- 编号后，结点i(1~n),有：  
>（1）若i>1,则结点i双亲编号为i/2（向下取整）；否则，结点i是根结点  
>（2）若2i<=n,则结点i的左孩子为2i;否则，无左孩子  
>（3）若2i+1<=n,则结点i的右孩子为2i+1；否则，无右孩子  

## 1.3. 二叉树的存储结构

- 顺序存储结构：数组，容易造成空间的浪费，一般仅用于存储完全二叉树
- 二叉链表（binary linked list）
  
**二叉链表的结点：**

```cpp
template<class DataType>
struct BiNode
{
    DataType data;
    BiNode<DataType>* lchild, *rchild;
}
```

**二叉树的实现：**

为了避免类的调用者访问类的私有变量root，在构造函数、析构函数和遍历函数中调用了相应的私有函数

```cpp
template<class DataType>
class BiTree
{
private:
    BiNode<DataType>* root;//指向根结点的头指针
    BiNode<DataType>* Creat(BiNode<DataType>* bt);//构造函数调用
    void Release(BiNode<DataType>* bt);//析构函数调用
    void PreOrder(BiNode<DataType>* bt);//前序遍历函数调用
    void InOrder(BiNode<DataType>* bt);//中序遍历函数调用
    void PostOrder(BiNode<DataType>* bt);//后序遍历函数调用
public:
    BiTree() {root = Creat(root);}//构造函数
    ~BiTree() {Release(root);}//析构函数，释放各结点的存储空间
    void PreOrder() {PreOrder(root);}//前序遍历函数
    void InOrder() {InOrder(root);}//中序遍历函数
    void PostOrder() {PostOrder(root);}//后序遍历函数
    void LeverOrder();//层序遍历函数
}
```

**构造函数：** 假设从输入来构造二叉树

```cpp
template<class DataType>
BiNode<DataType>* BiTree<DataType>::Creat(BiNode<DataType>* bt)
{
    std::cin >> ch;//输入结点的数据，假设为字符
    if(ch == '#') bt = null;//假设输入字符#表示空二叉树
    else
    {
        bt = new BiNode;
        bt->data = ch;//生成新结点
        bt->lchild = Creat(bt->lchild);//递归建立左子树
        bt->rchild = Creat(bt->rchild);//递归建立右子树
    }
    return bt;
}
```

**析构函数：**

```cpp
template<class DataType>
void BiTree<DataType>::Release(BiNode<DataType>* bt)
{
    if(bt != null)
    {
        Release(bt->lchild);//递归释放左子树
        Release(bt->rchild);//递归释放右子树
        delete bt;
    }
}
```

## 1.4. 二叉树的遍历操作

**注意：**

- 中序 + 前序or后序 才可以确定一个唯一的二叉树，否则没办法确定左右子树的情况
- 前中后三种遍历都有递归形式（比较简单），但是非递归版本（使用栈和队列）更为重要

### 1.4.1. 前序遍历

**递归实现：**

```cpp
template<class DataType>
void BiTree<DataType>::PreOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    if(bt == null) return;
    else
    {
        vec.push_back(bt->data);//保存数据
        PreOrder(bt->lchild, vec);//递归遍历左子树
        PreOrder(bt->rchlid, vec);//递归遍历右子树
    }
}
```

**非递归版本：** 关键是在前序遍历完某结点的左子树后，如何找到该结点的右子树的根指针  
（1）把二叉树的根结点root入栈  
（2）若栈不空，出栈一个结点；若该结点的右子树存在，则右子树指针入栈；若该结点的左子树存在，则把左子树指针入栈。（由于栈的特性，所以先入右子树再入左子树，这样出栈就是先左子树在右子树）  
（3）重复步骤（2），直到栈为空

```cpp
template<class DataType>
void BiTree<DataType>::PreOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    stack<BiNode<DataType>*> st;
    if(bt == null) return;
    st.push(bt);//根结点入栈
    while(!st.empty())//栈为空才结束
    {
        BiNode<DataType>* tmp = st.pop();
        vec.push_back(tmp->data);
        if(tmp->rchild != null) st.push(tmp->rchild);//右子树入栈
        if(tmp->lchild != null) st.push(tmp->lchild);//左子树入栈
    }
}
```

**另一种非递归版本：**  
（1）bt!=null时，表明当前二叉树不为空，此时输出根结点bt，并将根结点bt入栈，继续遍历bt的左子树  
（2）bt=null时，表明以bt为根结点指针的二叉树遍历完毕，且bt是栈顶指针所指结点的左子树，若栈不空，则令bt=出栈指针的右子树指针，继续遍历右子树；若栈空，则表明整个二叉树遍历完毕  

```cpp
template<class DataType>
void BiTree<DataType>::PreOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    stack<BiNode<DataType>*> st;//栈初始化
    while(bt!=null || !st.empty())//bt和栈同时为空才停止遍历
    {
        while(bt != null)//指针没到底，就继续入栈
        {
            vec.push_back(bt->data);//输出根结点
            st.push(bt);//根结点指针入栈
            bt = bt->lchild;//利用循环，遍历bt的左子树
        }
        if(!st.empty())//栈没空，说明还没遍历完二叉树
        {
            bt = st.pop();//出栈
            bt = bt->rchild;//利用循环,遍历bt的右子树
        }
    }
}
```


### 1.4.2. 中序遍历

**递归版本：**

```cpp
template<class DataType>
void BiTree<DataType>::InOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    if(bt == null) return;
    else
    {
        InOrder(bt->lchild, vec);//递归遍历左子树
        vec.push_back(bt->data);//保存数据
        InOrder(bt->rchlid, vec);//递归遍历右子树
    }
}
```

**非递归版本：** 遍历过程中遇到某结点时，不能立即访问它，而是将它入栈，等它的左子树遍历完后，再出栈该结点访问  
（1）把根结点极其左子树的根结点、左子树的左子树的根结点……按顺序入栈  
（2）出栈一个结点，若该结点的存在右子树，则把该结点的右子树及其右子树的所有左子树结点……按顺序入栈
（3）重复（2），直至栈为空且当前结点也为空

```cpp
template<class DataType>
void BiTree<DataType>::InOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    stack<BiNode<DataType>*> st;//栈初始化
    while(bt!=null || !st.empty())//bt和栈同时为空才停止遍历
    {
        while(bt != null)//指针没到底，就继续入栈
        {
            st.push(bt);//根结点指针入栈
            bt = bt->lchild;//利用循环，遍历bt的左子树
        }
        if(!st.empty())//栈没空，说明还没遍历完二叉树
        {
            bt = st.pop();//出栈
            vec.push_back(bt->data);//输出根结点
            bt = bt->rchild;//利用循环,遍历bt的右子树
        }
    }
}
```

### 1.4.3. 后序遍历

后序遍历的难点在于：需要判断上次访问的节点是位于左子树，还是右子树。若是位于左子树，则需跳过根节点，先进入右子树，再回头访问根节点；若是位于右子树，则直接访问根节点  
**递归版本：**

```cpp
template<class DataType>
void BiTree<DataType>::PostOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    if(bt == null) return;
    else
    {
        InOrder(bt->lchild, vec);//递归遍历左子树
        InOrder(bt->rchlid, vec);//递归遍历右子树
        vec.push_back(bt->data);//保存数据
    }
}
```

**非递归版本：**  
（1）把二叉树根结点root入栈  
（2）若栈不空，则出栈一个结点，把该结点插入到容器的头部；若该结点的左子树不为空，则把该结点的左子树入栈；若该结点的右子树不为空，则把右子树入栈  
（3）重复（2），直至栈为空，遍历结束

```cpp
template<class DataType>
void BiTree<DataType>::PostOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    stack<BiNode<DataType>*> st;//栈初始化
    st.push(bt);
    while(!st.empty())
    {
        BiNode<DataType>* tmp = st.pop();
        vec.insert(vec.begin(), tmp->data);//在vec头部插入结点
        if(tmp->lchild != null) st.push(tmp->lchild);//先左子树
        if(tmp->rchild != null) st.push(tmp->rchild);//再右子树
    }
}
```

**另一非递归版本：**

```cpp
template<class DataType>
void BiTree<DataType>::PostOrder(BiNode<DataType>* bt， vector<DataType> &vec)
{
    stack<BiNode<DataType>*> st;//栈初始化
    BiNode<DataType>* pre = null;//记录当前根结点的右结点是否已经保存过
    while(bt != null && !st.empty())//bt和栈同时为空才停止遍历
    {
        while(bt!=null)//指针没到底，就继续入栈
        {
            st.push(bt);
            bt = bt->lchild;
        }
        bt = st.top();
        if(bt->rchild==null || pre==bt->rchlid)
        {
            vec.push_back(bt->data);
            st.pop();
            pre = bt;
            bt = null;
        }
        else
        {
            bt = bt->rchild;
            pre = null;
        }
    }
}
```

### 1.4.4. 层序遍历

**思路：**  
（1）将根指针入队  
（2）从队列头部取出一个元素  访问该指针所指的结点  
（3）若该指针所指结点的左、右孩子结点非空， 则将其左、右孩子指针入队  
（4）重复（2）（3）直至队列为空  

```cpp
template<class DataType>
void BiTree<DataType>::LeverOrder(vector<DataType> &vec)
{
    queue<BiNode<DataType>*> q;//队列初始化
    if(root == null) return;//二叉树为空，遍历结束
    q.push(root);//二叉树不是空二叉树，根指针入队
    while(!q.empty())//循环直到队列为空
    {
        root = q.front();//取队头元素
        q.pop();//队头元素出队，即删除
        vec.push_back(root->data);
        if(root->lchild != null) q.push(root->lchild);
        if(root->rchild != null) q.push(root->rchild);
    }
}
```

**分析：** 从层序遍历代码可以看出，把整个二叉树从上到下，每一层的结点都从左到右的放入队列中去，然后从队头来遍历每个结点即可  

## 1.5. 构建二叉树

### 1.5.1. 通过 前序+中序 构建二叉树

**思路：** 分析遍历的特点：

- 前序遍历第一个元素一定是这个二叉树的根结点  
- 中序遍历中，根结点将序列分为了左右两个区间。左区间对应左子树，右区间对应右子树  

**能够找到根结点，就能找到左子树和右子树的集合**，那么就有了一个二叉树大致的样子，可以使用**递归**去构建出左子树和右子树  

```cpp
template<class DataType>
BiNode<DataType>* BuildTree(vector<DataType> preorder, vector<DataType> inoreder)
{//使用hashmap存储中序遍历，目的是查找方便。因为从前序遍历找到根结点后，还要寻找根结点在中序遍历中的位置
    unordered_map<int, DataType> inmap;
    for(int i=0; i<inoreder.size(); i++)
    {
        inmap.insert(pair<DataType, int>{inorder[i], i});
    }
    return Build(preorder, inmap, 0, preorder.size()-1, 0);
}

//传入五个参数：前序序列，中序序列，前序序列的开始，前序序列的结束，中序序列的开始
template<class DataType>
BiNode<DataType>* Build(vector<DataType> preorder, unordered_map<DataType,int> inmap, int preStart, int preEnd, int inStart)
{
    if(preEnd < preStrat) return null;//递归边界
    BiNode<DataType>* root = new BiNode(preorder[preStart]);//从先序序列首位生成根结点
    int rootIndex = inmap.at(root->data);//在中序序列中找到根结点对应的索引
    int len = rootIndex - inStart;//左子树的结点个数
    root->lchild = build(preorder, inmap, preStart+1, preStart+len, inStart);//左子树递归调用
    root->rchild = build(preorder, inmap, preStart+len+1, preEnd, rootIndex+1);//右子树递归调用
    return root;
}
```

### 1.5.2. 通过 后序+中序 构建二叉树

**思路：**  
（1）找到根结点（后序序列的最后一位）  
（2）在中序序列中，找到根结点的位置(既可以使用hash表来查找，也可以使用向量遍历得方式来寻找)，划分左右子树，进行递归构建  

```cpp
template<class DataType>
BiNode<DataType>* BuildTree(vector<DataType> postorder, vector<DataType> inoreder)
{//使用hashmap存储中序遍历，目的是查找方便。因为从前序遍历找到根结点后，还要寻找根结点在中序遍历中的位置，也可以通过循环向量遍历来查找
    hash_map<int, DataType> inmap;
    for(int i=0; i<inoreder.size(); i++)
    {
        inmap.insert(pair<DataType, int>{inorder[i], i});
    }
    return Build(postorder, inmap, 0, postorder.size()-1, 0);
}

//传入五个参数：前序序列，中序序列，前序序列的开始，前序序列的结束，中序序列的开始
template<class DataType>
BiNode<DataType>* Build(vector<DataType> postorder,unordered_map<DataType,int> inmap, int postStart, int postEnd, int inStart)
{
    if(postEnd < postStrat) return null;//递归边界
    BiNode<DataType>* root = new BiNode(postorder[posEnd]);//从后序序列最后位生成根结点
    int rootIndex = inmap.at(root->data);//在中序序列中找到根结点对应的索引
    int len = rootIndex - inStart;//左子树的结点个数
    root->lchild = build(postorder, inmap, postStart, postStart+len-1, inStart);//左子树递归调用
    root->rchild = build(postorder, inmap, postStart+len+1, postEnd-1, rootIndex+1);//右子树递归调用
    return root;
}
```

**另一个版本：** 清晰明了，没用使用hash表查找根结点，而是遍历查找

```cpp
template<class DataType>
BiNode<DataType>* BuildTree(vector<DataType> post, vector<DataType> vin)
{
    if(!vin.empty()) return null;//递归边界
    vector<DataType> postleft, postright, vinleft, vinright;
    BiNode<DataType>* root = new BiNode<DataType>(post[post.size()-1])//生成根结点
    int rootindex = 0;//查找根结点在中序序列中的位置
    for(int i=0; i<vin.size(); i++)
    {
        if(root->data == vin[i])
        {
            rootindex = i;
            break;
        }
    }

    //找到root所指结点的左子树对应的后序和中序
    for(int i=0; i<rootindex; i++)
    {
        postleft.push_back(post[i]);
        vinleft.push_back(vin[i]);
    }
    //找到root所指结点的右子树对应的后序和中序
    for(int i=rootindex+1; i<vin.size(); i++)
    {
        postright.push_back(post[i-1]);
        vinright.push_back(vin[i]);
    }
    //递归构建左右子树
    root->left = BuildTree(postleft, vinleft);
    root->right = BuildTree(postright, vinright);

    return root;

}
```