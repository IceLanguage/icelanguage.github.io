
---
layout: page
title: ECS架构 Entitas-CSharp学习之路（三）
category: 
    - blogs
---

第三个教程 -实现一个多反应系统
## 教程地址
https://github.com/sschmid/Entitas-CSharp/wiki/MultiReactiveSystem-Tutorial

## 配置
打开之前的Entitas编辑器，就是之前用来generate的那个。
在那个Jenny->Context 配置Game, Input, Ui,然后generate
加入教程所说的DestroyedComponent组件,Generate.
我之前用的是1.4.0,一直在报错，说array out of range，怕是因为不能识别多个属性，然后换成1.4.2版本的Entitas就没事了。

## MultiDestroySystem
官方代码有问题，是这样的
```
using System.Collections.Generic;
using Entitas;
using Entitas.Unity;
using UnityEngine;

// IDestroyed: "I'm an Entity, I can have a DestroyedComponent"
public interface IDestroyEntity : IEntity, IDestroyedEntity{ }

// tell the compiler that our context-specific entities implement IDestroyed
public partial class GameEntity : IDestroyEntity { }
public partial class InputEntity : IDestroyEntity { }
public partial class UiEntity : IDestroyEntity { }

// inherit from MultiReactiveSystem using the IDestroyed interface defined above
public class MultiDestroySystem : MultiReactiveSystem<IDestroyEntity, Contexts>
{
    // base class takes in all contexts, not just one as in normal ReactiveSystems
    public MultiDestroySystem(Contexts contexts) : base(contexts)
    {
    }

    // return an ICollector[] with a collector from each context
    protected override ICollector[] GetTrigger(Contexts contexts)
    {
        return new ICollector[] {
            contexts.game.CreateCollector(GameMatcher.Destroyed),
            contexts.input.CreateCollector(InputMatcher.Destroyed),
            contexts.ui.CreateCollector(UiMatcher.Destroyed)
        };
    }

    protected override bool Filter(IDestroyEntity entity)
    {
        return entity.isDestroyed;
    }

    protected override void Execute(List<IDestroyEntity> entities)
    {
        foreach (var e in entities)
        {
            Debug.Log("Destroyed Entity from " + e.contextInfo.name + " context");
            e.Destroy();
        }
    }
}
```
注意：系统类继承的不是ReactiveSystem< GameEntity>，而是MultiReactiveSystem< IDestroyEntity, Contexts>

然后自己加个GameController和Feature，Feature 里 Add(new MultiDestroySystem(contexts));

运行游戏
可以发现实体一被添加销毁组件，就会销毁，并输出Destroyed Entity from Game context

## MultiDestroySystem with View
照着做
这个示例主要教我们怎么在多反应系统访问多个组件

## Performing context-specific actions in multi-reactive systems
通过使用扩展方法，可以编写使用其名称检索上下文引用的功能。我们可以在多反应系统中使用此扩展来获取每个实体的IContext参考。

发现问题很多，大概是版本的问题，我修正了版本(我的版本是1.4.2）
```
using System.Collections.Generic;
using Entitas;
using Entitas.Unity;
using UnityEngine;

// IViewEntity: "I am an Entity, I can have an AssignViewComponent and a ViewComponent"
public interface IViewedEntity : IAssignViewEntity, IViewEntity, IEntity { }
public partial class GameEntity : IViewedEntity { }
public partial class InputEntity : IViewedEntity { }
public partial class UiEntity : IViewedEntity { }

public class MultiAddViewSystem : MultiReactiveSystem<IViewedEntity, Contexts>
{
    private readonly Transform _topViewContainer = new GameObject("Views").transform;
    private readonly Dictionary<string, Transform> _viewContainers = new Dictionary<string, Transform>();
    private readonly Contexts _contexts;

    public MultiAddViewSystem(Contexts contexts) : base(contexts)
    {
        _contexts = contexts;
        // create a view container for each context name
        foreach (var context in contexts.allContexts)
        {
            string contextName = context.contextInfo.name;
            Transform contextViewContainer = new GameObject(contextName + " Views").transform;
            contextViewContainer.SetParent(_topViewContainer);
            _viewContainers.Add(contextName, contextViewContainer);
        }
    }

    protected override ICollector[] GetTrigger(Contexts contexts)
    {
        return new ICollector[] {
            contexts.game.CreateCollector(GameMatcher.AssignView),
            contexts.input.CreateCollector(InputMatcher.AssignView),
            contexts.ui.CreateCollector(UiMatcher.AssignView)
        };
    }

    protected override bool Filter(IViewedEntity entity)
    {
        return entity.isAssignView && !entity.hasView;
    }

    protected override void Execute(List<IViewedEntity> entities)
    {
        foreach (var e in entities)
        {
            string contextName = e.contextInfo.name;
            GameObject go = new GameObject(contextName + " View");
            go.transform.SetParent(_viewContainers[contextName]);
            e.AddView(go);
            go.Link(e, _contexts.GetContextByName(contextName));
            e.isAssignView = false;
        }
    }
}
```

## 总结
就是学会了如何利用多反应系统和扩展方法为多个系统构建相同的方法（不过吐槽一句，教程没有之前2个用心了，教程中间突然叫你运行游戏，GameController还没创建呢，我以为只是省略了，叫读者自己创建，可教程后面又让你创建GameController。。。。）
