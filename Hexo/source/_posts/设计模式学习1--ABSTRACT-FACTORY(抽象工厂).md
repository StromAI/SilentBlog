title: 设计模式学习1--ABSTRACT FACTORY(抽象工厂)
date: 2014-09-11 03:42:02
tags: [Code,Software]
categories: Design-Patterns
---
最近开始了设计模式的学习，在这里做简单的记录
这里我用的是《设计模式：可复用面向对象软件的基础》这本书，用到的例子都是书中的内容
（基本就是抄书了= =。  咳咳 不要在意这些细节）
So，you got it~

---

这里看到的是创建型模式中的ABSTRACT FACTORY(抽象工厂)

抽象工厂可以为我们提供一个创建一系列相关相互依赖对象的接口，在我们使用时无需指定这些接口具体的类。（个人理解为C++的动态绑定技术）

在以下几种情况下我们可以使用ABSTRACT FACTORY模式
* 一个系统要独立于它的产品创建、组合和表示时
* 一个系统要由多个产品系列中的一个来配置时
* 当你要强调一系列相关的产品对象的设计以便进行联合使用时
* 当你提供一个产品类库，而只想显示它们的借口而不是现实

这里是结构图：

![](http://stromaiblog.qiniudn.com/abstractfactory.png)


关于ABSTRACT FACTORY的应用，CE3实体系统似乎就是用的类似的模式实现，同时书上也给出了几个例子供大家参考这里就不做说明了。

```c
class MazeFactory //AbstractFactory&ConcreteFactory
{
public:
    MazeFactory();
  
    virtual Maze* MakeMaze() const
        { return new Maze; }
    virtual Maze* MakeWall() const
        { return new Wall; }
    virtual Maze* MakeRoom(int n) const
        { return new Room(n); }
    virtual Maze* MakeDoor(Room* r1, Room * r2) const
        { return new Door(r1, r2); }
};
  
class BombedMazeFactory : public MazeFactory //ConcreteFactory
{
public:
    BombedMazeFactory();

    virtual Room* MakeRoom(int n) const
        { return new RoomWithBomb(); }
    virtual Room* MakeWall () const
        { return new BombedWall(); }
};
```
上面这段即为书上给出的例子，其中MazeFactory作为*AbstractFactory*的同时又充当了*ConcreteFactory*角色，而BombedMazeFactory则只是作为*ConcreteFactory*在其中。

在我们使用*AbstractFactory*时，使用MazeFactory的引用来创建实体，调用时我们只需通过不同的对象参数既可产生不同的实体。
具体而言就是这样一段代码
```c
Maze* MazeGame::CreateMaze (MazeFactory& factory)
{
	//...
	//这是一个生成迷宫的函数MazeGame，为这个游戏中的一个类
	//这里只是说明问题所以具体的细节就不在这里显示了
	//具体的代码在书中有提及（虽然也不是完整的）
}
 
MazeGame game;
 
 
MazeFactory factory;
//BombedMazeFactory factory;
//通过对这两行不同的类型的factory的调用，下面的函数会得到不同的结果
 
game.CreateMaze(factory);
```

描述中应该是有很多问题的，所以有错误的话希望大家指出作为纯正菜鸡再次感谢Orz