# 队列(queue)

<!-- TOC -->

- [队列(queue)](#队列queue)
    - [1 队列的逻辑结构](#1-队列的逻辑结构)
    - [2 队列的物理结构](#2-队列的物理结构)
        - [2.1 循环队列(circular queue)](#21-循环队列circular-queue)
            - [2.1.1 循环队列的实现](#211-循环队列的实现)
            - [2.1.2 入队](#212-入队)
            - [2.1.3 出队](#213-出队)
            - [2.1.4 读取队头](#214-读取队头)
        - [2.2 链队列(linked queue)](#22-链队列linked-queue)
            - [2.2.1 链队列的实现](#221-链队列的实现)
            - [2.2.2 构造函数](#222-构造函数)
            - [2.2.3 入队](#223-入队)
            - [2.2.4 出队](#224-出队)
            - [2.2.5 取队头元素](#225-取队头元素)
            - [2.2.6 析构函数](#226-析构函数)
    - [3 循环队列与链队列比较](#3-循环队列与链队列比较)

<!-- /TOC -->

## 1 队列的逻辑结构

- 队列指的是只能在一端进行插入（入队，队尾），另一端进行删除操作（出队，队头）的线性表
- 先进先出(first in first out)

## 2 队列的物理结构

### 2.1 循环队列(circular queue)

- 采用普通数组构成的队列，随着队列的插入和删除操作的进行，整个队列会向数组中下标较大的位置移动过去，从而产生队列的“单向移动性”，这样当数组被插入到数组中最大下标后，队列的空间就用尽了，而数组的低端还有空闲空间，这种现象叫做**假溢出**
- 将存储队列的数组看成是首尾相接的循环结构，这种**头尾相接的顺序存储结构**称为循环队列

#### 2.1.1 循环队列的实现

- 队头指针front指向队头元素的前一个位置，队尾指正rear指向队尾元素
- 队空条件：front == rear
- 队满条件：(rear+1)%QueueSize == front

```cpp
const int QueueSize = 100;//100只是作为示例
template<class DataType>
class CirQueue
{
    public:
        CirQueue() {front = rear = QueueSize-1;}//构造函数，初始化空队列,front和rear指向数组中同一个位置
        ~CirQueue() {}//类中没有用到真正的指针，所以析构函数为空
        void EnQueue(DataType x);//入队
        DataType DeQueue();//出队
        DataType GetQueue();//取队头元素（并不删除）
        int Empty() {return (rear+1)%QueueSize == front ? 1 : 0;}//判断队列是否为空
    private:
        DataType data[QueueSize];
        int front, rear;
};
```
  
- 循环队列基本操作的算法时间复杂度均为$O(1)$

#### 2.1.2 入队

- 思路：rear+1,然后在队尾插入元素

```cpp
template<class DataType>
void CirQueue<DataType>::EnQueue(DataType x)
{
    if((rear+1)%QueueSize == front) throw "上溢";//检查边界
    rear = (rear+1)%QueueSize;//队尾指针在循环意义下+1
    data[rear] = x;//队尾插入元素
}
```

#### 2.1.3 出队

- 思路：front+1，然后读取并返回队头元素

```cpp
template<class DataType>
DataType CirQueue<DataType>::DeQueue()
{
    if(front == rear) throw "下溢";
    front = (front+1)%QueueSize;
    return data[front];
}
```

#### 2.1.4 读取队头

- 思路：与出队类似，只是不要改变队头指针front

```cpp
template<class DataType>
DataType CirQueue<DataType>::GetQueue()
{
    if(front == rear) throw "下溢";
    int i = (front+1)%QueueSize;//引入i避免改变front
    return data[i];
}
```

### 2.2 链队列(linked queue)

- 队列的链接存储结构称为链队列

#### 2.2.1 链队列的实现

- 链队列使用单链表（保存头结点，使空队列和非空队列操作一致）
- 队头指针front指向头结点
- 队尾指针rear指向终端结点

```cpp
template<class DataType>
class LinkQueue
{
    public:
        LinkQueue();
        ~LinkQueue();
        void EnQueue(DataType x);//入队
        DataType DeQueue();//出队
        DataType GetQueue();//读取队头
        int Empty() {return (front==rear) ? 1 : 0;}//判断队列是否为空
    private:
        Node<DataType> *front, *rear;
};
```

- 链队列所有基本操作的算法（除了析构函数）时间复杂度均为$O(1)$

#### 2.2.2 构造函数

- 思路：构造函数的作用就是初始化一个空的链队列，只需申请一个头结点，然后让front和rear均指向头结点即可

```cpp
template<class DataType>
LinkQueue<DataType>::LinkQueue()
{
    Node<DataType>* tmp = new Node<DataType>;
    tmp->next = nullptr;//创建头结点
    front = rear = tmp;//队头指针和队尾指针都指向头结点
}
```

#### 2.2.3 入队

- 思路：链队列的插入操作就是在链表的尾部插入一个结点，然后将队尾指针后移

```cpp
template<class DataType>
void LinkQueue<DataType>::EnQueue(DataType x)
{
    Node<DataType>* tmp = new Node<DataType>;
    tmp->data = x;
    tmp->next = nullptr;//创建新结点保存待插入元素
    rear->next = tmp;//新结点插入队尾
    rear = tmp;//更新队尾指针
}
```

#### 2.2.4 出队

- 思路：只在队列的头部进行删除操作，注意队列长度为1的特殊情况（需要调整队尾指针rear）

```cpp
template<class DataType>
DataType LinkQueue<DataType>::DeQueue()
{
    if(front == rear) throw "下溢";//检查边界
    Node<DataType>* tmp = front->next;
    DataType x = tmp->data;//暂存队头元素
    front->next = tmp->next;//队头元素所在结点摘链
    if(tmp->next == nullptr) rear = front;//判断出出队前队列长度是否为1
    delete tmp;
    return x;
}
```

#### 2.2.5 取队头元素

- 思路：只需返回front->next->data

```cpp
template<class DataType>
DataType LinkQueue<DataType>::GetQueue()
{
    if(front == rear) throw "下溢";
    return front->next->data;
}
```

#### 2.2.6 析构函数

- 将链队列中所有结点的存储空间释放

```cpp
template<class DataType>
LinkQueue<DataType>::~LinkQueue()
{
    while(front != nullptr)
    {
        Node<DataType>* tmp = front;
        front = front->next;
        delete tmp;
    }
}
```

## 3 循环队列与链队列比较

||循环队列 vs 链队列|
|-|-|
|时间性能|$O(1)$|
|空间性能|同顺序栈与链栈的空间性能的比较类似，只是循环队列不能像顺序栈那样共享空间（不能在一个数组中存储两个队列）
