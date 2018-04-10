
---
layout: page
title: ECS架构 Entitas-CSharp学习之路（二）
category: 
    - blogs
---

接下来第二篇教程。主要是教我们如何在Entitas下使用组件表示游戏状态，以及其他系统如何响应用户输入执行相应逻辑。

## 官方教程
https://github.com/sschmid/Entitas-CSharp/wiki/Unity-Tutorial---Simple-Entity-View-and-Movement
完全按教程做就能将demo跑起来，注意要将camera的Projection属性设为Orthographic

## 实体定义层
教程将定义实体数据的组件类全部定义到一个文件内，然后通过generate生成实体类。
可以注意到每个类内至多只有一个字段。
MoverComponent，MoveCompleteComponent这种组件并没有按我想象中定义一个bool字段作为标志，而是直接以实体的存在与否作为标志，作者的设计相当简洁。

LeftMouse和RightMouse则是利用[Input, Unique]2个属性定义2个实体区分左右键，利用Input属性创建了3个实体区分鼠标的三个状态，很有意思很简洁的定义

## 视图系统
### AddViewSystem
功能如下：
筛选所有Sprite实体后
创建GameObject与实体Sprite绑定，让创建的GameObject作为实体的View
同时将所有与实体Sprite绑定的GameObject放到GameObject("Game Views")下统一管理

### 渲染系统
用RenderSpriteSystem，RenderPositionSystem，RenderDirectionSystem分别修改实体绑定的GameObject的各项属性

### 功能
不多说
```
Add(new AddViewSystem(contexts));
Add(new RenderSpriteSystem(contexts));
Add(new RenderPositionSystem(contexts));
Add(new RenderDirectionSystem(contexts));
```
## 移动系统
这个系统只负责移动，只要存在的move实体且target和本身实体position之间有较长的距离，就实现自动移动
```
using Entitas;
using UnityEngine;

public class MoveSystem : IExecuteSystem, ICleanupSystem
{
    readonly IGroup<GameEntity> _moves;
    readonly IGroup<GameEntity> _moveCompletes;
    const float _speed = 4f;

    public MoveSystem(Contexts contexts)
    {
        _moves = contexts.game.GetGroup(GameMatcher.Move);
        _moveCompletes = contexts.game.GetGroup(GameMatcher.MoveComplete);
    }

    public void Execute()
    {
        foreach (GameEntity e in _moves.GetEntities())
        {
            Vector2 dir = e.move.target - e.position.value;
            Vector2 newPosition = e.position.value + dir.normalized * _speed * Time.deltaTime;
            e.ReplacePosition(newPosition);

            float angle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg;
            e.ReplaceDirection(angle);

            float dist = dir.magnitude;
            if (dist <= 0.5f)
            {
                e.RemoveMove();
                e.isMoveComplete = true;
            }
        }
    }

    public void Cleanup()
    {
        foreach (GameEntity e in _moveCompletes.GetEntities())
        {
            e.isMoveComplete = false;
        }
    }
}
```
移动系统最核心的就是这个类，同时继承 IExecuteSystem, ICleanupSystem2个接口，分别实现反应系统和清理系统

同时反应系统每帧执行完后都会销毁，所以Execute()里是也是利用Time.deltaTime做每帧的运动
而当距离足够小时，就会移除move实体，将isMoveComplete 设置为true，从而结束游戏对象的运动状态


## 输入系统

### 输入监听执行系统
EmitInputSystem

Initialize()中将左右键实体标志位设置为true，同时在类中存储2个实体

Execute()则是利用unity自带的方法获取鼠标状态，从而在相应状态时将鼠标位置传递给实体


### CreateMoverSystem
总觉得这个名字不好，这个就是用来创建游戏对象的，在右键点击时将相应数据传递给实体

### CommandMoveSystem
构造函数获取了所有Mover实体，在鼠标左键点击时随机选择一个Mover实体获取鼠标点击位置，添加Move

## GameController
没什么好说的就是，就是获取上下文，创建功能，初始化，在update中执行反应系统和清理系统

## 总结
Entitas将unity输入控制用ECS的方法重写了一遍，个人感觉工程量有些大，不够灵活，不过细化了输入的控制，功能分的很清晰，如果工程架构大了，这样做反而容易管理。
