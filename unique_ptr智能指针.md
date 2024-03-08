---
UID: 2403081256
link: ""
cssclasses: 
tags:
---
## 1 `std::unique_ptr<T>` 的原理和使用

- `std::unique_ptr`独占性的实现

其不能拷贝和赋值，对应拷贝构造函数和赋值运算符函数已定义删除。

```cpp
// Disable copy from lvalue.
unique_ptr(const unique_ptr&) = delete;
unique_ptr& operator=(const unique_ptr&) = delete;

```

- C++14通过`std::make_unique`创建unique_ptr，是一种更加异常安全的做法。  
    
- 释放所有权 `sp.release()`, 返回raw pointer  
    
```cpp
unique_ptr<int> p1 = make_unique<int>(1);
int* a = p1.release();
std::cout<<*a<<std::endl;
delete a; // you need to delete it manually
```

- 重置所有权 `sp.reset()`, 释放所有权，指向空指针

```cpp
unique_ptr<int> p1 = make_unique<int>(1);
p1.reset();
std::cout<<p1.get()<<std::endl; // 0
```

## 2 `shared_ptr` 和 `unique_ptr` 共有操作

|方法|用途|
|---|---|
|p.get()|返回p中保存的指针，不会影响p的引用计数。|
|p.reset()|释放p指向的对象，将p置为空。|
|p.reset(q)|释放p指向的对象，令p指向q。|
|p.reset(new T)|释放p指向的对象，令p指向一个新的对象。|
|p.swap(q)|交换p和q中的指针。|
|swap(p, q)|交换p和q中的指针。|
|p.operator*()|解引用p。|
|p.operator->()|成员访问运算符，等价于(*p).member。|
|p.operator bool()|检查p是否为空指针。|

## 3 如何转移控制权？

`std::move()` 可以将一个`unique_ptr`转移给另一个`unique_ptr`或者`shared_ptr`。转移后，原来的`unique_ptr`将不再拥有对内存的控制权，将变为空指针。

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(0);
std::unique_ptr<int> p2 = std::move(p1); 
// now, p1 is nullptr
```
## 4 什么时候用`std::unique_ptr<T>`？

`std::unique_ptr<T>`比`std::shared_ptr<T>`具有更小的内存，而且不需要维护引用计数，因此它的性能更好。当我们需要一个独占的指针时，应该优先使用`std::unique_ptr<T>`。
