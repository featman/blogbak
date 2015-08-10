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
> 4. 抽象工厂模式
> 5. 策略模式
> 6. 装饰者模式
> 7. 观察者模式

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
- 每次需要新的工厂，比如说中国pizaa，在加上一个派生类，同时修改下simplepizzafacotry的判断条件。
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
>工厂模式与简单工厂模式相比有何优势？
> > 简单工厂模式下，如果多了一个内容，比如说有北京，有日本，那么需要在simplePizzaFatory下添加判断，然后再去添加相关的产品生产类。这样就改动了原有的simplePizzaFactory类。不符合面向对象的思想。如果我们不改变原有的simplePizzaFactory,而是再加一层抽象，把这个类也变成抽象父类，最初一开始就指定要xxxx工厂帮我判断，然后xxxx工厂自行判断好，去告诉相关的生产商去生产产品就可以了。

> >如下图所示：这时候如果想要添加一个合肥的pizza厂商帮我生产，那么分为两步：1.把HefeiPizzaFactory填写好，然后让其去判断是吃cheese还是吃veggie，然后再去写cheese还是veggie，这时候虽然麻烦一点，但是并不没有触动以前的类和方法。 


3.1 UML图
![image description](http://7xkz95.com1.z0.glb.clouddn.com/15-8-10/90160599.jpg)
3.2 实现

```
/*
工厂模式
*/
#include <iostream>
#include <string>
using namespace std;

class Pizza
{
public:
	virtual void Production() = 0;
};

class BeijingCheesePizza:public Pizza
{
public:

	void Production()
	{
		cout << "Beijing CheesePizza production.\n";
	}

};

class BeijingVeggiePizza:public Pizza
{
public:

	void Production()
	{
		cout << "Beijing VeggiePizza Prapare.\n";
	}
};

class JinanCheesePizza:public Pizza
{
public:

	void Production()
	{
		cout << "Jinan CheesePizza production.\n";
	}

};

class JinanVeggiePizza:public Pizza
{
public:

	void Production()
	{
		cout << "Jinan VeggiePizza Prapare.\n";
	}
};

class PizzaFactory
{
public:
	virtual Pizza* CreatePizza(string type) = 0;

};

class BeijingPizzaFactory:public PizzaFactory
{
public:
	Pizza* CreatePizza(string type)
	{
		if (0 == type.compare("cheese"))
		{
			pizza_ = new BeijingCheesePizza;
		}
		else if (0 == type.compare("veggie"))
		{
			pizza_ = new BeijingVeggiePizza;
		}
		return pizza_;
	}

private:

	Pizza* pizza_;
};

class JinanPizzaFactory:public PizzaFactory
{
public:
	Pizza* CreatePizza(string type)
	{
		if (0 == type.compare("cheese"))
		{
			pizza_ = new JinanCheesePizza;
		}
		else if (0 == type.compare("veggie"))
		{
			pizza_ = new JinanVeggiePizza;
		}
		return pizza_;
	}

private:

	Pizza* pizza_;
};

int main()
{
	PizzaFactory *beijing = new BeijingPizzaFactory;
	beijing->CreatePizza("veggie")->Production();

	PizzaFactory* jinan = new JinanPizzaFactory;
	jinan->CreatePizza("cheese")->Production();
	system("pause");
	return 0;
}


```

### 参考文献

http://www.jellythink.com/archives/82
http://blog.chinaunix.net/uid-25510439-id-3073218.html
http://www.cnblogs.com/cxjchen/p/3143633.html