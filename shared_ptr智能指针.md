---
UID: 2403072122
link: ""
cssclasses: 
tags:
  - C加/智能指针
关联文件: "[[智能指针]]"
---

## 1 定义

**特点**：
和 unique_ptr、weak_ptr 不同之处在于，**多个 shared_ptr 智能指针可以共同使用同一块堆内存**。

由于该类型智能指针在实现上采用的是**引用计数机制**，即便有一个 shared_ptr 指针放弃了堆内存的“使用权”（引用计数减 1），也不会影响其他指向同一堆内存的 shared_ptr 指针（只有引用计数为 0 时，堆内存才会被自动释放）。**shared_ptr内部的引用计数是安全的，但是对象的读取需要加锁**。





## 2 常见操作

### 2.1 构造

1). `shared_ptr< T > sp1; `空的`shared_ptr`，可以指向类型为T的对象

```cpp
shared_ptr<Person> sp1;
Person *person1 = new Person(1);
sp1.reset(person1);	// 托管person1
```


2). `shared_ptr< T > sp2(new T());` 定义 `shared_ptr`, 同时指向类型为 T 的对象 

```cpp
shared_ptr<Person> sp2(new Person(2));
shared_ptr<Person> sp3(sp1);
```


3). `shared_ptr<T[]> sp4;` 空的`shared_ptr`，可以指向类型为`T[]`的数组对象 **C++17后支持**

```cpp
shared_ptr<Person[]> sp4;
```


4). `shared_ptr<T[]> sp5(new T[] { … });` 指向类型为 T 的数组对象 **C++17后支持**

```cpp
shared_ptr<Person[]> sp5(new Person[5] { 3, 4, 5, 6, 7 });
```


5). `shared_ptr< T > sp6(NULL, D());` 空的 shared_ptr，接受一个 D 类型的删除器，使用 D 释放内存

```cpp
shared_ptr<Person> sp6(NULL, DestructPerson());
```


6). `shared_ptr< T > sp7(new T(), D());` 定义 shared_ptr,指向类型为 T 的对象，接受一个 D 类型的删除器，使用 D 删除器来释放内存

```cpp
shared_ptr<Person> sp7(new Person(8), DestructPerson());
```



### 2.2 初始化


初始化方式：
- 方式一：使用构造函数
	- 构造出 `shared_ptr<T> ` 类型的空智能指针，**裸指针直接初始化，但不能通过隐式转换来构造**，因为 shared_ptr 构造函数被声明为 explicit；在构建 shared_ptr 智能指针，也可以明确其指。允许使用拷贝构造函数和移动构造函数。
- 方式二：使用自定义所指堆内存的释放规则。
	- 在某些场景中，自定义释放规则是很有必要的。比如，对于申请的动态数组来说，shared_ptr 指针默认的释放规则是不支持释放数组的，只能自定义对应的释放规则，才能正确地释放申请的堆内存。
- 方式三：使用 `make_shared` 初始化对象，分配内存效率更高(推荐使用)


```cpp
/*方式一：使用构造函数*/
std::shared_ptr<int> p1; //不传入任何实参
std::shared_ptr<int> p2(nullptr); //传入空指针 nullptr

std::shared_ptr<int> p3(new int(10));//构建了一个 shared_ptr 智能指针，其指向一块存有 10 这个 int 类型数据的堆内存空间。

std::shared_ptr<int> p4(p3);//或者 std::shared_ptr<int> p4 = p3;//调用拷贝构造函数
std::shared_ptr<int> p5(std::move(p4)); //或者 std::shared_ptr<int> p5 = std::move(p4);//调用移动构造函数

/*方式二：使用自定义所指堆内存的释放规则*/

//指定 default_delete 作为释放规则
std::shared_ptr<int> p6(new int[10], std::default_delete<int[]>());

//自定义释放规则
void deleteInt(int*p) {
delete []p;
}
//初始化智能指针，并自定义释放规则
std::shared_ptr<int> p7(new int[10], deleteInt);

std::shared_ptr<int> p7(new int[10], [](int* p) {delete[]p; });//借助 lambda 表达式初始化 p7

/*方式三：使用 make_shared 初始化对象*/
shared_ptr<int> up3 = make_shared<int>(2); // 多个参数以逗号','隔开，最多接受十个
shared_ptr<string> up4 = make_shared<string>("字符串");
shared_ptr<Person> up5 = make_shared<Person>(9);

```

