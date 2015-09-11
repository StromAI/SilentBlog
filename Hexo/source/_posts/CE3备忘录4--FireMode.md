title: CE3备忘录4--FireMode
date: 2014-10-24 07:08:18
tags: [Code,Game]
categories: CE3
---
关于备忘录：
这里主要是自己记录一些CE3(Cryengine3)的常用方法，包括代码编写以及API功能，如果有什么问题感谢指出，这是我的邮箱albstein2@gmail.com

---
这里简单介绍FireMode的写法。

首先说明下CE3武器系统的调用结构：
在CE3中当触发射击事件的时候一般来说是这样的流程，首先调用到当前武器，在武器代码中通过m_fm成员调用FireMode的方法之后，通过FireMode完成射击动作，发射出的弹头造成伤害。

以上的这些关联均在xml文件中定义，可以在允许的范围内自由调用。

接下来就说FireMode了，因为之前一直没有怎么用过所以没怎么研究。最开始使用的时候直接继承下来，然后重载要用的函数，但是实践后发现会造成一个断言失败，之后调试的结论是FireMode的每个成员函数中都有自己的内存管理方法，由于调用了**this**指针所以所有的子类都需要定义这一方法，以保证内存的合理使用，当然在其中还使用一个宏定义来辅助完整这一任务。不过这里确实是给我造成了一些困扰，毕竟宏很少用啊。其中代码块分别使用了两组宏定义实现了这一方法的*声明与定义*。

好的~
接下来话不多说，咱们来看代码：
```c
#ifndef __MYFM_H__
#define __MYFM_H__

#include "Single.h"
//这里从Single中继承下来
class CLock : public CSingle
{

private:
	typedef CSingle BaseClass;

public:
	CRY_DECLARE_GTI(CMyFM);//迷之宏定义的声明部分
	//从注释来说大概是获取静态和运行时的信息
	//由于免费版源代码并没有完全开放，这里个人推测获取的信息在之后的GetMemoryUsage方法中需要使用

	CMyFM();
	virtual ~CMyFM(){};//注意这里的虚析构函数
	virtual void GetMemoryUsage(ICrySizer * s) const;
	//前面说的内存管理


	//你想要重载的函数	
private:
	//所需的成员
};

#endif 
```

cpp的定义
```c
#include "StdAfx.h"
#include "Lock.h"
#include "Projectile.h"

CRY_IMPLEMENT_GTI(CMyFM, CSingle);//迷之宏定义的实现部分

void CMyFM::GetMemoryUsage(ICrySizer * s) const
{ 
	s->AddObject(this, sizeof(*this));	
	BaseClass::GetInternalMemoryUsage(s);		// collect memory of parent class
}
//内存的管理

//你所需要的的方法的实现
```

使用的时候需要改写使用这一FireMode的武器xml中的FireMode标签，改成在GameFactory.cpp中注册的名称即可使用，关于注册方法与Weapon的注册方法相同。
这样整个武器系统的编写就完成了，当然关于一些奇怪武器的编写还要了解其他的API这里就不再赘述。

以上