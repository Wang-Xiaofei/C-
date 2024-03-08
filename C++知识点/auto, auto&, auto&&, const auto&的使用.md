---
UID: 2401091245
link: ""
cssclasses: 
tags:
  - C加/auto
---
## 1 c++中的 auto

1. auto的使用   
    c++11引入了auto类型说明符，auto让编译器通过初始值来推算变量的类型，所以auto定义的变量必须有初始值。   
    使用auto也能在一条语句中声明多个变量，因为一条声明语句只能有一个基本数据类型，所以该语句中所有变量的初始基本数据类型都必须一样：   
2. 范围for循环，遍历给定序列中的每个元素并对序列中的每个值执行某种操作。

```c
for(declaration:expression)    statement   
```

>expression 部分是一个对象，用于表示一个序列，declaration部分负责定义一个变量，该变量被用于访问序列中的基础元素，每次迭代declaration部分的变量会被初始化为expression部分的下一个元素值。

## 2 auto，auto&， auto&&， const auto&的使用

标题中auto，auto&， auto&&， const auto&的使用，其实更多的是C++中值传递，引用传递，右值引用等知识，使用以下代码进行了测试，以及测试结果。

```C
#include <stdio.h>
#include <iostream>
#include <vector>
#include <numeric>
#include <string>
using std::vector;
 
int main()
{
 
//测试1：auto c, 创建拷贝，每次遍历都是copy一次，不会对str中的元素进行修改
	std::string str("str ing1");
	for (auto c : str)
	{
		c+=1;
		std::cout<<c;  //输出tus!joh2，c每次获取str的元素后，加1，再打印
	}
	std::cout<<'\n';
	std::cout<<str<<'\n';  //这时再看原str字符串，仍然输出str ing1，没变化。
 
//测试2：auto& c, reference, 可以通过c给str2中的元素赋值
	std::string str2("str ing1");
	for (auto& c : str2)
	{
		c+=1;
		std::cout<<c;  //输出tus!joh2，c每次获取str的元素后，加1，再打印
	}
	std::cout<<'\n';
	std::cout<<str2<<'\n';  //这时再看原str2字符串，输出tus!joh2，被重新赋值了。
 
//测试3：auto&& c，右值引用， 可以修改str3中的元素
	std::string str3("str ing1");
	for (auto&& c : str3)
	{
		c+=1;
		std::cout<<c;  //输出tus!joh2，c每次获取str的元素后，加1，再打印
	}
	std::cout<<'\n';
	std::cout<<str3<<'\n'; //这时再看原str3字符串，输出tus!joh2，被重新赋值了。
 
//测试4：const auto&， 只读str4中的元素
	std::string str4("str ing1");
	for (const auto& c : str4)
	{
		//c+=1;  //报错，assignment of read-only reference 'c'，const限定c只读
		std::cout<<c;
	}
	std::cout<<'\n';
	std::cout<<str4<<'\n';
}
```


学习时用的参考链接：
[C++ 遍历循环表达示 auto, auto&, auto&&-CSDN博客](https://blog.csdn.net/songbaiyao/article/details/105814251?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161915851016780271526809%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=161915851016780271526809&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-2-105814251.pc_search_result_no_baidu_js&utm_term=auto%26)
[C++ 中的auto、auto&、auto&&、const auto&\_auto &&-CSDN博客](https://blog.csdn.net/weixin_42442319/article/details/116056114)
[Range-based for loop  (since C++11) - cppreference.com](https://en.cppreference.com/w/cpp/language/range-for)

## 3 for(auto x : vector)

auto会**拷贝**一份容器内的vector，在修改x时**不会改变**原容器当中的vector值，只会改变拷贝的vector。  


```cpp
#include <iostream>
#include <vector>
 
using namespace std;
 
void printVec(vector<int>& vec) {
	for (int i = 0; i < vec.size(); i++)
		cout << vec[i] << endl;
}
 
int main() {
	vector<int> vec = {4,3,2,1,0 };
	cout<<" for(auto x : vector)"<<endl;
    //这个时候每个取出来的iter只是原来vec元素的副本，不会改变vec中任何元素
    //iter可以作为左值进行赋值，但只是在副本范围
	for (auto iter : vec) {
		iter = iter + 100;
	}
	printVec(vec);
 
	cout<<"for(auto& x : vector)"<<endl;
    //这个时候取出来的是 &iter，代表原来vec中元素的引用
    //如果对iter进行任何改变都会作用在vec上
	for (auto& iter : vec) {	//如果vec返回临时对象，用auto&&
		iter = iter + 100;
	}
	printVec(vec);
}
```

### 3.1 【重点1】for(auto& x : vector)，左值引用

当需要对原数据进行同步修改时，就需要添加&，即vector的引用。

会在改变x的同时修改vector。

  

### 3.2 【重点2】for(auto&& x : vector)，右值引用

当vector返回临时对象，使用auto&会编译错误，临时对象不能绑在non-const l-value reference （非常量左值引用），需使用auto&&（非常量右值引用），初始化右值时也可捕获

  

### 3.3 【重点3】for(const auto& x : vector)，常量左值引用

使用 const 可以避免无意中修改数据的编程错误；

使用 const 能让函数接收 const 和非 const 类型的实参，否则将只能接收非 const 类型的实参；

使用 const 引用能够让函数正确生成并使用临时变量（当传入实参与形参符合自动转换要求时可以通过临时变量进行自动类型转换）。

  

const （常类型），不能作为左值

& （引用），不拷贝，不申请新空间，会对原vector修改

当我们不希望拷贝原vector（拷贝需要申请新的空间），同时不愿意随意改变原vector，那么我们可以使用for(const auto& x : vector)，这样我们可以很方便的在不拷贝的情况下读取vector，同时不会修改vector。一般用在只读操作。

  

### 3.4 const auto x : vector，常量左值引用

该操作相对于const auto& x : vector只是少了引用（&），**即会申请新的空间（拷贝），不经常使用**。

  

### 3.5 const auto&& x：vector）,常量右值引用无实际意义，可以被常量左值引用替代

常量与非常量的左值右值引用可以参考：C++11右值引用

- 参考[[右值引用与左值引用的区别]]
























