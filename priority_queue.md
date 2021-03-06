### priority_queue优先级队列

priority_queue是一个容器适配器。与queue类似，仅允许在尾部插入元素push，在首部取出元素pop，只不过内部元素具有优先级，pop时根据接收的compare函数对象决定输出顺序。

priority_queue原型如下：

```c++
#include <queue>

template<class T, // element type
         class Container = std::vector<T>, // container type
         class Compare = std::less<typename Container::value_type> > // compare function obj
class priority_queue;
```

priority_queue的底层容器默认为vector，比较函数默认为less。**此时priority_queue默认形成大顶堆**。

<br>

### 疑问

**优先级队列传入的比较函数为less，直觉上看应该形成小顶堆，但为什么实际上形成了大顶堆？**

<br>

### 源码

**less源码：**

```c++
template<typename T>
struct less: public binary_function<T, T, bool> {
    bool operator() (const T& x, const T& y) const {
        return x < y;
    }
};
```

**make_heap中调用的push_heap源码(为了简洁性去掉部分下划线)：**

```c++
template<typename T>
void push_heap(Iterator first,     // 随机访问迭代器
               Distance holeIndex, // 尾部新添加节点
               Distance topIndex,  // 首部节点索引
               T value,            // 新节点的值
               Compare comp)       // 比较函数, 传入less
{
    Distance parent = (holeIndex - 1) / 2; // 上浮,寻找新节点的父节点索引
    // 堆排序, 将新添加元素插入到合适位置
    while (holeIndex > topIndex && comp(*(first + parent), value)) {
        *(first + holeIndex) = *(first + parent); // 父节点值小于新节点,父节点下沉
        holeIndex = parent; // 新节点继续上浮
        parent = (holeIndex - 1) / 2; // 新父节点的索引
    }
    *(first + holeIndex) = value; // 插入新节点
}
```

根据上述源码可以看出，传入compare函数对象是为了比较新节点与父节点间的大小，返回true则交换两元素的位置。

- **传入less，当父节点小于子节点时返回true，此时进行交换，值更大的节点排在前面，所以会形成大顶堆。**
- **传入greater，当父节点大于子节点时返回true，此时进行交换，值更小的节点排在前面，所以会形成小顶堆。**

<br>

### 自定义比较函数

基本数据类型的比较函数使用greater或less即可。对于自定义类型，则需要定义相应的比较函数。

**运算符重载**

```c++
struct Node {
    int id;
    int value;
};

bool operator< (const Node& a, const Node& b) {
    return a.value < b.value;
}

priority_queue<Node> que; // 默认使用less, 大顶堆
```

这里需要注意的是，若重载operator>，则无法通过编译：

```c++
bool operator> (const Node& a, const Node& b) {
    return a.value > b.value;
}

priority_queue<Node> que; // compile error.
```

priority_queue默认使用less，less中使用的是运算符<，因此仅重载运算符>会出现错误。

<br>

**自定义仿函数**

```c++
struct cmp {
    bool operator() (const Node& a, const Node& b) {
        return a.value < b.value;
    }
};

priority_queue<Node, vector<Node>, cmp> heap;
```

<br>

**类成员函数重载**

```c++
bool Node::operator< (const Node& other) const {
	return this->value < other.value;    
}
```

<br>

**友元函数重载**

与普通重载函数相比，友元函数重载可以访问类的非公有成员

```c++
struct Node {
    friend bool operator< (const Node&, const Node&);
    int id;
private:
    int value;
};

bool operator< (const Node& a, const Node& b) {
    return a.value < b.value; // 访问Node的私有成员
}
```

