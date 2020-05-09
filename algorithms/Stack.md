# 栈(stack)

<!-- TOC -->

- [栈(stack)](#栈stack)
    - [1 栈的逻辑结构](#1-栈的逻辑结构)
    - [2 栈的物理结构](#2-栈的物理结构)
        - [2.1 顺序栈(sequential stack)](#21-顺序栈sequential-stack)
            - [2.1.1 顺序栈的实现](#211-顺序栈的实现)
            - [2.1.2 入栈（插入操作）](#212-入栈插入操作)
            - [2.1.3 出栈（删除操作）](#213-出栈删除操作)
        - [2.2 顺序栈之两栈共享空间](#22-顺序栈之两栈共享空间)
            - [2.2.1 两栈共享空间的实现](#221-两栈共享空间的实现)
            - [2.2.2 入栈](#222-入栈)
            - [2.2.3 出栈](#223-出栈)
            - [2.2.4 取栈顶元素](#224-取栈顶元素)
            - [2.2.5 判断栈是否为空](#225-判断栈是否为空)
        - [2.3 链栈(linked stack)](#23-链栈linked-stack)
            - [2.3.1 链栈的实现](#231-链栈的实现)
            - [2.3.2 链栈的入栈](#232-链栈的入栈)
            - [2.3.4 链栈的出栈](#234-链栈的出栈)
            - [2.3.5 链栈的析构函数](#235-链栈的析构函数)
    - [3 顺序栈与链栈的比较](#3-顺序栈与链栈的比较)

<!-- /TOC -->

## 1 栈的逻辑结构

- 栈指的是限定在表尾（栈顶）进行插入和删除操作的线性表
- **先进后出(first in last out)**，注意栈只是对线性表插入和删除操作的位置（空间上）进行了限定，而时间上却没有限定，即**出栈可以随时进行**

## 2 栈的物理结构

- 按存储方式有**顺序栈**和**链栈**两种

### 2.1 顺序栈(sequential stack)

- 栈的顺序存储结构称为顺序栈

#### 2.1.1 顺序栈的实现

```cpp
const int StackSize = 10;//10作为示例，可以自己修改
template<class DataType>
class SeqStack
{
    public:
        SeqStack() {top = -1;}//构造函数，初始化栈
        ~SeqStack() {};//析构函数
        void Push(DataType x);//入栈
        DataType Pop();//出栈
        DataType GetTop() {if(top != -1) return data[top];}//取栈顶元素但不出栈，不修改栈顶指针
        int Empty() {return top==-1 ? 1 : 0;}//判断栈是否为空栈
    private:
        DataType data[StakSize];//存放栈元素的顺序表
        int top;//栈顶指针，为栈顶元素在数组中的下标
}
```

- 顺序栈的基本操作时间复杂度均为$O(1)$

#### 2.1.2 入栈（插入操作）

- 思路：只需将栈顶指针top+1，然后在top指向处填入待插入元素

```cpp
template<class DataType>
void SeqStack<DataType>::Push(DataType x)
{
    if(top == StackSize-1) throw "上溢";//检查边界
    data[++top] = x;
}
```

#### 2.1.3 出栈（删除操作）

- 思路：只需删除栈顶元素，然后top-1

```cpp
template<class DataType>
DataType SeqStack<DataType>::Pop()
{
    if(top == -1) throw "下溢";//检查边界
    x = data[top--];
    return x;
}
```

### 2.2 顺序栈之两栈共享空间

当程序中需要同时使用两个具有相同数据类型的栈时，可以使用一个数组来存储这两个栈（**顺序栈的单向延伸性**），一个栈的栈底为该数组的起始端，另一个栈的栈底为该数组的末尾端，两个栈由各自的栈底向数组的中间延伸。  

#### 2.2.1 两栈共享空间的实现

```cpp
const int StackSize = 100;//同上，100只是示例
template<class DataType>
class BothStack
{
    public:
        BothStack() {top1 = -1; top2 = StackSize;}
        ~BothStack() {};
        void Push(int i, DataType x);//入栈，将x入栈i
        DataType Pop(int i);//对栈i进行出栈
        DataType GetTop(int i);//取栈i的栈顶元素
        int Empty(int i);//判断栈i是否为空
    private:
        DataType data[StackSize];
        int top1, top2;
}
```

#### 2.2.2 入栈

- 思路：入栈操作时，top1对应的应该是+，top2对应的应该是-，栈满时对应的是top1==top2-1

```cpp
template<class DataType>
void BothStack<DataType>::Push(int i, DataType x)
{
    if(top1 == top2-1) throw "上溢";//检查边界
    if(i == 1) {data[++top1] = x;}//在栈1入栈
    if(i == 2) {data[--top2] = x;}//在栈2入栈
}
```

#### 2.2.3 出栈

- 思路：栈1为空对应top1 == -1, 栈2为空对应top2 == StackSize

```cpp
template<class DataType>
DataType BothStack<DataType>::Pop(int i)
{
    if(i == 1)
    {
        top1 == -1 ? throw "下溢" ： return data[top1--];
    }
    if(i == 2)
    {
        top2 == StackSize ? throw "下溢" : return data[top2++];
    }
}
```

#### 2.2.4 取栈顶元素

```cpp
template<class DataType>
DataType BothStack<DataType>::GetTop(int i)
{
    if(i == 1)
    {
        top1 == -1 ? throw "栈空" : return data[top1];
    }
    if(i == 2)
    {
        top1 == StackSize ? throw "栈空" : return data[top2];
    }
}
```

#### 2.2.5 判断栈是否为空

```cpp
template<class DataType>
int BothStack<DataType>::Empty(int i)
{
    if(i == 1) return top1 == -1 ? 1 : 0;
    if(i == 2) return top2 == StackSize ? 1 : 0;
}
```

### 2.3 链栈(linked stack)

- 栈的链接存储结构称为链栈
- 由于只能在栈顶进行插入和删除操作，显然以单链表的头部做栈顶是最方便的，而且没有必要像单链表那样附加一个是头结点

#### 2.3.1 链栈的实现

- 链栈的结点可以直接使用单链表的结点结构，所以链栈的基本操作都是单链表基本操作的简化，除析构函数外，所有的算法的时间复杂度均为$O(1)$

```cpp
template<class DataType>
class LinkStack
{
    public:
        LinkStack() {top = Null;}//构造函数，初始化一个空链栈
        ~LinkStack();//析构函数
        void Push(DataType x);
        DataType Pop();
        DataType GetTop() {if(top != null) return top->data;}
        int Empty() {return top == null ? 1 : 0;}
    private:
        Node<DataType>* top;
}
```

#### 2.3.2 链栈的入栈

- 思路：链栈的栈顶插入一个结点即可，然后修改栈顶指针top指向新的栈顶
  
```cpp
template<class DataType>
void LinkStack<DataType>::Push(DataType x)
{
    Node<DataType>* tmp = new Node<DataType>;
    tmp->data = x;
    tmp->next = top;//插入后，新的栈顶元素的指针域要指向原来的栈顶，才能构成链表
    top = tmp;//更新栈顶指针
}
```

#### 2.3.4 链栈的出栈

- 思路：只需先暂存栈顶元素，然后将栈顶结点摘链（同时更新栈顶元素指针top）

```cpp
template<class DataType>
DataType LinkStack<DataType>::Pop()
{
    if(top == null) throw "下溢";//检查边界
    Node<DataType>* tmp = new Node<DataType>;
    DataType x = top->data;
    top = top->next;//完成摘链
    delete tmp;//删除结点
    return x;
}
```

#### 2.3.5 链栈的析构函数

- 将链栈中所有结点的存储空间释放

```cpp
template<class DataType>
LinkStack<DataType>::~LinkStack()
{
    while(top != nullptr)
    {
        Node<DataType>* tmp = top;
        top = top->next;
        delete tmp;
    }
}
```

## 3 顺序栈与链栈的比较

||顺序栈|链栈|
|:-:|:-:|:-:|
|时间性能|$O(1)$|$O(1)$
|空间性能|顺序表有固定的长度，所以有存储元素的个数的限制和空间浪费的问题|没有栈满问题，直到内存没有可用空间栈才算满了，但是每个元素都需要一个指针域，从而产生了结构性开销|
|适用情况|栈的使用过程中元素个数变化不大|栈的使用过程中元素个数变换较大|