如上所示，p3 和 p4 都是 shared_ptr 类型的智能指针，因此可以用 p3 来初始化 p4，由于 p3 是左值，因此会调用**拷贝构造函数**。需要注意的是，如果 p3 为空智能指针，则 p4 也为空智能指针，其引用计数初始值为 0；反之，则表明 p4 和 p3 指向同一块堆内存，同时该堆空间的引用计数会加 1。  
  
而对于 `std::move(p4)` 来说，该函数会强制将 p4 转换成对应的右值，因此初始化 p5 调用的是**移动构造函数**。另外和调用拷贝构造函数不同，用 `std::move(p4)` 初始化 p5，会使得 p5 拥有了 p4 的堆内存，而 p4 则变成了空智能指针。

### 2.3 赋值

```cpp
shared_ptrr<int> up1(new int(10));  // int(10) 的引用计数为1
shared_ptr<int> up2(new int(11));   // int(11) 的引用计数为1
up1 = up2;	// int(10) 的引用计数减1,计数归零内存释放，up2共享int(11)给up1, int(11)的引用计数为2
```

### 2.4 主动释放对象

```cpp
shared_ptrr<int> up1(new int(10));
up1 = nullptr ;	// int(10) 的引用计数减1,计数归零内存释放 
// 或
up1 = NULL; // 作用同上 
```


### 2.5 重置

```cpp
p.reset() ; //将p重置为空指针，所管理对象引用计数 减1
p.reset(p1); //将p重置为p1（的值）,p 管控的对象计数减1，p接管对p1指针的管控
p.reset(p1,d); //将p重置为p1（的值），p 管控的对象计数减1并使用d作为删除器
```


### 2.6 成员方法

为了方便用户使用 shared_ptr 智能指针，`shared_ptr<T> ` 模板类还提供有一些实用的成员方法。
  
|成员方法名|功能|
|---|---|
|operator=()|重载赋值号，使得同一类型的 shared_ptr 智能指针可以相互赋值。|
|operator*()|重载 * 号，获取当前 shared_ptr 智能指针对象指向的数据。|
|operator->()|重载 -> 号，当智能指针指向的数据类型为自定义的结构体时，通过 -> 运算符可以获取其内部的指定成员。|
|swap()|交换 2 个相同类型 shared_ptr 智能指针的内容。|
|reset()|当函数没有实参时，该函数会使当前 shared_ptr 所指堆内存的引用计数减 1，同时将当前对象重置为一个空指针；当为函数传递一个新申请的堆内存时，则调用该函数的 shared_ptr 对象会获得该存储空间的所有权，并且引用计数的初始值为 1。|
|get()|获得 shared_ptr 对象内部包含的普通指针。|
|use_count()|返回同当前 shared_ptr 对象（包括它）指向相同的所有 shared_ptr 对象的数量。|
|unique()|判断当前 shared_ptr 对象指向的堆内存，是否不再有其它 shared_ptr 对象再指向它。|
|operator bool()|判断当前 shared_ptr 对象是否为空智能指针，如果是空指针，返回 false；反之，返回 true。|

>除此之外，C++11 标准还支持同一类型的 shared_ptr 对象，或者 shared_ptr 和 nullptr 之间，进行 `\==，!=，<，<=，>，>=` 运算。



## 3 注意点

1. 在使用 shared_ptr 智能指针时，**要注意避免对象交叉使用智能指针的情况！** 否则会导致内存泄露！若使用对象交叉使用智能指针，可使用**weak_ptr**弱指针
























