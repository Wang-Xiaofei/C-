---
UID: 2403081325
link: ""
cssclasses: 
tags:
---
## 1 定义

标准库还定义了一个名为`weak_ptr`的伴随类，它是一种弱引用，**指向shared_ptr所管理的对象**，而不影响所指对象的生命周期，也就是将一个`weak_ptr`绑定到一个`shared_ptr`不会改变`shared_ptr`的引用计数。不论是否有`weak_ptr`指向，一旦最后一个指向对象的`shared_ptr`被销毁，对象就会被释放。


## 2 常用操作
### 2.1 读取引用对象

`weak_ptr`对它所指向的`shared_ptr`所管理的对象没有所有权，不能对它解引用，因此若要读取引用对象，必须要转换成`shared_ptr`。 C++中提供了`lock`函数来实现该功能。如果对象存在，`lock()`函数返回一个指向共享对象的`shared_ptr`，否则返回一个空`shared_ptr`。

### 2.2 判断`weak_ptr`指向对象是否存在

`weak_ptr`提供了一个成员函数`expired()`来判断所指对象是否已经被释放。如果所指对象已经被释放，expired()返回true，否则返回false。

程序示例：

```cpp
std::shared_ptr<int> sp1(new int(22));
std::shared_ptr<int> sp2 = sp1;
std::weak_ptr<int> wp = sp1; // point to sp1
std::cout<<wp.use_count()<<std::endl; // 2
if(!wp.expired()){
    std::shared_ptr<int> sp3 = wp.lock();
    std::cout<<*sp3<<std::endl; // 22
}
```

### 2.3 `weak_ptr`可以作为`shared_ptr`的构造函数参数

`std::weak_ptr`可以作为`std::shared_ptr`的构造函数参数，但如果`std::weak_ptr`指向的对象已经被释放，那么`std::shared_ptr`的构造函数会抛出`std::bad_weak_ptr`异常。

```cpp
std::shared_ptr<int> sp1(new int(22));
std::weak_ptr<int> wp = sp1; // point to sp1
std::shared_ptr<int> sp2(wp);
std::cout<<sp2.use_count()<<std::endl; // 2
sp1.reset();
std::shared_ptr<int> sp3(wp); // throw std::bad_weak_ptr
```

### 2.4  `weak_ptr` 内置方法

|方法|用途|
|---|---|
|use_count()|返回与之共享对象的shared_ptr的数量|
|expired()|检查所指对象是否已经被释放|
|lock()|返回一个指向共享对象的shared_ptr，若对象不存在则返回空shared_ptr|
|owner_before()|提供所有者基于的弱指针的排序|
|reset()|释放所指对象|
|swap()|交换两个weak_ptr对象|

## 3 使用场景

### 3.1 用于实现缓存

weak_ptr可以用来缓存对象，当对象被销毁时，weak_ptr也会自动失效，不会造成野指针。

假设我们有一个Widget类，我们需要从文件中加载Widget对象，但是Widget对象的加载是比较耗时的。

```cpp
std::shared_ptr<Widget> loadWidgetFromFile(int id); 
// a factory function which returns a shared_ptr, which is expensive to call
// may perform file or database I/O
```

因此，我们希望Widget对象可以缓存起来，当下次需要Widget对象时，可以直接从缓存中获取，而不需要重新加载。这个时候，我们就可以使用`std::weak_ptr`来缓存Widget对象，实现快速访问。如以下代码所示：

```cpp
std::shared_ptr<Widget> fastLoadWidget(int id) {
    static std::unordered_map<int, std::weak_ptr<Widget>> cache;
    auto objPtr = cache[id].lock(); 
    if (!objPtr) {
        objPtr = loadWidgetFromFile(id);
        cache[id] = objPtr; // use std::shared_ptr to construct std::weak_ptr
    }
    return objPtr;
}
```

当对应id的Widget对象已经被缓存时，`cache[id].lock()`会返回一个指向Widget对象的`std::shared_ptr`，否则`cache[id].lock()`会返回一个空的`std::shared_ptr`，此时，我们就需要重新加载Widget对象，并将其缓存起来，这一步会由`std::shared_ptr`构造`std::weak_ptr`。

为什么不直接存储`std::shared_ptr`呢？因为这样会导致缓存中的对象永远不会被销毁，因为`std::shared_ptr`的引用计数永远不会为0。而`std::weak_ptr`不会增加对象的引用计数，因此，当缓存中的对象没有被其他地方引用时，`std::weak_ptr`会自动失效，从而导致缓存中的对象被销毁。

### 3.2 避免循环引用问题

- 什么是循环引用问题 ？

循环引用是指两个或多个对象之间通过`shared_ptr`相互引用，形成了一个环，导致它们的引用计数都不为0，从而导致内存泄漏。

在观察者模式中使用shared_ptr可能会出现循环引用，在下面的程序中，Observer对象和Subject对象相互引用，导致它们的引用计数都不为0，从而导致内存泄漏。

![](https://pic4.zhimg.com/80/v2-91d7772bd74c4187e9fe646f2c840e9f_720w.webp)

```cpp
class IObserver {
public:
    virtual void update(const string& msg) = 0;
};

class Subject {
public:
    void attach(const std::shared_ptr<IObserver>& observer) {
        observers_.emplace_back(observer);
    }
    void detach(const std::shared_ptr<IObserver>& observer) {
        observers_.erase(std::remove(observers_.begin(), observers_.end(), observer), observers_.end());
    }
    void notify(const string& msg) {
        for (auto& observer : observers_) {
            observer->update(msg);
        }
    }
private:
    std::vector<std::shared_ptr<IObserver>> observers_;
};

class ConcreteObserver : public IObserver {
public:
    ConcreteObserver(const std::shared_ptr<Subject>& subject) : subject_(subject) {}
    void update(const string& msg) override {
        std::cout << "ConcreteObserver " << msg<< std::endl;
    }
private:
    std::shared_ptr<Subject> subject_;
};

int main() {
    std::shared_ptr<Subject> subject = std::make_shared<Subject>();
    std::shared_ptr<IObserver> observer = std::make_shared<ConcreteObserver>(subject);
    subject->attach(observer);
    subject->notify("update");
    return 0;
}
```

- 避免循环引用的方法

将Observer类中的subject_成员变量改为`weak_ptr`，这样就打破循环引用，不会导致内存无法正确释放了。

### 3.3 用于实现单例模式

单例模式是指一个类只能有一个实例，且该类能自行创建这个实例的一种模式。单例模式的实现方式有很多种，其中一种就是使用`std::weak_ptr`。

```cpp
class Singleton {
public:
    static std::shared_ptr<Singleton> getInstance() {
        std::shared_ptr<Singleton> instance = m_instance.lock();
        if (!instance) {
            instance.reset(new Singleton());
            m_instance = instance;
        }
        return instance;
    }
private:
    Singleton() {}
    static std::weak_ptr<Singleton> m_instance;
};

std::weak_ptr<Singleton> Singleton::m_instance;
```

用`std::weak_ptr`实现单例模式的优点：

1. 避免循环应用：避免了内存泄漏。
2. 访问控制：可以访问对象，但是不会延长对象的生命周期。
3. 可以在单例对象不被使用时，自动释放对象。



参考：

1. [万字长文全面详解现代C++智能指针：原理、应用和陷阱 - 知乎](https://zhuanlan.zhihu.com/p/672745555)