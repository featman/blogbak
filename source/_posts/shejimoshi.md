title: 程序设计的几种设计模式    
date: 2015-8-10 12:31:49
categories: 程序设计
tags: [设计模式,C++实现] 
---
**在程序设计中往往想到哪里就写到哪里，涉及到设计模式的思想用到实现中确实不多，总结下常用的，对于后期需要的只能之后用到再补充，这样做以免自己一次性学的太多，记的不牢靠。每个设计模式后都有C++实现，并配图便于理解。**
<!--more-->
### 目录
> 1. 单例模式  
> 2. 简单工厂模式
> 3. 工厂模式
> 4. 观察者模式
> 5. 策略模式
> 6. 装饰者模式
> 7. 抽象工厂模式

### 1.单例模式
> 单例模式就是在程序的运行周期内，只有一个类的实例在运行。  
> > 他为了解决某一个程序执行相同的操作，但是每次使用，都需要分配内存，然后销毁，这样给系统带来了不必要的开销。所以只需要一个相同的实例，每次调用一次就完成工作。
> > 
单例模式适合应用在工具类，常常与工厂模式配合使用。

1.1 UML图


![image description](http://7xkz95.com1.z0.glb.clouddn.com/15-8-10/5097401.jpg)  
1.2 实现  

- 考虑防止创建多个实例，这样构造函数设置为private可解决。
- 考虑防止多个线程操作类的时候，还是会创建多个实例。用lock同步

```
#include <iostream>
using namespace std;

class Singleton
{
public:
	static Singleton *GetInstance()
	{
		if (m_Instance == NULL )
		{
			Lock(); 
			if (m_Instance == NULL )
			{
				m_Instance = new Singleton ();
			}
			UnLock(); 
		}
		return m_Instance;
	}

	static void DestoryInstance()
	{
		if (m_Instance != NULL )
		{
			delete m_Instance;
			m_Instance = NULL ;
		}
	}

	int GetTest()
	{
		return m_Test;
	}

private:
	Singleton(){ m_Test = 0; }
	static Singleton *m_Instance;
	int m_Test;
};

Singleton *Singleton ::m_Instance = NULL;

int main(int argc , char *argv [])
{
	Singleton *singletonObj = Singleton ::GetInstance();
	cout<<singletonObj->GetTest()<<endl;
	Singleton ::DestoryInstance();

	return 0;
}
```
### 2.简单工厂模式
> 简单工厂模式，它的主要特点是需要在工厂类中做判断，从而创造相应的产品。当增加新的产品时，就需要修改工厂类。  

2.1 UML图
![image description](http://7xkz95.com1.z0.glb.clouddn.com/15-8-10/58525325.jpg)
2.2 实现

```
#include <iostream>
#include <string>
using namespace std;

class Pizza
{
public:
	virtual void Prapare() = 0;
	virtual void Bake() = 0;
};

class CheesePizza:public Pizza
{
public:

	void Prapare()
	{
		cout << "CheesePizza Prapare.\n";
	}

	void Bake()
	{
		cout << "CheesePizza Bake.\n";
	}
};

class VeggiePizza:public Pizza
{
public:

	void Prapare()
	{
		cout << "VeggiePizza Prapare.\n";
	}

	void Bake()
	{
		cout << "VeggiePizza Bake.\n";
	}
};


class SimplePizzaFactory
{
public:

	static Pizza* CreatePizza(string type)
	{
		if (0 == type.compare("cheese"))
		{
			pizza_ = new CheesePizza;
		}
		else if (0 == type.compare("veggie"))
		{
			pizza_ = new VeggiePizza;
		}

		return pizza_;
	}

private:
	static Pizza* pizza_;
};

Pizza* SimplePizzaFactory::pizza_ = NULL;

int main()
{
	SimplePizzaFactory::CreatePizza("cheese")->Prapare();
	SimplePizzaFactory::CreatePizza("veggie")->Prapare();

	return 0;
}


```
### 3.工厂模式

### 参考文献

http://www.jellythink.com/archives/82
http://blog.chinaunix.net/uid-25510439-id-3073218.html